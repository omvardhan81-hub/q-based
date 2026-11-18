# Q-Based Exam Platform — Merged Full-Stack Project

This project merges **two previously separate versions** of the Q-Based app
(one had real JWT auth + student exam flow + leaderboard; the other had a
JSON Paper Merger with MCQ/MSQ/NAT question types + bulk upload) into a
**single working full-stack app**: `backend/` (Express + MongoDB) and
`frontend/` (React + Vite), fully wired together.

## What was merged

**Backend** — combined both APIs into one Express server:
- Auth: register / login / JWT sessions (`/api/auth`)
- Question Bank CRUD for admins (`/api/questions`)
- **New:** JSON Paper Merger bulk upload + fetch, supporting MCQ, MSQ, and
  NAT question types (`/api/admin/upload-json`, `/api/admin/fetch-questions`)
- Exam flow: random question fetch, server-side grading (now supports
  MCQ/MSQ/NAT), per-user result storage (`/api/exam`)
- Leaderboard ranked by best percentage score (`/api/leaderboard`)
- **New:** Student profile management — view/edit name, mobile, and email,
  change password, and upload a profile picture (`/api/profile`), merged in
  from a previously separate profile-management backend and rewired onto the
  shared `User` model/JWT auth instead of its own duplicate auth system.
  Uploaded pictures are served statically from `/uploads`.

All admin-only routes now require a logged-in admin account (previously the
Paper Merger backend had no auth at all).

**Frontend** — used the version that was already wired to the real backend
(login/register/exam/leaderboard via `src/api/client.js`) as the base, and
merged in the newer UI features from the other version on top of it:
- `ExamInterface.jsx` now supports MCQ (radio), MSQ (checkboxes), and NAT
  (numeric input) questions, fetched from and graded by the real backend.
- `QuestionBank.jsx` gained an MSQ option (comma-separated correct answers).
- `PaperMerger.jsx` gained a "☁️ Upload to Database" button that posts the
  compiled paper straight into MongoDB via the new admin upload endpoint.
- `ResultPage.jsx` now renders the richer `totalScore / maximumMarks /
  detailedResult` breakdown instead of the older percentage-only result.
- `Profile.jsx` is now a fully working page instead of a static mockup: it
  loads the logged-in student's real details, lets them inline-edit
  name/mobile/email, change their password, and upload a new profile
  picture — all wired to the new `/api/profile` endpoints.

## Project structure

```
backend/     Express + MongoDB API (see backend/README below-inline)
frontend/    React + Vite app
```

## Running it locally

### 1. Backend

```bash
cd backend
npm install
cp .env.example .env   # edit MONGO_URI etc. if needed (a working local default is already set)
npm run seed            # creates sample MCQ/MSQ/NAT questions + an admin + a demo student
npm run dev              # starts on http://localhost:5000
```

Seeded accounts:
- Admin: `admin@qbased.com` / `admin1234`
- Student: `student@qbased.com` / `student123`

You need a MongoDB instance reachable at `MONGO_URI` (local `mongod`, Docker,
or a free MongoDB Atlas cluster).

### 2. Frontend

```bash
cd frontend
npm install
npm run dev    # starts on http://localhost:5173
```

The frontend talks to `http://localhost:5000/api` by default. Override it
with a `.env` file in `frontend/` containing `VITE_API_URL=...` if your
backend runs elsewhere.

### 3. Try it out

1. Log in as `admin@qbased.com` and open **Test Management & Merger** →
   upload/merge an MCQ+MSQ+NAT paper → click "Upload to Database".
2. Log out, register/log in as a student, and take the exam from
   `/dashboard` → `/exam`. It pulls random questions from whatever is in the
   database (seeded questions + anything the admin uploaded).
3. Check `/admin` → Leaderboard tab, or the student Result page, to see the
   real graded results.
