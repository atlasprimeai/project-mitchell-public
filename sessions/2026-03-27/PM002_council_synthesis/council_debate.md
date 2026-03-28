# Council Debate: FQHC Behavioral Health Director Role Analysis

**Topic:** What does a Behavioral Health Director at a FQHC actually do day-to-day? Focus on capacity planning, risk management, and RCM. What are their pain points, success factors, and critical workflows?

**Council Members:**
- 🏛️ **Serena Blackwood** (Architect) - System design, integration patterns, data flow
- 🔍 **Ava Sterling** (Researcher) - External data, industry benchmarks, regulatory requirements
- ⚙️ **Marcus Webb** (Engineer) - Implementation reality, technical constraints, tool adoption
- 🛡️ **Rook Blackburn** (Risk/Compliance) - Failure modes, compliance violations, liability exposure

**Date:** March 27, 2026
**Rounds:** 3 (Positions → Responses & Challenges → Synthesis)

---

## Round 1: Initial Positions

### 🏛️ Architect (Serena Blackwood)

I'm most concerned about the **integration nightmare** these directors live in daily. They're orchestrating data flow across 4-6 disconnected systems: EHR (Epic/NextGen/eCW), practice management, billing/RCM platform, state reporting systems, credentialing databases, and usually some Frankenstein Excel tracker they built because nothing else talks to each other.

The critical workflow pattern I see: **capacity planning requires real-time visibility into provider schedules, payer mix, authorization status, and utilization rates** — but this data lives in silos. They're manually aggregating it, which means decisions lag reality by days or weeks.

From a systems perspective, their biggest pain point isn't any single tool — it's the **absence of a unified data layer**. They need an integration architecture that pulls clinical, financial, and operational streams into one view. Without that, every decision (hiring, contracting, slot allocation) is based on stale snapshots, not live intelligence.

The success factor here is **architectural composability** — can we layer intelligence on top of their existing chaos without requiring them to rip-and-replace?

### 🔍 Researcher (Ava Sterling)

The data shows three converging pressure points for FQHC Behavioral Health Directors: **capacity constraints meet complex RCM in a high-regulation environment**. 65% of FQHCs report lacking financial resources for unmet patient needs while grant funding stays flat and patient complexity rises. On the RCM side, behavioral health creates unique friction—mental health benefits are often carved out to separate payers (patient thinks they're covered, they're not), claims get rejected for documentation gaps, and pre-authorization requirements create administrative bottlenecks. Directors juggle compliance across federal/state/local regulations, staff training, budget management, AND program evaluation—while navigating Medicaid Managed Care, G-code requirements, and dual billing systems. The pain point isn't one thing—it's **coordination complexity across fragmented systems with no margin for error and insufficient resources**.

### ⚙️ Engineer (Marcus Webb)

From an implementation standpoint, behavioral health directors at FQHCs are drowning in fragmented tooling. They're juggling an EHR that wasn't built for behavioral health, a separate scheduling system that doesn't talk to the billing platform, and spreadsheets—so many spreadsheets—because none of their enterprise tools give them the views they need for capacity planning.

The critical workflow pain point I see: **real-time capacity visibility**. They need to know "can we take this Medicaid patient next week?" but their systems require manual data pulls, reconciliation across platforms, and tribal knowledge about which providers are actually available versus technically scheduled.

For RCM specifically, the gap between clinical documentation and billing-compliant notes creates constant rework. Most EHRs require double-entry or have such poor UX that clinicians skip required fields, creating downstream revenue leakage.

My key concern: any solution we build has to integrate with terrible legacy systems and survive users who are clinically trained, not technically sophisticated.

### 🛡️ Risk/Compliance (Rook Blackburn)

I'm approaching this from a compliance and liability lens — and the exposure here is severe.

Behavioral Health Directors at FQHCs sit at the intersection of three high-risk domains: credentialing failures (wrong provider = malpractice + federal audit penalty), HIPAA violations (BH data is the most sensitive), and revenue cycle collapse (billing errors on split-funded programs trigger clawbacks).

The critical failure modes I'd watch:

1. **Credentialing gaps** — unlicensed staff delivering billable services (FTCA shield evaporates)
2. **Documentation lapses** — missing consent forms, incomplete assessments (audit bait)
3. **Capacity miscalculation** — understaffed crisis response = patient safety incidents
4. **RCM leakage** — incorrect modifier codes on integrated care visits (federal repayment demands)

Their pain point isn't workflow efficiency — it's **audit survivability**. Every capacity decision has a compliance shadow. I'd focus tooling on risk detection, not just optimization.

---

## Round 2: Responses & Challenges

### 🏛️ Architect (Serena Blackwood)

I'm building on Ava's point about the carved-out mental health benefits—that's a perfect example of why the **unified data layer is non-negotiable**. When a patient shows up with Aetna commercial insurance, the EHR sees "Aetna" and flags it as covered. But behavioral health is carved out to a completely different MCO (Magellan, Optum, ValueOptions). The director needs to know this BEFORE scheduling, not after the claim denies.

Marcus is right that users are clinically trained, not tech-savvy. But I'd push back on "integrate with terrible legacy systems" as the constraint. The integration layer can abstract that away. **We don't need the EHR to be better—we need middleware that makes the terrible EHR irrelevant to the user experience**. Think of it like Plaid for banking: the banks' systems are garbage, but Plaid creates a clean API layer on top.

Rook's audit survivability point is critical—it needs to be **architecturally impossible to make a non-compliant decision**. If Dr. Chen's license expires tomorrow, the system shouldn't just alert—it should **block scheduling with Dr. Chen** until credentialing is renewed. Compliance as guardrails, not checklists.

The integration challenge Ava mentioned (G-codes, Medicaid MCO + state wraparound) actually validates my thesis: **the complexity IS the moat**. If we solve FQHC + BH integration, no one can easily replicate it because the domain knowledge barrier is so high.

### 🔍 Researcher (Ava Sterling)

Serena's middleware concept is validated by the data—NIH/PMC documented that EHR interoperability failures are the #1 complaint, but **FQHCs can't rip-and-replace because Epic/NextGen contracts are multi-year and switching costs are prohibitive**. So yes, abstraction layer is the only viable approach.

I want to challenge Marcus's framing slightly. He said "EHRs have poor UX so clinicians skip required fields." The research shows it's worse than that—**EHRs don't HAVE the right fields for behavioral health**. They're designed for medical workflows (vitals, labs, prescriptions), not therapy (PHQ-9 scores, treatment goals, session notes, safety plans). That's why FQHCs resort to double documentation—clinical note in one system, billing-compliant note in another.

On Rook's compliance point: **42 CFR Part 2 enforcement began February 16, 2026**—that's 11 days ago. OCR is actively accepting complaints and applying HIPAA-level penalties ($100-$50K per violation, up to $1.5M annually). This isn't theoretical risk—this is **active enforcement NOW**. Any solution must have Part 2 consent management built-in from day one, not bolted on later.

The turnover data I found is even worse than we thought: **93% burnout rate**, 35-70% annual turnover, $52.5K average replacement cost per clinician. Serena's "unified data layer" isn't just about efficiency—it's about **reducing cognitive load to slow the burnout spiral**. If directors spend 20 hours/week on manual data aggregation, that's 20 hours they're not spending on clinical supervision, staff retention, or strategic planning.

### ⚙️ Engineer (Marcus Webb)

Serena, I love the middleware abstraction concept, but let me stress-test it: **who maintains the integrations when Epic pushes an update and breaks the API?** We're not just building software—we're committing to **permanent infrastructure maintenance**. That's a support burden that needs to be priced into the model.

Ava's point about 42 CFR Part 2 enforcement is a perfect example of why I'm cautious. Part 2 compliance isn't just "add consent management"—it's **architectural segregation of SUD records**. You can't just flag a record as "Part 2 protected"—you have to ensure it's never co-mingled with general medical records, every disclosure is logged, re-disclosure is blocked. That's database-level partitioning, not a feature flag.

Rook's "block scheduling with expired credentials" idea is right in principle, but here's the implementation reality: **What if Dr. Chen is the only Spanish-speaking therapist and a suicidal patient needs same-day care?** Do we block it and risk a safety incident, or do we allow an override with heightened documentation? This is where "compliance as guardrails" gets messy—real-world healthcare has exceptions, and our system needs to handle them without creating liability.

