# 🌱 Green Skills Platform

A full-stack sustainability skills assessment platform with **SSO login**, **personalised feedback dashboards**, and **anonymised admin analytics** — all offline-capable with no external API keys.

---

## Architecture

```
green-skills-platform/
├── jac/                  ← Jaseci JAC domain logic
│   ├── schemas.jac       ← Node / edge definitions
│   ├── llm_tasks.jac     ← Scoring + Ollama LLM calls
│   ├── employee_flow.jac ← Employee walkers
│   ├── admin_flow.jac    ← Admin walkers (anonymised only)
│   └── main.jac          ← Entry point + data seeding
├── app/                  ← React frontend
│   └── src/pages/
│       ├── Landing.js
│       ├── Login.js           ← SSO / demo login
│       ├── Profile.js         ← Role/dept profile
│       ├── Questionnaire.js   ← Dynamic assessment
│       ├── EmployeeDashboard.js ← Personal feedback
│       ├── AdminDashboard.js  ← Aggregated analytics
│       └── AdminQuestionnaires.js ← Manage assessments
├── server/               ← Node.js/Express API
│   ├── db.js             ← PostgreSQL schema + pool
│   ├── index.js          ← Server entry
│   ├── middleware/auth.js ← JWT + SSO middleware
│   ├── routes/           ← auth / employee / admin / seed
│   └── services/scoring.js ← Scoring + LLM service
├── config/jaseci.yaml    ← Jaseci runtime config
├── deployment/
│   ├── docker-compose.yml
│   └── nginx.conf
└── scripts/
    ├── dev.sh            ← Local development
    └── deploy.sh         ← Production deployment
```

---

## Quick Start

### Prerequisites
- Docker + Docker Compose
- Node.js 18+ (for local dev)
- (Optional) Ollama for local AI feedback

### 1. Clone & configure
```bash
git clone <repo>
cd green-skills-platform
cp .env.example .env
# Edit .env — set DB_PASS and JWT_SECRET at minimum
```

### 2. Development mode
```bash
chmod +x scripts/dev.sh
./scripts/dev.sh
```

### 3. Production deployment
```bash
chmod +x scripts/deploy.sh
./scripts/deploy.sh
```

Access:
- **Frontend**: http://localhost:3000
- **API**: http://localhost:4000
- **Via Nginx**: http://localhost:80

---

## SSO Integration

The platform supports **Azure AD SSO** out of the box. To enable:

1. Register an app in Azure Active Directory
2. Set in `.env`:
   ```
   AZURE_CLIENT_ID=your-client-id
   AZURE_CLIENT_SECRET=your-secret
   AZURE_TENANT_ID=your-tenant-id
   AZURE_REDIRECT_URI=https://yourdomain.com/api/auth/callback
   ```
3. The `/api/auth/sso` endpoint accepts `{email, name, sso_token}` from your SSO provider

For **demo/dev**, use the email login — it creates a user in Postgres automatically.

---

## Admin vs Employee Data Access

| Data Type | Employee | Admin |
|-----------|----------|-------|
| Own answers | ✓ Full | ✗ Never |
| Own scores | ✓ Full | ✗ Never |
| Own feedback | ✓ Full | ✗ Never |
| Dept avg scores | ✗ | ✓ Aggregated |
| Skill gap heatmap | ✗ | ✓ Aggregated |
| Role breakdowns | ✗ | ✓ Aggregated |
| Individual names | N/A | ✗ Never |

Admin analytics require **minimum 1 response** to appear. The system computes aggregations on demand (Refresh Analytics button) or via `POST /api/admin/compute-aggregations`.

---

## LLM Configuration (Ollama)

The platform uses **Ollama** for AI-generated feedback — fully local, no API key needed.

```bash
# Install Ollama
curl https://ollama.ai/install.sh | sh

# Pull the model
ollama pull mistral

# Start (runs on localhost:11434)
ollama serve
```

If Ollama is unavailable, the system **automatically falls back** to rule-based scoring and feedback generation.

---

## Jaseci JAC Walkers

The core logic lives in `.jac` files and can be run via Jaseci server:

```bash
# Install jaseci
pip install jaseci jaseci_serv

# Start Jaseci server
jsserv -c config/jaseci.yaml

# Seed questionnaires
jsctl walker run seed_questionnaires

# Employee submit
jsctl walker run submit_response -ctx '{"user_id":"...","questionnaire_id":"...","answers":{}}'

# Admin aggregation
jsctl walker run compute_aggregations
```

---

## Default Departments & Questions

| Department | Assessment Focus |
|------------|-----------------|
| Operations | Carbon targets, energy tracking, ESG reporting |
| Finance | Green finance, TCFD, carbon pricing |
| Engineering | Energy-efficient design, LCA, scope emissions |
| HR | Green talent acquisition, sustainability training |
| Supply Chain | Supplier risk, circular economy, scope 3 |
| General | Baseline sustainability awareness (fallback) |

---

## API Reference

### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/sso` | SSO login |
| POST | `/api/auth/demo-login` | Demo login |
| GET | `/api/auth/me` | Current user |

### Employee
| Method | Endpoint | Description |
|--------|----------|-------------|
| PUT | `/api/employee/profile` | Save role/dept |
| GET | `/api/employee/questionnaire` | Get tailored questionnaire |
| POST | `/api/employee/submit` | Submit answers |
| GET | `/api/employee/dashboard` | Personal dashboard |

### Admin (requires `is_admin=true`)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/admin/dashboard` | Aggregated analytics |
| POST | `/api/admin/compute-aggregations` | Refresh aggregations |
| GET | `/api/admin/questionnaires` | List all |
| POST | `/api/admin/questionnaire` | Create questionnaire |
| PUT | `/api/admin/questionnaire/:id` | Update questionnaire |
| GET | `/api/admin/heatmap` | Dept × category heatmap |

---

## Security Notes

- All JWT tokens expire in **8 hours**
- Admin endpoints require `is_admin: true` in the JWT
- Admin analytics are **anonymised** — no user IDs, names, or raw answers exposed
- Rate limiting: 300 requests per 15 minutes per IP
- CORS restricted to `FRONTEND_URL` in production

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Domain Logic | Jaseci JAC |
| API | Node.js + Express |
| Database | PostgreSQL 15 |
| Frontend | React 18 + Recharts |
| Auth | JWT + Azure AD SSO |
| LLM | Ollama (local Mistral) |
| Deployment | Docker + Nginx |
