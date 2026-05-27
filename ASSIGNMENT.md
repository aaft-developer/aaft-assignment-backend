# AAFT Online — Backend Developer Technical Assignment

**Enterprise Learning Management System (LMS)**

| | |
|---|---|
| **Stack** | Node.js + TypeScript |
| **Database** | PostgreSQL (required) |
| **Auth** | JWT + RBAC |
| **ORM** | Prisma or TypeORM |
| **Reference frontend** | Included in this repository (Next.js 15) |
| **Submission** | Public GitHub repo link → **aaft@akashmbair** |

*Confidential — For hiring use only. Do not distribute.*

---

## 1. Assignment overview

You are required to design and implement a **production-ready backend API** for an Enterprise Learning Management System (LMS). AAFT has already built a **reference Admin + Student frontend** (Next.js, Redux Toolkit) that ships in this repository. Your job is to **replace the in-app mock API** with a real NestJS (or Express) service backed by PostgreSQL.

The frontend must work end-to-end against your API without UI changes. Treat the mock Route Handlers under `src/app/api/` and the TypeScript types in `src/types/` as the **contract you must satisfy**.

### What you deliver

| Deliverable | Requirement |
|-------------|-------------|
| Backend service | New `backend/` directory (or separate repo linked in README) |
| Database | Normalized PostgreSQL schema + versioned migrations |
| API | All **Part A** endpoints (frontend contract) fully functional |
| Docs | README with setup, env vars, architecture; Swagger or Postman collection |
| Demo | Screen recording or short Loom showing Admin + Student flows against your API |

### What you do **not** need to build

- A new frontend (provided)
- Mobile apps (API should remain stateless and mobile-ready, but no app is required)

### AI tools policy

Use of AI coding assistants (ChatGPT, Copilot, Cursor, Tabnine, or similar) is **strictly prohibited**. Submitted code must be entirely your own. Reviewers will conduct a code walkthrough; inability to explain your implementation results in disqualification.

---

## 2. Tech stack (mandatory)

| Component | Requirement | Notes |
|-----------|-------------|--------|
| Runtime | Node.js + TypeScript | `strict` mode enabled |
| Framework | **NestJS** (preferred) or Express | NestJS strongly recommended |
| Database | **PostgreSQL** | No substitutes |
| ORM | Prisma or TypeORM | Either acceptable |
| Authentication | JWT | Access + refresh token pattern |
| Authorization | RBAC | Guard-level enforcement |
| File storage | S3-compatible (mock allowed) | Local disk mock is acceptable |
| Caching | Redis | Optional — bonus credit |

Deviation from the database, TypeScript, or auth requirements may result in disqualification.

---

## 3. Getting started with the provided frontend

```bash
# Clone this repository
git clone https://github.com/aaft-developer/aaft-assignment-backend.git
cd aaft-assignment-backend

# Run the reference UI (mock API — for studying the contract only)
npm install
npm run dev
# → http://localhost:3000
```

**Demo accounts (mock seed):**

| Role | Email | Password |
|------|-------|----------|
| Admin | `admin@aaft.com` | `Admin@123` |
| Student | `student@aaft.com` | `Student@123` |

### Connecting your backend

1. Implement your API on a separate port (e.g. `3001`) with base path `/api`.
2. Enable **CORS** for `http://localhost:3000`.
3. In the frontend root, create `.env.local`:

```env
NEXT_PUBLIC_API_URL=http://localhost:3001/api
```

4. Disable mock-only behavior: the frontend uses `localStorage` mock DB sync when `NEXT_PUBLIC_API_URL=/api`. With an external URL, it talks directly to your server — **this is the integration mode we evaluate**.

Full request/response shapes: see [`docs/API-CONTRACT.md`](docs/API-CONTRACT.md).

---

## 4. User roles

### Part A — Frontend integration (required)

The reference UI implements two roles:

| Role | Capabilities in UI |
|------|-------------------|
| **Admin** | Student CRUD, course & lesson management, bulk enrollments, analytics |
| **Student** | Enrolled course library, lesson player, video progress (90% completion rule), dashboard |

Your API must enforce RBAC on every route. Only `POST /api/auth/login` is public.

### Part B — Enterprise extensions (required for senior bar; weighted in rubric)

Extend the system to three roles as defined in the enterprise spec:

| Role | Capabilities |
|------|--------------|
| **Admin** | User management, course runs, bulk enroll, reporting |
| **Instructor** | Content hierarchy, quizzes, learner progress for their courses |
| **Student** | Enroll in course runs, content access, quiz attempts, certificates |

The frontend does not yet expose Instructor or quiz flows; these endpoints are evaluated via API tests, Swagger, or Postman.

---

## 5. Part A — Frontend API contract (required)

Implement every endpoint below. Match HTTP methods, auth headers, status codes, and JSON shapes documented in [`docs/API-CONTRACT.md`](docs/API-CONTRACT.md).

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/login` | Login; returns `{ token, user }` |

Use real JWTs in production. The mock uses `Bearer aaft-token-{userId}`; your API must accept `Authorization: Bearer <access_token>` and return the same response envelope.

### Admin

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/admin/students` | Paginated list (`page`, `limit`, `search`, `sort`) |
| POST | `/api/admin/students` | Create student |
| GET | `/api/admin/students/:id` | Student detail |
| PATCH | `/api/admin/students/:id` | Update student |
| DELETE | `/api/admin/students/:id` | Delete student |
| GET | `/api/admin/courses` | List courses |
| POST | `/api/admin/courses` | Create course |
| PATCH | `/api/admin/courses/:id` | Update course |
| DELETE | `/api/admin/courses/:id` | Delete course |
| POST | `/api/admin/courses/:id/lessons` | Create lesson |
| PATCH | `/api/admin/courses/:id/lessons` | Update lesson (`lessonId` in body) |
| DELETE | `/api/admin/courses/:id/lessons` | Delete lesson (`lessonId` query param) |
| GET | `/api/admin/enrollments` | List enrollments with progress |
| POST | `/api/admin/enrollments` | Bulk assign `{ studentIds, courseIds }` |
| GET | `/api/admin/reports/overview` | Dashboard aggregates |
| GET | `/api/admin/reports/students/:id` | Per-student report |

### Student

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/student/courses` | Enrolled courses + progress (`search`, `status` filters) |
| GET | `/api/student/courses/:id` | Course detail + lessons + per-lesson progress |
| POST | `/api/student/progress` | Update video progress |
| GET | `/api/student/progress/:courseId` | Course-level progress |
| GET | `/api/student/dashboard` | Dashboard stats |

### Response conventions

- **Success:** JSON body directly (not wrapped in `{ data: ... }` unless the contract specifies `items`).
- **Error:** `{ "message": "Human-readable error" }` with appropriate HTTP status.
- **Pagination:** `{ items: T[], meta: { page, limit, totalItems, totalPages } }`.
- **List endpoints:** Always paginated where the contract specifies `meta`.

### Business rules (Part A)

| Rule | Implementation |
|------|----------------|
| Video completion | Mark lesson complete when `percentage >= 90` |
| Progress upsert | One progress record per student per lesson; update in place |
| Course progress | Derived from completed lessons / total lessons |
| Enrollment | Student sees only enrolled courses; admin assigns via bulk endpoint |
| Passwords | bcrypt, minimum 10 rounds (seed demo users in migration/seed script) |

---

## 6. Part B — Enterprise modules (from AAFT LMS spec)

Implement the following in addition to Part A. Modules marked **REQUIRED** must be functional; **BONUS** items improve your score.

### Module 1 — User management (REQUIRED)

- CRUD users with role assignment (Admin | Instructor | Student)
- Admin can change roles
- Soft delete — `is_active = false`, never hard-delete users

### Module 2 — Course management (REQUIRED)

Four-level content hierarchy:

```
Course
└── Section
    └── Subsection
        └── Unit (video | text | attachment | quiz)
