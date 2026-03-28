# Serena Blackwood - Architecture Document
## PM003: Dashboard Prototype Technical Architecture

**Date:** 2026-03-27
**Session:** PM003_deliverables
**Document Type:** Technical architecture and system design
**Source:** DELIVERABLE-3-DASHBOARD-PROTOTYPE.md (Technical Architecture section)
**Status:** Developer-ready

---

## Executive Summary

The dashboard architecture is a Progressive Web App (PWA) designed to aggregate data from 4-6 disconnected FQHC systems (EHR, scheduling, billing, credentialing) into a unified, compliance-aware, action-oriented interface. The system prioritizes audit survivability over efficiency, embeds compliance checks into every workflow, and gracefully handles integration failures.

**Core Architecture Decisions:**
- **Platform:** PWA (offline capability, works on any device, no app store approval)
- **Frontend:** React 18+ with TanStack Query (caching, offline sync)
- **Backend:** FastAPI (Python) with PostgreSQL (ACID compliance for audit trails)
- **Integration:** HL7 FHIR + proprietary APIs (NextGen, athenahealth, Epic)
- **Security:** OAuth 2.0 + SAML, HIPAA + 42 CFR Part 2 compliant

---

## System Architecture Overview

### High-Level Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    Frontend (PWA)                                │
│  React 18 + Zustand + React Router + TanStack Query            │
│  - Morning Brief        - RCM Performance                        │
│  - Capacity Planning    - Team Management                        │
│  - Risk Compliance      - Analytics                              │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     │ HTTPS (TLS 1.3)
                     │
┌────────────────────▼────────────────────────────────────────────┐
│              API Layer (FastAPI)                                 │
│  - Authentication   - Dashboard Aggregation                      │
│  - Authorization    - Compliance Checks                          │
│  - Audit Logging    - Report Generation                          │
└────────┬───────────┬────────────┬───────────┬────────────┬──────┘
         │           │            │           │            │
         │           │            │           │            │
┌────────▼──┐ ┌──────▼───┐ ┌─────▼────┐ ┌────▼─────┐ ┌───▼──────┐
│PostgreSQL │ │  Redis   │ │Elastic-  │ │ Celery   │ │   S3     │
│ (Primary  │ │ (Cache/  │ │  search  │ │ (Background│ │(Reports) │
│   Data)   │ │ Session) │ │(Audit Log)│ │  Tasks)   │ │          │
└─────┬─────┘ └──────────┘ └──────────┘ └───────────┘ └──────────┘
      │
      │
┌─────▼────────────────────────────────────────────────────────────┐
│               Integration Layer                                   │
│  - EHR Connector (HL7 FHIR)                                      │
│  - Scheduling API (NextGen, athenahealth)                        │
│  - Billing System (X12 EDI 837/835)                             │
│  - Credentialing DB (Manual + scraping)                          │
└──────────────────────────────────────────────────────────────────┘
```

### Data Flow

**1. Morning Brief Generation (5-minute aggregation)**

```
User opens dashboard
  → API: GET /dashboard/morning-brief
  → Check cache (Redis): Valid < 5 minutes old?
     → YES: Return cached data
     → NO: Aggregate from sources:
        - PostgreSQL: Compliance alerts (credentialing, authorizations)
        - EHR API: Crisis capacity status, overnight crisis calls
        - Billing API: Denials from yesterday, aging A/R
        - Scheduling API: Available slots this week
  → Generate narrative summary
  → Cache result (5-minute TTL)
  → Return to frontend
```

**2. Patient Assignment Workflow (embedded compliance checks)**

```
User clicks "Assign Patient"
  → Modal opens with patient info + provider list
  → User selects provider + date
  → API: POST /patients/:id/assign
  → Compliance validation (BEFORE scheduling):
     1. Insurance verified? (Query billing system)
     2. Authorization approved? (Query billing system)
     3. Provider credentialed? (Query PostgreSQL credentialing table)
     4. 42 CFR Part 2 consent? (Query PostgreSQL SUD flag + consent date)
  → IF ANY CHECK FAILS:
     - Return error with specific reason
     - Frontend shows modal: "Cannot schedule - [reason]"
     - Offer resolution workflow
  → IF ALL CHECKS PASS:
     - Create appointment (PostgreSQL)
     - Log action (Audit Log)
     - Trigger background job (sync to EHR/scheduling system)
     - Return success
