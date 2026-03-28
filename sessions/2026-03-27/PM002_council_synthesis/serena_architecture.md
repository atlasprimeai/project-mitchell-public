# Serena Blackwood - Architecture Perspective
## PM002 Council Debate: FQHC Behavioral Health Director Role Analysis

**Agent:** Serena Blackwood (Architect)
**Focus:** System design, integration patterns, data flow, architectural composability
**Date:** March 27, 2026
**Session:** PM002_council_synthesis

---

## Executive Summary

The core problem FQHC Behavioral Health Directors face is not workflow inefficiency—it's the **absence of a unified data layer** connecting 4-6 disconnected systems. Directors orchestrate data flow across EHR, practice management, billing, credentialing, and state reporting platforms—making every decision based on stale snapshots rather than live intelligence. The solution requires **intelligent orchestration** (not just data aggregation) that answers complex questions like "can we take this patient?" in under 10 seconds across 6 upstream systems.

**Key Architectural Principle:** Compliance as guardrails, not checklists. The system should make it **architecturally impossible to make a non-compliant decision**.

---

## Round 1: Initial Position - The Integration Nightmare

### The Problem: 4-6 Disconnected Systems

Directors orchestrate data flow across:

1. **EHR** (Epic, NextGen, eClinicalWorks, Athena)
   - Patient demographics, diagnoses, treatment plans
   - Clinical notes, assessments, outcome measures
   - BUT: Designed for medical workflows, not behavioral health

2. **Practice Management / Scheduling**
   - Provider availability, appointments, cancellations
   - Wait times, utilization rates
   - BUT: Doesn't show payer eligibility or authorization status

3. **Billing / RCM Platform**
   - Claims status, denials, authorizations
   - Insurance eligibility verification
   - BUT: Disconnected from clinical documentation quality

4. **State Reporting Systems**
   - UDS data extracts, performance metrics
   - Grant reporting requirements
   - BUT: Manual aggregation from multiple sources

5. **Credentialing Databases**
   - License expiration dates, NPDB queries
   - Privileging status, peer reviews
   - BUT: Often maintained in Excel or separate HR system

6. **Excel Trackers** (Frankenstein workarounds)
   - Built because nothing else talks to each other
   - Authorization expiration tracking
   - Capacity planning spreadsheets
   - Compliance audit checklists

### The Critical Workflow Pattern

**Capacity planning requires real-time visibility into:**
- Provider schedules
- Payer mix
- Authorization status
- Utilization rates

**But this data lives in silos.**

Directors manually aggregate it, which means **decisions lag reality by days or weeks**.

### Root Cause: Absence of Unified Data Layer

From a systems perspective, the biggest pain point isn't any single tool—it's the **absence of a unified data layer**.

They need an integration architecture that pulls clinical, financial, and operational streams into one view. Without that, every decision (hiring, contracting, slot allocation) is based on stale snapshots, not live intelligence.

### Success Factor: Architectural Composability

**Can we layer intelligence on top of their existing chaos without requiring them to rip-and-replace?**

This is the key architectural question. FQHCs are locked into multi-year EHR contracts. They can't afford migration downtime. They have federal reporting dependencies (UDS extracts from EHR).

**Rip-and-replace is not an option.**

We need **middleware that abstracts the terrible legacy systems** and creates a clean, intelligent layer on top.

---

## Round 2: Response & Challenges - Middleware Abstraction Strategy

### Building on Ava's Carved-Out Benefits Example

Ava mentioned carved-out mental health benefits as a perfect example of why the **unified data layer is non-negotiable**.

**Scenario:**
- Patient shows up with **Aetna commercial insurance**
- EHR sees "Aetna" and flags it as covered
- BUT: Behavioral health is **carved out** to a completely different MCO (Magellan, Optum, ValueOptions)
- Director needs to know this **BEFORE scheduling**, not after the claim denies

**Current Reality:**
- Schedule appointment
- Submit claim 30 days later
- Claim denies (wrong payer)
- 65% never resubmitted (revenue lost)
- Patient billed incorrectly (satisfaction hit)

