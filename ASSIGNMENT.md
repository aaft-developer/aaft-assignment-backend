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

Full request/response shapes: see **Appendix A — Frontend API Contract**.

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

Implement every endpoint below. Match HTTP methods, auth headers, status codes, and JSON shapes documented in **Appendix A**.

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
├── ASSIGNMENT.md            ← this document
├── README.md
├── backend/                 ← YOUR implementation
│   ├── src/
│   ├── prisma/
│   └── README.md
└── src/                     ← reference frontend (do not rewrite)
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

---

# Appendix A — Frontend API Contract

This document defines the **exact API** the reference Next.js frontend expects. Your backend must implement these routes so the Admin and Student portals work without code changes.

**Base URL:** configured via `NEXT_PUBLIC_API_URL` (default `/api` for mock; use `http://localhost:3001/api` for your server).

**Auth header:** `Authorization: Bearer <token>`

**Error format:** `{ "message": string }`

---

### Types (reference)

Source of truth: `src/types/index.ts`.

```typescript
type UserRole = 'admin' | 'student';

type User = {
  id: string;
  name: string;
  email: string;
  role: UserRole;
  avatar: string;
};

type AuthResponse = { token: string; user: User };

type Lesson = {
  id: string;
  courseId: string;
  title: string;
  description: string;
  videoUrl: string;
  duration: number;
  order: number;
};

type Course = {
  id: string;
  name: string;
  description: string;
  thumbnail: string;
  lessons: Lesson[];
  createdAt: string;
};

type Student = {
  id: string;
  name: string;
  email: string;
  avatar: string;
  enrolledCourseIds: string[];
  createdAt: string;
};

type VideoProgress = {
  lastWatched: number;
  percentage: number;
  isCompleted: boolean;
  watchedSegments?: [number, number][];
  totalWatchTime?: number;
};

type CourseProgress = {
  completedVideos: number;
  totalVideos: number;
  percentage: number;
};

type PaginatedMeta = {
  page: number;
  limit: number;
  totalItems: number;
  totalPages: number;
};
```

---

### POST `/api/auth/login`

**Auth:** none

**Body:**

```json
{ "email": "admin@aaft.com", "password": "Admin@123" }
```

