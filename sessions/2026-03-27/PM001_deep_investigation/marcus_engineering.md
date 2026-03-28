# Marcus Webb - Engineering Reality Check
# Project Mitchell PM001 - Build Constraints & Implementation Analysis

**Agent:** Marcus Webb (Engineer)
**Session:** PM001_deep_investigation
**Date:** 2026-03-27
**Focus:** Build feasibility, technical constraints, implementation reality extracted from research

---

## Executive Summary

The FQHC behavioral health technology opportunity is real, but building for this market means embracing **extreme complexity tolerance**. Directors manage providers with 15+ credential types across 50+ state licensing regimes, billing to 20+ payers with different rules per state, under 4+ federal regulatory frameworks that change annually. Any platform that simplifies this into generic workflows will fail in production.

**Engineering philosophy required:** Embrace complexity at the data model level, abstract it at the UI level. Don't pretend FQHC billing is "just healthcare billing with extra fields."

---

## Build Complexity Assessment

### Complexity Layer 1: Multi-Payer Billing Logic

**Problem:** FQHC billing rules vary by:
- Payer type (Medicare, Medicaid FFS, Medicaid MCO, Commercial, Uninsured)
- State (Medicaid T-codes vary by state)
- Service type (medical, behavioral health, dental, pharmacy)
- Date (PPS rates adjust annually, retroactive adjustments common)

**Engineering implication:**

```python
# This DOESN'T work (generic billing logic):
def calculate_claim_amount(service, patient_insurance):
    return service.base_rate * patient_insurance.coverage_percentage

# This DOES work (FQHC-specific):
def calculate_fqhc_claim_amount(encounter, patient, provider, service_date):
    payer = get_payer(patient.insurance, service_date)

    if payer.type == "MEDICARE":
        base_rate = get_pps_rate(year=service_date.year)
        productivity_adjustment = get_productivity_factor(encounter.fqhc)
        geographic_adjustment = get_geographic_factor(encounter.fqhc.zip)
        return base_rate * productivity_adjustment * geographic_adjustment

    elif payer.type == "MEDICAID_MCO":
        mco_rate = get_mco_encounter_rate(payer, encounter.fqhc, service_date)
        pps_rate = get_state_pps_rate(encounter.fqhc.state, service_date.year)
        wraparound_payment = pps_rate - mco_rate
        return {"mco": mco_rate, "wraparound": wraparound_payment}

    elif payer.type == "MEDICAID_FFS":
        return get_state_pps_rate(encounter.fqhc.state, service_date.year)

    elif payer.type == "COMMERCIAL":
        # Negotiated rate per contract
        return get_contract_rate(payer, encounter.fqhc, encounter.service_codes)

    elif payer.type == "UNINSURED":
        base_cost = calculate_cost_based_rate(encounter)
        discount = calculate_sliding_fee_discount(patient.income, patient.household_size)
        return base_cost * (1 - discount)
```

**Key insight:** You can't abstract payer logic into a generic "insurance" object. FQHC billing is a state machine with 50+ states (literally, US states).

**Build estimate:** 6-9 months just for billing engine (if starting from scratch). Faster if partnering with existing FQHC RCM platform.

### Complexity Layer 2: PPS Encounter Qualification

**Problem:** Medicare/Medicaid pay per "encounter," not per service. An encounter must meet specific criteria:

**Qualifying services for FQHC encounter:**
- Face-to-face visit with physician, NP, PA, CNM, CP, CSW, LMFT, LMHC
- Medical or mental health services
- May include services on same day (medical + BH = one encounter, if properly documented)

**Non-qualifying services:**
- Lab tests alone (must be part of visit)
- Immunizations alone
- Case management (unless part of visit)

**Engineering implication:**

```python
def qualifies_as_encounter(visit):
    # Check provider type
    if visit.provider.type not in QUALIFYING_PROVIDER_TYPES:
        return False

    # Check service type
    if not any(service.is_qualifying for service in visit.services):
        return False

    # Check face-to-face requirement
    if not visit.is_face_to_face and not visit.is_telehealth_compliant:
        return False

    # Check documentation completeness
    if not visit.has_complete_documentation():
        return False  # Most common denial reason

    # Same-day consolidation check
    if same_day_visit := get_same_day_visit(visit.patient, visit.date):
        if not can_consolidate(visit, same_day_visit):
            return False  # Duplicate billing risk

    return True
```

**Key insight:** Encounter qualification is not just a billing rule, it's a clinical documentation requirement. The EHR must guide providers to document in a way that meets PPS criteria.

**Build estimate:** 3-6 months for encounter qualification engine + clinical documentation templates.

