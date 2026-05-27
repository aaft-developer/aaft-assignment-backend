# AAFT LMS — Backend Developer Assignment

[![Node.js](https://img.shields.io/badge/Node.js-TypeScript-339933?style=flat-square&logo=node.js)]()
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-required-4169E1?style=flat-square&logo=postgresql)]()
[![NestJS](https://img.shields.io/badge/NestJS-preferred-E0234E?style=flat-square&logo=nestjs)]()

Build a **production-ready REST API** for AAFT’s Enterprise LMS. This repository includes a **complete reference frontend** (Next.js 15, Admin + Student portals). Your task is to implement the backend in the `backend/` folder and connect the UI to your API.

| Document | Purpose |
|----------|---------|
| **[ASSIGNMENT.md](./ASSIGNMENT.md)** | Full brief, rubric, Part A + Part B requirements |
| **[docs/API-CONTRACT.md](./docs/API-CONTRACT.md)** | Exact endpoints and JSON shapes the frontend expects |
| **[docs/FRONTEND.md](./docs/FRONTEND.md)** | Reference UI setup and architecture (read-only for candidates) |
| **[backend/README.md](./backend/README.md)** | Where to place your NestJS/Express service |

**Submit your GitHub repo link to:** [aaft@akashmbair](mailto:aaft@akashmbair)

---

## Quick start (reference frontend)

Study the mock API contract before building your backend:

```bash
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

| Role | Email | Password |
|------|-------|----------|
| Admin | `admin@aaft.com` | `Admin@123` |
| Student | `student@aaft.com` | `Student@123` |

Mock API routes live under `src/app/api/`. Types are in `src/types/index.ts`.

---

## Your backend (what you build)

1. Create the `backend/` directory (see [backend/README.md](./backend/README.md)).
2. Implement all endpoints in [docs/API-CONTRACT.md](./docs/API-CONTRACT.md) (**Part A**).
3. Extend with enterprise modules in [ASSIGNMENT.md](./ASSIGNMENT.md) (**Part B**).
4. Point the frontend at your server:

```env
# .env.local (repository root)
NEXT_PUBLIC_API_URL=http://localhost:3001/api
```

5. Enable CORS on your API for `http://localhost:3000`.
6. Record a short demo of Admin + Student flows against your API.

---

## Tech stack (required)

- Node.js + TypeScript (strict)
- **NestJS** (preferred) or Express
- **PostgreSQL** + Prisma or TypeORM
- JWT (access + refresh) + RBAC
- bcrypt password hashing

---

## Repository layout

```
.
├── ASSIGNMENT.md          # Full assignment spec
├── README.md              # This file
├── docs/
│   ├── API-CONTRACT.md    # Frontend integration contract
│   └── FRONTEND.md        # Reference UI documentation
├── backend/               # ← Your implementation goes here
└── src/                   # Reference frontend (provided)
    ├── app/api/           # Mock API (study, do not depend on in production)
    └── ...
```

---

## Rules

- **Do not** rebuild the frontend.
- **Do not** use AI coding assistants — see [ASSIGNMENT.md](./ASSIGNMENT.md).
- **Do** document setup, env vars, and architecture in `backend/README.md`.
- **Do** include Swagger or a Postman collection.

---

## Questions

Email **aaft@akashmbair** with subject: `Backend Assignment — <Your Name>`.

*AAFT Online — Confidential hiring material.*