```

**3. One-Click Compliance Report (pre-computed, <3 seconds)**

```
User clicks "Generate Full Audit Report"
  → API: POST /compliance/generate-report
  → Background job (Celery) triggered:
     1. Query credentialing table (all providers, license expirations, NPDB dates)
     2. Query random sample of 20 charts (documentation completeness)
     3. Query SUD patients (42 CFR Part 2 consent coverage)
     4. Query services/sites (Scope of Project alignment)
     5. Query UDS data tables (preview for Feb 15 submission)
  → Generate PDF (Puppeteer/WeasyPrint)
  → Upload to S3
  → Notify user (WebSocket or polling)
  → Return download link
```

---

## Data Model

### Core Entities

**1. Provider**
```sql
CREATE TABLE providers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  name VARCHAR(255) NOT NULL,
  npi VARCHAR(10) UNIQUE NOT NULL,
  role VARCHAR(50) NOT NULL, -- psychologist, lcsw, lmft, lpc, psychiatrist
  specialties TEXT[], -- array of specialties
  languages TEXT[], -- array of languages
  license_number VARCHAR(50) NOT NULL,
  license_expiry DATE NOT NULL,
  license_state VARCHAR(2) NOT NULL,
  npdb_query_date DATE NOT NULL,
  board_approval_date DATE,
  credentialing_status VARCHAR(20) NOT NULL, -- current, expiring_soon, expired
  telehealth BOOLEAN DEFAULT FALSE,
  total_slots_per_week INT DEFAULT 0,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_providers_org ON providers(organization_id);
