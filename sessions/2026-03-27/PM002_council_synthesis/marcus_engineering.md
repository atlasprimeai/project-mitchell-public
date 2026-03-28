# Marcus Webb - Engineering Perspective
## PM002 Council Debate: FQHC Behavioral Health Director Role Analysis

**Agent:** Marcus Webb (Engineer)
**Focus:** Implementation reality, technical constraints, user adoption, build feasibility
**Date:** March 27, 2026
**Session:** PM002_council_synthesis

---

## Executive Summary

FQHC Behavioral Health Directors are drowning in fragmented tooling—juggling an EHR not built for behavioral health, a scheduling system that doesn't talk to billing, and spreadsheets to fill the gaps. The critical pain point is **real-time capacity visibility**: answering "can we take this Medicaid patient next week?" currently requires 30 minutes of manual work when it should take 10 seconds. The solution must integrate with legacy systems and survive users who are clinically trained, not technically sophisticated. **Build for 80% automation, 20% human judgment**—don't try to automate every edge case.

**Key Engineering Principle:** Graceful degradation when upstream systems fail. Partial information that's honest about its limitations is better than no information.

---

## Round 1: Initial Position - Fragmented Tooling Reality

### The Problem: Directors Are Drowning

From an implementation standpoint, behavioral health directors at FQHCs are drowning in fragmented tooling. They're juggling:

1. **EHR that wasn't built for behavioral health**
   - Designed for medical workflows (vitals, labs, prescriptions)
   - NOT therapy workflows (PHQ-9 scores, treatment goals, session notes, safety plans)
   - Bolt-on BH modules feel like afterthoughts

2. **Separate scheduling system that doesn't talk to billing**
   - Shows "scheduled" but not actual provider availability
   - Doesn't show payer eligibility
   - Doesn't surface authorization expiration dates

3. **Spreadsheets—so many spreadsheets**
   - Built because enterprise tools don't give them the views they need
   - Authorization tracking
   - Capacity planning
   - Compliance checklists
   - Manual reconciliation across platforms

### The Critical Workflow Pain Point: Real-Time Capacity Visibility

**Question:** "Can we take this Medicaid patient next week?"

**Should be:** 10-second answer

**Actually requires:**
- Manual data pulls from EHR
- Reconciliation across scheduling platform
- Tribal knowledge about which providers are "really" available vs technically scheduled
- Authorization status check in billing system
- Credentialing verification in HR database
- 30 minutes of spreadsheet archaeology

**No unified view possible with current tools.**

### Gap Between Clinical Documentation and Billing-Compliant Notes

**For RCM specifically:** The gap between clinical documentation and billing-compliant notes creates constant rework.

**Problem:**
- Therapist writes a clinical note (what happened in session, patient progress)
- Billing system requires: diagnostic codes, medical necessity statement, treatment plan updates, encounter qualification criteria
- Most EHRs require **double-entry** or have such poor UX that clinicians skip required fields
- Creates downstream revenue leakage (15-25% denial rate)

**Root Cause:**
- EHRs are medical-first, behavioral health bolt-on
- Forms don't have the right fields
- Clinicians don't know they're supposed to write for billing compliance, not just clinical documentation

### User Reality: Clinically Trained, Not Technically Sophisticated

**Critical constraint:** Any solution we build has to:
1. **Integrate with terrible legacy systems** (Epic, NextGen, eCW, athenahealth)
2. **Survive users who are clinically trained, not technically sophisticated**

Directors have Master's degrees in social work, not computer science. They know therapy, not APIs.

**If the system is confusing, they'll route around it:**
- Phone calls instead of system workflows
- Paper tracking instead of digital
- Verbal approvals instead of audit trails

**Usability is non-negotiable.**

---

## Round 2: Response & Challenges - Stress-Testing the Architecture

### Middleware Abstraction: Who Maintains It?

Serena, I love the middleware abstraction concept, but let me stress-test it:

**Who maintains the integrations when Epic pushes an update and breaks the API?**

We're not just building software—we're committing to **permanent infrastructure maintenance**.

