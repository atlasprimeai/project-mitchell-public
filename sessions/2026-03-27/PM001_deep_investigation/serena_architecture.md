# Serena Blackwood - Architecture Analysis
# Project Mitchell PM001 - System Design Patterns & Integration Requirements

**Agent:** Serena Blackwood (Architect)
**Session:** PM001_deep_investigation
**Date:** 2026-03-27
**Focus:** Architecture patterns, integration requirements, technical constraints extracted from Ava's research

---

## Executive Summary

The FQHC behavioral health technology landscape reveals a **fundamental architecture failure**: existing platforms were designed for either medical workflows (EHR-first) or behavioral health workflows (BH-first), then retrofitted for the other domain via bolt-on modules. This creates fragmentation at the data model level, not just the UI level.

**Core architectural challenge:** Build a unified data model that natively understands both FQHC billing logic (PPS encounter qualification, wraparound payment reconciliation) AND behavioral health treatment workflows (multi-session episodes, 42 CFR Part 2 consent, measurement-based care).

---

## Current State Architecture Analysis

### The 4-6 System Fragmentation Pattern

**Typical FQHC BH Director technology stack:**

```
┌─────────────────────────────────────────────────────────────┐
│  Clinical Layer                                              │
│  - EHR (Epic, NextGen, eClinicalWorks, athenahealth)       │
│  - Patient records, clinical documentation, treatment plans │
└─────────────────────────────────────────────────────────────┘
                          ↓ (manual reconciliation)
┌─────────────────────────────────────────────────────────────┐
│  Scheduling Layer                                            │
│  - Practice Management (often integrated with EHR)          │
│  - Provider availability, appointment booking, waitlists    │
└─────────────────────────────────────────────────────────────┘
                          ↓ (manual reconciliation)
┌─────────────────────────────────────────────────────────────┐
│  Billing Layer                                               │
│  - RCM Platform (Cantata, Qualifacts, NextGen billing)     │
│  - Claims generation, authorization tracking, A/R           │
└─────────────────────────────────────────────────────────────┘
                          ↓ (manual reconciliation)
┌─────────────────────────────────────────────────────────────┐
│  Compliance Layer                                            │
│  - Credentialing (manual/spreadsheet)                       │
│  - FTCA tracking (manual/spreadsheet)                       │
│  - 42 CFR Part 2 consent logs (paper/spreadsheet)          │
│  - UDS reporting (manual data aggregation)                  │
└─────────────────────────────────────────────────────────────┘
```

**Integration failures:**
1. **No shared patient identifier** - Each system maintains separate patient records
2. **No real-time sync** - Batch updates (nightly) or manual entry
3. **No unified authorization model** - Authorization status lives in billing, but scheduling needs it
4. **No credentialing-aware scheduling** - Can schedule with expired provider licenses
5. **No Part 2 consent enforcement** - SUD records can leak across systems without proper consent

### The Bolt-On Module Problem

**Medical EHR → BH Bolt-On (Epic, athenahealth, NextGen):**

```
Core Medical Data Model:
- Visit = discrete encounter (acute care paradigm)
- Billing = fee-for-service CPT codes
- Documentation = SOAP notes optimized for medical specialties

Behavioral Health Bolt-On:
- Treatment episodes ≠ discrete visits (poor fit)
- Therapy notes forced into SOAP format (poor fit)
- 42 CFR Part 2 consent added as separate module (leakage risk)
```

**Result:** "60% of athenahealth FQHC users request better behavioral health modules" - the data model doesn't natively support BH workflows.

**BH Platform → FQHC Billing Bolt-On (Qualifacts, Valant):**

```
Core BH Data Model:
- Treatment episodes = multi-session care plans
- Therapy notes = narrative, measurement-based care (PHQ-9, GAD-7)
- Group therapy = native concept

FQHC Billing Bolt-On:
- PPS encounter qualification logic ≠ standard CPT billing (poor fit)
- Wraparound payment reconciliation added as separate module (poor fit)
- Sliding fee scale calculation bolted on (poor fit)
```

**Result:** Great for BH workflows, weak for FQHC financial sustainability.

---

## Integration Patterns & Anti-Patterns

### Anti-Pattern 1: Paper Transport

