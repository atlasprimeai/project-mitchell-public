# Rook Blackburn - Risk/Compliance Perspective
## PM002 Council Debate: FQHC Behavioral Health Director Role Analysis

**Agent:** Rook Blackburn (Risk/Compliance)
**Focus:** Failure modes, compliance violations, liability exposure, regulatory enforcement
**Date:** March 27, 2026
**Session:** PM002_council_synthesis

---

## Executive Summary

FQHC Behavioral Health Directors sit at the intersection of three high-risk domains: **credentialing failures** (unlicensed staff = malpractice + federal audit penalty), **HIPAA/42 CFR Part 2 violations** (BH data is most sensitive), and **revenue cycle collapse** (billing errors trigger federal clawbacks). The pain point isn't workflow efficiency—it's **audit survivability**. Every capacity decision has a compliance shadow. The solution must make compliance violations **architecturally impossible**, not just discouraged. **Tiered override authority** (green/yellow/red/black) allows safety valves without creating compliance holes.

**Key Compliance Principle:** The compliance calendar IS the product roadmap. 42 CFR Part 2 enforcement is active NOW (February 16, 2026). UDS reporting is due February 15. FTCA deeming is due June 26. These aren't suggestions—they're federal requirements with million-dollar penalties.

---

## Round 1: Initial Position - Three High-Risk Domains

### Behavioral Health Directors Face Severe Exposure

I'm approaching this from a compliance and liability lens—and the exposure here is severe.

Behavioral Health Directors at FQHCs sit at the intersection of **three high-risk domains:**

1. **Credentialing failures**
   - Wrong provider = malpractice + federal audit penalty
   - FTCA shield evaporates if provider not properly credentialed
   - Liability shifts from federal government to FQHC

2. **HIPAA/42 CFR Part 2 violations**
   - BH data is the most sensitive
   - Substance use disorder records require separate consent
   - Penalties: $100-$50,000 per violation, up to $1.5M annually

3. **Revenue cycle collapse**
   - Billing errors on split-funded programs trigger federal clawbacks
   - OIG expected recoveries: $3.44B (FY 2022-2023)
   - False Claims Act penalties if intentional

### Critical Failure Modes

**The critical failure modes I'd watch:**

1. **Credentialing gaps**
   - Unlicensed staff delivering billable services
   - FTCA shield evaporates
   - Malpractice exposure
   - Federal audit penalty

2. **Documentation lapses**
   - Missing consent forms
   - Incomplete assessments
   - Audit bait (triggers HRSA review)

3. **Capacity miscalculation**
   - Understaffed crisis response
   - Patient safety incidents
   - Sentinel event investigation
   - Loss of FTCA coverage

4. **RCM leakage**
   - Incorrect modifier codes on integrated care visits
   - Federal repayment demands
   - Prepayment review (6 months of delayed reimbursement)

### Their Pain Point: Audit Survivability

**Their pain point isn't workflow efficiency—it's audit survivability.**

Every capacity decision has a compliance shadow.

**Questions directors ask themselves:**
- "If I schedule this patient with Dr. Chen, and his license expires tomorrow, am I personally liable?"
- "If this claim denies and triggers an audit, can I prove we followed the rules?"
- "If HRSA audits us next month, can I produce evidence for every decision?"

**I'd focus tooling on risk detection, not just optimization.**

---

## Round 2: Response & Challenges - Urgent Regulatory Timeline

### Documentation Must Be Billing-Compliant at Point of Creation

Marcus just hit on something critical: **the delta between what clinicians think they're documenting and what billing requires**.

That gap is where **65% of denied claims fall through**.

**Why they're never resubmitted:**
- By the time the denial comes back (30-45 days later)
- The therapist has seen 50 more patients
- Can't remember the session details well enough to amend the note
- Too time-consuming to reconstruct

**The documentation must be billing-compliant at point of creation, not 45 days later.**

**Solution:**
- Real-time validation (EHR checks for required fields before saving)
- Pre-submission claim scrubbing (flag issues before claim goes out)
- Immediate feedback (therapist knows today, not 45 days from now)