CREATE INDEX idx_providers_credentialing_status ON providers(credentialing_status);
CREATE INDEX idx_providers_license_expiry ON providers(license_expiry);
```

**2. Patient (PHI - encrypted at rest)**
```sql
CREATE TABLE patients (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  first_name_encrypted BYTEA NOT NULL,
  last_name_encrypted BYTEA NOT NULL,
  date_of_birth_encrypted BYTEA NOT NULL,
  insurance_primary_id UUID REFERENCES insurances(id),
  insurance_secondary_id UUID REFERENCES insurances(id),
  is_sud BOOLEAN DEFAULT FALSE,
  part2_consent_date DATE,
  part2_consent_expiry DATE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_patients_org ON patients(organization_id);
CREATE INDEX idx_patients_sud ON patients(is_sud) WHERE is_sud = TRUE;
```

**3. Appointment**
```sql
CREATE TABLE appointments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  patient_id UUID NOT NULL REFERENCES patients(id),
  provider_id UUID NOT NULL REFERENCES providers(id),
  appointment_date TIMESTAMPTZ NOT NULL,
  duration_minutes INT NOT NULL,
  service_type VARCHAR(100) NOT NULL,
  status VARCHAR(20) NOT NULL, -- scheduled, completed, no_show, cancelled
  cpt_codes TEXT[],
  diagnosis_codes TEXT[],
  authorization_id UUID REFERENCES authorizations(id),
  compliance_flags TEXT[], -- array of flags
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_appointments_org ON appointments(organization_id);
CREATE INDEX idx_appointments_date ON appointments(appointment_date);
CREATE INDEX idx_appointments_provider ON appointments(provider_id);
CREATE INDEX idx_appointments_patient ON appointments(patient_id);
```

**4. Authorization**
```sql
CREATE TABLE authorizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  patient_id UUID NOT NULL REFERENCES patients(id),
  insurer VARCHAR(255) NOT NULL,
  service_type VARCHAR(100) NOT NULL,
  cpt_code VARCHAR(10),
  approved_sessions INT NOT NULL,
  used_sessions INT DEFAULT 0,
  approval_date DATE NOT NULL,
  expiry_date DATE NOT NULL,
  status VARCHAR(20) NOT NULL, -- approved, pending, expired, denied
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_authorizations_org ON authorizations(organization_id);
CREATE INDEX idx_authorizations_patient ON authorizations(patient_id);
CREATE INDEX idx_authorizations_expiry ON authorizations(expiry_date);
CREATE INDEX idx_authorizations_status ON authorizations(status);
```

**5. Claim**
```sql
CREATE TABLE claims (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  patient_id UUID NOT NULL REFERENCES patients(id),
  provider_id UUID NOT NULL REFERENCES providers(id),
  service_date DATE NOT NULL,
  submission_date DATE,
  cpt_codes JSONB NOT NULL, -- array of {code, modifier, charge_amount}
  diagnosis_codes TEXT[],
  charge_amount NUMERIC(10,2) NOT NULL,
  paid_amount NUMERIC(10,2) DEFAULT 0,
  status VARCHAR(20) NOT NULL, -- draft, submitted, paid, denied, appealed
  denial_reason TEXT,
  denial_code VARCHAR(10),
  authorization_id UUID REFERENCES authorizations(id),
  risks TEXT[], -- array of risk flags
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_claims_org ON claims(organization_id);
CREATE INDEX idx_claims_service_date ON claims(service_date);
CREATE INDEX idx_claims_status ON claims(status);
CREATE INDEX idx_claims_denial ON claims(status) WHERE status = 'denied';
```

**6. Audit Log (immutable, append-only)**
```sql
CREATE TABLE audit_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID NOT NULL REFERENCES organizations(id),
  timestamp TIMESTAMPTZ DEFAULT NOW() NOT NULL,
  user_id UUID NOT NULL REFERENCES users(id),
  user_name VARCHAR(255) NOT NULL,
  action VARCHAR(100) NOT NULL,
  entity VARCHAR(50) NOT NULL,
  entity_id UUID,
  details JSONB,
  ip_address INET,
  user_agent TEXT,
  session_id UUID,
  signature VARCHAR(255) NOT NULL -- cryptographic hash
);

CREATE INDEX idx_audit_log_org ON audit_log(organization_id);
CREATE INDEX idx_audit_log_timestamp ON audit_log(timestamp DESC);
CREATE INDEX idx_audit_log_user ON audit_log(user_id);
CREATE INDEX idx_audit_log_entity ON audit_log(entity, entity_id);

-- Prevent updates/deletes (immutable)
CREATE RULE audit_log_no_update AS ON UPDATE TO audit_log DO INSTEAD NOTHING;
CREATE RULE audit_log_no_delete AS ON DELETE TO audit_log DO INSTEAD NOTHING;
```

---

## API Specifications

### Authentication

**POST /auth/login**
```json
Request:
{
  "email": "director@example-fqhc.org",
  "password": "hashed_password",
  "mfa_code": "123456" // optional if MFA enabled
}

Response (200 OK):
{
  "access_token": "eyJhbGciOiJIUzI1...",
  "refresh_token": "eyJhbGciOiJIUzI1...",
  "expires_in": 3600,
  "user": {
    "id": "uuid",
    "name": "Dr. Jane Smith",
    "email": "director@example-fqhc.org",
    "role": "director",
    "organization_id": "uuid"
  }
}
```

### Dashboard

**GET /dashboard/morning-brief**
```json
Response (200 OK):
{
  "timestamp": "2026-03-27T08:00:00Z",
  "summary": "3 urgent items, 12 decisions needed, 87% capacity this week",
  "crisis_capacity": {
    "status": "stable",
    "available_today": 2,
    "overnight_calls": 0
  },
  "compliance_alerts": [
    {
      "type": "credentialing_expiring",
      "severity": "high",
      "message": "Dr. Alex Johnson license expires in 28 days",
      "action_required": "Initiate re-credentialing"
    }
  ],
  "revenue_cycle": {
    "denials_yesterday": 3,
    "total_amount": 621.44,
    "aging_over_90_days": 12450.00
  },
  "capacity": {
    "available_slots_this_week": 67,
    "utilization_rate": 0.87
  }
}
```

**GET /dashboard/capacity**
```json
Response (200 OK):
{
  "providers": [
    {
      "id": "uuid",
      "name": "Dr. Sarah Chen",
      "role": "psychologist",
      "availability": {
        "mon": 6,
        "tue": 4,
        "wed": 8,
        "thu": 3,
        "fri": 2
      },
      "utilization_rate": 0.92,
      "credentialing_status": "current"
    }
  ],
  "waitlist": {
    "total": 23,
    "by_urgency": {
      "crisis": 2,
      "high": 7,
      "routine": 14
    },
    "by_insurance": {
      "medicaid": 15,
      "medicare": 4,
      "commercial": 3,
      "uninsured": 1
    }
  }
}
```

### Compliance

**GET /compliance/audit-readiness**
```json
Response (200 OK):
{
  "score": 93.5,
  "grade": "A-",
  "breakdown": {
    "credentialing": {
      "weight": 30,
      "score": 29,
      "details": "29/30 providers current (96.7%)"
    },
    "authorization": {
      "weight": 25,
      "score": 22.5,
      "details": "90/100 visits authorized (90%)"
    },
    "documentation": {
      "weight": 20,
      "score": 17,
      "details": "85/100 charts complete (85%)"
    },
    "scope_of_project": {
      "weight": 15,
      "score": 15,
      "details": "100% aligned"
    },
    "part2_consent": {
      "weight": 10,
      "score": 10,
      "details": "100% consents on file"
    }
  },
  "alerts": [
    {
      "category": "credentialing",
      "severity": "high",
      "message": "1 provider with expired license",
      "action": "Suspend from schedule immediately"
    }
  ]
}
```

**POST /compliance/generate-report**
```json
Request:
{
  "report_type": "full_audit_evidence",
  "include_sections": [
    "credentialing",
    "documentation",
    "part2",
    "scope",
    "uds"
  ],
  "patient_sample_size": 20
}

Response (202 Accepted):
{
  "job_id": "uuid",
  "status": "processing",
  "estimated_completion": "2026-03-27T08:03:00Z"
}

// Poll: GET /compliance/reports/:job_id
Response (200 OK):
{
  "job_id": "uuid",
  "status": "completed",
  "download_url": "https://s3.amazonaws.com/reports/audit_2026_03_27_001.pdf",
  "expires_at": "2026-03-28T08:00:00Z"
}
```

---

## Integration Architecture

### EHR Integration (HL7 FHIR)

**Epic, NextGen, eClinicalWorks** all support FHIR R4 (to varying degrees).

**Data to Fetch:**
- Patients (demographics, insurance, diagnoses)
- Practitioners (providers, credentials, specialties)
- Appointments (scheduled, completed, no-shows)
- Observations (PHQ-9, GAD-7 scores)

**Implementation:**
```python
import httpx
from fhir.resources.patient import Patient
from fhir.resources.appointment import Appointment

class FHIRClient:
    def __init__(self, base_url: str, auth_token: str):
        self.base_url = base_url
        self.headers = {
            "Authorization": f"Bearer {auth_token}",
            "Accept": "application/fhir+json"
        }

    async def get_patients(self, organization_id: str):
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{self.base_url}/Patient",
                params={"organization": organization_id},
                headers=self.headers
            )
            response.raise_for_status()
            bundle = response.json()
            return [Patient(**entry["resource"]) for entry in bundle.get("entry", [])]

    async def get_appointments(self, date_range: tuple):
        start, end = date_range
        async with httpx.AsyncClient() as client:
            response = await client.get(
                f"{self.base_url}/Appointment",
                params={
                    "date": f"ge{start.isoformat()}&date=le{end.isoformat()}"
                },
                headers=self.headers
            )
            response.raise_for_status()
            bundle = response.json()
            return [Appointment(**entry["resource"]) for entry in bundle.get("entry", [])]