**What it is:**
- Print report from System A
- Manually enter data into System B
- No API, no data sync

**Where it happens:**
- Authorization status (payer portal → billing system)
- Credentialing expiration dates (licensing board → HR spreadsheet)
- UDS reporting (EHR → manual aggregation → HRSA submission)

**Why it persists:**
- Vendor APIs don't exist or are prohibitively expensive
- Custom integration development not feasible for small FQHCs
- Compliance risk accepted as cost of doing business

### Anti-Pattern 2: Tribal Knowledge

**What it is:**
- Critical data stored in staff memory, not systems
- "Don't schedule with Dr. X on Fridays, she's not really available"
- "Patient Y needs special authorization, call billing first"

**Why it happens:**
- Systems show technical status, not operational reality
- EHR shows "available," but provider is on leave/training/burnout
- Authorization "approved" in billing, but expired yesterday

**Architecture implication:** Systems must model operational state, not just data-entry state.

### Anti-Pattern 3: Excel Shadow Systems

**What it is:**
- Critical operational data maintained outside formal systems
- Authorization expiration tracking spreadsheet
- Credentialing renewal calendar
- Capacity planning workbook

**Why it happens:**
- Formal systems don't provide the views/filters/alerts needed
- Directors need real-time operational dashboards
- Vendor platforms designed for data entry, not decision support

**Architecture implication:** Platform must provide operational intelligence layer, not just CRUD operations.

### Integration Pattern: Real-Time Eligibility Gateway

**What it should be:**

```
Scheduling Event (User schedules Patient X with Provider Y)
        ↓
Real-Time Validation Layer:
        ↓
Checks (parallel):
1. Provider credentialing current? (Licensing board API)
2. Patient insurance active? (Payer eligibility API)
3. Authorization on file? (Billing system API)
4. 42 CFR Part 2 consent signed? (Compliance DB)
5. Provider capacity available? (Scheduling system)
        ↓
Result:
- ✓ All checks pass → Schedule confirmed
- ✗ Any check fails → Block scheduling, surface reason
```

**Current reality:**
- All checks happen AFTER scheduling, if at all
- Claims denied weeks later when authorization missing
- Patient scheduled with expired provider license (FTCA risk)

**Architecture requirement:** Event-driven validation layer with real-time API integrations.

---

## Data Model Requirements

### Core Entities (FQHC BH Specific)

**1. Patient (Extended)**
```
Standard fields: Name, DOB, Address, Phone, MRN
FQHC-specific fields:
- Sliding fee discount % (income-verified)
- Income verification date (annual requirement)
- Service area eligibility (Scope of Project validation)
BH-specific fields:
- 42 CFR Part 2 consent status (SUD records)
- Consent effective dates (TPO consent)
- Disclosure log (Part 2 tracking)
```

**2. Provider (Extended)**
```
Standard fields: Name, License, Specialty, Schedule
FQHC-specific fields:
- FTCA deeming status (active/pending/expired)
- Credentialing cycle date (every 2 years)
- NPDB query date (last verification)
- Privileging scope (approved services)
- Scope of Project approval (which sites/services)
BH-specific fields:
- License type (LCSW, LMFT, LPC)
- Medicare recognition status (Jan 2024 expansion)
- Prescribing authority (MAT, controlled substances)
```

**3. Encounter (FQHC BH Paradigm)**
```
Standard fields: Date, Provider, Diagnosis, Procedures
FQHC-specific fields:
- PPS encounter qualification status (meets criteria?)
- Same-day consolidation flag (medical + BH on one claim)
- Wraparound payment eligibility (Medicaid MCO + state)
- Sliding fee discount applied (patient responsibility calculation)
BH-specific fields:
- Treatment episode ID (multi-session care plan)
- Measurement-based care scores (PHQ-9, GAD-7, AUDIT)
- Treatment modality (individual, group, family, telehealth)
- 42 CFR Part 2 record flag (SUD-related)
```

**4. Authorization (Multi-Payer Complexity)**
```
Standard fields: Payer, Service, Approval date, Expiration date
FQHC-specific fields:
- Payer type (Medicare PPS, Medicaid MCO, wraparound)
- Encounter qualification criteria (what services qualify)
BH-specific fields:
- Pre-auth required services (psychiatric eval, IOP, MAT)
- Session count approved/used (treatment episode tracking)
- Clinical review dates (for ongoing authorization)
```

