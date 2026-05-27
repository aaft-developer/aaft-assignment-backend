# Frontend API Contract

This document defines the **exact API** the reference Next.js frontend expects. Your backend must implement these routes so the Admin and Student portals work without code changes.

**Base URL:** configured via `NEXT_PUBLIC_API_URL` (default `/api` for mock; use `http://localhost:3001/api` for your server).

**Auth header:** `Authorization: Bearer <token>`

**Error format:** `{ "message": string }`

---

## Types (reference)

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

## POST `/api/auth/login`

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

## Admin — Students

### GET `/api/admin/students`

**Auth:** admin

**Query:** `page`, `limit`, `search`, `sort` (`name` | `email`)

**Response 200:**

```json
{
  "items": [ /* Student[] */ ],
  "meta": { "page": 1, "limit": 10, "totalItems": 25, "totalPages": 3 }
}
```

### POST `/api/admin/students`

**Body:** `{ "name": string, "email": string }`

**Response 201:** `Student`

### GET `/api/admin/students/:id`

**Response 200:** `Student` | **404**

### PATCH `/api/admin/students/:id`

**Body:** partial `Student` fields

**Response 200:** `Student`

### DELETE `/api/admin/students/:id`

**Response 200:** success (empty or `{ success: true }`)

---

## Admin — Courses

### GET `/api/admin/courses`

**Response 200:** `{ "items": Course[] }`

### POST `/api/admin/courses`

**Body:** `{ "name": string, "description": string, "thumbnail"?: string }`

**Response 201:** `Course`

### PATCH `/api/admin/courses/:id`

**Body:** partial course fields

**Response 200:** `Course`

### DELETE `/api/admin/courses/:id`

**Response 200:** success

### POST `/api/admin/courses/:id/lessons`

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

### PATCH `/api/admin/courses/:id/lessons`

**Body:** `{ "lessonId": string, ...updates }`

**Response 200:** `Lesson`

### DELETE `/api/admin/courses/:id/lessons?lessonId=:lessonId`

**Response 200:** success

---

## Admin — Enrollments

### GET `/api/admin/enrollments`

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

### POST `/api/admin/enrollments`

**Body:**

```json
{
  "studentIds": ["student_1", "student_2"],
  "courseIds": ["course_1"]
}
```

**Response 200:** `{ "success": true, "enrollments": Enrollment[] }`

---

## Admin — Reports

### GET `/api/admin/reports/overview`

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

### GET `/api/admin/reports/students/:id`

**Response 200:**

```json
{
  "student": { /* Student */ },
  "enrolledCourses": [{ /* Course + progress */ }],
  "totalWatchTime": 3600
}
```

---

## Student — Courses

### GET `/api/student/courses`

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

### GET `/api/student/courses/:id`

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

## Student — Progress

### POST `/api/student/progress`

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

### GET `/api/student/progress/:courseId`

**Response 200:** `CourseProgress`

---

## Student — Dashboard

### GET `/api/student/dashboard`

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

## Mock-only (do not implement in production)

| Method | Endpoint | Notes |
|--------|----------|--------|
| GET/POST | `/api/mock/state` | Dev snapshot sync — not part of your backend |

---

## CORS

Allow:

- Origin: `http://localhost:3000`
- Methods: `GET, POST, PATCH, DELETE, OPTIONS`
- Headers: `Authorization, Content-Type`

---

## Implementation notes

1. **Student vs user:** The mock maps login user email to a `Student` record. Your schema may use a single `users` table with `role=student`; return the shapes above.
2. **Avatar URLs:** May be generated (e.g. UI Avatars) on create if omitted.
3. **IDs:** String IDs are fine (`uuid` or prefixed strings like `course_1`).
4. **Latency:** Mock adds ~300ms delay; your API should be faster but need not artificial delay.

Reference implementation: `src/mocks/db.ts` and `src/app/api/**/route.ts`.
