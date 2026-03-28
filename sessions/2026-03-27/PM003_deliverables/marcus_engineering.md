# Marcus Webb - Engineering Plan
## PM003: Build Plan and Implementation Roadmap

**Date:** 2026-03-27
**Session:** PM003_deliverables
**Document Type:** Engineering implementation plan
**Extracted From:** DELIVERABLE-3 (Technical Architecture + Component Specifications)
**Status:** Ready for sprint planning

---

## Executive Summary

The dashboard is buildable in 4-6 months with a team of 5 engineers (2 frontend, 2 backend, 1 DevOps). MVP can ship in 8 weeks with limited integrations. Total estimated effort: 120-180 engineering weeks.

**Technology Stack Chosen:**
- **Frontend:** React 18 + Tailwind CSS + TanStack Query (battle-tested, large community)
- **Backend:** FastAPI (Python) - fast async, automatic OpenAPI docs, strong typing
- **Database:** PostgreSQL (ACID compliance critical for audit trails)
- **Integrations:** HL7 FHIR (EHR), X12 EDI (billing), proprietary APIs

**Key Engineering Risks:**
1. Integration complexity (4-6 vendor APIs, each with quirks)
2. 42 CFR Part 2 consent enforcement (strict legal requirement)
3. Audit trail immutability (cannot be wrong, cannot be lost)

---

## MVP Scope (8 weeks, 40 engineering weeks)

**Included:**
1. **Morning Brief module** (unified dashboard view)
2. **Capacity Planning module** (real-time availability by provider)
3. **Risk Compliance module** (credentialing alerts, authorization tracking)
4. **Basic audit logging** (who did what, when)
5. **One EHR integration** (Epic FHIR or NextGen)

**Explicitly Excluded from MVP:**
- Full RCM module (denials, claims aging) - Phase 2
- Team Management module - Phase 2
- Analytics/reporting - Phase 2
- Multi-FQHC support - Phase 2
- Mobile app (PWA only in MVP)

**MVP Success Criteria:**
- Director can answer "Can we take this patient next week?" in <5 minutes (vs 30 minutes baseline)
- Compliance score visible in <3 seconds
- Zero compliance failures during 90-day pilot

---

## Phase 1: Foundation (Weeks 1-4, 20 engineering weeks)

### Sprint 1-2: Core Infrastructure
**Backend (2 engineers x 2 weeks = 4 weeks):**
- [ ] FastAPI project setup (authentication, database, migrations)
- [ ] PostgreSQL schema implementation (providers, patients, appointments, authorizations, audit_log)
- [ ] OAuth 2.0 + JWT authentication
- [ ] RBAC (role-based access control) middleware
- [ ] Audit logging middleware (auto-log all API calls)
- [ ] Docker setup (Dockerfile, docker-compose for local dev)

**Frontend (2 engineers x 2 weeks = 4 weeks):**
- [ ] React 18 + Vite setup (fast dev server, hot reload)
- [ ] Tailwind CSS configuration (design system tokens)
- [ ] React Router setup (pages: Login, Dashboard, Capacity, Risk, Settings)
- [ ] TanStack Query setup (caching, background sync)
- [ ] Authentication flow (login, logout, token refresh)
- [ ] Layout components (Header, Sidebar, MainContent, Footer)

**DevOps (1 engineer x 2 weeks = 2 weeks):**
- [ ] AWS account setup (VPC, subnets, security groups)
- [ ] RDS PostgreSQL (Multi-AZ, automated backups)
- [ ] ElastiCache Redis (single-node for dev, replicated for prod)
- [ ] S3 bucket (reports, encrypted at rest)
- [ ] CI/CD pipeline (GitHub Actions: lint, test, build, deploy)

**Deliverable:** Deployed skeleton app (login works, empty dashboard shows)

### Sprint 3-4: Morning Brief Module
**Backend (2 engineers x 2 weeks = 4 weeks):**
- [ ] API endpoint: GET /dashboard/morning-brief
- [ ] Data aggregation logic (crisis capacity, compliance alerts, revenue snapshot, capacity)
- [ ] Narrative generation (convert data → human-readable summary)
- [ ] Caching layer (Redis, 5-minute TTL)
- [ ] Background sync job (Celery task, runs every 5 minutes)

**Frontend (2 engineers x 2 weeks = 4 weeks):**
- [ ] Morning Brief page (dashboard layout)
- [ ] Summary card (narrative text + key metrics)
- [ ] Alert cards (credentialing, authorization, denial alerts)
- [ ] Interactive actions ("Review 3 providers needing assignment" → link to Capacity page)
- [ ] Loading states, error states, empty states