**5. Claim (FQHC Billing Logic)**
```
Standard fields: Service date, Provider, Diagnosis, Procedures, Amount
FQHC-specific fields:
- G-code/T-code (FQHC-specific billing codes)
- PPS rate applied (Medicare base rate + adjustments)
- Wraparound payment calculation (MCO + state reconciliation)
- Same-day consolidation logic (medical + BH on one claim)
BH-specific fields:
- CoCM billing components (care manager, psychiatrist, registry)
- Denial reason (if rejected)
- Resubmission status (65% never resubmitted - track this)
```

---

## Integration Architecture

### Option 1: Greenfield Platform (Ground-Up Design)

**Pros:**
- Unified data model from day one
- Native FQHC + BH logic, not bolt-ons
- Modern tech stack (cloud-native, API-first)
- No legacy technical debt

**Cons:**
- Must build everything (clinical documentation, scheduling, billing, compliance)
- Compete with established EHR vendors (Epic, NextGen)
- Long time-to-market (2-3 years minimum)
- High capital requirements

**Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│  Unified Platform                                            │
│  - Shared data model (Patient, Provider, Encounter, Claim)  │
│  - Event-driven architecture (validation hooks)             │
│  - Real-time sync (no batch updates)                        │
│  - Compliance-aware (42 CFR Part 2 baked in)                │
│                                                              │
│  Modules:                                                    │
│  - Clinical (EHR)                                            │
│  - Scheduling (capacity planning)                           │
│  - Billing (PPS encounter qualification, wraparound)        │
│  - Compliance (FTCA, credentialing, UDS, Part 2)            │
│  - Analytics (denial prediction, capacity forecasting)      │
└─────────────────────────────────────────────────────────────┘
```

### Option 2: Integration Layer (Middleware Approach)

**Pros:**
- Work with existing EHRs (Epic, NextGen, athenahealth)
- Faster time-to-market (6-12 months)
- Lower capital requirements
- Directors keep familiar EHR, add intelligence layer

**Cons:**
- Dependent on vendor APIs (limited, expensive, unreliable)
- Can't fix underlying data model problems
- Integration maintenance burden (vendor changes break integrations)
- Limited to data available via APIs (many workarounds remain)

**Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│  Existing EHR (Epic, NextGen, athenahealth)                 │
│  - Clinical documentation                                    │
│  - Patient records                                           │
└─────────────────────────────────────────────────────────────┘
                          ↓ API Integration
┌─────────────────────────────────────────────────────────────┐
│  Intelligence Layer (New Platform)                           │
│  - Real-time validation (authorization, credentialing)      │
│  - Denial prediction (pre-submission checks)                │
│  - Capacity analytics (demand forecasting)                  │
│  - Compliance dashboard (FTCA, Part 2, UDS tracking)        │
└─────────────────────────────────────────────────────────────┘
                          ↓ API Integration
┌─────────────────────────────────────────────────────────────┐
│  Existing Billing Platform (Cantata, Qualifacts)            │
│  - Claims generation                                         │
│  - A/R management                                            │
└─────────────────────────────────────────────────────────────┘
```

### Option 3: Hybrid (Compliance + Analytics Module, Partner for EHR/Billing)

**Pros:**
- Focus on highest-value pain points (denial prevention, compliance tracking)
- Partner with established EHR vendors (co-sell opportunity)
- Faster time-to-market than greenfield
- Lower technical risk than full platform build

**Cons:**
- Still dependent on partner APIs
- Revenue share with partners reduces margin
- Brand visibility lower (white-label risk)

**Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│  Partner EHR (e.g., NextGen, athenahealth)                  │
│  - Clinical + Scheduling + Billing (core platform)          │
└─────────────────────────────────────────────────────────────┘
                          ↓ Deep Integration (certified partner)
