# Learnings & Retrospective
# Project Mitchell PM001 - Deep Investigation Phase

**Session:** PM001_deep_investigation
**Date:** 2026-03-27
**Retrospective Type:** Process reflection + domain insights

---

## What Worked Well

### 1. Parallel 6-Domain Research Approach

**What we did:**
- Divided research into 6 distinct domains (job profiles, FQHC operations, capacity planning, RCM, technology landscape, compliance)
- Ava researched all 6 domains with deep, focused investigation per domain
- Each domain report standalone valuable, but synthesis across domains revealed critical insights

**Why it worked:**
- Comprehensive coverage without shallow skimming
- Clear scope per domain (prevented scope creep)
- Cross-domain patterns emerged naturally (e.g., "Can we schedule this patient?" requires data from 4 domains)

**Evidence of success:**
- 80+ sources per domain (high-quality, not just Googling)
- 35,000+ total words of research (depth, not superficial)
- Key findings quantified (denial rate 15-25%, turnover 35-70%, revenue leakage $390K)

**Lesson:** For complex multi-faceted problems, parallel domain research with synthesis phase > sequential investigation.

### 2. Depth Over Breadth Strategy

**What we did:**
- 80+ sources across 6 domains (vs typical "quick research" of 10-20 sources)
- Read full regulatory documents (HRSA Compliance Manual, 42 CFR Part 2 Final Rule, UDS reporting specs)
- Traced primary sources (HRSA, CMS, SAMHSA official docs) vs relying on secondary summaries

**Why it worked:**
- Uncovered non-obvious insights (e.g., "Scope of Project violations = #1 HRSA citation" - not in vendor marketing materials)
- Quantification accuracy (denial rates, turnover costs, penalty amounts from authoritative sources)
- Regulatory timeline precision (42 CFR Part 2 enforcement began Feb 16, 2026 - exact date matters)

**Evidence of success:**
- High confidence ratings (90% overall, 95% for compliance/RCM domains)
- Specific actionable findings (not generic "healthcare is complex")
- Direct citations to federal agencies (HRSA, CMS, SAMHSA, OIG)

**Lesson:** In regulated industries (healthcare, finance), depth of regulatory research > breadth of industry surveys. Primary sources beat secondary summaries.

### 3. Quantification Focus (Every Pain Point Has a Dollar Value)

**What we did:**
- Denial rate: 15-25% → $390K revenue leakage per FQHC annually
- Turnover: 35-70% → $562K cost per FQHC annually
- Compliance penalties: $100-$50K per violation, up to $1.5M annually
- ROI calculation: Pay $125K, get $300K-$475K value = 2.5-3.8x ROI

**Why it worked:**
- Moves from "this is a problem" to "this is a $390K problem" (sales conversation vs complaint)
- ROI justification for $100K+ annual spend (FQHCs are cost-conscious, need proof)
- Prioritization clarity (denial prevention = $215K value vs compliance = risk mitigation)

**Evidence of success:**
- Rich's monetization analysis builds directly on quantified pain points
- Pricing model (base + performance) tied to value created, not arbitrary
- Customer ROI story clear (2.5-3.8x) without hand-waving

**Lesson:** For B2B enterprise sales, quantify EVERYTHING. Pain points without dollar values are anecdotes, not business cases.

### 4. Cross-Domain Synthesis Reveals Non-Obvious Insights