### Complexity Layer 3: 42 CFR Part 2 Consent Management

**Problem:** Substance use disorder (SUD) records are protected by 42 CFR Part 2, which is MORE restrictive than HIPAA. Every disclosure requires patient consent, and patients can revoke consent at any time.

**Engineering implication:**

```python
class Part2Record:
    """SUD record with consent-aware disclosure tracking"""

    def can_disclose_to(self, recipient, purpose):
        # Check if patient has signed consent for this recipient + purpose
        consent = self.patient.get_part2_consent(effective_on=datetime.now())

        if not consent:
            return False

        if recipient not in consent.approved_recipients:
            return False

        if purpose not in consent.approved_purposes:
            return False

        # Check revocation status
        if consent.revoked:
            return False

        return True

    def disclose(self, recipient, purpose, user):
        if not self.can_disclose_to(recipient, purpose):
            raise Part2ViolationError("Disclosure not authorized by patient consent")

        # Log disclosure (required by Part 2)
        DisclosureLog.create(
            record=self,
            recipient=recipient,
            purpose=purpose,
            disclosed_by=user,
            timestamp=datetime.now()
        )

        # Return record with prohibition on re-disclosure notice
        return self.with_redisclosure_prohibition_notice()
```

**Key insight:** Part 2 compliance can't be bolted on. It must be enforced at the database layer (row-level security), API layer (every read/write), and UI layer (redaction, access warnings).

**Build estimate:** 4-6 months for Part 2 consent engine + disclosure tracking + UI redaction.

**Penalty for getting this wrong:** $100-$50,000 per violation, up to $1.5M annually per violation type (effective Feb 16, 2026).

### Complexity Layer 4: Credentialing Lifecycle Management

**Problem:** Providers must be re-credentialed every 2 years, with primary source verification (NPDB query, state license verification, DEA verification). FQHCs lose FTCA malpractice coverage if they fail to credential properly.

**Engineering implication:**

```python
class ProviderCredentialing:
    """2-year credentialing cycle with automated alerts"""

    def __init__(self, provider):
        self.provider = provider
        self.cycle_start = None
        self.cycle_end = None
        self.status = "pending"

    def initiate_credentialing(self):
        # Primary source verification checklist
        tasks = [
            ("LICENSE", self.verify_state_license()),
            ("NPDB", self.query_npdb()),
            ("DEA", self.verify_dea()),
            ("MALPRACTICE", self.review_claims_history()),
            ("REFERENCES", self.collect_peer_references()),
        ]

        all_complete = all(status == "complete" for _, status in tasks)

        if all_complete:
            self.status = "approved"
            self.cycle_start = datetime.now()
            self.cycle_end = datetime.now() + timedelta(days=730)  # 2 years
            self.schedule_renewal_alerts()

        return self.status

    def schedule_renewal_alerts(self):
        # Alert 180 days before expiration (6 months)
        alert_180 = self.cycle_end - timedelta(days=180)
        schedule_task(send_credentialing_alert, run_at=alert_180, args=[self.provider, "180_days"])

        # Alert 90 days before expiration (3 months)
        alert_90 = self.cycle_end - timedelta(days=90)
        schedule_task(send_credentialing_alert, run_at=alert_90, args=[self.provider, "90_days"])

        # Alert 30 days before expiration (URGENT)
        alert_30 = self.cycle_end - timedelta(days=30)
        schedule_task(send_credentialing_alert, run_at=alert_30, args=[self.provider, "30_days_URGENT"])

    def is_current(self, on_date=None):
        on_date = on_date or datetime.now()
        return self.status == "approved" and self.cycle_start <= on_date <= self.cycle_end
```

**Key insight:** Credentialing is a time-series problem with scheduled workflows. Can't just store "credentialed: true/false" - need full lifecycle tracking.

**Build estimate:** 2-3 months for credentialing workflow engine + alert system.

**Penalty for getting this wrong:** FTCA coverage revoked, malpractice exposure, cannot bill for services provided by non-credentialed providers.

### Complexity Layer 5: UDS Reporting (Adaptive to Federal Changes)

**Problem:** HRSA restructured UDS reporting in 2026 ("one of the largest restructurings in decades"). Table 4 eliminated, Table 5 restructured, Table 6A measures changed. This happens periodically, and platforms must adapt quickly.

**Engineering implication:**