┌─────────────────────────────────────────────────────────────┐
│  Specialized FQHC BH Module (New Platform)                  │
│  - AI denial prevention (pre-submission validation)         │
│  - 42 CFR Part 2 consent automation (SUD record protection) │
│  - FTCA credentialing tracker (2-year renewal cycle)        │
│  - UDS reporting automation (adaptive to federal changes)   │
│  - Capacity forecasting (demand prediction)                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Critical Architecture Decisions

### Decision 1: Event-Driven vs Request-Response

**Event-Driven (Recommended):**
- Every action (schedule appointment, submit claim, sign consent) emits event
- Validation services subscribe to events (real-time checks)
- Compliance audit trail automatic (every event logged)

**Request-Response (Traditional):**
- User requests action → system checks rules → allow/deny
- No automatic audit trail
- Validation gaps (checks only happen when explicitly coded)

**Recommendation:** Event-driven architecture with validation hooks for FTCA, Part 2, authorization, credentialing.

### Decision 2: Monolith vs Microservices

**Monolith:**
- Simpler initial development
- Single deployment unit
- Shared database (easier data consistency)

**Microservices:**
- Independent scaling (analytics service separate from clinical documentation)
- Technology diversity (Python for ML, Node for APIs)
- Organizational alignment (teams own services)

**Recommendation:** Start monolith for MVP (faster iteration), plan for microservice extraction as specific services prove high-value (e.g., denial prediction service).

### Decision 3: Cloud-Native vs On-Premises

**Cloud-Native (Recommended):**
- Elasticity (scale during UDS reporting season, scale down after)
- Security posture (AWS/Azure HIPAA compliance built-in)
- Disaster recovery (multi-region replication)
- Lower ops burden (managed services)

**On-Premises:**
- Some FQHCs have data residency requirements (rare)
- Perceived control (not actual security benefit)
- Higher ops burden

**Recommendation:** Cloud-native (AWS or Azure), with HIPAA BAA, encryption at rest/transit, audit logging.

### Decision 4: API-First vs Monolithic UI

**API-First (Recommended):**
- Mobile apps, integrations, partner ecosystems all use same API
- Validation logic in API layer (enforced everywhere)
- Third-party integrations easier

**Monolithic UI:**
- Simpler initial development
- Business logic in UI layer (harder to reuse)

**Recommendation:** API-first design, with separate web UI, mobile app, and integration API endpoints.

---

## Data Flow Patterns (Critical Use Cases)

### Use Case 1: Real-Time Scheduling Validation

**User action:** Front desk schedules Patient X with Provider Y for service Z

**Data flow:**
```
1. UI submits scheduling request to API
2. API emits "SchedulingRequested" event
3. Validation services (parallel checks):
   a. CredentialingService: Is Provider Y licensed + FTCA-deemed?
   b. AuthorizationService: Does Patient X have active auth for service Z?
   c. EligibilityService: Is Patient X insurance active today?
   d. ConsentService: If SUD service, does Patient X have Part 2 consent?
   e. CapacityService: Does Provider Y have actual availability?
4. If all checks pass → confirm scheduling
5. If any check fails → block scheduling, surface reason to user
6. Audit log: Record all checks + results (compliance trail)
```

**Current reality:** All checks happen manually or after-the-fact, resulting in claim denials weeks later.

### Use Case 2: Pre-Submission Claim Validation

**User action:** Billing staff submits claim for encounter

**Data flow:**
```
1. Billing system sends claim to API
2. API emits "ClaimSubmissionRequested" event
3. Validation services (parallel checks):
   a. PPS_EncounterService: Does clinical documentation meet PPS qualification?
   b. AuthorizationService: Was service pre-authorized (if required)?
   c. CodingService: Are G-codes/T-codes correct for service type?
   d. Part2Service: If SUD service, was proper consent obtained?
4. If all checks pass → submit to payer
5. If any check fails → block submission, surface correction needed
6. AI DenialPredictionService: Calculate denial risk score
7. If risk > threshold → flag for manual review before submission
```

**Current reality:** Claims submitted without validation, 15-25% denial rate, 65% never resubmitted.

### Use Case 3: Compliance Health Monitoring

**User action:** Director opens compliance dashboard (daily)