```

- Lifecycle: `draft` → `published` → `archived`
- Only published courses can be assigned to a course run
- Instructors edit only their own courses

*Note:* Part A uses a simplified `Course → Lesson` model for the UI. Map lessons to **video units** in your schema, or expose an adapter layer. Document your mapping in README.

### Module 3 — Course runs (REQUIRED)

A **course run** is a scheduled batch instance of a course.

| Column | Type |
|--------|------|
| id | UUID PK |
| course_id | UUID FK |
| run_name | VARCHAR |
| start_date / end_date | DATE |
| enrollment_type | ENUM `free` \| `paid` \| `honor` |
| price | DECIMAL(10,2) nullable |
| pass_threshold | DECIMAL |
| is_active | BOOLEAN |

- Admin can clone a run (copy content references)
- Enrollments are scoped to **course run**, not course
- Cannot delete a run with active enrollments

### Module 4 — Enrollment system (REQUIRED)

| Type | Description |
|------|-------------|
| Free / Audit | Student self-enroll or admin enroll |
| Paid | Access after payment confirmation flag |
| Honor | Admin-issued complimentary access |

- Single self-enroll endpoint
- Bulk enroll (admin): array of student IDs
- Unenroll: mark inactive; retain history
- Unique constraint: one enrollment per student per course run

### Module 5 — Content delivery (REQUIRED)

- Video: URL, title, duration (seconds)
- Text: HTML or Markdown
- Attachment: file URL, name, MIME type
- Access limited to enrolled students and owning instructor

### Module 6 — Video progress tracking (REQUIRED)

- `last_watched_at` (seconds), `completion_percent` per enrollment per unit
- Upsert on update — no duplicate rows
- Auto-complete at ≥ 90%
- Aggregate to overall course progress %

### Module 7 — Assessment system (REQUIRED)

| Table | Key fields |
|-------|------------|
| quizzes | unit_id, title, pass_percent, max_attempts |
| questions | quiz_id, text, type (`mcq_single` \| `mcq_multi`) |
| options | question_id, text, is_correct |
| attempts | quiz_id, enrollment_id, answers (JSONB), score, passed |

- Server-side scoring only
- No partial credit per question
- Respect `max_attempts`

### Module 8 — Grading system (REQUIRED)

- `grading_config`: component weights per course run (sum = 100)
- Weighted aggregate per enrollment; recalculate on new quiz attempt
- `GET` grade breakdown per enrollment

### Module 9 — Certificates (REQUIRED)

- Auto-issue on pass
- Unique verification code
- Public `GET /api/certificates/:code` (no auth)

### Module 10 — Reporting (REQUIRED)

| Endpoint | Purpose |
|----------|---------|
| `GET /api/admin/reports/progress` | Progress across runs |
| `GET /api/admin/reports/completions` | Completion counts per run |
| `GET /api/admin/reports/time-spent` | Aggregated watch time |

Date range filters + pagination on all report endpoints.

### Part B — Additional API surface

```
POST   /api/auth/register
POST   /api/auth/refresh
POST   /api/auth/logout
POST   /api/admin/users
GET    /api/admin/users
PATCH  /api/admin/users/:id
DELETE /api/admin/users/:id
POST   /api/admin/courses
PATCH  /api/admin/courses/:id/publish
POST   /api/admin/course-runs
POST   /api/admin/course-runs/:id/clone
POST   /api/admin/enrollments
POST   /api/instructor/courses/:id/sections
POST   /api/instructor/sections/:id/subsections
POST   /api/instructor/subsections/:id/units
POST   /api/instructor/units/:id/quiz
POST   /api/instructor/quizzes/:id/questions
GET    /api/instructor/course-runs/:id/progress
POST   /api/student/enroll
GET    /api/student/enrollments
GET    /api/student/enrollments/:id/content
POST   /api/student/attempts
GET    /api/student/grades/:enrollmentId
GET    /api/student/certificates
GET    /api/certificates/:code
```

---

## 7. Database design

### Minimum tables (Part A + Part B combined)

| Table | Notes |
|-------|--------|
| users | email unique, password_hash, role, is_active |
| roles | Admin, Instructor, Student |
| courses | status, instructor_id |
| sections, subsections, units | Ordered hierarchy |
| unit_content | Type-specific payload |
| lessons *(or units)* | Bridge to frontend `Lesson` model |
| course_runs | Scheduling + enrollment_type + pass_threshold |
| enrollments | student_id + course_run_id unique |
| video_progress | enrollment_id + unit_id |
| quizzes, questions, options, attempts | Assessment |
| grading_config, grades | Weighted scoring |
| certificates | verification_code unique |

### Mandatory standards

- UUID primary keys
- `created_at` / `updated_at` on every table
- Explicit foreign keys with indexes on `user_id`, `course_run_id`, `enrollment_id`
- Soft deletes for users and enrollments
- Unique: `users.email`, `certificates.verification_code`, `(student_id, course_run_id)`

---

## 8. Security (non-negotiable)

| Requirement | Standard |
|-------------|----------|
| Passwords | bcrypt, ≥ 10 rounds |
| Auth | JWT access (~15 min) + refresh (~7 days) |
| Authorization | RBAC guards on every route except `/auth/*` and `/certificates/:code` |
| Validation | DTO validation; reject unknown fields |
| SQL | ORM parameterized queries only |

Endpoints without proper RBAC are treated as a **critical failure**.

---

## 9. Architecture

Organize by feature module:

```
backend/src/
  modules/
    auth/
    users/
    courses/
    course-runs/
    enrollments/
    content/
    assessments/
    grading/
    certificates/
    reports/
  common/
    guards/
    filters/
    interceptors/
  config/
  main.ts
```

- Controller → Service → Repository
- Strict TypeScript (no `any`)
- No N+1 queries on list endpoints
- All lists paginated

---

## 10. Bonus features

| Feature | Details |
|---------|---------|
| Swagger / OpenAPI | Document all DTOs |
| Redis | Cache catalog, rate limits |
| Bull / BullMQ | Certificate generation, email jobs |
| Structured logging | Correlation IDs, JSON logs |
| Rate limiting | On `/auth` (bonus: global) |

---

## 11. Submission

### Deliverables

1. **Public GitHub repository** with meaningful commit history
2. **`backend/README.md`** — setup, env vars, how to run with the frontend
3. **API documentation** — Swagger UI (preferred) or Postman export
4. **Email** repository link to **aaft@akashmbair** before the stated deadline

### Suggested repo layout

```
aaft-assignment-backend/     ← this repo
├── src/                     ← reference frontend (do not rewrite)
├── docs/
│   └── API-CONTRACT.md
├── backend/                 ← YOUR implementation
│   ├── src/
│   ├── prisma/
│   └── README.md
├── ASSIGNMENT.md            ← this document
└── README.md
```

### Evaluation rubric

| Criterion | Weight | What reviewers look for |
|-----------|--------|-------------------------|
| Frontend integration (Part A) | 25% | All contract endpoints work with provided UI |
| Database schema design | 20% | Normalization, constraints, indexes, UUIDs |
| API design & correctness | 15% | REST conventions, pagination, errors |
| Business logic (Part B) | 20% | Enrollments, grading, certificates |
| Code quality & architecture | 10% | Modules, strict TS, no N+1 |
| Security | 5% | bcrypt, JWT, RBAC, validation |
| Testing | 5% | Service unit tests + core integration tests |

---

## 12. Timeline & support

- Late submissions are not reviewed.
- Questions: **aaft@akashmbair** (subject: `Backend Assignment — <Your Name>`).

Good luck.

*AAFT Online — Engineering Hiring*