```

### Billing System Integration (X12 EDI)

**athenahealth, Kareo, Tebra** use X12 EDI transactions.

**Data to Fetch:**
- 837 (Claim submission)
- 835 (Payment/remittance advice)
- 270/271 (Eligibility inquiry/response)
- 278 (Prior authorization request/response)

**Implementation:**
```python
from pydantic import BaseModel
from typing import List

class Claim837(BaseModel):
    claim_id: str
    patient_id: str
    provider_id: str
    service_date: str
    cpt_codes: List[str]
    diagnosis_codes: List[str]
    charge_amount: float

class Denial835(BaseModel):
    claim_id: str
    denial_code: str
    denial_reason: str
    denied_amount: float

class BillingSystemClient:
    async def get_denials_since(self, date: str):
        # Query 835 transactions with denial codes
        denials = await self.query_835_denials(date)
        return [Denial835(**d) for d in denials]

    async def get_authorization_status(self, authorization_id: str):
        # Query 278 responses
        response = await self.query_278(authorization_id)
        return {
            "status": response.get("status"),
            "approved_sessions": response.get("approved_sessions"),
            "expiry_date": response.get("expiry_date")
        }
```

### Sync Strategy

**Challenge:** 4-6 systems, each with different API capabilities, sync frequencies.

**Solution: Tiered Sync**
1. **Real-time (WebSocket/Webhook):** Crisis alerts, urgent compliance flags
2. **High-frequency (Every 5 minutes):** Appointment availability, authorization status
3. **Medium-frequency (Every hour):** Claims, denials, credentialing status
4. **Low-frequency (Daily):** UDS data, financial reports

**Implementation:**
```python
from celery import Celery
from celery.schedules import crontab

celery_app = Celery("dashboard")

@celery_app.task
def sync_appointments():
    """Sync appointments every 5 minutes"""
    fhir_client = FHIRClient(...)
    appointments = await fhir_client.get_appointments(date_range=(today, today + 7))
    # Update PostgreSQL
    for appointment in appointments:
        upsert_appointment(appointment)