**Data flow:**
```
1. Dashboard queries ComplianceHealthService API
2. Service aggregates status from:
   a. CredentialingService: % providers with current licenses, upcoming expirations
   b. FTCAService: Deeming status, QI/QA meeting schedule (6x/year)
   c. Part2Service: % SUD patients with valid consent, disclosure logs complete
   d. ScopeOfProjectService: Approved vs actual service delivery (geographic, service types)
   e. UDSService: Data completeness for upcoming February 15 submission
3. Health score calculated (0-100)
4. Risk alerts surfaced (license expires in 30 days, QI/QA meeting overdue)
5. Audit trail: Dashboard access logged (who checked when)
```

**Current reality:** Director checks multiple spreadsheets, discovers compliance gaps reactively.

---

## Technology Stack Recommendations

### Frontend
- **Framework:** React (web), React Native (mobile)
- **State management:** Redux or Zustand
- **UI library:** Material-UI or Tailwind (accessible, WCAG 2.1 AA)

### Backend
- **API:** Node.js (Express or Fastify) or Python (FastAPI)
- **Database:** PostgreSQL (relational, strong ACID guarantees for billing)
- **Cache:** Redis (session state, real-time data)
- **Message queue:** RabbitMQ or AWS SQS (event-driven architecture)
- **Search:** Elasticsearch (patient/provider search, audit logs)

### AI/ML
- **Denial prediction:** Python (scikit-learn, XGBoost)
- **Demand forecasting:** Python (Prophet, ARIMA)
- **Ambient documentation:** OpenAI Whisper (speech-to-text) + GPT-4 (structuring)

### Infrastructure
- **Cloud:** AWS or Azure (HIPAA BAA required)
- **Containers:** Docker + Kubernetes (or ECS/EKS)
- **Monitoring:** DataDog or New Relic (HIPAA-compliant)
- **Logging:** AWS CloudWatch or Splunk (audit trail for compliance)

### Security
- **Encryption:** TLS 1.3 (transit), AES-256 (rest)
- **Authentication:** OAuth 2.0 + SAML (SSO integration with FQHC IdP)
- **Authorization:** RBAC (role-based access control)
- **Audit logging:** Immutable logs for HIPAA/Part 2 compliance

---

## Architecture Risk Analysis

### Risk 1: Vendor API Reliability (Integration Layer Approach)

**Risk:** EHR vendor APIs are unreliable, rate-limited, or change without notice

**Mitigation:**
- Design for API failure (graceful degradation, cached data)
- Contractual SLAs with vendor partners
- Fallback to manual entry if API unavailable

### Risk 2: Regulatory Change Velocity

**Risk:** UDS 2026 restructuring, 42 CFR Part 2 Feb 2026 enforcement - regulations change faster than software

**Mitigation:**
- Rules engine for compliance logic (externalize from code)
- Rapid update cycle (CI/CD pipeline for compliance patches)
- Subscription to HRSA/CMS/SAMHSA regulatory alerts

### Risk 3: Data Breach (42 CFR Part 2 + HIPAA)

**Risk:** SUD records leaked = civil penalties up to $1.5M annually per violation type

**Mitigation:**
- Encryption at rest + transit
- Role-based access control (least privilege)
- Audit logging (every access to Part 2 records logged)
- Annual penetration testing

### Risk 4: Scalability (1,400 FQHCs, 19,000 Sites)

**Risk:** Platform can't scale to full market

**Mitigation:**
- Cloud-native architecture (elastic scaling)
- Multi-tenancy design (single codebase, isolated data per FQHC)
- Load testing early (simulate 100+ concurrent FQHCs)

---

## Next Steps (Phase 2 Council Debate)

**Architecture questions for council:**
1. **Build vs Buy vs Partner?** Greenfield platform vs integration layer vs hybrid?
2. **Monolith vs Microservices?** Start simple or plan for scale from day one?
3. **Which pain point first?** Denial prevention, compliance tracking, or capacity planning?
4. **Partnership strategy?** White-label for existing EHR vendors vs standalone brand?

**Technical feasibility assessment needed:**
- API availability/quality from Epic, NextGen, athenahealth
- Data model compatibility (can we map their schemas to unified model?)
- Regulatory approval timeline (FTCA-approved vendors list, ONC certification)

— Serena Blackwood, Architect
Project Mitchell PM001
2026-03-27