**Testing (all engineers, 2 weeks):**
- [ ] Unit tests (backend: 80% coverage)
- [ ] Integration tests (API endpoints)
- [ ] E2E tests (Playwright: login → morning brief loads)

**Deliverable:** Morning Brief shows real data (even if some is mocked)

---

## Phase 2: Capacity & Risk (Weeks 5-8, 20 engineering weeks)

### Sprint 5-6: Capacity Planning Module
**Backend (2 engineers x 2 weeks = 4 weeks):**
- [ ] API endpoints: GET /providers, GET /providers/:id/availability, POST /patients/:id/assign
- [ ] Provider availability calculation (total_slots - scheduled - PTO)
- [ ] Waitlist management (triage by urgency/insurance)
- [ ] Patient assignment workflow (compliance checks before scheduling)
- [ ] EHR integration (Epic FHIR: sync appointments every 5 minutes)

**Frontend (2 engineers x 2 weeks = 4 weeks):**
- [ ] Capacity Planning page (provider list + availability grid)
- [ ] Provider cards (name, role, utilization, availability by day)
- [ ] Waitlist panel (patients awaiting assignment, sorted by urgency)
- [ ] Assignment modal (select provider + date, compliance checks run)
- [ ] Compliance error handling (modal: "Cannot schedule - reason + resolution")

**Testing (all engineers, 2 weeks):**
- [ ] Unit tests (availability calculation, assignment logic)
- [ ] Integration tests (EHR sync, assignment workflow)
- [ ] E2E tests (assign patient flow, compliance blocks work)

**Deliverable:** Directors can assign patients with embedded compliance checks

### Sprint 7-8: Risk Compliance Module
**Backend (2 engineers x 2 weeks = 4 weeks):**
- [ ] API endpoints: GET /compliance/audit-readiness, GET /compliance/credentialing
- [ ] Compliance score calculation (100-point scale)
- [ ] Credentialing tracker (license expirations, NPDB queries)
- [ ] Authorization tracking (expirations in <30 days)
- [ ] Alert system (email/SMS when critical alert)

**Frontend (2 engineers x 2 weeks = 4 weeks):**
- [ ] Risk Compliance page (audit readiness score + alerts)
- [ ] Compliance scorecard (breakdown by category)
- [ ] Alert list (credentialing, authorization, documentation gaps)
- [ ] Provider detail panels (credentialing status, upcoming expirations)
- [ ] One-click report generation (button → backend job → PDF download)

**Testing (all engineers, 2 weeks):**
- [ ] Unit tests (score calculation, alert logic)
- [ ] Integration tests (compliance queries)
- [ ] E2E tests (audit readiness page loads, report downloads)

**Deliverable:** Audit readiness visible in <3 seconds, one-click reports work

---

## Phase 3: Polish & Pilot (Weeks 9-12, optional if MVP ships at Week 8)

### Sprint 9-10: Integration Hardening
**Backend (2 engineers x 2 weeks = 4 weeks):**
- [ ] Retry logic with exponential backoff (1s, 2s, 4s, 8s, 16s)
- [ ] Graceful degradation (show cached data with staleness warning)
- [ ] Manual fallback mode (allow manual data entry when integration fails)
- [ ] Integration status dashboard (admin-only: see sync health)

**Frontend (2 engineers x 2 weeks = 4 weeks):**
- [ ] Integration failure notifications (toast: "Billing system sync failed")
- [ ] Staleness badges ("Data may be stale (last sync 2 hours ago)")
- [ ] Manual mode UI (form to manually enter authorization data)
- [ ] Performance optimization (lazy loading, code splitting, image optimization)

### Sprint 11-12: Pilot Preparation
**All engineers (5 engineers x 2 weeks = 10 weeks):**
- [ ] Security audit (OWASP Top 10, HIPAA checklist)
- [ ] Performance testing (load test: 100 concurrent users)
- [ ] Accessibility audit (WCAG 2.1 AA compliance)
- [ ] Documentation (API docs, admin guide, user guide)
- [ ] Pilot FQHC onboarding (data migration, training materials)

**Deliverable:** Production-ready MVP deployed to pilot FQHC

---

## Technology Stack Details

### Frontend Stack

**Core:**
- **React 18.2+** (component-based, hooks, Suspense)
- **Vite 4+** (fast dev server, native ESM, optimized builds)
- **React Router 6+** (declarative routing, code splitting)
- **Tailwind CSS 3+** (utility-first, responsive, customizable)

**State & Data:**
- **Zustand** (lightweight state management, simpler than Redux)
- **TanStack Query** (data fetching, caching, background sync, offline support)
- **React Hook Form** (performance, validation, accessibility)

