# 🔐 AIMS — Agent Identity Management System

### 🛡️ AI Agent Identity • Security • Authorization • Governance

> **AIMS is an end-to-end identity and governance platform built to securely manage AI agents, their credentials, permissions, lifecycle, risk, and activities.**

As AI agents become part of modern software systems, simply creating an agent is not enough.

Every agent needs to answer:

```text
┌─────────────────────────────────────────────┐
│ 👤 WHO AM I?                                │
│ 🔑 WHAT AM I ALLOWED TO ACCESS?             │
│ 🛡️ IS MY CREDENTIAL VALID?                 │
│ ⚠️ AM I STILL TRUSTED?                      │
│ 📋 WHAT HAVE I DONE?                        │
└─────────────────────────────────────────────┘
```

**AIMS answers all five.**

It provides unique agent identities, scoped JWT credentials, least-privilege authorization, credential rotation, lifecycle management, risk monitoring, automated revocation, and complete auditability.

---

# 🚀 Why AIMS?

Imagine an organization running multiple AI agents:

```text
                🤖 AI AGENTS
                    │
       ┌────────────┼────────────┐
       ▼            ▼            ▼
   FinanceBot     HRBot      AuditBot
       │            │            │
       ▼            ▼            ▼
      READ       READ/WRITE     ADMIN
```

Without proper identity governance:

* ❌ Agents may receive excessive permissions
* ❌ Credentials may remain active indefinitely
* ❌ Inactive agents may remain trusted
* ❌ Compromised credentials become difficult to control
* ❌ Security teams lack visibility into agent activity
* ❌ There may be no clear audit trail

AIMS introduces a controlled identity layer between **AI agents and protected resources**.

---

# 🧠 How AIMS Works

The complete lifecycle looks like this:

```text
                    👤 ADMIN
                      │
                      ▼
              ┌───────────────┐
              │ Register Agent│
              └───────┬───────┘
                      │
                      ▼
              🆔 Agent Identity
                      │
                      ▼
              🔐 Generate JWT
                      │
                      ▼
             🎯 Assign Scopes
             READ / WRITE / ADMIN
                      │
                      ▼
              💾 Store Credential
              SHA-256 Token Hash
                      │
                      ▼
              🚀 Agent Uses API
                      │
                      ▼
             🛡️ Scope Checker
                      │
             ┌────────┴────────┐
             │                 │
           VALID             INVALID
             │                 │
             ▼                 ▼
        ✅ ALLOW             ❌ DENY
             │
             ▼
        📋 Audit Log
             │
             ▼
       🕒 Activity Tracking
             │
             ▼
       ⚠️ Risk Evaluation
             │
             ▼
       💤 Stale Detection
             │
             ▼
       🚫 Auto Revocation
```

This lifecycle combines identity provisioning, authorization, credential management, monitoring, and governance.

---

# 🏗️ System Architecture

```text
┌─────────────────────────────────────────────────────────────┐
│                    🌐 FRONTEND                             │
│                                                             │
│        React + TypeScript + Material UI + Vite             │
│                                                             │
│  Dashboard │ Agents │ Register │ Reviews │ Audit Logs      │
└─────────────────────────────┬───────────────────────────────┘
                              │
                         HTTP / REST
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    ⚡ FASTAPI BACKEND                       │
│                                                             │
│  Agent Routes │ Protected APIs │ Governance Routes         │
│                                                             │
│                    ↓                                        │
│              🛡️ Scope Checker                              │
│                                                             │
│     JWT → Hash → Agent → Status → Scope → Audit            │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    ⚙️ CORE SERVICES                         │
│                                                             │
│   🔑 Token Service    📊 Review Service    🚫 Revoke       │
│                                                             │
│   JWT Issue/Rotate    Risk & Stale        Auto-Revoke      │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    💾 DATABASE                              │
│                                                             │
│       AGENTS        CREDENTIALS        AUDIT LOGS           │
│                                                             │
│                SQLite / PostgreSQL                          │
└─────────────────────────────────────────────────────────────┘
```