**Ideal State:**
- System checks: Aetna commercial → BH carved out to Magellan
- Surface to scheduler: "Need Magellan authorization, not Aetna"
- Block scheduling until authorization obtained
- OR: Offer cash-pay / sliding scale option

### Pushback on "Integrate with Terrible Legacy Systems"

Marcus is right that users are clinically trained, not tech-savvy. But I'd push back on **"integrate with terrible legacy systems" as the constraint**.

**The integration layer can abstract that away.**

We don't need the EHR to be better—we need **middleware that makes the terrible EHR irrelevant to the user experience**.

**Think of it like Plaid for banking:**
- Banks' systems are garbage (COBOL, mainframes, no APIs)
- But Plaid creates a **clean API layer on top**
- Users never interact with the bank's terrible system
- They interact with Plaid's elegant abstraction

**Same principle here:**
- Epic's BH modules are bolt-ons and clunky
- We create a clean orchestration layer on top
- Directors never log into Epic
- They interact with our intelligent dashboard
- We handle the Epic API integration behind the scenes

### Compliance as Guardrails, Not Checklists

Rook's audit survivability point is critical—it needs to be **architecturally impossible to make a non-compliant decision**.

**Example: License Expiration**

**Bad Design (Checklist Model):**
- System shows: "⚠️ Dr. Chen's license expires tomorrow"
- Hopes the director notices and takes action
- Dr. Chen gets scheduled anyway
- Violation occurs

**Good Design (Guardrail Model):**
- Dr. Chen's license expires 6/15
- System **blocks scheduling with Dr. Chen** on 6/16
- Not "warns"—**blocks**
- Until credentialing is renewed, Dr. Chen is unavailable in the system
- Override requires Medical Director approval + audit trail entry

**Architecture Principle:** Compliance checks run **before** actions, not after.

### The Complexity IS the Moat

Ava mentioned the integration challenge (G-codes, Medicaid MCO + state wraparound, carved-out benefits) as validation of the thesis.

**I agree completely: the complexity IS the moat.**

If we solve FQHC + BH integration, no one can easily replicate it because:
- Domain knowledge barrier is very high
- Integration maintenance is ongoing (Epic pushes updates, APIs break)
- Compliance requirements change faster than software
- You need healthcare + billing + compliance + software expertise

**This is defensible.**

It's not a feature anyone can copy in 3 months. It's deep, hard, complex work that takes years to master.

---

## Round 3: Final Synthesis - Intelligent Orchestration Architecture

### Where the Council Agrees

- Integration complexity is the core problem ✓ (all 4 of us centered on this)
- Current systems are fundamentally inadequate ✓ (no one defended the status quo)
- Compliance must be architectural, not aspirational ✓ (Rook and I are aligned)
- The solution must work with legacy systems ✓ (Marcus and I agree on abstraction layer)

### Where I Still Push: Intelligent Orchestration

The middleware abstraction layer is **table stakes, not the differentiator**.

The differentiator is **intelligent orchestration**.

### The 10-Second Patient Scheduling Use Case

**Question:** "Can we schedule Patient A for therapy next week?"

**Current Reality:** 30 minutes of manual work
- Check EHR for patient demographics
- Verify insurance in billing system
- Check if authorization required (payer-specific rules)
- Look up authorization status (approved? expired?)
- Match to appropriate provider (language, specialty, cultural competence)
- Verify provider credentialing is current (HR database)
- Find next available slot (scheduling system)
- Check if encounter qualifies for PPS billing (documentation requirements)
- Generate billing prep (correct G-code, T-code, modifiers)

**System Orchestration (Under 10 Seconds):**

```
1. Check insurance eligibility (EHR integration)
2. Identify carved-out benefits (payer database lookup)
3. Verify authorization status (billing platform API)
4. Match to culturally/linguistically appropriate provider (credentialing + patient preference)
5. Confirm provider credentialing is current (HR database check)
6. Find next available slot (scheduling system query)
7. Pre-qualify encounter for PPS billing (documentation requirements check)
8. Generate compliant scheduling workflow (correct codes, modifiers, consent requirements)
```