```python
class UDSReportingEngine:
    """Adaptive reporting engine that adjusts to federal spec changes"""

    def __init__(self, reporting_year):
        self.reporting_year = reporting_year
        self.spec = self.load_uda_spec(reporting_year)

    def load_uda_spec(self, year):
        """Load reporting spec for given year (external config, not hardcoded)"""
        return UDSSpec.load_from_config(f"uda_spec_{year}.yaml")

    def generate_report(self, fqhc):
        report = {}

        for table in self.spec.tables:
            if table.deprecated_in_year and table.deprecated_in_year <= self.reporting_year:
                continue  # Skip deprecated tables (e.g., Table 4 eliminated 2026)

            report[table.name] = self.generate_table(fqhc, table)

        return report

    def generate_table(self, fqhc, table):
        """Generate table data based on spec"""
        data = {}

        for field in table.fields:
            if field.added_in_year and field.added_in_year > self.reporting_year:
                continue  # Skip fields not yet required

            data[field.name] = self.calculate_field(fqhc, field)

        return data

    def calculate_field(self, fqhc, field):
        """Calculate field value using query defined in spec"""
        query = field.query_template.format(
            fqhc_id=fqhc.id,
            reporting_year=self.reporting_year
        )
        return execute_query(query)
```

**Key insight:** Don't hardcode UDS table structure in application code. Externalize to config so you can update without code deployment when HRSA changes specs.

**Build estimate:** 2-4 months for adaptive reporting engine + historical spec versioning.

**Penalty for getting this wrong:** FQHCs can't submit UDS reports on time (February 15 deadline), risk losing Section 330 grant funding.

---

## Technology Stack Reality Check

### Ava's Research Says: "AI denial prevention, only 14% adoption, 82% want it"

**My read:** This is marketing speak, not engineering reality. Let's define what "AI denial prevention" actually means in code.

**Naive approach (will fail):**
```python
# Train model on historical denials
model = train_model(historical_claims_data)

# Predict denial risk for new claim
risk_score = model.predict(new_claim)

if risk_score > 0.7:
    flag_for_review(new_claim)
```

**Why this fails:**
1. **Class imbalance:** 15-25% denial rate means 75-85% claims are approved. Model will just predict "approve" for everything.
2. **Payer policy changes:** Model trained on 2024 data doesn't know about 2025 policy changes.
3. **No actionable feedback:** "This claim will be denied" doesn't tell billing staff HOW to fix it.

**Engineering-grade approach:**

```python
class DenialPreventionEngine:
    """Rule-based validation + ML risk scoring"""

    def validate_claim(self, claim):
        # Phase 1: Rule-based validation (deterministic)
        validation_results = []

        # Check 1: Authorization on file?
        if claim.requires_authorization and not claim.has_authorization():
            validation_results.append({
                "check": "authorization",
                "status": "FAIL",
                "reason": "Pre-authorization required but not on file",
                "fix": f"Obtain authorization from {claim.payer.name} before submitting"
            })

        # Check 2: PPS encounter qualification?
        if not claim.encounter.qualifies_as_pps_encounter():
            validation_results.append({
                "check": "pps_qualification",
                "status": "FAIL",
                "reason": "Encounter does not meet PPS qualification criteria",
                "fix": "Review clinical documentation for completeness"
            })

        # Check 3: Coding accuracy?
        if not claim.codes_are_valid_for_service():
            validation_results.append({
                "check": "coding",
                "status": "FAIL",
                "reason": f"G-code {claim.g_code} invalid for service type {claim.service_type}",
                "fix": f"Use G0470 for established patient mental health visit"
            })

        # Phase 2: ML risk scoring (probabilistic)
        risk_factors = self.extract_risk_features(claim)
        risk_score = self.ml_model.predict_proba(risk_factors)

        # Phase 3: Combine results
        if any(r["status"] == "FAIL" for r in validation_results):
            return {
                "submit": False,
                "reason": "Failed validation checks",
                "validations": validation_results,
                "risk_score": risk_score
            }
        elif risk_score > 0.5:
            return {
                "submit": True,
                "warning": "High denial risk",
                "risk_score": risk_score,
                "recommendations": self.get_recommendations(claim, risk_factors)
            }
        else:
            return {
                "submit": True,
                "risk_score": risk_score
            }
```

**Key insight:** "AI denial prevention" is 80% rule-based validation, 20% ML risk scoring. Don't sell AI magic, sell deterministic checks with ML augmentation.

**Build estimate:** 4-6 months for rule engine (this is the hard part) + 2-3 months for ML model training/deployment.

### Ava's Research Says: "Ambient listening for documentation"

**My read:** This is technically feasible (OpenAI Whisper for transcription, GPT-4 for structuring), but productionizing it for FQHC PPS encounter qualification is non-trivial.