**Response 200:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "admin_1",
    "name": "Avery Admin",
    "email": "admin@aaft.com",
    "role": "admin",
    "avatar": "https://..."
  }
}
```

**Response 401:** `{ "message": "Invalid credentials" }`

---

### Admin — Students

#### GET `/api/admin/students`

**Auth:** admin

**Query:** `page`, `limit`, `search`, `sort` (`name` | `email`)

**Response 200:**

```json
{
  "items": [ /* Student[] */ ],
  "meta": { "page": 1, "limit": 10, "totalItems": 25, "totalPages": 3 }
}
```

#### POST `/api/admin/students`

**Body:** `{ "name": string, "email": string }`

**Response 201:** `Student`

#### GET `/api/admin/students/:id`

**Response 200:** `Student` | **404**

#### PATCH `/api/admin/students/:id`

**Body:** partial `Student` fields

**Response 200:** `Student`

#### DELETE `/api/admin/students/:id`

**Response 200:** success (empty or `{ success: true }`)

---

### Admin — Courses

#### GET `/api/admin/courses`

**Response 200:** `{ "items": Course[] }`

#### POST `/api/admin/courses`

**Body:** `{ "name": string, "description": string, "thumbnail"?: string }`

**Response 201:** `Course`

#### PATCH `/api/admin/courses/:id`

**Body:** partial course fields

**Response 200:** `Course`

#### DELETE `/api/admin/courses/:id`

**Response 200:** success

#### POST `/api/admin/courses/:id/lessons`

**Body:**

```json
{
  "title": "string",
  "description": "string",
  "videoUrl": "string",
  "duration": 600
}
```

**Response 201:** `Lesson`

#### PATCH `/api/admin/courses/:id/lessons`

**Body:** `{ "lessonId": string, ...updates }`

**Response 200:** `Lesson`

#### DELETE `/api/admin/courses/:id/lessons?lessonId=:lessonId`

**Response 200:** success

---

### Admin — Enrollments

#### GET `/api/admin/enrollments`

**Response 200:**

```json
{
  "items": [
    {
      "id": "enr_1",
      "studentId": "student_1",
      "courseId": "course_1",
      "assignedAt": "2025-01-10",
      "studentName": "Jordan Miles",
      "courseName": "Cinematic Storytelling Lab",
      "progress": { "completedVideos": 1, "totalVideos": 3, "percentage": 33 }
    }
  ]
}
```

#### POST `/api/admin/enrollments`

**Body:**

```json
{
  "studentIds": ["student_1", "student_2"],
  "courseIds": ["course_1"]
}
```

**Response 200:** `{ "success": true, "enrollments": Enrollment[] }`

---

### Admin — Reports

#### GET `/api/admin/reports/overview`

**Response 200:**

```json
{
  "metrics": [
    { "label": "Total Students", "value": "12", "change": "+12%" }
  ],
  "completionTrend": [{ "month": "Jan", "rate": 62 }],
  "timeSpentByCourse": [{ "course": "Course Name", "hours": 24 }],
  "courseCompletion": [{ "course": "Course Name", "completed": 5, "total": 10 }]
}
```

#### GET `/api/admin/reports/students/:id`

**Response 200:**

```json
{
  "student": { /* Student */ },
  "enrolledCourses": [{ /* Course + progress */ }],
  "totalWatchTime": 3600
}
```

---

### Student — Courses

#### GET `/api/student/courses`

**Auth:** student

**Query:** `search`, `status` (`not_started` | `in_progress` | `completed` | `all`)

**Response 200:**

```json
{
  "items": [
    {
      "id": "course_1",
      "name": "...",
      "description": "...",
      "thumbnail": "...",
      "lessons": [],
      "createdAt": "2025-01-10",
      "progress": { "completedVideos": 1, "totalVideos": 3, "percentage": 33 },
      "status": "in_progress"
    }
  ]
}
```

#### GET `/api/student/courses/:id`

**Response 200:**

```json
{
  "course": { "id", "name", "description" },
  "progress": { "percentage", "completedVideos", "totalVideos" },
  "lessons": [
    {
      "id": "lesson_1",
      "title": "...",
      "description": "...",
      "videoUrl": "...",
      "duration": 720,
      "progress": {
        "lastWatched": 120,
        "percentage": 45,
        "isCompleted": false,
        "watchedSegments": [[0, 120]]
      }
    }
  ]
}
```

**Response 404:** not enrolled or not found

---

### Student — Progress

#### POST `/api/student/progress`

**Body:**

```json
{
  "videoId": "lesson_1_1",
  "courseId": "course_1",
  "lastWatched": 180,
  "percentage": 25,
  "duration": 720,
  "watchedSegments": [[0, 180]]
}
```

**Rules:**

- Set `isCompleted: true` when `percentage >= 90`
- Upsert by `(student, videoId)` — never duplicate rows
- Recompute `courseProgress` from all lessons in the course

**Response 200:**

```json
{
  "videoProgress": { /* VideoProgress */ },
  "courseProgress": { /* CourseProgress */ }
}
```

#### GET `/api/student/progress/:courseId`

**Response 200:** `CourseProgress`

---

### Student — Dashboard

#### GET `/api/student/dashboard`

**Response 200:**

```json
{
  "inProgress": 2,
  "completed": 1,
  "totalWatchTime": 5400,
  "courses": [ /* StudentCourseCard[] */ ],
  "weeklyProgress": [
    { "day": "Mon", "minutes": 45 }
  ]
}
```

---

### Mock-only (do not implement in production)

| Method | Endpoint | Notes |
|--------|----------|--------|
| GET/POST | `/api/mock/state` | Dev snapshot sync — not part of your backend |

---

### CORS

Allow:

- Origin: `http://localhost:3000`
- Methods: `GET, POST, PATCH, DELETE, OPTIONS`
- Headers: `Authorization, Content-Type`