The project separates the frontend, FastAPI API layer, authorization middleware, core governance services, background scheduler, and database models.

---

# 🔑 1. Agent Identity Provisioning

Every AI agent receives a unique machine identity.

Example:

```text
Agent Name  : FinanceBot
Agent ID    : AID-7F31A92B
Purpose     : Financial report generation
Team        : Finance
Status      : ACTIVE
```

AIMS generates IDs in the form:

```text
AID-XXXXXXXX
```

The identity card also stores ownership, purpose, scopes, lifecycle status, and activity information.

---

# 🔐 2. Scoped JWT Credentials

After registration, AIMS generates a cryptographically signed JWT.

A token contains claims such as:

```json
{
  "sub": "AID-7F31A92B",
  "agent_name": "FinanceBot",
  "scopes": ["read"],
  "jti": "unique-token-id",
  "iat": "issued-at",
  "exp": "expiration"
}
```

### Available scopes

```text
👁️ READ
   View permitted resources

✏️ WRITE
   Modify permitted resources

👑 ADMIN
   Administrative access
```

The `admin` scope acts as a privilege superset for protected resources.

---

# 🛡️ 3. Least-Privilege Authorization

AIMS follows the principle of:

> **Give every agent only the permissions it needs.**

Example:

```text
FinanceBot
   └── READ

HRBot
   ├── READ
   └── WRITE

AuditBot
   └── ADMIN
```

Therefore:

```text
FinanceBot → GET /reports
             ✅ 200 OK

FinanceBot → POST /reports
             ❌ 403 Forbidden

HRBot → POST /reports
        ✅ 200 OK

AuditBot → GET /admin
           ✅ 200 OK
```

The project's success-criteria tests explicitly verify these access patterns.

---

# ⚙️ 4. What Happens During an API Request?

This is the heart of AIMS.

Suppose:

```http
POST /reports
Authorization: Bearer <JWT>
```

The request passes through:

```text
          HTTP REQUEST
               │
               ▼
        🔐 Extract JWT
               │
               ▼
       Verify JWT Signature
               │
               ▼
       Extract Agent Identity
               │
               ▼
       Hash Presented Token
               │
               ▼
     Match ACTIVE Credential
               │
               ▼
       Check Agent Status
               │
               ▼
        Check Required Scope
               │
        ┌──────┴──────┐
        │             │
      VALID         INVALID
        │             │
        ▼             ▼
    ✅ ALLOW       ❌ DENY
        │             │
        └──────┬──────┘
               ▼
          📋 Audit Log
               │
               ▼
       Update Last API Call
```

The `ScopeChecker` performs token, credential, status, and scope validation before allowing protected operations.

---

# 🔒 5. Secure Token Storage

AIMS follows an important security rule:

> **Never store raw bearer credentials unnecessarily.**

Instead of storing:

```text
eyJhbGciOiJIUzI1NiIs...
```

AIMS stores:

```text
SHA-256(JWT)
```

During authentication:

```text
Presented JWT
      │
      ▼
SHA-256
      │
      ▼
Compare with stored hash
      │
      ▼
Match → Credential Valid
```

This means a database breach does not directly expose usable raw bearer credentials.

---

# 🔄 6. Credential Rotation

Credentials should not live forever.

When rotation is requested:

```text
          OLD JWT
             │
             ▼
        🔄 ROTATE
             │
             ▼
      Old → ROTATED
             │
             ▼
       Generate NEW JWT
             │
             ▼
       New → ACTIVE
```

The previous credential is immediately invalidated and a fresh credential is issued.

---

# ♻️ 7. Agent Lifecycle Management

AIMS manages the complete agent lifecycle:

```text
        ┌───────────┐
        │  REGISTER │
        └─────┬─────┘
              ▼
        ┌───────────┐
        │   ACTIVE  │
        └─────┬─────┘
              │
       ┌──────┼───────────────┐
       │      │               │
       ▼      ▼               ▼
   SUSPEND   STALE      DECOMMISSION
       │      │               │
       ▼      ▼               ▼
  SUSPENDED  ⚠️            REVOKED
       │
   REACTIVATE
       │
       ▼
    ACTIVE
```

Supported lifecycle operations include:

* Suspend
* Reactivate
* Decommission
* Credential revocation
* Fresh credential issuance after reactivation

---

# 💤 8. Stale Agent Detection

What happens when an AI agent hasn't been used for a long time?

AIMS detects it.

```text
ACTIVE
  │
  │ 30+ days inactive
  ▼
STALE ⚠️
```

The default stale threshold is:

```text
30 DAYS
```

The threshold is configurable.

---

# 🚫 9. Automatic Revocation

If an agent remains inactive for too long:

```text
ACTIVE
   │
   ▼
30 DAYS
   │
   ▼
STALE
   │
   ▼
90 DAYS
   │
   ▼
🚫 REVOKED
```

When automatic revocation occurs:

```text
Agent
  │
  ├── Status → REVOKED
  │
  ├── Active Credentials → REVOKED
  │
  └── Action → AUDIT LOG
```

The default auto-revocation threshold is **90 days**.

---

# 📊 10. Explainable Risk Scoring

AIMS calculates a **0–100 risk score**.

The score considers:

```text
🔑 Permission Breadth
⚠️ Agent Status
🕒 Inactivity
🚫 Unused Credentials
```

Example scoring:

```text
ADMIN  → +40
WRITE  → +20
READ   → +5

STALE      → +30
SUSPENDED  → +10

Idle time  → additional risk
Never used → additional risk
```

The final score remains within:

```text
0 ─────────────────────── 100
LOW                       HIGH
```

The model is intentionally explainable so that governance decisions can be traced back to specific factors.

---

# 📋 11. Audit Logging

Every important action leaves a trail.

Examples:

```text
REGISTER
ACCESS_ATTEMPT
ROTATE
SUSPEND
REACTIVATE
DECOMMISSION
AUTO_REVOKE
```

Example:

```text
┌─────────────────────────────────────────┐
│ 🔎 AUDIT EVENT                          │
├─────────────────────────────────────────┤
│ Agent   : FinanceBot                    │
│ Action  : ACCESS_ATTEMPT                │
│ Result  : DENIED                        │
│ Reason  : Missing 'write' scope         │
│ Time    : 2026-08-26 10:30:12           │
└─────────────────────────────────────────┘
```

Both successful and denied access attempts are recorded.

---

# ⏰ 12. Automated Governance

AIMS doesn't depend entirely on an administrator manually checking agents.

A background scheduler runs governance tasks daily:

```text
             ⏰ DAILY JOB
                  │
                  ▼
          Detect stale agents
                  │
                  ▼
           Refresh risk scores
                  │
                  ▼
          Find long-idle agents
                  │
                  ▼
          Auto-revoke if needed
                  │
                  ▼
        Generate notifications
```

The project uses **APScheduler** for the in-process scheduled governance sweep.

---

# 📊 13. Governance Dashboard

The React dashboard provides a centralized view of the entire system.

### Dashboard

```text
┌───────────────────────────────────────────────┐
│                 AIMS DASHBOARD                │
├──────────┬──────────┬──────────┬──────────────┤
│ Agents   │ Active   │ Stale    │ High Risk    │
│   24     │   18     │    4     │      2       │
└──────────┴──────────┴──────────┴──────────────┘

       📊 Agent Status Distribution

       ⚠️ Top Risk Agents

       💤 Stale Agent Alerts

       📋 Live Audit Activity
```

The dashboard includes KPI cards, status distribution, risk rankings, stale alerts, and an auto-refreshing audit activity stream.

---

# 🧭 14. Agent Directory

Administrators can:

```text
🔎 Search agents
🔽 Filter agents
📄 Paginate results
🏷️ View status
⚠️ View risk
🔑 View scopes
```

This creates a centralized identity directory for AI agents.

---

# 👤 15. Agent Detail View

Each agent has a detailed profile:

```text
Agent Identity
      │
      ├── 🆔 Agent ID
      ├── 👤 Name
      ├── 🏢 Team
      ├── 🎯 Purpose
      ├── 🔑 Scopes
      ├── ⚠️ Risk Score
      ├── 📊 Status
      ├── 🔄 Credential History
      └── 📋 Audit History
```

Administrators can also perform lifecycle actions from the agent detail view.

---

# 📝 16. Quarterly Governance Reviews

AIMS provides governance review information covering:

```text
🔄 Overdue credential rotations
💤 Stale agents
⚠️ Risk rankings
```

This allows administrators to periodically review the security posture of AI agents.

---

# 🧪 17. Automated Testing

The project includes **35+ unit and integration tests**.

```text
tests/
│
├── test_registration.py
├── test_scope_enforcement.py
├── test_rotation.py
├── test_stale_and_revoke.py
└── test_lifecycle.py
```

The tests cover:

```text
✅ Registration
✅ Validation
✅ Scope enforcement
✅ Invalid tokens
✅ 401 / 403 behavior
✅ Credential rotation
✅ Stale detection
✅ Auto-revocation
✅ Suspension
✅ Reactivation
✅ Decommissioning
✅ Audit events
```

The test suite uses isolated in-memory SQLite databases.

---

# 🧰 Tech Stack

| Layer              | Technologies                         |
| ------------------ | ------------------------------------ |
| 🎨 Frontend        | React, TypeScript, Vite, Material UI |
| ⚡ Backend          | Python, FastAPI, Uvicorn             |
| 🗄️ ORM            | SQLAlchemy                           |
| 💾 Database        | SQLite, PostgreSQL                   |
| 🔐 Authentication  | JWT, HS256                           |
| 🛡️ Authorization  | Scope-Based Access Control           |
| 🔒 Security        | SHA-256                              |
| ⏰ Scheduler        | APScheduler                          |
| 🧪 Testing         | Pytest                               |
| 📡 API Client      | Axios                                |
| 📊 Visualization   | Recharts                             |
| 🐳 DevOps          | Docker                               |
| 🔧 Version Control | Git, GitHub                          |

---

# 📁 Project Structure

```text
agent-identity-management/
│
├── app/
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   │
│   ├── models/
│   │   ├── agent.py
│   │   ├── credential.py
│   │   └── audit.py
│   │
│   ├── schemas/
│   ├── routes/
│   ├── services/
│   │   ├── token_service.py
│   │   ├── review_service.py
│   │   └── revoke_service.py
│   │
│   └── middleware/
│       └── scope_checker.py
│
├── frontend/
│   └── src/
│       ├── api/
│       ├── components/
│       ├── context/
│       ├── pages/
│       └── types/
│
├── tests/
├── scripts/
├── render.yaml
├── requirements.txt
└── README.md
```

The architecture separates API routes, schemas, database models, governance services, authorization middleware, frontend components, and automated tests.

---

# ⚡ Quick Start

## Backend

### 1. Create virtual environment

```bash
python -m venv venv
```

### 2. Activate

**Windows**

```bash
venv\Scripts\activate
```

**Linux / macOS**

```bash
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
pip install -r requirements-dev.txt
```

### 4. Configure environment

```bash
cp .env.example .env
```

### 5. Start FastAPI

```bash
uvicorn app.main:app --reload --port 8000
```

Backend:

```text
http://localhost:8000
```

Swagger:

```text
http://localhost:8000/docs
```

Health check:

```text
http://localhost:8000/health
```

These are the documented project startup commands and endpoints.

---

# 🎨 Frontend Setup

```bash
cd frontend
npm install
npm run dev
```