**This is a support burden that needs to be priced into the model.**

**Examples:**
- Epic pushes FHIR R4 update, changes Patient resource structure
- NextGen deprecates an API endpoint
- athenahealth changes OAuth flow
- eClinicalWorks rate-limits our requests

**Each of these requires:**
- Detection (monitoring/alerting)
- Diagnosis (what broke?)
- Fix (update our connector)
- Testing (did we break anything else?)
- Deployment (push to production)

**This is ongoing operational overhead.**

### 42 CFR Part 2 Compliance: Not Just a Feature Flag

Ava's point about 42 CFR Part 2 enforcement is a perfect example of why I'm cautious.

**Part 2 compliance isn't just "add consent management"—it's architectural segregation of SUD records.**

**You can't just flag a record as "Part 2 protected."**

**You have to ensure:**
- **Database-level partitioning** (SUD records never co-mingled with general medical records)
- **Every disclosure is logged** (who accessed, when, for what purpose)
- **Re-disclosure is blocked** (can't forward records without new consent)
- **Consent is tied to specific uses** (TPO consent doesn't allow research use)

**This is database-level partitioning, not a feature flag.**

**Implementation Complexity:**
- Separate tables for SUD records
- Audit log for every access
- Access control layer (RBAC + consent checks)
- Consent management UI (obtain, revoke, track)
- Disclosure logging (report generation)

**This adds 3-6 months to development timeline.**

### The Override Problem: Safety vs. Compliance

Rook's "block scheduling with expired credentials" idea is right in principle, but here's the implementation reality:

**What if Dr. Chen is the only Spanish-speaking therapist and a suicidal patient needs same-day care?**

**Do we:**
- **Block it** and risk a safety incident (patient leaves, harms themselves)
- **Allow an override** with heightened documentation (safety first, compliance second)

This is where "compliance as guardrails" gets messy—**real-world healthcare has exceptions**, and our system needs to handle them without creating liability.

**My answer: Tiered override authority**
- Front desk **can't** override credential blocks
- Director **can** override with documented justification
- Medical Director **can** override any block
- Every override creates an audit trail entry and escalates to QI/QA Committee

You can have safety valves without creating compliance holes.

### Denial Rate Root Cause: Encounter Qualification

On the denial rate problem (15-25% vs 5-10% medical):

**I'd hypothesize the root cause is encounter qualification under PPS rules.**

**FQHCs get paid per encounter, not per service.**

To qualify as an encounter, the documentation must meet specific criteria:
- Medical necessity explicitly stated
- Diagnostic codes documented
- Treatment plan changes noted
- Duration/intensity appropriate for level of service

**If the therapist's note doesn't meet criteria:**
- The claim gets downgraded from "encounter" to "not billable"
- Revenue lost

**Most therapists don't know they're supposed to write for billing compliance, not just clinical documentation.**

**This is a training problem disguised as a technology problem.**

**Solution:**
- Real-time documentation guidance (EHR prompt: "Did you document medical necessity?")
- Pre-submission validation (claim scrubbing before it goes out)
- Feedback loop (when claim denies, show therapist what was missing)

---

## Round 3: Final Synthesis - Build for 80% Automation, 20% Human Judgment

### Where the Council Agrees

- System fragmentation is the root problem ✓
- Real-time data is critical (not batch updates) ✓
- Compliance must be enforced, not optional ✓
- Integration with legacy systems is mandatory ✓

### Where I Remain Skeptical: Perfectionism is the Enemy of Reliability

Serena's "10-second orchestration" is **architecturally sound but operationally fragile**.

**Why?**

That orchestration depends on **6 upstream systems** (EHR, scheduling, billing, payer database, credentialing, HR) **all being available and responsive**.

**If any one has downtime, the entire workflow breaks.**

**We need graceful degradation.**

### Graceful Degradation Strategy

**If billing system is down:**
- Can we still schedule the appointment with a "pending authorization verification" flag?
- Show user: "⚠️ Billing system offline. Authorization status may be stale. Verify manually before claim submission."

**If payer database is stale:**
- Can we proceed with a warning that insurance info might not be current?
- Show user: "⚠️ Last insurance check: 6 hours ago. Recommend calling insurance to verify."

**If credentialing system unavailable:**
- Block high-risk actions (can't schedule with expired license)
- Allow low-risk actions with warnings (provider re-credentialing in-progress but current license valid)

**Principle: Partial information that's honest about its limitations is better than no information.**

**Perfectionism is the enemy of reliability in healthcare.**

### Perceived Speed Matters as Much as Actual Speed

Ava's point about speed is right, but let me add:

**Perceived speed matters as much as actual speed.**

**If the system takes 8 seconds but shows a progress indicator:**
```
Checking authorization... ✓
Verifying credentials... ✓
Finding slots... ⏳
```

**Users perceive it as fast and reliable.**

**If it takes 5 seconds but spins silently:**
- Users think it's frozen
- They click again (duplicate requests)
- They give up and go manual

**UX is as important as backend performance.**

**Implementation:**
- WebSocket connection (real-time updates)
- Progress indicator (show what's happening)
- Timeout handling (if >10 seconds, show "This is taking longer than usual, still working...")
- Cancel button (let user abort if they need to)

### Override Friction Must Be Proportional to Risk

Rook's tiered override system is brilliant in theory. In practice, here's my concern:

**Override friction must be proportional to risk.**

**Low-risk override:**
- Scheduling with a provider whose re-credentialing is in-progress but license is still valid
- Should be **one click** with auto-documentation

**High-risk override:**
- Scheduling with expired license
- Should require **Medical Director approval** + explicit justification

**If we make every override equally burdensome, users will route around the system entirely:**
- Phone calls instead of system
- Paper instead of digital
- Verbal approvals instead of audit trails

**We're back to spreadsheets and tribal knowledge.**

---

## Final Recommendation: Build for 80% Automation, 20% Human Judgment

### Don't Try to Automate Every Edge Case

**Don't try to automate every edge case—give humans good tools for handling the exceptions.**

**The dashboard should surface:**
- **3 patients need scheduling**
- **2 are straightforward (green)** → System can handle automatically
- **1 has a complication (yellow, requires your decision)** → Present to director with options

**Example:**

```
┌─────────────────────────────────────────────────────────────┐
│ NEW PATIENT REFERRALS (3)                                    │
├─────────────────────────────────────────────────────────────┤
│ 🟢 Patient A - Maria G. (READY TO SCHEDULE)                 │
│    Insurance verified, authorization approved, Dr. Reyes     │
│    available. [Schedule Automatically]                       │
│                                                              │
│ 🟢 Patient C - Sarah M. (CRISIS - READY FOR SAME-DAY)       │
│    Suicidal ideation, Dr. Parker has crisis slot 4pm today. │
│    Part 2 consent obtained. [Schedule Crisis Slot]          │
│                                                              │
│ 🟡 Patient B - John D. (NEEDS YOUR DECISION)                │
│    Insurance: Aetna commercial → BH carved out to Magellan  │
│    No authorization submitted yet.                           │
│    Options:                                                  │
│    • [Request Magellan Authorization] (3-5 days)            │
│    • [Refer to Magellan Provider Network]                   │
│    • [Offer Cash-Pay / Sliding Scale]                       │
└─────────────────────────────────────────────────────────────┘
```

**Director clicks one button for the green ones, makes a decision on the yellow one.**

**System handles 67% automatically (2/3), director handles 33% (1/3).**

**That's the right ratio.**

---

## Engineering Priorities & Roadmap

### What I'd Prioritize

**Phase 1: Prove Integration Reliability (Before Building Features)**

1. **Integration testing infrastructure**
   - Build test harness (mock EHR/billing/scheduling responses)
   - Automated integration tests (run on every deploy)
   - Monitoring + alerting (detect integration failures before users report them)
   - **Prove we can reliably connect** to Epic/NextGen/Athena before building workflows on top

2. **Error handling + graceful degradation**
   - What happens when upstream systems fail?
   - Fallback modes (proceed with warnings vs block entirely)
   - User messaging (clear, actionable error messages)

3. **Core workflows**
   - Patient scheduling (10-second orchestration)
   - Denial prevention (pre-submission claim scrubbing)
   - Credentialing tracking (90-day alerts, expiration blocks)

4. **Monitoring + alerting**
   - Detect integration failures before users report them
   - Track performance (P50, P95, P99 latency)
   - Error rates, timeout rates

### Technical Debt Management

**The engineering risk: technical debt from rushing to market.**

If we prioritize features over architecture quality:
- We'll ship fast
- But create a **maintenance nightmare**

**3 months from now:**
- Integration breaks daily
- Bugs pile up
- Users lose trust
- We're firefighting instead of building

**Mitigation:**

1. **Automated testing from day one**
   - Every integration has a test suite
   - Every feature has unit + integration tests
   - Regression testing on every deploy

2. **Feature flags**
   - We can disable broken features without redeploying
   - If Epic integration breaks, disable Epic-dependent features, rest of system keeps working

3. **Blameless postmortems**
   - When things break (they will), we learn instead of finger-point
   - Document failure mode, root cause, mitigation
   - Build institutional knowledge

---

## Engineering Risks & Mitigations

### Risk 1: Integration Maintenance Burden

**Problem:** Epic pushes an update and breaks the API. Who fixes it? How fast?

**Mitigation:**
- **Dedicated integrations team** (not "whoever's available")
- **Automated monitoring** (detect failures instantly)
- **Test harness** (validate fixes don't break other integrations)
- **SLAs with EHR vendors** (contractual stability commitments)

**Cost:** Budget for 2-3 full-time engineers on integrations.

### Risk 2: Upstream System Failures

**Problem:** If billing system is down, does entire workflow break?

**Mitigation:**
- **Graceful degradation** (proceed with warnings vs block entirely)
- **Caching** (Redis cache for authorization status, refresh every 15 minutes)
- **Fallback modes** (if real-time check fails, use cached data + warn user)

**Principle:** System should degrade gracefully, not fail catastrophically.

### Risk 3: User Adoption Resistance

**Problem:** Directors are used to Excel. Why switch to new system?

**Mitigation:**
- **Solve a painful problem immediately** (denial prevention, 15-25% → <5%)
- **Make onboarding effortless** (import existing data, minimal setup)
- **Training + support** (not just documentation, actual humans helping)
- **Champion strategy** (find 1-2 power users, make them successful, they'll advocate)

**Metric:** Time to first value <30 minutes (director logs in, sees immediate insight they didn't have before).

### Risk 4: 42 CFR Part 2 Implementation Complexity

**Problem:** Database-level partitioning adds 3-6 months to timeline.

**Mitigation:**
- **Phase it** (MVP without Part 2, add in Phase 2)
- **OR: Partner** (use existing Part 2-compliant platform, integrate via API)
- **OR: Limit scope** (initially support FQHCs that don't offer SUD services, expand later)

**My recommendation:** Phase it. MVP focuses on non-SUD behavioral health (depression, anxiety). Phase 2 adds SUD support with full Part 2 compliance.

### Risk 5: Denial Prevention Training Problem

**Problem:** Therapists don't know they're writing for billing compliance.

**Mitigation:**
- **Real-time guidance** (EHR prompts: "Did you document medical necessity?")
- **Pre-submission validation** (claim scrubbing, flag missing elements)
- **Feedback loop** (when claim denies, show therapist what was missing)
- **Training content** (video tutorials, templates, examples)

**This is a training problem disguised as a technology problem.**

Technology can help, but it's not 100% solvable with software alone.

---

## Technical Architecture Specifications

### Platform: Progressive Web App (PWA)

**Why PWA?**
- Works on any device (desktop, tablet, mobile)
- Offline mode with sync (for rural connectivity issues)
- No app store gatekeepers (deploy instantly)
- Real-time WebSocket updates (no manual refresh)

**Tech Stack:**
- **Frontend:** React + TypeScript (industry standard, large talent pool)
- **Backend:** Node.js + Express (fast iteration, JavaScript end-to-end)
- **Database:** PostgreSQL (structured data, ACID compliance)
- **Cache:** Redis (real-time capacity status, sync status)
- **API:** RESTful + GraphQL (REST for simple CRUD, GraphQL for complex queries)

### Integration Architecture

**HL7 FHIR (Healthcare Interoperability Standard):**
- Industry standard for health data exchange
- Epic, NextGen, Athena all support FHIR R4
- Patient, Practitioner, Appointment, Claim, Coverage resources

**RESTful APIs:**
- OAuth 2.0 authentication
- Rate limiting (prevent overload)
- Webhook support (real-time event notifications)

**Error Handling:**
- Retry logic (exponential backoff)
- Circuit breaker pattern (if upstream system fails repeatedly, stop hammering it)
- Fallback modes (cached data when real-time unavailable)

### Data Storage

**PostgreSQL (Structured Data):**
- Patients, appointments, providers, claims
- Credentialing records, authorization tracking
- Audit logs (immutable, timestamped)

**Schema Design:**
- Multi-tenant (1 database, tenant_id on every table)
- Row-level security (PostgreSQL RLS policies)
- Partitioning for audit logs (partition by month, archive old data)

**Redis (Real-Time Cache):**
- Capacity status (available slots by provider/date)
- Sync status (last successful integration by system)
- Session state (user context, preferences)

**TTL Strategy:**
- Capacity status: 15 minutes
- Sync status: 5 minutes
- Session state: 24 hours

### Security

**Authentication:**
- OAuth 2.0 with OpenID Connect
- Multi-factor authentication (TOTP, SMS, hardware keys)
- Session management (JWT tokens, 8-hour expiry)

**Authorization:**
- Role-based access control (RBAC)
- Roles: Director, Billing Staff, Front Desk, Clinical Staff, Read-Only
- Least privilege principle

**Data Protection:**
- Encryption at rest (AES-256)
- Encryption in transit (TLS 1.3)
- Key management (AWS KMS or HashiCorp Vault)

**HIPAA Compliance:**
- BAA with all customers
- Audit logging (all PHI access tracked)
- Breach notification plan
- Annual security risk assessment

**42 CFR Part 2 Compliance (Phase 2):**
- Database-level partitioning (SUD records separate)
- Consent management (obtain, track, enforce)
- Disclosure logging (who, when, why)
- Re-disclosure blocking

---

## Council Vote: Should We Build This?

**⚙️ Marcus: YES**

**Rationale:** Technically feasible if we prioritize infrastructure over features.

**Key Validation:**
- Integration with legacy systems is possible (FHIR standard, REST APIs)
- Graceful degradation handles upstream failures
- 80/20 automation ratio is right balance
- Tiered override system handles edge cases

**Confidence Level:** 85%

**Remaining Concerns:**
- Integration maintenance burden (requires dedicated team, ongoing cost)
- 42 CFR Part 2 complexity (adds 3-6 months to timeline)
- User adoption risk (training + support required)

**Recommendation:** Build Phase 1 MVP with 3-5 beta FQHCs. Focus on:
1. **Proving integration reliability** (can we connect to Epic/NextGen/Athena?)
2. **Measuring denial prevention impact** (can we reduce 15-25% to <5%?)
3. **Validating usability** (do clinically-trained users actually use it?)

If beta succeeds, proceed to Phase 2 (full build-out).

---

## Engineering KPIs (PM002 Session)

**Feasibility Assessments Completed:** 3
- Integration architecture (FHIR + REST APIs)
- Error handling + graceful degradation
- 42 CFR Part 2 implementation complexity

**User Reality Analysis:** 1
- Clinically trained, not technically sophisticated
- Usability is non-negotiable
- Resistance to adoption if not immediately valuable

**Technical Risk Assessments:** 5
- Integration maintenance burden
- Upstream system failures
- User adoption resistance
- 42 CFR Part 2 complexity
- Technical debt from rushing to market

**Tool Calls:** 18
**Reports Generated:** 1 (this document)

---

**Document Status:** Final
**Confidence Level:** 85%
**Last Updated:** March 27, 2026