**UI & Visualization:**
- **Recharts** (React-native charts, accessible, customizable)
- **date-fns** (lightweight date manipulation, tree-shakeable)
- **Lucide React** (icon library, tree-shakeable)

**Testing:**
- **Vitest** (fast, Vite-native, Jest-compatible)
- **React Testing Library** (user-centric testing)
- **Playwright** (E2E testing, cross-browser)

### Backend Stack

**Core:**
- **FastAPI 0.100+** (async, automatic OpenAPI docs, Pydantic validation)
- **Python 3.11+** (type hints, performance improvements)
- **SQLAlchemy 2.0** (ORM, async support)
- **Alembic** (database migrations)

**Background Jobs:**
- **Celery 5.3+** (distributed task queue)
- **Redis 7+** (message broker for Celery, cache)
- **Celery Beat** (periodic task scheduling)

**Integrations:**
- **fhir.resources** (FHIR R4 models)
- **httpx** (async HTTP client)
- **python-hl7** (HL7 v2 parsing if needed)

**Testing:**
- **pytest** (test framework)
- **pytest-asyncio** (async test support)
- **httpx-mock** (mock HTTP requests)

### Infrastructure

**AWS Services:**
- **ECS Fargate** (containerized FastAPI app, auto-scaling)
- **RDS PostgreSQL 15** (Multi-AZ, automated backups, encryption at rest)
- **ElastiCache Redis** (cache, session store)
- **S3** (PDF reports, encrypted at rest)
- **CloudFront** (CDN for static assets)
- **Route 53** (DNS)
- **ALB** (Application Load Balancer, TLS termination)
- **Secrets Manager** (API keys, database credentials)

**CI/CD:**
- **GitHub Actions** (automated testing, builds, deployments)
- **Docker** (containerization)
- **Terraform** (infrastructure as code, optional)

---

## Build Complexity Estimate

### By Module (Engineering Weeks)

| Module | Backend | Frontend | DevOps | Testing | Total |
|--------|---------|----------|--------|---------|-------|
| Foundation | 12 | 12 | 6 | 4 | 34 |
| Morning Brief | 8 | 8 | 2 | 4 | 22 |
| Capacity Planning | 12 | 10 | 2 | 6 | 30 |
| Risk Compliance | 10 | 8 | 2 | 6 | 26 |
| Integration Hardening | 8 | 6 | 4 | 4 | 22 |
| Pilot Prep | 6 | 6 | 4 | 8 | 24 |
| **TOTAL** | **56** | **50** | **20** | **32** | **158** |

**Adjusted Total (parallelization, overhead):** 120-180 engineering weeks

**Team Size:** 5 engineers (2 frontend, 2 backend, 1 DevOps)
**Timeline:** 24-36 weeks (6-9 months) for full product
**MVP Timeline:** 8 weeks (40 engineering weeks) with 5 engineers

---

## Integration Complexity

### Epic FHIR API
**Complexity:** Medium
**Effort:** 2-3 weeks
**Challenges:**
- OAuth 2.0 authentication (need Epic App Orchard approval)
- Rate limiting (100 requests/minute)
- Data format differences (FHIR → our schema mapping)

**Mitigation:**
- Use Epic FHIR sandbox for development
- Build abstraction layer (FHIRClient class)
- Implement retry logic + rate limiting

### NextGen API
**Complexity:** High
**Effort:** 3-4 weeks
**Challenges:**
- Proprietary API (no FHIR support in older versions)
- Inconsistent documentation
- Legacy SOAP endpoints (not REST)

**Mitigation:**
- Negotiate API access with NextGen early
- Build adapter pattern (NextGenClient → FHIRClient interface)
- Manual testing with pilot FQHC's NextGen instance

### athenahealth API
**Complexity:** Low-Medium
**Effort:** 2 weeks
**Challenges:**
- RESTful API (well-documented)
- OAuth 2.0 (standard)
- Good FHIR support

**Mitigation:**
- Use athenahealth developer sandbox
- Prioritize athenahealth if pilot FQHC uses it

---

## Risk Mitigation

### Risk 1: Integration Failures
**Probability:** High (external APIs will fail)
**Impact:** High (dashboard unusable without data)
**Mitigation:**
- Retry logic with exponential backoff
- Graceful degradation (cached data + staleness warning)
- Manual fallback mode (allow manual data entry)
- Integration status monitoring (alert when sync fails >5 times)

