# Backend implementation

Implement your LMS API in this directory.

## Requirements

- NestJS (preferred) or Express + TypeScript strict mode
- PostgreSQL with Prisma or TypeORM
- JWT auth + RBAC (admin, student for Part A; add instructor for Part B)
- Base path: `/api` (e.g. server on port `3001`)

## Suggested structure

```
backend/
├── src/
│   ├── modules/
│   │   ├── auth/
│   │   ├── students/
│   │   ├── courses/
│   │   ├── enrollments/
│   │   ├── progress/
│   │   └── reports/
│   ├── common/
│   └── main.ts
├── prisma/          # or typeorm migrations
├── .env.example
├── package.json
└── README.md        # ← expand with your setup steps
```

## Environment variables

Copy `.env.example` to `.env` and configure:

| Variable | Description |
|----------|-------------|
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | Access token signing secret |
| `JWT_REFRESH_SECRET` | Refresh token secret |
| `JWT_ACCESS_EXPIRES` | e.g. `15m` |
| `JWT_REFRESH_EXPIRES` | e.g. `7d` |
| `PORT` | Default `3001` |
| `CORS_ORIGIN` | `http://localhost:3000` |

## Seed data

Provide a seed script that creates at minimum:

- Admin: `admin@aaft.com` / `Admin@123`
- Student: `student@aaft.com` / `Student@123`
- Sample courses matching the frontend demo (see `../src/mocks/seed.ts`)

## Integration checklist

- [ ] All routes in `../docs/API-CONTRACT.md` implemented
- [ ] CORS enabled for the Next.js dev server
- [ ] Error responses use `{ "message": "..." }`
- [ ] Video progress: complete at ≥ 90%
- [ ] Swagger at `/api/docs` (recommended)
- [ ] Frontend runs with `NEXT_PUBLIC_API_URL=http://localhost:3001/api`

## Running with the frontend

```bash
# Terminal 1 — your API
cd backend
npm install
npm run start:dev

# Terminal 2 — reference UI (repo root)
cd ..
cp .env.example .env.local
# Edit .env.local: NEXT_PUBLIC_API_URL=http://localhost:3001/api
npm run dev
```

See [../ASSIGNMENT.md](../ASSIGNMENT.md) for the full specification and evaluation rubric.