**Engineering challenges:**

1. **Real-time transcription latency:** Can't wait 30 seconds after visit for note to appear. Need <5 second latency.
2. **HIPAA compliance:** Audio recording must be encrypted, transmitted securely, deleted after transcription.
3. **Speaker diarization:** Which utterance is patient vs provider? Critical for consent ("patient stated...").
4. **Medical terminology:** General speech-to-text models make errors on clinical terms. Need fine-tuning.
5. **PPS encounter qualification:** Model must know what documentation is required for PPS encounter (not just "write a SOAP note").

**Build estimate:** 6-9 months for production-grade ambient documentation system (assuming we use existing Whisper/GPT APIs, not training from scratch).

**Alternative:** Partner with existing ambient documentation vendors (Nuance DAX, Abridge, Suki) and add FQHC PPS logic on top of their transcription.

---

## User Technical Sophistication Assessment

**From Ava's research:**
- Directors manage 4-6 disconnected systems (EHR, scheduling, billing, compliance tracking)
- Manual data pulls, Excel tracking, tribal knowledge
- "Can we take this patient now?" requires 30 minutes of phone calls/spreadsheet checks

**My interpretation:** Directors are NOT low-tech. They're high-tech users trapped in low-tech systems.

**Evidence:**
1. They've learned to navigate Epic, NextGen, athenahealth (complex enterprise EHRs)
2. They build complex Excel models (capacity planning, authorization tracking)
3. They coordinate across 4-6 systems daily (integration by human)

**What this means for product design:**
- Don't build dumbed-down UI (they're not novices)
- Do build power-user features (keyboard shortcuts, bulk actions, custom views)
- Do provide API access (they WILL want to export data to Excel for custom analysis)
- Don't hide complexity (they understand the domain, they want visibility)

**Anti-pattern:** Consumer app simplicity ("swipe to approve claim!")
**Correct pattern:** Enterprise productivity app (think Bloomberg Terminal, not Instagram)

---

## Integration Reality Check

### Ava's Research Says: "60% of athenahealth FQHC users request better behavioral health modules"

**My read:** athenahealth has market presence in FQHCs, but their BH modules are weak. This is a partnership/acquisition opportunity.

**Option 1: Partner with athenahealth**
- Build specialized BH module that integrates with athenahealth
- Co-sell to their FQHC customer base (60% already asking for it)
- Faster time-to-market (6-12 months vs 2-3 years greenfield)

**Technical constraints:**
- Dependent on athenahealth API quality (need to audit)
- Revenue share with athenahealth (reduces margin)
- Must conform to their platform constraints