### Risk 2: 42 CFR Part 2 Compliance
**Probability:** Medium (complex legal requirement)
**Impact:** Critical (civil penalties up to $1.5M)
**Mitigation:**
- Legal review of consent enforcement logic
- Automated testing of SUD patient access controls
- Audit trail every disclosure (who accessed, when, why)
- Red team testing (try to bypass consent checks)

### Risk 3: Performance Degradation
**Probability:** Medium (complex queries, large datasets)
**Impact:** High (>3 second load time = directors abandon tool)
**Mitigation:**
- Database indexing (all foreign keys, frequently queried columns)
- Redis caching (5-minute TTL for dashboard aggregations)
- Query optimization (EXPLAIN ANALYZE all slow queries)
- Load testing (simulate 100 concurrent users)

### Risk 4: Audit Trail Corruption
**Probability:** Low (PostgreSQL is reliable)
**Impact:** Critical (legal defensibility lost)
**Mitigation:**
- Append-only table (no updates/deletes allowed)
- Cryptographic signatures (SHA-256 hash of each entry)
- Regular backups (hourly incremental, daily full)
- Replication (Multi-AZ RDS, read replicas)

---

## Deployment Strategy

### Environments

**Development (Local):**
- Docker Compose (PostgreSQL, Redis, FastAPI, React dev server)
- Seed data (synthetic patients, providers, appointments)
- Mock integrations (no real EHR/billing APIs)

**Staging (AWS):**
- Single-AZ RDS (smaller instance)
- ECS Fargate (1 task)
- Sandbox integrations (Epic FHIR sandbox)
- Updated nightly from `main` branch

**Production (AWS):**
- Multi-AZ RDS (automated backups, encryption)
- ECS Fargate (2+ tasks, auto-scaling)
- Real integrations (pilot FQHC's EHR/billing)
- Deployed manually after QA approval

### CI/CD Pipeline

**GitHub Actions Workflow:**
```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: |
          pip install -r requirements.txt
          pytest --cov=app tests/
          npm install
          npm run test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: docker build -t dashboard:latest .
      - name: Push to ECR
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin ecr
          docker tag dashboard:latest ecr/dashboard:latest
          docker push ecr/dashboard:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to ECS
        run: aws ecs update-service --cluster prod --service dashboard --force-new-deployment
```

---

## Timeline Summary

**MVP (8 weeks):**
- Weeks 1-4: Foundation + Morning Brief
- Weeks 5-8: Capacity + Risk modules

**Full Product (24 weeks):**
- Weeks 1-4: Foundation + Morning Brief
- Weeks 5-8: Capacity + Risk modules
- Weeks 9-12: RCM + Team Management modules
- Weeks 13-16: Analytics + Reporting
- Weeks 17-20: Mobile optimization + PWA features
- Weeks 21-24: Polish + security audit + pilot prep

**Post-Launch:**
- Weeks 25-28: Pilot FQHC deployment (1 org)
- Weeks 29-36: Iteration based on pilot feedback
- Weeks 37+: Scale to additional FQHCs

---

## Resource Requirements

**Team (MVP):**
- 2 Frontend Engineers (React, TypeScript, Tailwind)
- 2 Backend Engineers (Python, FastAPI, PostgreSQL)
- 1 DevOps Engineer (AWS, Docker, CI/CD)

**Team (Full Product):**
- 3 Frontend Engineers
- 3 Backend Engineers
- 1 DevOps Engineer
- 1 QA Engineer
- 1 Product Manager
- 1 Designer (part-time)

**Budget (MVP, 8 weeks):**
- Engineering: $80K (5 engineers x $160/hr x 40 hrs/week x 8 weeks = $256K)
- AWS: $2K/month x 2 months = $4K
- Third-party services: $1K (Epic sandbox, monitoring tools)
- **Total MVP: ~$260K**

**Budget (Full Product, 24 weeks):**
- Engineering: $384K (5 engineers → 8 engineers over 24 weeks, blended)
- AWS: $5K/month x 6 months = $30K
- Third-party services: $5K (multiple EHR sandboxes, monitoring, security tools)
- **Total Full Product: ~$420K**

---

## Conclusion

This is buildable in 8 weeks (MVP) or 24 weeks (full product). The technology stack is mature, the architecture is sound, and the risks are mitigated.

**Critical Path:**
1. Week 1-2: Foundation (deploy empty app)
2. Week 3-4: Morning Brief (first real value)
3. Week 5-6: Capacity Planning (solve "30-minute question")
4. Week 7-8: Risk Compliance (audit readiness)

**Next Steps:**
1. Assemble team (5 engineers)
2. Set up AWS account + CI/CD
3. Sprint 1 kickoff (Foundation)

---

**Marcus Webb**
Engineering Lead
PM003: Deliverables Phase
2026-03-27
