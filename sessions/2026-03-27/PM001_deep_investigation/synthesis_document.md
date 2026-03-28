# Synthesis Document: FQHC Behavioral Health Director Requirements
# Project Mitchell PM001 - Question & Answer Format

**Session:** PM001_deep_investigation
**Date:** 2026-03-27
**Format:** Executive briefing (1-page summary)

---

## Primary Research Question

**"What are the operational realities, pain points, and requirements for FQHC Behavioral Health Directors managing mental health and substance use disorder services?"**

---

## Answer: Five Critical Findings

### 1. Directors Navigate Extreme Operational Complexity

**Reality:**
- Manage 4-6 disconnected systems (EHR, scheduling, billing, compliance tracking)
- Manually reconcile data to answer basic questions: "Can we schedule this patient now?" (30 minutes per inquiry)
- Track provider availability, authorization status, credentialing expiration, Part 2 consent - all in separate systems
- Excel shadow systems for critical operational data (authorization tracking, capacity planning, compliance calendars)

**Source:** Research domains 3 (Capacity Planning) + 5 (Technology Landscape)

### 2. Revenue Leakage from High Denial Rates

**Reality:**
- **15-25% claim denial rate** for behavioral health (vs 5-10% for medical/surgical)
- **65% of denials never resubmitted** = $390K revenue leakage annually per FQHC
- Triple complexity stack: FQHC billing (PPS encounter qualification) + BH coding (G-codes/T-codes) + integration requirements (same-day consolidation)
- Only 14% of FQHCs using AI denial prevention (82% want it)

**Source:** Research domain 4 (RCM)

### 3. Workforce Crisis Driving Burnout & Turnover

**Reality:**
- **77% report behavioral health provider shortages** (up from 70% in 2018)
- **93% BH worker burnout rate**
- **35-70% turnover** in BH facilities ($562K annual cost per FQHC)
- #2 turnover driver: **Documentation burden exceeding feasible completion time**
- Directors spend 30-50% of time on administrative tasks that could be automated

**Source:** Research domains 1 (Job Profiles) + 2 (FQHC Domain)

### 4. Active Regulatory Enforcement with High-Stakes Penalties

**Reality:**
- **42 CFR Part 2 enforcement began February 16, 2026** - Civil penalties up to $1.5M annually per violation type
- **Scope of Project violations = #1 HRSA citation** - Billing recoupment, grant funding at risk
- **FTCA deeming deadline June 26, 2026** - Miss it, lose malpractice coverage for the year
- Credentialing every 2 years (primary source verification, NPDB query) - fail = FTCA revoked
- UDS reporting February 15 annually (2026 restructuring: "one of largest in decades")

**Source:** Research domain 6 (Compliance & Regulatory)

### 5. No Vendor Solves Both FQHC Billing AND BH Workflows

**Reality:**
- **General medical EHRs (Epic, athenahealth, NextGen):** Strong on FQHC billing ✓ | Weak on BH workflows ✗
  - 60% of athenahealth FQHC users request better BH modules
- **BH-specific platforms (Qualifacts, Valant):** Strong on BH workflows ✓ | Weak on FQHC billing ✗
  - Lack PPS encounter qualification logic, wraparound payment reconciliation
- **Market gap:** No platform designed ground-up for FQHC + BH simultaneously

**Source:** Research domain 5 (Technology Landscape)

---

## Quantified Pain Points (ROI Justification)

| Pain Point | Annual Cost per FQHC | Platform Impact | Value Created |
|------------|---------------------|-----------------|---------------|
| Claim denials (15-25% rate) | $420K | Reduce denial rate to <10% | $215K+ recovered |
| Staff turnover (35-70% rate) | $562K | Reduce turnover by 15% | $168K saved |
| Compliance risk (Part 2, FTCA) | $7.5K-$1.5M | Automated tracking, validation | Risk mitigation |
| Manual reconciliation (30-50% of Director time) | $50K+ | Real-time unified view | Efficiency gain |

**Total quantifiable value:** $383K+ per FQHC annually (denial recovery + turnover savings)

**Platform pricing:** $100K-$150K annually (base + performance)

**Customer ROI:** 2.5-3.8x

---

## Market Sizing

**Total Addressable Market (TAM):**
- 1,400 FQHCs, 70%+ offer mental health (~1,000 FQHCs)
- Average revenue per FQHC: $100K-$150K annually
- **TAM: $60-100M annually**

**Serviceable Addressable Market (SAM):**
- Target: Mid-sized FQHCs (5-20 locations) with established BH programs
- 400-600 FQHCs in target segment
- **SAM: $60M annually**

**Achievable ARR:**
- Year 1: $350K (5-10 pilot FQHCs)
- Year 3: $5M (50 FQHCs)
- Year 5: $19.5M (150 FQHCs)

---

## Strategic Recommendation

**Go-to-market:** Hybrid approach
1. **Phase 1 (0-12 months):** Integration layer on athenahealth (fastest path to market, validate demand)
2. **Phase 2 (12-24 months):** Expand features, build standalone platform in parallel (long-term independence)
3. **Phase 3 (24-36 months):** Scale standalone, migrate customers, consider acquisition (Qualifacts, Cantata)

**Pricing:** Base fee ($50K-$75K) + performance incentive (20-25% of recovered revenue) = $100K-$150K total

**Lead with:** Denial prevention (highest ROI, fastest payback) + compliance automation (risk mitigation, insurance premium logic)

**Capital required:** $7-10M over 5 years (seed $2-3M, Series A $5-7M)

---

## Cross-References to Full Research

**Detailed findings:**
- **Job profiles & role definition:** `/ava_research.md` (Domain 1, lines 12-169)
- **FQHC domain overview:** `/ava_research.md` (Domain 2, lines 171-346)
- **Capacity planning workflows:** `/ava_research.md` (Domain 3, lines 348-473)
- **Revenue cycle management:** `/ava_research.md` (Domain 4, lines 475-709)
- **Technology landscape:** `/ava_research.md` (Domain 5, lines 711-969)
- **Compliance & regulatory:** `/ava_research.md` (Domain 6, lines 971-1347)

**Architecture analysis:** `/serena_architecture.md` (integration patterns, data model requirements, build vs buy options)

**Engineering feasibility:** `/marcus_engineering.md` (build complexity, technology stack, timeline estimates)

**Risk assessment:** `/rook_risk.md` (regulatory timeline, compliance requirements, security threat model)

**Market & monetization:** `/rich_monetization.md` (TAM/SAM/SOM, pricing models, go-to-market strategy)

**Executive synthesis:** `/pi_synthesis.md` (cross-domain insights, strategic recommendations, Phase 2 debate topics)

---

## Next Phase

**Phase 2: Council Debate (PM002)**
- **Participants:** Serena (Architect), Marcus (Engineer), Rook (Risk), Rich (Monetization), Pi (Orchestrator)
- **Topics:** Build vs Buy vs Partner, architecture approach, pricing validation, go-to-market sequencing
- **Output:** Multi-perspective strategic recommendation with consensus + dissenting opinions

**Status:** Phase 1 (Deep Investigation) COMPLETE → Phase 2 PENDING

---

**Document prepared by Pi for Project Mitchell PM001**
**2026-03-27**