---

### Implementation notes

1. **Student vs user:** The mock maps login user email to a `Student` record. Your schema may use a single `users` table with `role=student`; return the shapes above.
2. **Avatar URLs:** May be generated (e.g. UI Avatars) on create if omitted.
3. **IDs:** String IDs are fine (`uuid` or prefixed strings like `course_1`).
4. **Latency:** Mock adds ~300ms delay; your API should be faster but need not artificial delay.

Reference implementation: `src/mocks/db.ts` and `src/app/api/**/route.ts`.

---

# Appendix B — Reference Frontend

> **For backend candidates:** This UI is provided for integration testing. Do not rewrite it. Implement the API described in **Appendix A** of this document and connect via `NEXT_PUBLIC_API_URL`.

A production-grade **Mini Learning Management System** with separate **Admin** and **Student** portals — built as a frontend assignment demonstrating modern React architecture, Redux Toolkit, and premium UI/UX.

![Next.js](https://img.shields.io/badge/Next.js-15-black?style=flat-square&logo=next.js)
![TypeScript](https://img.shields.io/badge/TypeScript-strict-blue?style=flat-square&logo=typescript)
![Redux](https://img.shields.io/badge/Redux_Toolkit-2.x-764ABC?style=flat-square&logo=redux)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-4-38B2AC?style=flat-square&logo=tailwind-css)

---

### Project summary

**AAFT Mini LMS** is a full-featured learning platform inspired by premium education products (Coursera / MasterClass aesthetics with **AAFT branding** — deep navy `#0A2540` and gold `#D4AF37`).

| Portal | Capabilities |
|--------|----------------|
| **Admin** | Student CRUD (TanStack Table), course & lesson management, bulk enrollments, analytics dashboard with Recharts |
| **Student** | Assigned course library with progress rings, lesson player (YouTube / Vimeo / MP4), resume playback, 90% completion rule, personal progress dashboard |

The app uses **Next.js 15 App Router** Route Handlers as a mock REST API, **axios** with auth interceptors, **five Redux slices** plus **RTK Query**, and **createAsyncThunk** for login, CRUD, and debounced video progress sync. All user-driven changes are **persisted in `localStorage`** and re-hydrated into the mock API on reload so data survives refresh without a real backend.

---

### Live demo & setup

```bash
git clone <your-repo-url>
cd aaft-mini-lms   # or aaft-lms-assignment
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

#### Demo accounts

Here are the default credentials to access and test both portals:

| Role | Email | Password |
|------|-------|----------|
| **Admin** | `admin@aaft.com` | `Admin@123` |
| **Student** | `student@aaft.com` | `Student@123` |

#### Environment

Copy `.env.example` to `.env.local`:

```env
NEXT_PUBLIC_API_URL=/api
```

---

### Tech stack

| Category | Tools |
|----------|--------|
| Framework | Next.js 15 (App Router), React 19, TypeScript (strict) |
| State | Redux Toolkit — `createSlice`, `createAsyncThunk`, RTK Query, typed hooks |
| Styling | Tailwind CSS 4, CSS variables, Radix UI primitives |
| UI/UX | shadcn-style components, Lucide icons, Framer Motion, Sonner |
| Data | TanStack Table, Recharts, React Hook Form + Zod |
| API | Axios + Next.js Route Handlers + in-memory mock DB |

---

### Features in detail

#### Authentication & roles

- Login with **Admin** or **Student** role (auto-detected from credentials)
- Protected routes: `/admin/*` and `/student/*`
- Persistent session via `localStorage` + Redux hydration (`aaft_auth`)
- Logout clears the session only; courses, students, enrollments, and watch progress remain stored
- 401 handling with redirect to login

#### Admin portal

- **Students** — search, sort, pagination, create/edit modals, profile with enrolled courses
- **Courses** — grid and table views, thumbnails, lesson CRUD
- **Enrollments** — multi-select bulk assignment
- **Reports** — completion trends, time-by-course, student-wise drill-down

#### Student portal

- **Course library** — filters: All / In Progress / Completed
- **Course view** — lesson list with completion indicators
- **Video player** — multi-provider support, auto-resume, segmented watched bar (MP4), playback speed, fullscreen, PiP, prev/next navigation
  - *Foolproof Auto-Resume*: Video playback progress is synchronously saved to `localStorage` as the video plays. If you accidentally close the tab, navigate away, or refresh, the video flawlessly resumes from the exact second you left off, independent of backend sync intervals.
- **Progress dashboard** — watch time, weekly activity chart, courses in progress

---

### Redux architecture

```
store/
├── slices/         auth | courses | students | progress | ui
├── thunks/         authThunks, coursesThunks, studentsThunks, progressThunks
├── store.ts        configureStore with all slices + RTK Query middleware
├── makeStore.ts    store factory for SSR-safe initialization
├── selectors.ts    memoized selectors
├── hooks.ts        useAppDispatch, useAppSelector
├── types.ts        RootState, AppDispatch types
└── persistListener.ts  middleware that syncs Redux → localStorage
```

| Slice | Responsibility |
|-------|----------------|
| `auth` | user, token, isAuthenticated, loading, error |
| `courses` | courses list, selectedCourse |
| `students` | students list, selectedStudent, pagination meta |
| `progress` | videoProgress & courseProgress maps |
| `ui` | sidebar, theme, notifications |
| `aaftApi` | RTK Query cache (lists, reports, mutations) |

**Pattern:** RTK Query for fetch/cache and tag invalidation; **thunks** for auth and complex video progress (debounce + Redux + localStorage + API). Redux slices and thunks are unchanged in shape; persistence is layered via storage helpers, axios interceptors, and a listener middleware.

#### localStorage persistence (no real backend)

Because the assignment uses a **mock in-memory API** (Next.js Route Handlers), data would otherwise reset on server restart or hard refresh. The app mirrors assignment requirements (Redux Toolkit + thunks + RTK Query) and adds a robust persistence layer:

| Key | Contents |
|-----|----------|
| `aaft_auth` | Logged-in user + JWT-style token |
| `aaft_progress` | `videoProgress` and `courseProgress` maps (student playback) |
| `aaft_mock_db` | Full mock DB snapshot: courses, students, enrollments, video progress, watch sessions |
| `aaft_ui_prefs` | Sidebar open state and theme preference |
| `video-progress-[lessonId]` | Exact last-watched timestamp per video, updated continuously for foolproof auto-resume |

**Flow:**

1. On first API request, the client hydrates the server mock DB from `aaft_mock_db` (and merges `aaft_progress` if needed) via `POST /api/mock/state`.
2. All CRUD, enrollments, and progress updates still go through existing Redux thunks / RTK Query → axios → Route Handlers (assignment pattern unchanged).
3. After mutating requests, a debounced export saves the latest server snapshot back to `aaft_mock_db`.
4. Video playback updates `localStorage` (`video-progress-[id]`) synchronously as the video plays for immediate, foolproof session recovery.
5. A Redux listener writes to `aaft_progress` periodically and schedules a mock DB export for backend sync.

**Relevant files:** `lib/mock-db-storage.ts`, `lib/mock-db-sync.ts`, `lib/progress-storage.ts`, `lib/auth-storage.ts`, `lib/ui-storage.ts`, `components/common/MockDbHydrator.tsx`, `components/student/VideoPlayer.tsx`, `app/api/mock/state/route.ts`.

---

### API endpoints (mock)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/login` | Authenticate admin or student |
| GET/POST | `/api/admin/students` | List / create students |
| PATCH/DELETE | `/api/admin/students/[id]` | Update / delete |
| GET/POST | `/api/admin/courses` | List / create courses |
| PATCH/DELETE | `/api/admin/courses/[id]` | Update / delete course |
| POST/PATCH/DELETE | `/api/admin/courses/[id]/lessons` | Lesson CRUD |
| GET/POST | `/api/admin/enrollments` | List all / bulk assign courses |
| GET | `/api/admin/reports/overview` | Analytics aggregates |
| GET | `/api/admin/reports/students/[id]` | Student report |
| GET | `/api/student/courses` | Enrolled courses + progress |
| GET | `/api/student/courses/[id]` | Course detail + lessons |
| POST | `/api/student/progress` | Update video progress |
| GET | `/api/student/progress/[courseId]` | Get progress for a specific course |
| GET | `/api/student/dashboard` | Student stats |
| GET/POST | `/api/mock/state` | Export / import mock DB snapshot (dev only; powers localStorage sync) |

---

### Project structure

```
src/
├── app/
│   ├── admin/          Admin portal pages (courses, students, enrollments, reports)
│   ├── student/        Student portal pages (courses, dashboard, lesson player)
│   ├── login/          Authentication page
│   └── api/            Route Handlers (mock REST API)
│
├── components/
│   ├── common/         Shells, guards, breadcrumbs, empty states, hydrators
│   ├── admin/          Admin-specific UI (tables, forms, overview)
│   ├── student/        Course cards, video player
│   ├── auth/           LoginForm
│   ├── charts/         ChartPanel (Recharts wrapper)
│   └── ui/             Shared primitives (button, card, dialog, input, etc.)
│
├── store/              Redux slices, thunks, selectors, persistListener
├── services/           apiClient.ts (axios), api.ts (RTK Query)
├── providers/          Redux + theme provider wrapper
├── mocks/              seed.ts, db.ts (mock DB with export/import snapshot)
├── types/              TypeScript models + mock-db snapshot type
├── utils/              video.ts (provider detection, segment merging)
└── lib/                validations, utils, api-helpers, chart-config,
                        *-storage.ts, mock-db-sync, mock-db-merge, mock-db-patch
```

---

### Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Development server |
| `npm run build` | Production build |
| `npm run start` | Start production server |
| `npm run lint` | ESLint |
| `npm run typecheck` | TypeScript check |

---

### Design decisions

- **Navy + gold** — trustworthy, premium education brand aligned with AAFT
- **Route Handlers over MSW** — simpler deployment, same-origin API, realistic REST
- **Global vs local state** — Redux for session and shared domain data; React state for modals, filters, and toggles
- **Hybrid video player** — full custom controls for MP4; embedded iframes for YouTube/Vimeo with progress tracked server-side in Redux
- **Mobile-first** — collapsible admin sidebar, student bottom navigation, responsive grids
- **localStorage over json-server** — matches the assignment note to use mock APIs when no backend exists; Redux remains the source of truth in memory while `localStorage` + `/api/mock/state` keep the mock DB consistent across reloads without changing slice/thunk contracts
- **Logout keeps learning data** — signing out clears auth only so students and admins can switch accounts without wiping CRUD or progress demos

#### Trade-offs

| Approach | Why |
|----------|-----|
| Snapshot sync vs redux-persist | Keeps all five slices + RTK Query structure identical to the PDF; mock API stays authoritative for RTK Query cache invalidation |
| Debounced export after mutations | Avoids serializing the full DB on every progress tick while still capturing CRUD within ~400ms |
| `/api/mock/state` dev-only | Prevents exposing raw DB export in production builds |

#### Resetting persisted data

Clear site data in the browser (or remove `aaft_auth`, `aaft_progress`, `aaft_mock_db`, `aaft_ui_prefs`, and all `video-progress-*` keys from Application → Local Storage) to return to seed data.

---

### Author

**Varun Pandey**  
Senior Front-End Engineer · React & Next.js

| | |
|---|---|
| Email | [varundwt@gmail.com](mailto:varundwt@gmail.com) |
| Portfolio | [varun-pandey.vercel.app](https://varun-pandey.vercel.app/) |
| LinkedIn | [linkedin.com/in/semiintrovert](https://www.linkedin.com/in/semiintrovert/) |
| GitHub | [github.com/varundwt](https://github.com/varundwt) |

Bundled as the **reference frontend** for the AAFT Backend Developer Assignment (Admin + Student portals).

---

### License

This project was submitted as an assignment submission. All rights reserved by the author unless otherwise agreed with AAFT.