@celery_app.task
def sync_denials():
    """Sync denials every hour"""
    billing_client = BillingSystemClient(...)
    denials = await billing_client.get_denials_since(yesterday)
    # Update PostgreSQL
    for denial in denials:
        upsert_denial(denial)

# Schedule tasks
celery_app.conf.beat_schedule = {
    "sync-appointments-every-5-minutes": {
        "task": "sync_appointments",
        "schedule": 300.0,  # 5 minutes
    },
    "sync-denials-every-hour": {
        "task": "sync_denials",
        "schedule": 3600.0,  # 1 hour
    },
}
```

### Failure Handling

**When integration fails:**
1. **Immediate:** Log error, retry with exponential backoff (1s, 2s, 4s, 8s, 16s)
2. **After 5 retries:** Mark system as "degraded" in dashboard
3. **User notification:** "Billing system sync failed. Authorization status may be stale."
4. **Graceful degradation:** Show cached data with staleness warning
5. **Manual fallback:** Allow manual data entry (flagged for verification when sync resumes)

---

## Security Architecture

### HIPAA Compliance

**Encryption:**
- At rest: AES-256 (PostgreSQL transparent data encryption)
- In transit: TLS 1.3 (enforce minimum version)

**Access Control:**
- Role-based permissions (director, admin, billing_staff, clinical_staff)
- Audit log every PHI access
- Automatic session timeout (15 minutes idle)

**Audit Trail:**
- Immutable log (append-only, no updates/deletes)
- Cryptographic signature (SHA-256 hash of entry)
- 7-year retention (HIPAA requirement)

### 42 CFR Part 2 Compliance

**SUD Patient Records:**
- Flag `is_sud` on patient record
- Segregate SUD data (separate query paths)
- Consent enforcement (cannot schedule without valid consent)
- Disclosure logging (who accessed, when, why)

**Implementation:**
```python
def can_access_sud_patient(user: User, patient: Patient) -> bool:
    if not patient.is_sud:
        return True  # Non-SUD patients: standard HIPAA

    # SUD patient: Check 42 CFR Part 2 consent
    consent = get_part2_consent(patient.id)
    if not consent:
        log_consent_violation(user, patient, "no_consent")
        return False

    if consent.expiry_date < date.today():
        log_consent_violation(user, patient, "consent_expired")
        return False

    # Log disclosure
    log_part2_disclosure(user, patient, "accessed_for_scheduling")
    return True
```

---

## Performance Requirements

**Load Time:**
- Initial page load: <2 seconds (3G connection)
- Navigation: <300ms
- Search/filter: <500ms
- Report generation: <3 seconds

**Scalability:**
- 100 concurrent users per FQHC
- 1,000+ providers per organization
- 10M+ audit log entries
- 50,000+ patients

**Caching Strategy:**
- Redis: Session data (15-minute TTL), dashboard aggregations (5-minute TTL)
- Browser: Static assets (1-year TTL), API responses (5-minute TTL)
- Service worker: Offline capability (cache critical views)

---

## Deployment Architecture

**Infrastructure:**
- **Cloud Provider:** AWS (HIPAA-eligible services)
- **Compute:** ECS Fargate (containerized FastAPI app)
- **Database:** RDS PostgreSQL (Multi-AZ, automated backups)
- **Cache:** ElastiCache Redis (replication enabled)
- **Search:** Elasticsearch Service (audit log queries)
- **Storage:** S3 (reports, encrypted at rest)
- **CDN:** CloudFront (static assets, certificate management)

**Environments:**
- **Production:** Multi-AZ, auto-scaling, 99.9% SLA
- **Staging:** Single-AZ, smaller instances
- **Development:** Local Docker Compose

---

## Conclusion

This architecture is developer-ready. Hand this to a team and they can build it.

**Key Decisions:**
- PWA for offline capability and device flexibility
- React + FastAPI for modern, maintainable stack
- PostgreSQL for ACID compliance (audit trails)
- Tiered sync strategy (real-time, high-freq, low-freq)
- HIPAA + 42 CFR Part 2 compliance embedded throughout

**Next Steps:**
1. Build clickable prototype (Figma → React)
2. Integrate with Epic FHIR sandbox (validate API)
3. Beta test with pilot FQHC (90 days)
4. Iterate based on real-world usage

---

**Serena Blackwood**
Architect
PM003: Deliverables Phase
2026-03-27