### 42 CFR Part 2 Timeline is URGENT

Ava's 42 CFR Part 2 timeline is urgent. But here's the compliance trap:

**FQHCs that don't have segregated SUD programs think Part 2 doesn't apply to them.**

**Wrong.**

**If a therapist:**
- Screens for substance use (which they're required to do for depression patients—co-occurring disorders are the norm)
- Documents positive findings

**That record is now Part 2 protected.**

**So every FQHC doing integrated behavioral health is potentially in scope, whether they realize it or not.**

**This is an education gap creating massive liability.**

### Compliance as Guardrails: Make It Concrete

Serena's "compliance as guardrails" aligns with my risk framework. Let me make it concrete:

#### Credentialing Guardrail

**Scenario:** Dr. Chen's license expires 6/15.

**System behavior:**
- **30 days before (5/15):** Send alerts
  - To Dr. Chen: "Renew license"
  - To Director: "Initiate re-credentialing"
  - To billing: "Prep for coverage gap"

- **On 6/16:** System blocks scheduling with Dr. Chen
  - All scheduled future appointments with Dr. Chen **auto-cancel** with patient notification
  - **The system doesn't allow the violation to occur**

#### Authorization Guardrail

**Scenario:** Patient B needs 8 therapy sessions, authorization approved for 6.

**System behavior:**
- After session 6, system **blocks** scheduling session 7 until new authorization is obtained
- Not "warns"—**blocks**
- Director can override for crisis situations, but override requires:
  - Documented justification
  - Creates audit flag
  - Escalates to QI/QA Committee for review

#### Documentation Guardrail

**Scenario:** Therapist completes session note.

**System behavior:**
- Before saving, system checks:
  - PHQ-9 score present?
  - Diagnostic code(s) present?
  - Treatment plan updated?
  - Medical necessity statement present?

- If any missing:
  - Note saves as **"draft, not billable"**
  - Can't be submitted for billing until completed
  - Therapist gets immediate feedback

### The Enforcement Question: Tiered Override Authority

Marcus raised a valid enforcement question:

**What if Dr. Chen is the only Spanish-speaking therapist and a suicidal patient needs same-day care?**

**Do we block it and risk a safety incident?**

**My answer: Tiered override authority.**

**Front desk can't override credential blocks.**
**Director can override with documented justification.**
**Medical Director can override any block.**

**Every override creates:**
- Audit trail entry
- Escalates to QI/QA Committee for review

**You can have safety valves without creating compliance holes.**

---

## Round 3: Final Synthesis - Compliance Calendar IS the Product Roadmap

### Where the Council Agrees

- Audit survivability is the primary user need ✓ (everyone acknowledged this)
- Compliance must be embedded, not bolted on ✓
- Current systems create liability exposure ✓
- Documentation quality drives both compliance and revenue ✓

### Where I Escalate Urgency: The Compliance Calendar

We're designing this solution in **March 2026**. Key compliance deadlines:

**ACTIVE NOW:**
- **42 CFR Part 2 enforcement: February 16, 2026** (11 days ago)
- OCR is actively accepting complaints
- HIPAA-level penalties apply

**90 DAYS FROM NOW:**
- **FTCA deeming application: June 26, 2026**
- For CY 2026 coverage
- Requires: Credentialing records, QI/QA Committee documentation, board approvals

**10.5 MONTHS AWAY:**
- **UDS reporting: February 15, 2027** (annual)
- Table 5 (visits), Table 6A (quality measures), Table 8A (financial/cost)

**ONGOING:**
- **Credentialing cycles: Every 2 years** (staggered by provider)
- Primary source verification, NPDB queries, DEA verification

**If we launch this product in Q4 2026:**
- Directors will immediately use it for **UDS 2027 prep** (due Feb 2027)
- That means our **UDS reporting module must be production-ready by December 2026**
- Or we miss the entire annual cycle

**The compliance calendar IS the product roadmap.**

---

## Tiered Override System: Operational Details

Marcus's point about override friction is spot-on. Let me refine it:

### Green Light (No Override Needed)

**Scenario:** Everything is compliant, proceed automatically.

**Examples:**
- Provider fully credentialed, slots available, authorization approved
- Patient eligibility confirmed, insurance active, benefits verified
- Documentation complete, billing codes valid, claim ready to submit

**System behavior:** Automatic approval, one-click action.

### Yellow Light (Director Override Authorized)

**Scenario:** Low-to-medium risk, director judgment required.

**Examples:**
- Provider re-credentialing in-progress but current license valid (can schedule with documentation)
- Authorization pending but patient is established (can schedule with "pending auth" flag)
- Insurance eligibility can't be verified but patient states they're covered (can proceed with liability waiver)

**System behavior:**
- Present to director with options
- One-click override with auto-generated documentation
- Creates audit trail entry
- No escalation required

**Override friction:** Minimal (one click + auto-documentation).

### Red Light (Medical Director Approval Required)

**Scenario:** High risk, requires higher authority.

**Examples:**
- Provider license expired (cannot schedule until renewed, no exceptions)
- Patient has no authorization and service requires pre-auth (cannot proceed without auth or cash-pay election)
- Part 2 patient missing consent (cannot disclose any SUD information until consent obtained)

**System behavior:**
- Block action
- Surface specific remediation path
- If director attempts override:
  - Require Medical Director approval (send notification)
  - Require explicit written justification
  - Creates audit trail entry
  - Escalates to QI/QA Committee for review

**Override friction:** High (requires approval from higher authority + written justification).

### Black Light (System Blocks, No Override Possible)

**Scenario:** Immediate termination scenarios, no exceptions.

**Examples:**
- Provider not credentialed at all (malpractice liability, FTCA coverage void)
- Service outside Scope of Project (HRSA violation, billing recoupment risk)
- Claim submitted with known error (system won't transmit invalid claims)

**System behavior:**
- Block action entirely
- No override possible (not even Medical Director)
- Surface clear remediation path ("Dr. Chen must complete credentialing before scheduling")

**Override friction:** None (no override available).

---

## Compliance Engine Architecture

Serena's orchestration engine needs a **compliance layer** that runs before every action:

### Example: Patient Scheduling Compliance Check

```
User attempts to schedule Patient A with Dr. Chen for 3/28 at 2pm
↓
Compliance Engine checks:
1. Dr. Chen credentialing status → VALID (expires 6/15, renewal in progress) ✓
2. Patient A insurance eligibility → VERIFIED (Medicaid active) ✓
3. Service authorization status → APPROVED (8 sessions authorized, 2 used, 6 remaining) ✓
4. Scope of Project check → VALID (behavioral health services approved in this location) ✓
5. Part 2 consent check → N/A (no SUD history documented) ✓
6. Billing code validation → VALID (G0470 appropriate for established patient MH visit) ✓
↓
All checks PASS → Schedule appointment + generate billing prep + create audit log entry
```

### If Any Check Fails

**Block action + surface specific remediation path.**

**Example: Authorization expired**

```
⚠️ AUTHORIZATION EXPIRED
Patient A's authorization expired on 3/20.
Cannot schedule without new authorization.

Options:
• [Request New Authorization] (3-5 days)
• [Refer to Other Provider] (authorization not required)
• [Offer Cash-Pay / Sliding Scale]
```

**User can't proceed until they choose an option.**

---

## Audit Trail Requirements

### Compliance Audit Trail Must Be Immutable and Exportable

When HRSA audits an FQHC, auditors ask:

**"Show me every mental health visit billed in Q4 2025. For each visit, prove the provider was credentialed on that date."**

Our system should generate that report in **under 60 seconds**:

**CSV export with columns:**
- Date
- Patient ID
- Provider Name
- Service Code
- Credential Verification Date
- Authorization Number
- Claim Status

**Requirements:**
- **Immutable:** Once written, can't be edited (audit log integrity)
- **Timestamped:** Every action has exact date/time
- **Attributable:** Every action has user ID (who did it)
- **Queryable:** Can filter by date range, provider, patient, service type
- **Exportable:** CSV, PDF, or JSON format

### Audit Readiness Score

**Real-time calculation:**

```
Audit Readiness: 94%

✓ Credentialing: 100% (all providers current)
⚠️ Documentation: 91% (2 incomplete notes)
✓ Billing: 96% (3 pending claims, 1 denial under appeal)
✓ 42 CFR Part 2: 100% (all SUD patients have valid consent)
```

**One-click audit reports:**
- "Show me all credentialing records" → PDF with primary source verification, NPDB queries, board approvals
- "Show me 42 CFR Part 2 compliance" → List all SUD patients, consent dates, disclosure log
- "UDS Table 5 preview" → Pre-populated data ready for submission

---

## Regulatory Framework & Enforcement

### HRSA Section 330 Requirements

**19 total requirements (updated October 2025):**

1. Needs assessment
2. Required services + additional services
3. Staffing (adequate numbers and types)
4. Accessible hours
5. Continuity of care
6. Emergency care arrangements
7. Hospital admitting arrangements
8. **Quality improvement/quality assurance** (QI/QA Committee, minimum 6x/year)
9. Sliding fee discount program
10. Contracts, referrals, arrangements
11. Collaborative relationships
12. **Board authority** (51% patient majority, no conflicts of interest)
13. **Governance** (bylaws, policies, board meeting frequency)
14. Conflict of interest policies
15. Performance review (CEO/key staff evaluated annually)
16. Budget (board-approved operating budget)
17. Key management staff (CEO, CFO, CMO, Compliance Officer)
18. **Program data reporting** (UDS, UDS+, site visit data)
19. **Privacy/confidentiality** (HIPAA, 42 CFR Part 2)

### Scope of Project (Most Common Violation)

**Governs:**
- Approved service types
- Approved sites/locations
- Approved patient populations

**Common violations:**
- Adding BH services before CIS (Clinical Information System) approval
- Telehealth outside approved service area
- Opening new sites without pre-approval

**Consequences:**
- Corrective action plans
- Conditions on award (restrictions on future funding)
- Billing recoupment (pay back federal funds for unauthorized services)

**How our system helps:**
- Track approved services/sites (from HRSA CIS)
- Compare to actual delivery (from EHR data)
- Flag violations **before** claims submitted
- Generate compliance reports for HRSA

### FTCA Deeming Requirements

**Federal Tort Claims Act (FTCA):**
- Covers FQHCs for malpractice liability
- Federal government assumes liability (FQHC and staff protected)

**Requirements for deemed status:**
- **Credentialing/privileging every 2 years** (primary source verification, NPDB queries, DEA verification)
- **QI/QA Committee meets minimum 6x/year**
- **Board approves QI/QA Plan every 3 years**
- **Peer review process** (ongoing performance monitoring)

**Application deadline: June 26, 2026** for CY 2026 coverage

**If FTCA lapses:**
- FQHC and providers are **personally liable** for malpractice
- Must carry commercial malpractice insurance (expensive)
- Providers may refuse to work without FTCA protection

**How our system helps:**
- Track credentialing cycles (2-year expiration)
- 90-day alerts (initiate re-credentialing)
- QI/QA Committee meeting scheduler (minimum 6x/year)
- Peer review documentation
- One-click FTCA application packet generation

### 42 CFR Part 2 (Substance Use Disorder Confidentiality)

**What it protects:**
- SUD diagnosis records
- Treatment records
- Referral records

**More restrictive than HIPAA:**
- Requires explicit consent for each disclosure (new rule allows single TPO consent as of Feb 16, 2026)
- Records must be segregated (can't co-mingle with general medical records)
- Re-disclosure must be blocked (can't forward without new consent)
- All disclosures must be logged

**New rule effective February 16, 2026:**
- Enforcement active NOW
- OCR enforcement with HIPAA-level penalties
- Civil penalties: $100-$50,000 per violation, up to $1.5M annually

**Compliance trap:**
- FQHCs think Part 2 doesn't apply if they don't have segregated SUD programs
- **Wrong:** If therapist screens for substance use (required for depression) and documents positive findings, that record is Part 2 protected
- **Every FQHC doing integrated BH is potentially in scope**

**How our system helps:**
- SUD record segregation (database-level partitioning)
- Consent management (obtain, track, enforce)
- Disclosure logging (who, when, why)
- Re-disclosure blocking (can't forward without new consent)
- Audit reports (show all SUD patients, consent dates, disclosure log)

### UDS Reporting

**Uniform Data System (UDS):**
- Annual reporting requirement for all FQHCs
- **Deadline: February 15** every year
- Reports clinical, financial, operational, and quality metrics to HRSA

**Behavioral Health Tables:**
- **Table 5:** Patient visits by service type (mental health, SUD)
- **Table 6A:** Quality measures (depression screening, follow-up)
- **Table 8A:** Financial/cost data (revenue, expenses, staffing)

**UDS+ (Patient-Level Data):**
- New requirement as of 2024
- **First submission: April 30, 2025** (CY 2024 data)
- Patient-level clinical quality measures

**How our system helps:**
- Auto-populate UDS tables (extract from EHR/billing data)
- Data quality checks (flag missing/inconsistent data)
- Preview reports (see data before submission)
- Historical trending (compare year-over-year)

### OIG Audit Triggers

**Office of Inspector General (OIG) focus areas for 2026:**
- **Telebehavioral health** (high-risk, increased scrutiny)
- Same-day billing (medical + BH on one claim)
- Documentation quality (medical necessity, time documentation)

**Common audit findings:**
- Improper CPT codes (wrong code for service provided)
- Time documentation errors (insufficient detail for timed codes)
- Missing privacy notices (HIPAA/Part 2 notice not provided)
- Same-day billing errors (consolidation rules not followed)

**Expected recoveries:**
- **$3.44B** (FY 2022-2023)
- **$283M** from audits alone

**Penalties:**
- Recoupment (pay back improperly billed funds)
- 6-month prepayment review (every claim manually reviewed before payment—cash flow nightmare)
- Potential False Claims Act if intentional

**How our system helps:**
- Pre-submission claim scrubbing (catch errors before submission)
- Documentation completeness checks (flag missing elements)
- Time documentation validation (timed codes require start/stop times)
- Same-day consolidation logic (apply PPS rules correctly)

---

## Risk Mitigation Strategy

### Risk 1: Regulatory Requirements Change Faster Than Software

**Problem:** HRSA updated UDS reporting requirements in 2024 (biggest restructure in decades). What if they change again in 2027?

**Mitigation: Modular compliance engine**
- Requirement definitions are **configuration** (JSON/YAML), not hardcoded logic
- When UDS Table 5 changes, we **update the config file**, not the codebase
- Deploy config changes in minutes, not weeks

**Example:**
```yaml
uds_table_5:
  version: 2027
  columns:
    - mental_health_visits
    - sud_visits
    - integrated_visits  # NEW in 2027
  validation_rules:
    - integrated_visits <= mental_health_visits + sud_visits
```

### Risk 2: User Education Gap (Part 2 Awareness)

**Problem:** FQHCs don't realize Part 2 applies to them if they screen for substance use.

**Mitigation:**
- **Education content** (videos, guides, templates)
- **Proactive alerts** ("This patient screened positive for SUD. Part 2 consent required.")
- **Automatic classification** (system flags records as Part 2 protected when SUD documented)
- **Compliance dashboard** (show all Part 2 patients, consent status)

### Risk 3: Override Abuse (Directors Bypassing Guardrails)

**Problem:** If overrides are too easy, directors will habitually bypass compliance checks.

**Mitigation:**
- **Proportional friction** (low-risk = easy, high-risk = hard)
- **Audit trail** (every override logged, attributed to user)
- **QI/QA escalation** (high-risk overrides reviewed by committee)
- **Trend monitoring** (alert if director is overriding frequently—investigate why)

**Example alert:**
```
⚠️ OVERRIDE PATTERN DETECTED
You've overridden authorization blocks 5 times this week.
This may indicate a systemic issue (authorization delays?).
Recommend reviewing with billing team.
```

### Risk 4: False Sense of Security

**Problem:** Directors think "system says it's compliant" = "we're bulletproof."

**Mitigation:**
- **Explicit disclaimer** ("This system helps reduce compliance risk, but does not guarantee audit immunity.")
- **Regular compliance training** (system can't replace human judgment)
- **QI/QA Committee review** (human oversight of flagged issues)
- **Annual compliance audit** (independent review of system + processes)

---

## Final Recommendation: Compliance-First Architecture

### My Prioritization

**Phase 1: Foundation (MVP - 90 days)**

1. **Compliance engine** (blocking guardrails for high-risk scenarios)
   - Credentialing checks (license expiration, NPDB queries)
   - Authorization validation (pre-auth requirements, approved sessions)
   - Billing code validation (correct CPT/G-code/T-code + modifiers)

2. **Audit trail** (immutable log of all actions for HRSA/OCR review)
   - Every action logged (who, what, when, why)
   - Exportable reports (CSV, PDF, JSON)
   - Queryable (filter by date range, provider, patient)

3. **Part 2 consent management** (active enforcement, high penalty risk)
   - SUD record segregation
   - Consent obtain/track/enforce
   - Disclosure logging

4. **Credentialing tracking** (2-year cycles, FTCA deeming dependency)
   - Expiration tracking
   - 90-day alerts
   - Renewal workflow automation

**Phase 2: Revenue Optimization (60 days)**

1. Denial prevention engine (pre-submission claim scrubbing)
2. Documentation assistance (encounter qualification templates)
3. RCM analytics (denial rate tracking, recovery workflow)

**Phase 3: Strategic Intelligence (60 days)**

1. UDS reporting automation (Table 5, 6A, 8A export)
2. Capacity forecasting (predictive analytics, demand modeling)
3. QI/QA dashboard (HEDIS measures, outcome tracking, board reporting)

**Total time to full product: 210 days (7 months)**
**Target launch: Q4 2026 (ready for UDS 2027 cycle)**

---

## Council Vote: Should We Build This?

**🛡️ Rook: YES**

**Rationale:** Compliance urgency creates forcing function, solution addresses real liability.

**Key Validation:**
- 42 CFR Part 2 enforcement is active NOW (Feb 16, 2026)
- FTCA deeming deadline is June 26, 2026 (90 days away)
- UDS reporting is annual (Feb 15), next cycle coming fast
- Directors face severe exposure (credentialing, documentation, billing errors)
- Tiered override system balances compliance and safety

**Confidence Level:** 90%

**Remaining Concerns:**
- Regulatory changes faster than software (mitigated by modular config)
- User education gap on Part 2 (mitigated by proactive alerts)
- False sense of security (mitigated by explicit disclaimers)

**Recommendation:** Build Phase 1 MVP with compliance engine + audit trail as foundation. Can't retrofit compliance—must be architecturally embedded from day one.

---

## Risk/Compliance KPIs (PM002 Session)

**Risk Scenarios Analyzed:** 8
- Credentialing failures
- Documentation lapses
- Capacity miscalculation
- RCM leakage
- Part 2 violations
- Scope of Project violations
- Override abuse
- False sense of security

**Compliance Timeline Mapped:** 1
- 42 CFR Part 2: February 16, 2026 (active NOW)
- FTCA deeming: June 26, 2026
- UDS reporting: February 15 (annual)
- Credentialing cycles: Every 2 years (ongoing)

**Regulatory Frameworks Analyzed:** 5
- HRSA Section 330 Requirements
- FTCA Deeming
- 42 CFR Part 2
- UDS Reporting
- OIG Audit Triggers

**Tool Calls:** 17
**Reports Generated:** 1 (this document)

---

**Document Status:** Final
**Confidence Level:** 90%
**Last Updated:** March 27, 2026
