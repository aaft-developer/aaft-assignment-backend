# AAFT Mini LMS — Reference Frontend

> **For backend candidates:** This UI is provided for integration testing. Do not rewrite it. Implement the API described in [API-CONTRACT.md](./API-CONTRACT.md) and connect via `NEXT_PUBLIC_API_URL`.

A production-grade **Mini Learning Management System** with separate **Admin** and **Student** portals — built as a frontend assignment demonstrating modern React architecture, Redux Toolkit, and premium UI/UX.

![Next.js](https://img.shields.io/badge/Next.js-15-black?style=flat-square&logo=next.js)
![TypeScript](https://img.shields.io/badge/TypeScript-strict-blue?style=flat-square&logo=typescript)
![Redux](https://img.shields.io/badge/Redux_Toolkit-2.x-764ABC?style=flat-square&logo=redux)
![Tailwind CSS](https://img.shields.io/badge/Tailwind-4-38B2AC?style=flat-square&logo=tailwind-css)

---

## Project summary

**AAFT Mini LMS** is a full-featured learning platform inspired by premium education products (Coursera / MasterClass aesthetics with **AAFT branding** — deep navy `#0A2540` and gold `#D4AF37`).

| Portal | Capabilities |
|--------|----------------|
| **Admin** | Student CRUD (TanStack Table), course & lesson management, bulk enrollments, analytics dashboard with Recharts |
| **Student** | Assigned course library with progress rings, lesson player (YouTube / Vimeo / MP4), resume playback, 90% completion rule, personal progress dashboard |

The app uses **Next.js 15 App Router** Route Handlers as a mock REST API, **axios** with auth interceptors, **five Redux slices** plus **RTK Query**, and **createAsyncThunk** for login, CRUD, and debounced video progress sync. All user-driven changes are **persisted in `localStorage`** and re-hydrated into the mock API on reload so data survives refresh without a real backend.

---

## Live demo & setup

```bash
git clone <your-repo-url>
cd aaft-mini-lms   # or aaft-lms-assignment
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

### Demo accounts

Here are the default credentials to access and test both portals:

| Role | Email | Password |
|------|-------|----------|
| **Admin** | `admin@aaft.com` | `Admin@123` |
| **Student** | `student@aaft.com` | `Student@123` |

### Environment

Copy `.env.example` to `.env.local`:

```env
NEXT_PUBLIC_API_URL=/api
```

---

## Tech stack

| Category | Tools |
|----------|--------|
| Framework | Next.js 15 (App Router), React 19, TypeScript (strict) |
| State | Redux Toolkit — `createSlice`, `createAsyncThunk`, RTK Query, typed hooks |
| Styling | Tailwind CSS 4, CSS variables, Radix UI primitives |
| UI/UX | shadcn-style components, Lucide icons, Framer Motion, Sonner |
| Data | TanStack Table, Recharts, React Hook Form + Zod |
| API | Axios + Next.js Route Handlers + in-memory mock DB |

---

## Features in detail

### Authentication & roles

- Login with **Admin** or **Student** role (auto-detected from credentials)
- Protected routes: `/admin/*` and `/student/*`
- Persistent session via `localStorage` + Redux hydration (`aaft_auth`)
- Logout clears the session only; courses, students, enrollments, and watch progress remain stored
- 401 handling with redirect to login

### Admin portal

- **Students** — search, sort, pagination, create/edit modals, profile with enrolled courses
- **Courses** — grid and table views, thumbnails, lesson CRUD
- **Enrollments** — multi-select bulk assignment
- **Reports** — completion trends, time-by-course, student-wise drill-down

### Student portal

- **Course library** — filters: All / In Progress / Completed
- **Course view** — lesson list with completion indicators
- **Video player** — multi-provider support, auto-resume, segmented watched bar (MP4), playback speed, fullscreen, PiP, prev/next navigation
  - *Foolproof Auto-Resume*: Video playback progress is synchronously saved to `localStorage` as the video plays. If you accidentally close the tab, navigate away, or refresh, the video flawlessly resumes from the exact second you left off, independent of backend sync intervals.
- **Progress dashboard** — watch time, weekly activity chart, courses in progress

---

## Redux architecture

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

### localStorage persistence (no real backend)

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

## API endpoints (mock)

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

## Project structure

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

## Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Development server |
| `npm run build` | Production build |
| `npm run start` | Start production server |
| `npm run lint` | ESLint |
| `npm run typecheck` | TypeScript check |

---

## Design decisions

- **Navy + gold** — trustworthy, premium education brand aligned with AAFT
- **Route Handlers over MSW** — simpler deployment, same-origin API, realistic REST
- **Global vs local state** — Redux for session and shared domain data; React state for modals, filters, and toggles
- **Hybrid video player** — full custom controls for MP4; embedded iframes for YouTube/Vimeo with progress tracked server-side in Redux
- **Mobile-first** — collapsible admin sidebar, student bottom navigation, responsive grids
- **localStorage over json-server** — matches the assignment note to use mock APIs when no backend exists; Redux remains the source of truth in memory while `localStorage` + `/api/mock/state` keep the mock DB consistent across reloads without changing slice/thunk contracts
- **Logout keeps learning data** — signing out clears auth only so students and admins can switch accounts without wiping CRUD or progress demos

### Trade-offs

| Approach | Why |
|----------|-----|
| Snapshot sync vs redux-persist | Keeps all five slices + RTK Query structure identical to the PDF; mock API stays authoritative for RTK Query cache invalidation |
| Debounced export after mutations | Avoids serializing the full DB on every progress tick while still capturing CRUD within ~400ms |
| `/api/mock/state` dev-only | Prevents exposing raw DB export in production builds |

### Resetting persisted data

Clear site data in the browser (or remove `aaft_auth`, `aaft_progress`, `aaft_mock_db`, `aaft_ui_prefs`, and all `video-progress-*` keys from Application → Local Storage) to return to seed data.

---

## Author

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

## License

This project was submitted as an assignment submission. All rights reserved by the author unless otherwise agreed with AAFT.