Frontend:

```text
http://localhost:5173
```

---

# 🧪 Run Tests

```bash
pytest tests/ -v
```

For end-to-end success criteria:

```bash
python -m scripts.run_success_criteria_tests
```

Expected authorization behavior:

```text
FinanceBot → READ   → ✅ 200
FinanceBot → WRITE  → ❌ 403

HRBot      → WRITE  → ✅ 200

AuditBot   → ADMIN  → ✅ 200
```

---

# ☁️ Deployment

AIMS includes a `render.yaml` deployment blueprint.

Production architecture:

```text
             GitHub Repository
                    │
                    ▼
              Render Blueprint
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
    FastAPI Backend      React Frontend
          │
          ▼
   PostgreSQL Database
```

The documented deployment setup uses a Render Python web service for the API, a static site for the Vite frontend, and PostgreSQL for persistent production data.

---

# 🔐 Security Philosophy

AIMS is built around four principles:

```text
             🛡️ AIMS SECURITY MODEL

        ┌──────────────────────────┐
        │      IDENTIFY            │
        │  Every agent has an ID   │
        └────────────┬─────────────┘
                     ▼
        ┌──────────────────────────┐
        │      AUTHENTICATE        │
        │   Verify the credential  │
        └────────────┬─────────────┘
                     ▼
        ┌──────────────────────────┐
        │       AUTHORIZE          │
        │   Enforce least privilege│
        └────────────┬─────────────┘
                     ▼
        ┌──────────────────────────┐
        │        GOVERN             │
        │ Monitor • Audit • Revoke │
        └──────────────────────────┘
```

---

# 🎯 Project Goals

AIMS is designed to provide:

* **Identity** — Every AI agent has a unique machine identity.
* **Security** — Credentials are cryptographically signed and securely managed.
* **Least Privilege** — Agents receive only the permissions they need.
* **Lifecycle Control** — Agents and credentials can be suspended, rotated, reactivated, or revoked.
* **Governance** — Inactive and risky agents are automatically identified.
* **Auditability** — Security and lifecycle actions are traceable.
* **Visibility** — Administrators have a centralized governance dashboard.

---

# 🌟 What Makes AIMS Different?

AIMS doesn't stop at:

```text
Authentication
```

It goes further:

```text
IDENTITY
   ↓
AUTHENTICATION
   ↓
AUTHORIZATION
   ↓
CREDENTIAL MANAGEMENT
   ↓
LIFECYCLE MANAGEMENT
   ↓
RISK MONITORING
   ↓
AUTOMATED GOVERNANCE
   ↓
AUDITABILITY
```

> **AIMS treats AI agents as managed identities rather than simply treating them as API clients.**

---

# 🚀 Future Improvements

Potential extensions include:

* Distributed background workers for multi-instance deployments
* Advanced behavioral anomaly detection
* More granular permission models
* External identity-provider integrations
* Expanded compliance reporting
* Stronger production secret management
* More sophisticated risk analytics
* Real-time security alerts

---

# 👨‍💻 Project

**AIMS — Agent Identity Management System**

**PS-2.1 — AI Agent Identity Card & Governance Platform**

Built as an end-to-end full-stack AI-agent security and governance platform.

---

## ⭐ Core Flow

```text
          🤖 AI AGENT
               │
               ▼
        🆔 GET IDENTITY
               │
               ▼
        🔐 GET CREDENTIAL
               │
               ▼
       🎯 GET PERMISSIONS
               │
               ▼
          🚀 USE API
               │
               ▼
        🛡️ ACCESS CHECK
          ↙          ↘
       ALLOW         DENY
         │             │
         └──────┬──────┘
                ▼
           📋 AUDIT
                │
                ▼
          ⚠️ RISK CHECK
                │
                ▼
          💤 INACTIVITY
                │
                ▼
          🚫 REVOCATION
```

### **AIMS — Identity you can trust. Access you can control. Activity you can audit.**