**Option 2: Build standalone, integrate via API**
- Independence (don't depend on athenahealth roadmap)
- Full control of UX/features
- Longer time-to-market (2-3 years)

**Technical constraints:**
- Must integrate with Epic, NextGen, eClinicalWorks, athenahealth (4 different APIs)
- Maintenance burden (vendor API changes break integrations)
- HL7/FHIR standards help, but BH data models not fully standardized yet

**My recommendation:** Hybrid approach
- Start with athenahealth partnership (fastest path to market)
- Build standalone platform in parallel (longer-term independence)
- Use HL7 FHIR as integration layer (standardized, vendor-agnostic)

### Ava's Research Says: "Federal USCDI+ Behavioral Health initiative ($20M)"

**My read:** Federal government is investing in standardized BH data exchange. This is a tailwind for interoperability, but don't wait for it to be perfect.

**Timeline estimate:**
- USCDI+ BH data standards: 2025-2026 (in progress)
- FHIR BH Implementation Guide: 2026-2027
- EHR vendor adoption: 2027-2028 (lagging indicator)

**Engineering implication:** Build for current state (HL7 v2, proprietary APIs) with forward compatibility for FHIR BH. Don't block on federal standards being finalized.

---

## Build vs Buy vs Partner Decision Framework

### Build (Greenfield Platform)

**Pros:**
- Full control of data model, UX, feature roadmap
- Native FQHC + BH logic (not bolt-ons)
- Modern tech stack, no legacy debt

**Cons:**
- 2-3 years to production-grade platform
- $5-10M capital requirement (team of 10-15 engineers)
- Must compete with Epic, NextGen (established vendors)

**Engineering feasibility:** ✅ Feasible, but high risk/high reward

### Buy (Acquire Existing FQHC/BH Platform)

**Candidates:**
- Qualifacts (BH-specific, strong in CMH, weak in FQHC billing)
- Cantata Health (FQHC RCM, weak in BH workflows)
- NextGen (integrated EHR + billing, C- satisfaction rating)

**Pros:**
- Instant customer base, revenue, team
- Faster time-to-market (6-12 months to improve vs 2-3 years to build)

**Cons:**
- Acquisition cost ($50-200M for established platform)
- Technical debt (legacy codebase, integration challenges)
- Cultural integration (team alignment)

**Engineering feasibility:** ✅ Feasible if capital available

### Partner (Integration Layer)

**Candidates:**
- athenahealth (60% FQHC users want better BH modules)
- NextGen (integrated platform, needs UX/feature improvement)

**Pros:**
- Fastest time-to-market (6-12 months)
- Lower capital requirement ($2-5M for integration layer)
- Leverage partner's market presence

**Cons:**
- Dependent on partner API quality
- Revenue share reduces margin
- Limited control over roadmap

**Engineering feasibility:** ✅ Feasible, lowest technical risk

---

## My Recommendation (Engineering POV)

**Phase 1 (0-12 months): Partner + Validate**
- Build integration layer on top of athenahealth (or NextGen)
- Focus on highest-pain point: denial prevention (pre-submission validation)
- Prove product-market fit with 5-10 pilot FQHCs
- Capital requirement: $2-3M (team of 5-7 engineers)

**Phase 2 (12-24 months): Expand + Standalone**
- Add compliance dashboard, capacity planning, 42 CFR Part 2 automation
- Build standalone platform in parallel (for long-term independence)
- Expand to 50-100 FQHCs
- Capital requirement: $5-7M (team of 10-15 engineers)

**Phase 3 (24-36 months): Scale + Acquisition**
- Standalone platform GA (general availability)
- Migrate from integration layer to standalone (phased)
- Consider acquisition of complementary platform (Qualifacts for BH, Cantata for RCM)
- Target: 200-300 FQHCs, $20-50M ARR

**Why this sequencing:**
1. Validate demand before committing to greenfield build (de-risk)
2. Generate revenue during build phase (fund development)
3. Learn from integration layer experience (informs standalone design)
4. Preserve optionality (can stay integration layer if standalone doesn't pencil)

---

## Build Estimate Summary

**Phase 1 (Integration Layer):**
- Denial prevention engine: 4-6 months
- athenahealth API integration: 2-3 months
- Basic UI (claims dashboard): 2-3 months
- **Total: 8-12 months, $2-3M**

**Phase 2 (Standalone Platform MVP):**
- Core data model (Patient, Provider, Encounter, Claim): 3-4 months
- PPS billing engine: 6-9 months
- 42 CFR Part 2 consent engine: 4-6 months
- Credentialing lifecycle: 2-3 months
- Compliance dashboard: 3-4 months
- UDS reporting engine: 2-4 months
- **Total: 20-30 months, $5-10M**

**Phase 3 (Full Platform):**
- Ambient documentation: 6-9 months
- Capacity forecasting (ML): 4-6 months
- Mobile apps: 4-6 months
- Advanced analytics: 4-6 months
- **Total: 18-27 months additional, $5-7M additional**

**Grand total (all phases):** 46-69 months, $12-20M

**Reality check:** This assumes experienced team (senior engineers who've built healthcare platforms before). Junior team would add 50% to timeline.

---

## Technical Risks

**Risk 1: API quality (athenahealth, NextGen)**
- Mitigation: Audit APIs before committing to partnership

**Risk 2: Regulatory change velocity**
- Mitigation: Externalize compliance logic to config (not hardcoded)

**Risk 3: Data breach (42 CFR Part 2 + HIPAA)**
- Mitigation: Security-first design, annual pentesting, bug bounty

**Risk 4: Scalability (1,400 FQHCs)**
- Mitigation: Cloud-native, multi-tenancy, load testing

**Risk 5: Team experience**
- Mitigation: Hire engineers with healthcare domain experience (not just generic web dev)

---

## Next Steps (Phase 2 Council Debate)

**Engineering questions for council:**
1. **Build vs Buy vs Partner?** Standalone vs integration layer vs acquisition?
2. **Which pain point first?** Denial prevention (highest ROI) vs compliance (highest risk) vs capacity planning (highest UX appeal)?
3. **Tech stack?** Node vs Python, AWS vs Azure, monolith vs microservices?
4. **Team composition?** How many engineers with healthcare domain experience available?

**Technical due diligence needed:**
- API audit (athenahealth, NextGen) - quality, rate limits, SLA
- Compliance certification requirements (ONC, FTCA-approved vendor list)
- Security requirements (HIPAA BAA, SOC 2 Type II timeline)

— Marcus Webb, Engineer
Project Mitchell PM001
2026-03-27
