# 🔐 AIMS — Agent Identity Management System

> **AI Agent Identity & Governance Platform**

AIMS is an end-to-end **AI Agent Identity Management and Governance platform** designed to securely manage AI-agent identities, credentials, permissions, and lifecycle.

The platform provides **scoped JWT authentication, least-privilege authorization, credential rotation, inactivity monitoring, risk scoring, automatic revocation, and comprehensive audit logging** through a modern web-based governance dashboard.

## 🚀 What AIMS Does

AIMS answers five critical questions for every AI agent:

**👤 Who is the agent?**
Unique machine identity with ownership, purpose, and lifecycle status.

**🔑 What can the agent access?**
Scoped permissions such as `read`, `write`, and `admin`.

**🛡️ Is the agent authorized?**
Real-time JWT validation, credential verification, agent-status checks, and scope enforcement.

**⚠️ Is the agent still safe and active?**
Stale-agent detection, explainable risk scoring, and automated revocation.

**📋 What did the agent do?**
Comprehensive audit logs for access attempts, credential operations, and lifecycle events.

## ✨ Key Features

* 🆔 **Agent Identity Provisioning** — Generates unique `AID-XXXXXXXX` identities.
* 🔐 **JWT Authentication** — Cryptographically signed scoped credentials.
* 🛡️ **Least-Privilege Authorization** — `read`, `write`, and `admin` scope hierarchy.
* 🔄 **Credential Rotation** — Immediately invalidates old credentials and issues new ones.
* 🚫 **Credential Revocation** — Revokes credentials during lifecycle changes or governance actions.
* 💤 **Stale Detection** — Identifies agents inactive for 30+ days.
* ⚠️ **Risk Scoring** — Explainable 0–100 risk assessment.
* 🤖 **Automated Governance** — Automatically revokes agents inactive for 90+ days.
* 📋 **Audit Logging** — Records successful and denied access attempts and system operations.
* 📊 **Governance Dashboard** — Real-time KPIs, risk rankings, stale alerts, and audit activity.
* 🧪 **Automated Testing** — 35+ unit and integration tests using Pytest.

## 🏗️ Architecture

```text
┌─────────────────────────────────────────────────────┐
│              React + TypeScript + MUI               │
│  Dashboard │ Agents │ Reviews │ Audit Logs │ Admin  │
└──────────────────────┬──────────────────────────────┘
                       │ REST / HTTP
                       ▼
┌─────────────────────────────────────────────────────┐
│                    FastAPI Backend                   │
│                                                     │
│  Agent Routes │ Protected APIs │ Governance Routes  │
└──────────────────────┬──────────────────────────────┘
                       │
                       ▼
              ┌──────────────────┐
              │  Scope Checker   │
              │ JWT + Hash +     │
              │ Status + Scope   │
              └────────┬─────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
    Token Service  Review Service  Revoke Service
          │            │            │
          └────────────┼────────────┘
                       ▼
              SQLAlchemy ORM
                       │
                       ▼
           SQLite / PostgreSQL
          ┌────────┬────────┬─────────┐
          ▼        ▼        ▼
       Agents Credentials Audit Logs
```

The backend also includes a **daily background governance scheduler** that performs stale detection, risk evaluation, and automatic revocation.

## 🔑 Security Model

AIMS uses **scoped JWT credentials**.

Example:

```text
FinanceBot
    │
    └── Scope: read

HRBot
    │
    ├── Scope: read
    └── Scope: write

AuditBot
    │
    └── Scope: admin
```

Every protected request passes through the authorization flow:

```text
Request
   ↓
Bearer JWT
   ↓
Validate JWT
   ↓
Identify Agent
   ↓
Verify Credential
   ↓
Check Agent Status
   ↓
Check Required Scope
   ↓
Allow / Deny
   ↓
Audit Log
```

AIMS stores only the **SHA-256 hash of issued tokens**, rather than storing raw bearer tokens in the database.

## 📊 Risk & Governance

AIMS uses an explainable **0–100 risk scoring model** based on:

* Scope privilege level
* Agent lifecycle status
* Inactivity duration
* Unused credentials

Agents inactive for **30+ days** are flagged as stale, while agents inactive for **90+ days** can be automatically revoked. Both thresholds are configurable.

## 🛠️ Tech Stack

### Frontend

`React` `TypeScript` `Vite` `Material UI` `Axios` `Recharts`

### Backend

`Python` `FastAPI` `Uvicorn` `Pydantic` `SQLAlchemy`

### Security

`JWT` `HS256` `SHA-256` `Scope-Based Authorization`

### Database

`SQLite` `PostgreSQL`

### Testing & DevOps

`Pytest` `Docker` `Git` `GitHub`

### Automation

`APScheduler`

## 🧪 Testing

The project includes **35+ automated unit and integration tests** covering:

* Agent registration
* Scope enforcement
* Invalid token rejection
* Credential rotation
* Stale-agent detection
* Automatic revocation
* Agent suspension/reactivation
* Decommissioning
* Audit events

Run the test suite:

```bash
pytest tests/ -v
```

The project also includes an end-to-end success-criteria verification script.

## 🎯 Success Criteria

Example authorization scenarios:

```text
FinanceBot → READ   → ✅ HTTP 200
FinanceBot → WRITE  → ❌ HTTP 403

HRBot      → WRITE  → ✅ HTTP 200

AuditBot   → ADMIN  → ✅ HTTP 200
```

These scenarios demonstrate the project's **least-privilege and scope-based authorization model**.

## 📁 Project Structure

```text
agent-identity-management/
│
├── app/
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── models/
│   ├── schemas/
│   ├── routes/
│   ├── services/
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

The repository structure separates the **API layer, business services, authorization middleware, frontend, database models, and automated tests**.

## 🌐 Deployment

AIMS includes a Render deployment configuration for:

```text
React/Vite Frontend
        +
FastAPI Backend
        +
PostgreSQL Database
```

The included `render.yaml` can configure the backend web service and frontend static site.

---

### 💡 Project Goal

> **AIMS aims to provide a secure, auditable, and automated identity governance layer for AI agents, ensuring that every agent has a verified identity, only the permissions it needs, controlled credentials, monitored lifecycle, and traceable activity.**