**What we did:**
- After 6 domain reports, Pi synthesized cross-domain patterns
- Example: "Can we schedule this patient?" problem spans capacity planning + RCM + compliance + technology
- Example: Staff turnover (#2 driver = documentation burden) connects to technology fragmentation (4-6 systems)

**Why it worked:**
- Insights emerge at intersections (capacity planning alone doesn't reveal the authorization tracking failure)
- Product requirements become clear (real-time validation layer addresses 4 domains simultaneously)
- Competitive positioning sharpens (no vendor solves FQHC billing + BH workflows = market gap)

**Evidence of success:**
- Pi's synthesis document identified 4 cross-domain insights (scheduling validation, denial prevention ROI, turnover connection, compliance-as-insurance)
- Strategic recommendations built on synthesis (hybrid go-to-market, lead with denial prevention, bundle compliance)

**Lesson:** Don't skip the synthesis phase. Domain experts see trees, synthesis reveals forest.

---

## What Could Be Better

### 1. Earlier User Interviews (Direct Practitioner Validation)

**What we did:**
- Secondary research (job postings, industry reports, regulatory docs)
- No direct interviews with FQHC Behavioral Health Directors

**What we should have done:**
- 5-10 user interviews early in Phase 1 (before deep domain research)
- Validate assumptions: "Is 30 minutes to answer 'Can we schedule this patient?' accurate, or is it worse/better?"
- Uncover non-obvious pain points (what are we missing by only reading documentation?)

**Why it matters:**
- Risk: We quantify the wrong pain points (denial prevention may not be #1, capacity planning might be)
- Risk: Pricing model misaligned (Directors may not value performance-based pricing)
- Risk: We miss critical workflow nuances (e.g., maybe warm handoffs from primary care are the real bottleneck)

**Next step for Phase 2:**
- Conduct 5-10 Director interviews before council debate
- Validate top 3 pain points (denial prevention, staff turnover, compliance risk)
- Test pricing model (base + performance vs flat fee)

**Lesson:** Secondary research builds market understanding, primary research validates product-market fit. Don't confuse the two.

### 2. Competitive Vendor Demos (Hands-On Experience)

**What we did:**
- Read vendor documentation (athenahealth, NextGen, Qualifacts, Cantata)
- Reviewed market analysis reports (Netsmart 2025)
- No hands-on demos of competing platforms

**What we should have done:**
- Request demos from athenahealth, NextGen, Qualifacts
- Identify specific workflow failures (where does the bolt-on module break?)
- Assess API quality (if integration layer strategy, can we actually integrate with their APIs?)

**Why it matters:**
- Risk: We overestimate vendor gaps ("they're terrible at BH workflows") when reality is more nuanced
- Risk: Integration layer strategy assumes good API quality (what if athenahealth API is unusable?)
- Risk: We miss differentiation opportunities (what specific features are missing that we can build?)

**Next step for Phase 2:**
- Request demos from top 3 vendors (athenahealth, NextGen, Qualifacts)
- Conduct API audit (documentation quality, rate limits, data model compatibility)
- Document specific workflow failures (video recordings of where platform breaks)

**Lesson:** Reading about competitors ≠ using competitors. Hands-on experience reveals implementation details documentation hides.

### 3. API Quality Assessment (Integration Layer Feasibility)

**What we did:**
- Recommended hybrid strategy (integration layer on athenahealth → standalone platform)
- Assumed athenahealth API is sufficient quality (no validation)

**What we should have done:**
- API audit BEFORE recommending integration layer strategy
- Test API calls (can we actually pull clinical data, scheduling data, authorization status?)
- Assess API reliability (rate limits, uptime SLA, change frequency)

**Why it matters:**
- Risk: Integration layer strategy fails if API quality is poor (Marcus flagged this in engineering analysis)
- Risk: Revenue depends on partner relationship (if athenahealth changes API, we break)
- Risk: Timeline assumptions (6-12 months) invalid if API integration takes 12-18 months

**Next step for Phase 2:**
- Request API documentation from athenahealth, NextGen
- Build proof-of-concept integration (1-2 week spike)
- Assess feasibility before committing to integration layer strategy

**Lesson:** "We'll integrate via API" is easy to say, hard to do. Validate API quality before strategic commitment.

### 4. Pricing Validation (Willingness-to-Pay Survey)

**What we did:**
- Proposed hybrid pricing model (base $50K-$75K + performance 20-25% of recovered revenue)
- Benchmarked against competitors ($15K-$50K for BH-only, $100K-$200K for general EHRs)
- No direct validation with target customers

**What we should have done:**
- Survey 10-20 Directors on willingness-to-pay
- Test pricing model variations (base + performance vs flat fee vs seat-based)
- Identify price sensitivity (at what price point do they say "too expensive"?)

**Why it matters:**
- Risk: $100K-$150K pricing is too high (FQHCs are cost-constrained, grants only 20% of budget)
- Risk: Performance-based pricing is confusing (Directors prefer predictable budgeting)
- Risk: We leave money on table (maybe they'd pay $200K+ for this value)

**Next step for Phase 2:**
- Pricing survey (10-20 Directors, anonymized)
- A/B test pricing models (base + performance vs flat fee)
- Identify willingness-to-pay ceiling

**Lesson:** ROI justification ≠ willingness-to-pay. Even if value is 3x cost, budget constraints may prevent purchase.

---

## Patterns Observed (Domain Insights)

### Pattern 1: Compliance > Efficiency

**Observation:**
- Directors prioritize avoiding audit failures over productivity gains
- 42 CFR Part 2 penalty ($1.5M) > denial prevention savings ($215K)
- FTCA coverage loss = existential risk (can't operate without malpractice coverage)

**Implication for product:**
- Lead sales pitch with compliance (risk mitigation, insurance premium logic)
- Bundle compliance with efficiency features (not standalone)
- Prioritize compliance features in roadmap (Part 2 consent, credentialing, Scope of Project)

**Evidence:**
- Rook's risk analysis: "Compliance is not a feature, it's the foundation"
- Research finding: "Scope of Project violations = #1 HRSA citation"

**Lesson:** In regulated industries, risk avoidance > ROI optimization. Sell insurance before efficiency.

### Pattern 2: Integration Pain is Universal (4-6 Systems is the Norm)

**Observation:**
- Every FQHC BH Director manages 4-6 disconnected systems
- Manual reconciliation across systems is standard workflow (not edge case)
- Excel shadow systems for critical data (authorization tracking, capacity planning)

**Implication for product:**
- "Unified platform" is differentiator (vs "better BH module")
- Real-time operational intelligence = killer feature (vs batch reporting)
- API integration strategy must work with 4-6 existing systems (not just one EHR)

**Evidence:**
- Serena's architecture analysis: "4-6 system fragmentation pattern"
- Marcus's engineering reality check: "Directors manually reconcile to answer 'Can we schedule this patient?'"

**Lesson:** Healthcare fragmentation is not a bug, it's the environment. Product must embrace integration complexity, not pretend it away.

### Pattern 3: Bolt-On Modules Fail (Legacy Architecture Problem)

**Observation:**
- General medical EHRs add BH as bolt-on → workflow friction, weak BH features
- BH-specific platforms add FQHC billing as bolt-on → weak PPS logic, no wraparound reconciliation
- "Legacy platforms treat behavioral health as an add-on—tacked onto a medical platform via bolt-on modules"

**Implication for product:**
- Greenfield platform opportunity (designed for FQHC + BH from day one)
- Data model must natively support both (not EHR-first with BH fields, or BH-first with billing fields)
- Differentiation = "not a bolt-on" (marketing message)

**Evidence:**
- Research finding: "60% of athenahealth FQHC users request better BH modules"
- Serena's architecture analysis: "The bolt-on module problem - data model doesn't natively support BH workflows"

**Lesson:** Product architecture reveals strategic positioning. Bolt-ons signal "we didn't design for your use case." Greenfield signals "we built this for you."

### Pattern 4: Performance-Based Pricing Aligns Incentives (Directors Pay for Outcomes)

**Observation:**
- Directors care about denial rate reduction, not seat count
- ROI justification required for $100K+ spend (grant funding only 20% of budget)
- Performance-based pricing differentiates from competitors (most charge flat fee or seat-based)

**Implication for product:**
- Hybrid pricing (base + performance) recommended
- Track denial rate before/after platform (prove value)
- Sales narrative: "Pay $125K, recover $300K = 2.4x ROI"

**Evidence:**
- Rich's monetization analysis: "Optimal pricing = base $50K-$75K + performance 20-25% of recovered revenue"
- Research finding: "Only 14% of FQHCs using AI denial prevention (82% want it)" - demand exists, pricing model may be barrier

**Lesson:** In value-based care era, pricing models should reflect outcomes, not inputs (seats, storage, transactions). Align pricing with customer's success metric.

---

## Next Steps for Validation (Phase 2 Prerequisites)

### 1. User Interviews (5-10 Directors at Target FQHCs)

**Questions to validate:**
- Is "Can we schedule this patient?" actually a 30-minute question, or is it worse/better?
- Is denial prevention the #1 pain point, or is capacity planning / staff turnover / compliance more urgent?
- Would you pay $100K-$150K annually for 50% denial rate reduction + compliance automation?
- Do you prefer base + performance pricing, or flat fee, or seat-based?

**Target interviewees:**
- Mid-sized FQHCs (5-15 sites) with high BH volume (20%+ of visits)
- Directors who recently experienced audit (know compliance pain)
- Directors with high turnover (know staff retention pain)

### 2. Competitive Demos (athenahealth, NextGen, Qualifacts)

**What to observe:**
- Workflow failures: Where does the platform break? (e.g., scheduling doesn't check authorization)
- User frustration: What do users complain about? (e.g., "I have to log into 3 systems to check this")
- Feature gaps: What features are missing? (e.g., no automated Part 2 consent management)

**Deliverable:** Competitive analysis matrix (athenahealth vs NextGen vs Qualifacts vs us)

### 3. API Audit (athenahealth, NextGen Integration Quality)

**What to test:**
- Can we pull clinical data? (patient records, encounters, diagnoses)
- Can we pull scheduling data? (provider availability, appointments)
- Can we pull billing data? (claims status, authorization, A/R)
- What are rate limits? (can we support real-time validation?)
- What is API reliability? (uptime SLA, change frequency, versioning)

**Deliverable:** API feasibility report (integration layer strategy viable: yes/no)

### 4. Pricing Survey (10-20 Directors, Anonymized)

**Survey questions:**
- What do you currently pay for EHR + billing + compliance platforms? (total annual spend)
- If a platform reduced BH denial rate from 20% to <10%, what would you pay annually?
- Would you prefer: (A) Base $75K + 25% of recovered revenue, (B) Flat $125K, (C) $500/user/month?
- At what price point is this "too expensive to consider"? ($50K, $100K, $150K, $200K+)

**Deliverable:** Pricing validation report (optimal price point, model preference)

### 5. Regulatory Consultation (HRSA, State Medicaid on Certification)

**Questions to ask:**
- What certifications are required for FQHC vendor platforms? (ONC, SOC 2, FTCA-approved list?)
- What is timeline for certification? (how long to get on FTCA-approved vendor list?)
- What are audit triggers for new vendors? (will FQHCs using our platform get extra scrutiny?)

**Deliverable:** Regulatory roadmap (certifications required, timeline, cost)

---

## Process Improvements for Future Research Sessions

### 1. Parallel Research + Synthesis Works, Keep Doing It

**What worked:** 6 domains researched in parallel (Ava), then synthesized (Pi)

**Keep doing:** Divide complex problems into domains, research deeply, synthesize insights across domains

### 2. Add User Validation Earlier (Don't Wait Until End of Phase 1)

**What didn't work:** Finished all research before talking to users

**Improvement:** Conduct 3-5 user interviews BEFORE deep domain research, then 5-10 more AFTER to validate findings

### 3. Prototype/Demo Before Strategic Commitment

**What didn't work:** Recommended integration layer strategy without API validation

**Improvement:** Build proof-of-concept (1-2 week spike) before strategic decision (build vs buy vs partner)

### 4. Quantification is King, But Need Primary Validation

**What worked:** Every pain point has dollar value (denial rate → $390K revenue leakage)

**Improvement:** Validate quantifications with user interviews ("Is $390K accurate for your FQHC?")

---

## Conclusion

**Phase 1 (Deep Investigation) delivered:**
- ✅ Validated market opportunity ($60-100M TAM, $4-20M ARR achievable)
- ✅ Quantified pain points ($383K+ value per FQHC annually)
- ✅ Identified vendor gaps (no platform solves FQHC billing + BH workflows)
- ✅ Defined strategic options (build vs buy vs partner)

**Phase 1 did NOT deliver:**
- ❌ User validation (no Director interviews)
- ❌ Competitive demos (no hands-on experience with athenahealth, NextGen)
- ❌ API quality assessment (no integration testing)
- ❌ Pricing validation (no willingness-to-pay survey)

**Phase 2 (Council Debate) should:**
1. Complete validation steps BEFORE strategic decision (interviews, demos, API audit, pricing survey)
2. Debate build vs buy vs partner with full information (not assumptions)
3. Output: Strategic recommendation with evidence (not just opinion)

**Bottom line:** Phase 1 research is strong foundation, but strategic decision requires validation. Don't skip user interviews and API audit.

---

**Retrospective complete.**

— Project Mitchell PM001
2026-03-27