**In under 10 seconds.**

That's not a dashboard—that's an **intelligent agent** that orchestrates across 6 systems to answer one question.

**The UI is just the presentation layer.**

### Recommended System Architecture

**Layer 1: Integration Foundation**

**HL7 FHIR Connectors for EHRs:**
- Epic (FHIR R4 support)
- NextGen (FHIR R4 support)
- eClinicalWorks (FHIR STU3)
- Athena (FHIR R4 support)

**RESTful APIs for Billing Platforms:**
- athenahealth (API Gateway)
- Kareo (REST API)
- Cantata (HL7 integration)
- Qualifacts (CareLogic API)

**Payer Database Integration:**
- Availity (eligibility verification)
- Change Healthcare (authorization management)

**HR/Credentialing System Connectors:**
- Custom integrations (most FQHCs use in-house systems or Excel)
- NPDB API (National Practitioner Data Bank queries)

**Layer 2: Compliance Engine**

**Credentialing Status Checks:**
- License expiration tracking (2-year cycles)
- NPDB query dates (required every 2 years)
- Privileging dates (board approval)
- DEA verification (for prescribers)

**Authorization Validation:**
- Pre-auth requirements (payer-specific rules)
- Approved sessions (track utilization)
- Expiration dates (proactive alerts)

**42 CFR Part 2 Consent Management:**
- SUD record segregation (database-level partitioning)
- Disclosure logging (all disclosures tracked)
- Re-disclosure blocking (can't forward without new consent)

**Scope of Project Tracking:**
- Approved services/sites (from HRSA CIS)
- Actual delivery (from EHR data)
- Flag violations before claims submitted

**Documentation Completeness Validation:**
- Encounter qualification requirements (PPS rules)
- PHQ-9 scores present
- Diagnostic codes documented
- Treatment plan updated
- Medical necessity statement

**Layer 3: Orchestration Intelligence**

**Patient Scheduling Workflow:**
```
User: "Schedule Patient A"
↓
Orchestration Engine:
  - Query EHR (demographics, diagnosis)
  - Query billing (insurance eligibility, authorization status)
  - Query payer database (carved-out benefits check)
  - Query credentialing (provider match by language/specialty)
  - Query HR (verify credentialing current)
  - Query scheduling (find next available slot)
  - Run compliance checks (all pass?)
  - Generate billing prep (correct codes/modifiers)
↓
Output: "Ready to schedule with Dr. Reyes, Tue 2pm. All checks passed."
[Schedule] button appears
```

**Denial Prevention Pre-Submission Checks:**
- Coding validation (correct CPT/G-code/T-code)
- Modifier validation (required modifiers present)
- Documentation requirements (encounter qualifies for PPS?)
- Authorization confirmation (services authorized, not expired)
- Eligibility verification (patient covered on date of service)

**Credentialing Cycle Automation:**
- 90-day warnings (license expires in 90 days)
- Renewal workflows (initiate re-credentialing)
- NPDB scheduling (query NPDB 60 days before expiration)
- Board approval tracking (escalate to QI/QA Committee)

**Capacity Forecasting:**
- Historical trends (seasonal patterns, referral velocity)
- Predictive alerts (capacity dropping below threshold)
- Mitigation recommendations (hire, telehealth, extend hours)

**Layer 4: Presentation (UI)**

**Morning Brief Dashboard:**
- Crisis vs stable status (traffic light: green/yellow/red)
- Decisions needed (patient referrals, authorization declines)
- Proactive alerts (credentialing expiring, capacity thresholds)

**Conversational Action Paths:**
- Narrative presentation (not data grids)
- "You have 3 decisions to make" (not "Utilization: 87%")
- One-click actions with embedded compliance checks

**Compliance Sidebar:**
- Audit readiness score (percentage)
- Upcoming deadlines (UDS, FTCA, QI/QA meeting)
- One-click reports (credentialing records, Part 2 compliance, UDS preview)

**Crisis Mode:**
- Auto-activation when capacity thresholds crossed
- Immediate action recommendations (telehealth, partner referrals, extend hours)
- Leadership notifications (email + SMS)

---

## Architectural Priorities & Roadmap

### What I'd Prioritize

**Phase 1: Integration Layer (First)**
1. **Integration testing infrastructure**
   - Prove we can reliably connect to Epic/NextGen/Athena
   - Build test harness (mock EHR responses)
   - Validate API stability across vendor updates

2. **Compliance engine**
   - Credentialing checks (license expiration, NPDB)
   - Authorization validation (pre-auth requirements, approved sessions)
   - Part 2 consent management (SUD record segregation)

3. **Orchestration logic**
   - Patient scheduling use case (10-second answer)
   - Denial prevention pre-checks (claim scrubbing)

4. **Dashboard UI**
   - Narrative presentation of orchestrated data
   - Compliance sidebar (audit readiness)
   - Crisis mode (auto-activation)

### Backend Sophistication, Frontend Simplicity

**Most software projects fail because they build UI-first and bolt on logic later.**

We need **backend sophistication with frontend simplicity**.

The intelligence lives in the orchestration engine. The UI is just presenting the results in a way that reduces cognitive load.

**Don't build a pretty dashboard with dumb backend.**
**Build a smart backend with a simple, elegant frontend.**

---

## Architectural Risks & Mitigations

### Risk 1: Vendor Lock-In to EHR Platforms

**Problem:** If Epic changes their API and breaks our integrations, we're at their mercy.

**Mitigation:**
- **HL7 FHIR as the integration standard** (not vendor-specific APIs)
- FHIR is an industry standard, Epic/NextGen/Athena all support it
- Contractual SLAs with EHR vendors for API stability
- Build abstraction layer (so if Epic API changes, we only update our Epic connector, not entire codebase)

### Risk 2: Perfectionism vs. Reliability

**Problem:** Marcus raised this—orchestration depends on 6 upstream systems all being available. If any one has downtime, the entire workflow breaks.

**Mitigation: Graceful Degradation**

**If billing system is down:**
- Proceed with scheduling
- Show warning: "Authorization status may be stale (billing system offline)"
- Flag for manual verification before claim submission

**If payer database is stale:**
- Proceed with warning: "Insurance info might not be current"
- Recommend calling insurance to verify

**If credentialing system unavailable:**
- Block high-risk actions (can't schedule with expired license)
- Allow low-risk actions with warnings (provider re-credentialing in-progress)

**Principle:** Partial information that's honest about its limitations is better than no information.

**Perfectionism is the enemy of reliability in healthcare.**

### Risk 3: Integration Maintenance Burden

**Problem:** Marcus raised this—who maintains the integrations when Epic pushes an update and breaks the API?

**Mitigation:**
- **Automated integration testing** (every integration has a test suite)
- **Monitoring + alerting** (detect integration failures before users report them)
- **Feature flags** (disable broken features without redeploying)
- **Dedicated support team** (integration maintenance is ongoing, not one-time)

**This is a support burden that needs to be priced into the model.**

Integration maintenance is **permanent infrastructure work**. It's not a project with an end date—it's ongoing operations.

**We need to budget for a dedicated integrations team.**

### Risk 4: Override Friction

**Problem:** Rook's "block scheduling with expired credentials" is right in principle, but Marcus raised implementation reality:

**What if Dr. Chen is the only Spanish-speaking therapist and a suicidal patient needs same-day care?**

Do we block it and risk a safety incident, or allow an override with heightened documentation?

**Mitigation: Tiered Override Authority**

**Green Light (No Override Needed):**
- Provider fully credentialed, slots available, authorization approved
- Patient eligibility confirmed, insurance active
- Documentation complete, billing codes valid

**Yellow Light (Director Override Authorized):**
- Provider re-credentialing in-progress but current license valid
- Authorization pending but patient is established
- Insurance eligibility can't be verified but patient states they're covered
- **Requires:** Documented justification + audit trail entry

**Red Light (Medical Director Approval Required):**
- Provider license expired (high risk)
- Patient has no authorization and service requires pre-auth
- Part 2 patient missing consent
- **Requires:** Medical Director approval + audit trail + QI/QA escalation

**Black Light (System Blocks, No Override Possible):**
- Provider not credentialed at all (FTCA coverage void, malpractice liability)
- Service outside Scope of Project (HRSA violation)
- Claim submission with known error (won't transmit invalid claims)

**Principle:** Override friction must be proportional to risk.

Real-world healthcare has exceptions. Our system needs to handle them without creating compliance holes.

---

## Technical Architecture Specifications

### Platform: Progressive Web App (PWA)

**Why PWA?**
- Works on any device (desktop, tablet, mobile)
- Offline mode with sync (rural connectivity issues)
- Real-time updates (WebSocket connections)
- No app store approval delays

### Integration: API-First Design

**RESTful APIs:**
- OAuth 2.0 authentication
- Rate limiting (prevent system overload)
- Webhook support (real-time event notifications)

**HL7 FHIR:**
- Industry standard for health data interoperability
- Epic, NextGen, Athena all support FHIR R4
- Patient, Practitioner, Appointment, Claim resources

### Data Storage

**PostgreSQL (Structured Data):**
- Patients, appointments, claims
- Credentialing records, authorization tracking
- Audit logs (immutable, timestamped)

**Redis (Real-Time Cache):**
- Capacity status (available slots)
- Sync status (last successful integration)
- Session state (user context)

**Encryption at Rest:**
- HIPAA/42 CFR Part 2 compliance
- AES-256 encryption
- Key management (AWS KMS or HashiCorp Vault)

### Security

**SOC 2 Type II Certified Infrastructure:**
- Annual audit of security controls
- Penetration testing
- Incident response plan

**HIPAA Business Associate Agreement (BAA):**
- Signed with all FQHCs
- Covers PHI handling, breach notification

**42 CFR Part 2 Compliance:**
- SUD record segregation (database-level partitioning)
- Consent management (explicit consent for disclosures)
- Disclosure logging (all accesses tracked)

**Multi-Factor Authentication (MFA):**
- Required for all users
- SMS, authenticator app, or hardware token

**Role-Based Access Control (RBAC):**
- Director, billing staff, front desk, clinical staff
- Least privilege principle

---

## Council Vote: Should We Build This?

**🏛️ Serena: YES**

**Rationale:** Market gap is real, architectural approach is sound.

**Key Validation:**
- No platform solves both FQHC billing + BH workflows
- Middleware abstraction layer is technically feasible
- Intelligent orchestration is the differentiator
- Compliance as guardrails is architecturally sound

**Confidence Level:** 90%

**Remaining Concerns:**
- Integration maintenance burden (requires dedicated team)
- Vendor lock-in risk (mitigated by FHIR standard)
- Graceful degradation complexity (needs careful design)

**Recommendation:** Build Phase 1 (MVP) with 3-5 beta FQHCs to validate integration reliability and orchestration performance.

---

## Architecture KPIs (PM002 Session)

**System Design Patterns Developed:** 4
- Middleware abstraction layer
- Intelligent orchestration engine
- Compliance guardrail architecture
- Tiered override system

**Integration Analyses Completed:** 6
- EHR platforms (Epic, NextGen, Athena, eCW)
- Billing platforms (athenahealth, Kareo)
- Payer databases (Availity, Change Healthcare)

**Architectural Risk Assessments:** 4
- Vendor lock-in
- Perfectionism vs. reliability
- Integration maintenance burden
- Override friction

**Tool Calls:** 20
**Reports Generated:** 1 (this document)

---

**Document Status:** Final
**Confidence Level:** 90%
**Last Updated:** March 27, 2026