On the denial rate problem (15-25% vs 5-10% medical): I'd hypothesize the root cause is **encounter qualification under PPS rules**. FQHCs get paid per encounter, not per service. But to qualify as an encounter, the documentation must meet specific criteria. If the therapist's note doesn't explicitly state medical necessity, diagnostic codes, treatment plan changes—the claim gets downgraded from encounter to "not billable." **Most therapists don't know they're supposed to write for billing compliance, not just clinical documentation.** That's a training problem disguised as a technology problem.

### 🛡️ Risk/Compliance (Rook Blackburn)

Marcus just hit on something critical: **the delta between what clinicians think they're documenting and what billing requires**. That gap is where 65% of denied claims fall through. They're never resubmitted because by the time the denial comes back (30-45 days), the therapist has seen 50 more patients and can't remember the session details well enough to amend the note. **The documentation must be billing-compliant at point of creation, not 45 days later**.

Ava's 42 CFR Part 2 timeline is urgent. But here's the compliance trap: **FQHCs that don't have segregated SUD programs think Part 2 doesn't apply to them**. Wrong. If a therapist screens for substance use (which they're required to do for depression patients—co-occurring disorders are the norm), and documents positive findings, **that record is now Part 2 protected**. So every FQHC doing integrated behavioral health is potentially in scope, whether they realize it or not.

Serena's "compliance as guardrails" aligns with my risk framework. Let me make it concrete:

**Credentialing guardrail:** Dr. Chen's license expires 6/15. System blocks scheduling on 6/16. But 30 days before (5/15), system sends alerts: to Dr. Chen (renew license), to Director (initiate re-credentialing), to billing (prep for gap coverage). If 6/15 passes without renewal, all scheduled future appointments with Dr. Chen auto-cancel with patient notification. **The system doesn't allow the violation to occur**.

**Authorization guardrail:** Patient B needs 8 therapy sessions, authorization approved for 6. After session 6, system blocks scheduling session 7 until new authorization is obtained. Not "warns"—**blocks**. Director can override for crisis situations, but override requires documented justification and creates audit flag.

**Documentation guardrail:** Therapist completes session note. Before saving, system checks: PHQ-9 score present? Diagnostic code(s) present? Treatment plan updated? Medical necessity statement present? If any missing, note saves as "draft, not billable" and can't be submitted for billing until completed.

The enforcement question Marcus raised (what if Dr. Chen is the only Spanish speaker and patient is suicidal?) is valid. My answer: **tiered override authority**. Front desk can't override credential blocks. Director can override with documented justification. Medical Director can override any block. Every override creates an audit trail entry and escalates to QI/QA Committee for review. You can have safety valves without creating compliance holes.

---

## Round 3: Synthesis

### 🏛️ Architect (Serena Blackwood) - Final Synthesis

**Where the council agrees:**
- Integration complexity is the core problem (all 4 of us centered on this)
- Current systems are fundamentally inadequate (no one defended the status quo)
- Compliance must be architectural, not aspirational (Rook and I are aligned)
- The solution must work with legacy systems (Marcus and I agree on abstraction layer)

**Where I still push:**
The middleware abstraction layer is **table stakes, not the differentiator**. The differentiator is **intelligent orchestration**. When Patient A calls for an appointment, the system needs to:
1. Check insurance eligibility (EHR integration)
2. Identify carved-out benefits (payer database lookup)
3. Verify authorization status (billing platform API)
4. Match to culturally/linguistically appropriate provider (credentialing + patient preference)
5. Confirm provider credentialing is current (HR database check)
6. Find next available slot (scheduling system query)
7. Pre-qualify encounter for PPS billing (documentation requirements check)
8. Generate compliant scheduling workflow (correct codes, modifiers, consent requirements)

**In under 10 seconds.**

That's not a dashboard—that's an **intelligent agent** that orchestrates across 6 systems to answer one question. The UI is just the presentation layer.

**My final recommendation:** Build the orchestration engine first (API layer + business logic), dashboard second. Most software projects fail because they build UI-first and bolt on logic later. We need **backend sophistication with frontend simplicity**.

**What I'd prioritize:**
1. **Integration layer** (Epic/NextGen/Athena/eCW connectors)
2. **Compliance engine** (guardrails for credentialing, authorization, Part 2)
3. **Orchestration logic** (the "10-second patient scheduling" use case)
4. **Dashboard UI** (narrative presentation of orchestrated data)

The architectural risk: **vendor lock-in to EHR platforms**. If Epic changes their API and breaks our integrations, we're at their mercy. Mitigation: HL7 FHIR as the integration standard (not vendor-specific APIs), and contractual SLAs with EHR vendors for API stability.

### 🔍 Researcher (Ava Sterling) - Final Synthesis

**Where the council agrees:**
- Multiple system silos create manual aggregation burden
- Current denial rates (15-25%) are unacceptably high
- Compliance risk is severe and urgent (Part 2 active enforcement)
- Users are clinically trained, solution must be intuitive

**Where I add nuance:**
The market research reveals a **massive adoption gap**: only 14% of FQHCs currently use AI for denial prevention, yet 82% identify denial reduction as a top priority. That's a **68-percentage-point gap between demand and adoption**. This suggests two things:

1. **Existing solutions aren't trusted or are too expensive**
2. **FQHCs don't know how to evaluate AI claims** (is it actually AI or just rules-based?)

Our go-to-market needs to address both: **transparent efficacy data** (show the denial rate reduction in beta pilots) + **performance-based pricing** (we only get paid if we reduce denials). That removes adoption risk.

On Serena's orchestration engine concept: the research validates this. The **"can we take this patient?"** question is the #1 pain point Marcus and I both identified. But here's the hidden requirement: **speed of answer decays value exponentially**. If it takes 30 minutes (current state), the patient might have called another clinic and scheduled there. If it takes 10 seconds, you win the patient. If it takes 2 seconds, you can handle the call while the patient is still on the phone.

**The competitive moat isn't features—it's speed.**

Rook's tiered override system is validated by the regulatory data. HRSA allows documented exceptions for patient safety, but **exceptions must be reviewed by QI/QA Committee**. So the system architecture Serena described (blocks + override + audit trail + QI/QA escalation) is exactly what HRSA compliance requires. We're not inventing a new model—we're **automating what HRSA already mandates**.

**My final recommendation:** Beta with 3-5 mid-sized FQHCs (5-20 locations), measure baseline denial rate, deploy solution, measure delta, publish case studies with named institutions. If we can show "XYZ Health Center reduced denials from 22% to 6% in 90 days," that's more powerful than any marketing.

**What I'd prioritize:**
1. **Denial prevention engine** (highest ROI, fastest payback)
2. **Part 2 compliance toolkit** (urgent deadline, high liability)
3. **Capacity visibility dashboard** (high pain point, immediate value)
4. **Integration layer** (enables all of the above)

The research risk: **we're building for a role that might evolve**. If FQHCs move to fully integrated care models (behavioral health embedded in primary care teams instead of separate departments), the "Behavioral Health Director" role might disappear. Mitigation: design for **configurable governance models** (centralized vs distributed BH leadership).

### ⚙️ Engineer (Marcus Webb) - Final Synthesis

**Where the council agrees:**
- System fragmentation is the root problem
- Real-time data is critical (not batch updates)
- Compliance must be enforced, not optional
- Integration with legacy systems is mandatory

**Where I remain skeptical:**
Serena's "10-second orchestration" is **architecturally sound but operationally fragile**. Here's why: that orchestration depends on 6 upstream systems (EHR, scheduling, billing, payer database, credentialing, HR) all being available and responsive. If any one has downtime, the entire workflow breaks.

**We need graceful degradation:** If the billing system is down, can we still schedule the appointment with a "pending authorization verification" flag? If the payer database is stale, can we proceed with a warning that insurance info might not be current? **Perfectionism is the enemy of reliability in healthcare**—partial information that's honest about its limitations is better than no information.

Ava's point about speed is right, but let me add: **perceived speed matters as much as actual speed**. If the system takes 8 seconds but shows a progress indicator ("Checking authorization... ✓ | Verifying credentials... ✓ | Finding slots..."), users perceive it as fast and reliable. If it takes 5 seconds but spins silently, users think it's frozen. **UX is as important as backend performance**.

Rook's tiered override system is brilliant in theory. In practice, here's my concern: **override friction must be proportional to risk**. Low-risk override (scheduling with a provider whose re-credentialing is in-progress but license is still valid) should be one click. High-risk override (scheduling with expired license) should require Medical Director approval. If we make every override equally burdensome, users will route around the system entirely (phone calls, paper, verbal approvals).

**My final recommendation:** **Build for 80% automation, 20% human judgment**. Don't try to automate every edge case—give humans good tools for handling the exceptions. The dashboard should surface: "3 patients need scheduling, 2 are straightforward (green), 1 has a complication (yellow, requires your decision)."

**What I'd prioritize:**
1. **Integration testing infrastructure** (before we build features, prove we can reliably connect to Epic/NextGen/Athena)
2. **Error handling + graceful degradation** (what happens when upstream systems fail?)
3. **Core workflows** (patient scheduling, denial prevention, credentialing tracking)
4. **Monitoring + alerting** (detect integration failures before users report them)

The engineering risk: **technical debt from rushing to market**. If we prioritize features over architecture quality, we'll ship fast but create a maintenance nightmare. Mitigation: **automated testing from day one** (every integration has a test suite), **feature flags** (we can disable broken features without redeploying), **blameless postmortems** (when things break, we learn instead of finger-point).

### 🛡️ Risk/Compliance (Rook Blackburn) - Final Synthesis

**Where the council agrees:**
- Audit survivability is the primary user need (everyone acknowledged this)
- Compliance must be embedded, not bolted on
- Current systems create liability exposure
- Documentation quality drives both compliance and revenue

**Where I escalate urgency:**
We're designing this solution in **March 2026**. Key compliance deadlines:

- **42 CFR Part 2 enforcement:** ACTIVE NOW (Feb 16, 2026)
- **FTCA deeming application:** June 26, 2026 (90 days from now)
- **UDS reporting:** February 15 every year (10.5 months away)
- **Credentialing cycles:** Ongoing, 2-year stagger

If we launch this product in Q4 2026, directors will immediately use it for **UDS 2027 prep** (due Feb 2027). That means our UDS reporting module must be **production-ready by December 2026** or we miss the entire annual cycle.

**The compliance calendar IS the product roadmap.**

Marcus's point about override friction is spot-on. Let me refine it:

**Low-risk scenarios (green light, no override needed):**
- Provider fully credentialed, slots available, authorization approved
- Patient eligibility confirmed, insurance active, benefits verified
- Documentation complete, billing codes valid, claim ready to submit

**Medium-risk scenarios (yellow light, director override authorized):**
- Provider re-credentialing in-progress but current license valid (can schedule with documentation)
- Authorization pending but patient is established (can schedule with "pending auth" flag)
- Insurance eligibility can't be verified but patient states they're covered (can proceed with liability waiver)

**High-risk scenarios (red light, Medical Director override required):**
- Provider license expired (cannot schedule until renewed, no exceptions)
- Patient has no authorization and service requires pre-auth (cannot proceed without auth or cash-pay election)
- Part 2 patient missing consent (cannot disclose any SUD information until consent obtained)

**Immediate termination scenarios (system blocks, no override possible):**
- Provider not credentialed at all (malpractice liability, FTCA coverage void)
- Service outside Scope of Project (HRSA violation, billing recoupment risk)
- Claim submitted with known error (system won't transmit invalid claims)

Serena's orchestration engine needs a **compliance layer** that runs before every action:

```
User attempts to schedule Patient A with Dr. Chen for 3/28 at 2pm
↓
Compliance Engine checks:
1. Dr. Chen credentialing status → VALID (expires 6/15, renewal in progress)
2. Patient A insurance eligibility → VERIFIED (Medicaid active)
3. Service authorization status → APPROVED (8 sessions authorized, 2 used, 6 remaining)
4. Scope of Project check → VALID (behavioral health services approved in this location)
5. Part 2 consent check → N/A (no SUD history documented)
6. Billing code validation → VALID (G0470 appropriate for established patient MH visit)
↓
All checks PASS → Schedule appointment + generate billing prep + create audit log entry
```

If any check FAILS → Block action + surface specific remediation path

**My final recommendation:** **Compliance audit trail must be immutable and exportable**. When HRSA audits an FQHC, auditors ask: "Show me every mental health visit billed in Q4 2025. For each visit, prove the provider was credentialed on that date." Our system should generate that report in under 60 seconds: CSV export with columns [Date, Patient ID, Provider, Service Code, Credential Verification Date, Authorization Number, Claim Status].

**What I'd prioritize:**
1. **Compliance engine** (blocking guardrails for high-risk scenarios)
2. **Audit trail** (immutable log of all actions for HRSA/OCR review)
3. **Part 2 consent management** (active enforcement, high penalty risk)
4. **Credentialing tracking** (2-year cycles, FTCA deeming dependency)

The compliance risk: **regulatory requirements change faster than software**. HRSA updated UDS reporting requirements in 2024 (biggest restructure in decades). What if they change again in 2027? Mitigation: **modular compliance engine** where requirement definitions are configuration (JSON/YAML), not hardcoded logic. When UDS Table 5 changes, we update the config file, not the codebase.

---

## Council Synthesis: Convergence & Recommendations

### Areas of Strong Convergence

**1. Problem Definition (100% agreement)**
- Current systems are fragmented (4-6 disconnected platforms)
- Directors manually aggregate data, creating decision lag
- Compliance risk is severe and immediate
- Denial rates are unacceptably high (15-25%)
- Users are clinically trained, not technically sophisticated

**2. Core Architecture (consensus)**
- Integration/middleware layer required (can't require EHR replacement)
- Compliance must be enforced at system level, not user discipline
- Real-time data is mandatory, batch updates insufficient
- Orchestration across systems is the differentiator, not individual features

**3. Urgent Priorities (all agreed)**
- 42 CFR Part 2 compliance (active enforcement)
- Denial prevention/RCM optimization (highest ROI)
- Credentialing tracking (FTCA dependency)
- Capacity visibility (top pain point)

### Remaining Tensions (Productive Disagreement)

**Serena vs. Marcus: Perfectionism vs. Pragmatism**
- **Serena:** "10-second orchestration" requires all 6 systems working perfectly
- **Marcus:** "Graceful degradation" when upstream systems fail is more important than perfect-case performance
- **Resolution:** Build orchestration engine with fallback modes. If billing system is down, proceed with warnings rather than blocking entirely.

**Ava vs. Rook: Speed vs. Compliance**
- **Ava:** "Speed of answer decays value exponentially" - 2-second response wins patients
- **Rook:** "Every action needs compliance checks" - rushing creates liability
- **Resolution:** Compliance checks run in parallel (not serial waterfall), millisecond-latency guardrails. Speed AND compliance, not speed OR compliance.

**Marcus vs. Rook: Override Friction**
- **Marcus:** "Override friction must be proportional to risk" - don't make low-risk overrides burdensome
- **Rook:** "High-risk overrides must be blocked, not just warned" - can't rely on user judgment for critical compliance
- **Resolution:** Tiered system - green (no override needed), yellow (director override with documentation), red (Medical Director approval required), black (system blocks, no override possible).

### Recommended Solution Architecture

**Layer 1: Integration Foundation**
- HL7 FHIR connectors for EHRs (Epic, NextGen, eCW, Athena)
- RESTful APIs for billing platforms (athenahealth, Kareo, Cantata, Qualifacts)
- Payer database integration (Availity, Change Healthcare)
- HR/credentialing system connectors

**Layer 2: Compliance Engine**
- Credentialing status checks (license expiration, NPDB queries, privileging dates)
- Authorization validation (pre-auth requirements, approved sessions, expiration dates)
- Part 2 consent management (SUD record segregation, disclosure logging)
- Scope of Project tracking (approved services/sites vs actual delivery)
- Documentation completeness validation (encounter qualification requirements)

**Layer 3: Orchestration Intelligence**
- Patient scheduling workflow (insurance → authorization → provider match → slot availability)
- Denial prevention pre-submission checks (coding validation, documentation requirements)
- Credentialing cycle automation (90-day warnings, renewal workflows, NPDB scheduling)
- Capacity forecasting (historical trends + seasonal patterns + referral velocity)

**Layer 4: Presentation (UI)**
- Morning brief dashboard (crisis vs stable status, decisions needed, proactive alerts)
- Conversational action paths (narrative presentation, not data grids)
- Compliance sidebar (audit readiness score, upcoming deadlines, one-click reports)
- Crisis mode (auto-activation when capacity thresholds crossed)

### Prioritized Roadmap (Council Consensus)

**Phase 1 (MVP - 90 days):**
1. Integration layer (Epic + NextGen connectors, prove concept)
2. Compliance engine (credentialing + authorization + Part 2)
3. Patient scheduling orchestration ("can we take this patient?" in <10 sec)
4. Basic dashboard (morning brief, compliance sidebar)

**Phase 2 (Revenue Optimization - 60 days):**
1. Denial prevention engine (pre-submission validation, claim scrubbing)
2. RCM analytics (denial rate tracking, recovery workflow, root cause analysis)
3. Documentation assistance (encounter qualification templates, billing compliance checks)

**Phase 3 (Strategic Intelligence - 60 days):**
1. Capacity forecasting (predictive analytics, demand modeling)
2. UDS reporting automation (Table 5, Table 6A, Table 8A export)
3. QI/QA dashboard (HEDIS measures, outcome tracking, board reporting)

**Total time to full product: 210 days (7 months)**
**Target launch: Q4 2026 (ready for UDS 2027 cycle)**

### Go-to-Market Strategy (Council Input)

**Target Market:**
- Mid-sized FQHCs (5-20 locations)
- Currently experiencing high denial rates (>10%)
- Heavy Medicaid/Medicare mix (>60% payer mix)
- Behavioral health as strategic priority (not afterthought)
- Existing EHR: Epic, NextGen, or eCW (integration validation sites)

**Pricing Model (Performance-Based):**
- Base SaaS: $500-$1,000/month per site
- Denial recovery: 10-15% of delta (current rate vs target <5%)
- Efficiency value-share: 20-30% of labor savings

**Beta Program (Ava's recommendation):**
- 3-5 FQHCs, 90-day pilot
- Measure baseline (denial rate, manual hours, compliance gaps)
- Deploy solution, measure delta
- Publish case studies with named institutions
- Conversion to paid: 80%+ target (if we don't deliver value, we don't charge)

**Competitive Positioning:**
- **vs. athenahealth:** "We're the behavioral health module they should have built"
- **vs. Qualifacts:** "We're the FQHC billing engine they don't have"
- **vs. Epic:** "We're the middleware that makes Epic work for behavioral health"
- **vs. consultants:** "We're the automated compliance officer that never sleeps"

### Key Risks & Mitigations

**Risk 1: EHR vendor lock-in (Serena)**
- **Mitigation:** HL7 FHIR standard (not vendor-specific APIs), contractual SLAs for API stability

**Risk 2: Regulatory changes faster than software (Rook)**
- **Mitigation:** Modular compliance engine where rules are configuration (JSON/YAML), not hardcoded

**Risk 3: Integration maintenance burden (Marcus)**
- **Mitigation:** Automated integration testing, monitoring/alerting, dedicated support team

**Risk 4: Market adoption gap (Ava)**
- **Mitigation:** Performance-based pricing (remove buyer risk), published case studies (social proof)

### Final Council Vote

**Should we build this?**
- 🏛️ Serena: **YES** - "Market gap is real, architectural approach is sound"
- 🔍 Ava: **YES** - "Data validates demand, pricing model mitigates risk"
- ⚙️ Marcus: **YES** - "Technically feasible if we prioritize infrastructure over features"
- 🛡️ Rook: **YES** - "Compliance urgency creates forcing function, solution addresses real liability"

**Unanimous: 4-0 in favor of building this solution.**

**Estimated market size:** 1,400 FQHCs × 70% offering BH = 980 potential customers
**Revenue potential (at 30% market penetration):** 294 FQHCs × $36K-$72K annual = $10.6M-$21.2M ARR

**This is a revenue opportunity. Build it.**

---

**Debate completed:** March 27, 2026
**Recommendation:** PROCEED TO BUILD
**Next step:** Validate with beta partners, begin Phase 1 development
