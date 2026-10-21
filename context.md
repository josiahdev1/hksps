# HKSPS Context Document — Legacy PHP to Next.js + Supabase Migration

> **100% SELF-CONTAINED REPOSITORY NOTICE**:
> This repository (`hksps`) is completely self-contained. You do **NOT** need access to, nor should you look for, any external PHP folders or files.
> Every single function, database table, column, validation rule, user role, and UI workflow from the legacy PHP system is already fully extracted, analyzed, and documented inside this file and `architecture.md`.
> All development, specification drafting (`docs/SPEC.md`), and ticket implementation must focus 100% exclusively on this Next.js + Supabase repository.

This document serves as the single source of truth for the **Hawak Kamay Scholarship & Student Portal System**.
* **`HKSPS` (UPPERCASE)** = The legacy PHP/MySQL codebase reference.
* **`hksps` (lowercase)** = The modern Next.js 16 + Supabase codebase (`hksps` repository).

---

## 1. Executive Summary & Domain Overview

* **Application Name**: Hawak Kamay Scholarship & Student Portal System
* **Domain**: College / University Scholarship Management & Student Duty Hour Tracking
* **Objective**: Rebuild the legacy PHP system into Next.js (App Router) and hosted Supabase (PostgreSQL, Auth, RLS, Storage) with 100% feature parity, improved architecture, tamper-proof timer logging, and modern UI.

---

## 2. Legacy PHP System Analysis (`HKSPS`)

### 2.1 Roles & Permissions

| Role | Domain Responsibilities |
|---|---|
| **Student** (Scholar) | • Browse scholarship duties posted by departments/teachers.<br>• Submit applications with cover letter.<br>• Track status (`pending`, `approved`, `rejected`).<br>• Real-time duty tracking: clock in/out with automated elapsed time calculation.<br>• View history of rendered duty hours.<br>• Submit issue/concern reports. |
| **Teacher** (Facilitator / Employer) | • Post, edit, close, and delete duties (schedule, location, deadline, max applicants).<br>• Review student applicants (approve/reject).<br>• Monitor student facilitators assigned to their duties and view rendered hours logs.<br>• Submit reports and view activity logs. |
| **Admin** (Scholarship Coordinator) | • Manage academic departments and courses.<br>• User CRUD (students, teachers, admins), role toggling, deletion.<br>• Review, triage, and resolve submitted incident/issue reports.<br>• Audit system-wide activities.<br>• Manage global system settings & feature flags (`reports_enabled`, `job_applications_enabled`). |

### 2.2 Legacy File Tree Structure (`HKSPS/`)

```
HKSPS/
├── index.php                      # Dynamic router based on $_SESSION['role'] and $_GET['page']
├── landing.php                    # Public landing page
├── hawak_kamay.sql                # MySQL Database schema & seed data
├── index.css & main.js            # Base front-end styling and scripts
│
├── Auth/                          # Authentication module
│   ├── login.php                  # User login handler & view
│   ├── register.php               # User registration handler & view
│   ├── logout.php                 # Session destroy & logout
│   ├── middleware.php             # Role validation helper (requireRole)
│   ├── auth.php                   # Authentication routines
│   └── Unauthorized.php           # 403 access denied page
│
├── Students/                      # Student role module
│   ├── dashboard.php              # Student overview & active status
│   ├── browse-duty.php            # Duty listings with department filter
│   ├── apply.php                  # Duty application modal/form
│   ├── my-application.php         # Student's submitted applications
│   ├── duty.php                   # Check-in / Check-out timer API
│   ├── duties.php                 # Current assigned duties
│   ├── duty_logs.php              # History of rendered hours
│   ├── remove-duty.php            # Withdrawal action
│   └── actions/remove-application.php
│
├── Teachers/                      # Teacher / Facilitator role module
│   ├── dashboard.php              # Teacher metrics & applicant count
│   ├── my-post.php                # Posted duties overview
│   ├── duties/                    # Duty CRUD handlers
│   │   ├── create-duty.php        # New duty form & insert
│   │   ├── update-duty.php        # Edit duty details
│   │   └── delete-duty.php        # Delete duty
│   ├── applicants.php             # Review student applicants
│   ├── update-application.php     # Accept/reject applicant
│   ├── facilitators.php           # View active facilitators
│   ├── facilitator_logs.php       # Monitor facilitator rendered hours
│   ├── activities.php             # Activity audit trail for teacher
│   └── chart.php                  # Visual breakdown
│
├── admin/                         # Admin role module
│   ├── dashboard.php              # System overview, counts, analytics
│   ├── manage-departments.php     # Department listing & modal
│   ├── departments/               # Department CRUD (add, update, delete)
│   ├── users/                     # User management (update-user, delete-user)
│   ├── students.php               # Student directory
│   ├── teachers.php               # Teacher directory
│   ├── reports.php                # Triage & resolve submitted reports
│   ├── update_report_ajax.php     # AJAX status update for reports
│   ├── activities.php             # System-wide audit trail
│   ├── fetch_duty_logs.php        # Duty hours audit table
│   └── settings.php               # System settings & feature flags
│
├── Shared/                        # Role-agnostic components
│   ├── profile.php                # View user profile & photo
│   ├── update_profile.php         # Update profile details & picture
│   ├── report.php                 # Submit issue/problem report
│   ├── submit_report.php          # Action to insert report
│   └── fetch_reports.php          # Retrieve reports list
│
├── config/                        # Configuration & helpers
│   ├── conn.php                   # MySQL connection ($conn = new mysqli(...))
│   ├── init.php                   # Session start & auth guard
│   ├── helpers.php                # Utility functions
│   └── modal.php                  # Modal configuration
│
└── modals/                        # Department, student, and teacher modals
```

### 2.3 Legacy Database Tables (`hawak_kamay.sql`)

1. **`users`**: `id`, `name`, `email`, `password`, `role` (`student`, `teacher`, `admin`), `student_id`, `teacher_id`, `department_id`, `year_level`, `office_location`, `profile_image`, `phone`.
2. **`departments`**: `id`, `code` (`CITE`, `COE`, `CBA`, etc.), `name`, `description`.
3. **`courses`**: `id`, `department_code` / `department_id`, `name`, `major`.
4. **`job_postings`**: `id`, `title`, `description`, `requirements`, `schedule`, `location`, `posted_by_id`, `posted_date`, `deadline`, `status` (`open`, `closed`, `filled`), `max_applicants`, `current_applicants`, `department_id`.
5. **`applications`**: `id`, `job_id`, `student_id`, `applied_date`, `status` (`pending`, `approved`, `rejected`), `cover_letter`.
6. **`duty_hours`**: `id`, `user_id`, `duty_id`, `duty_date`, `start_time`, `end_time`, `total_seconds`.
7. **`reports`**: `id`, `submitted_by`, `role` (`student`, `teacher`), `title`, `description`, `status` (`pending`, `reviewed`, `resolved`), `resolved_by`.
8. **`activities`**: `id`, `user_id`, `job_id`, `action_type`, `description`, `created_at`.
9. **`settings`**: `id`, `site_name`, `admin_email`, `reports_enabled`, `job_applications_enabled`.

---

## 3. Target Modern Architecture (`hksps`)

### 3.1 Pinned Stack & Constraints (PHINMA AI Builder Template)
* **Framework**: Next.js 16 (App Router, Turbopack, React 19)
* **Language**: TypeScript (Strict Mode)
* **Styling & UI**: Tailwind CSS v4 + components in `src/components/ui/` (`Button`, `Card`, `Input`, `Label`, `Textarea`)
* **Backend**: Hosted Supabase (PostgreSQL with Row Level Security, Auth via `@supabase/ssr`, Storage)
* **Deployment**: Vercel (Production URL on `main` push)
* **Security Rules**:
  * **RLS is mandatory on every single table**.
  * No `service_role` key anywhere in code.
  * Browser code uses only `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`.

### 3.2 Target PostgreSQL Schema (`supabase/sql/`)

```sql
-- Role Enums
create type user_role as enum ('student', 'teacher', 'admin');
create type application_status as enum ('pending', 'approved', 'rejected');
create type job_status as enum ('open', 'closed', 'filled');
create type report_status as enum ('pending', 'reviewed', 'resolved');

-- 1. Departments
create table public.departments (
    id uuid primary key default gen_random_uuid(),
    code varchar(15) unique not null,
    name varchar(255) not null,
    description text,
    badge_url text,
    created_at timestamptz default now() not null,
    updated_at timestamptz default now() not null
);

-- 2. Courses
create table public.courses (
    id uuid primary key default gen_random_uuid(),
    department_id uuid references public.departments(id) on delete cascade not null,
    name varchar(255) not null,
    major varchar(255),
    created_at timestamptz default now() not null
);

-- 3. Profiles (Linked to auth.users)
create table public.profiles (
    id uuid primary key references auth.users(id) on delete cascade,
    email varchar(255) unique not null,
    full_name varchar(255) not null,
    role user_role not null default 'student',
    student_id varchar(50),
    teacher_id varchar(50),
    phone varchar(30),
    avatar_url text,
    department_id uuid references public.departments(id) on delete set null,
    course_id uuid references public.courses(id) on delete set null,
    year_level varchar(20),
    office_location varchar(255),
    created_at timestamptz default now() not null,
    updated_at timestamptz default now() not null
);

-- 4. Job Postings / Duties
create table public.job_postings (
    id uuid primary key default gen_random_uuid(),
    title varchar(255) not null,
    description text not null,
    requirements text,
    schedule varchar(255) not null,
    location varchar(255),
    department_id uuid references public.departments(id) on delete set null,
    posted_by_id uuid references public.profiles(id) on delete cascade not null,
    posted_date date default current_date not null,
    deadline date not null,
    status job_status default 'open' not null,
    max_applicants int default 1 not null check (max_applicants > 0),
    current_applicants int default 0 not null check (current_applicants >= 0),
    created_at timestamptz default now() not null,
    updated_at timestamptz default now() not null
);

-- 5. Applications
create table public.applications (
    id uuid primary key default gen_random_uuid(),
    job_id uuid references public.job_postings(id) on delete cascade not null,
    student_id uuid references public.profiles(id) on delete cascade not null,
    status application_status default 'pending' not null,
    cover_letter text,
    created_at timestamptz default now() not null,
    updated_at timestamptz default now() not null,
    constraint unique_student_job_application unique (job_id, student_id)
);

-- 6. Duty Sessions / Time Tracking
create table public.duty_sessions (
    id uuid primary key default gen_random_uuid(),
    user_id uuid references public.profiles(id) on delete cascade not null,
    duty_id uuid references public.job_postings(id) on delete cascade not null,
    duty_date date default current_date not null,
    start_time timestamptz default now() not null,
    end_time timestamptz,
    total_seconds int default 0 not null,
    created_at timestamptz default now() not null
);

-- 7. Reports
create table public.reports (
    id uuid primary key default gen_random_uuid(),
    submitted_by uuid references public.profiles(id) on delete cascade not null,
    role user_role not null,
    title varchar(255) not null,
    description text not null,
    attachment_url text,
    status report_status default 'pending' not null,
    resolved_by uuid references public.profiles(id) on delete set null,
    created_at timestamptz default now() not null,
    updated_at timestamptz default now() not null
);

-- 8. Activity Audit Logs
create table public.activities (
    id uuid primary key default gen_random_uuid(),
    user_id uuid references public.profiles(id) on delete cascade not null,
    job_id uuid references public.job_postings(id) on delete set null,
    action_type varchar(100) not null,
    description text,
    created_at timestamptz default now() not null
);

-- 9. System Settings
create table public.system_settings (
    id int primary key default 1 check (id = 1),
    site_name varchar(255) default 'HAWAK KAMAY SCHOLARSHIP PROGRAM SYSTEM' not null,
    admin_email varchar(255) default 'admin@example.com' not null,
    reports_enabled boolean default true not null,
    job_applications_enabled boolean default true not null,
    updated_at timestamptz default now() not null
);
insert into public.system_settings (id) values (1) on conflict do nothing;
```

### 3.3 Core Database Logic & Triggers

1. **Auto Profile on Signup (`handle_new_user`)**:
   Creates a row in `public.profiles` whenever a new user signs up in `auth.users`.
2. **Applicant Count Sync (`sync_applicant_count`)**:
   Automatically increments/decrements `current_applicants` on `job_postings` when an application is submitted or removed.

### 3.4 Row Level Security (RLS) Summary

* `profiles`: Authenticated users can view; users can only update their own profile; admins can edit all.
* `departments` & `courses`: Read by anyone authenticated; written only by admins.
* `job_postings`: Read by anyone authenticated; created/edited by teachers (own posts) or admins.
* `applications`: Students read/write their own applications; teachers read applications for their own duty posts; admins read/edit all.
* `duty_sessions`: Students insert/update their active sessions; teachers view their facilitators' logs; admins view all.
* `reports`: Users submit and view their own reports; admins view, review, and resolve all reports.
* `activities`: Authenticated users view relevant logs; writes triggered by system actions.

---

## 4. Feature Mapping Matrix (Legacy `HKSPS` -> Modern `hksps`)

| Legacy PHP File (`HKSPS/`) | Modern Next.js Route / Action (`hksps/`) | Description |
|---|---|---|
| `landing.php` | `src/app/page.tsx` | Modern landing page & portal introduction |
| `Auth/login.php` | `src/app/(auth)/login/page.tsx` | Supabase email/password login |
| `Auth/register.php` | `src/app/(auth)/register/page.tsx` | User signup with role & department picker |
| `Auth/logout.php` | `src/server/auth.ts` (`logout`) | Session termination & cookie clear |
| `Students/dashboard.php` | `src/app/(dashboard)/student/dashboard/page.tsx` | Student metric cards & active timer summary |
| `Students/browse-duty.php` | `src/app/(dashboard)/student/duties/page.tsx` | Searchable/filterable duty directory |
| `Students/apply.php` | `src/components/duty/ApplicationModal.tsx` | Cover letter application modal dialog |
| `Students/my-application.php` | `src/app/(dashboard)/student/applications/page.tsx` | Application status tracking table |
| `Students/duty.php` | `src/app/(dashboard)/student/tracker/page.tsx` | Live stopwatch timer (clock in/clock out) |
| `Students/duty_logs.php` | `src/app/(dashboard)/student/logs/page.tsx` | Completed duty sessions & total hours |
| `Teachers/dashboard.php` | `src/app/(dashboard)/teacher/dashboard/page.tsx` | Teacher duty metrics & applicants summary |
| `Teachers/my-post.php` | `src/app/(dashboard)/teacher/posts/page.tsx` | Manage posted duties (create/edit/close) |
| `Teachers/applicants.php` | `src/app/(dashboard)/teacher/applicants/page.tsx` | Review student applicants (approve/reject) |
| `Teachers/facilitator_logs.php` | `src/app/(dashboard)/teacher/logs/page.tsx` | Monitor facilitator rendered hours |
| `admin/manage-departments.php` | `src/app/(dashboard)/admin/departments/page.tsx` | Department & course management |
| `admin/users/` | `src/app/(dashboard)/admin/users/page.tsx` | User directory, role editing, deletion |
| `admin/reports.php` | `src/app/(dashboard)/admin/reports/page.tsx` | Incident reports triage & resolution |
| `admin/settings.php` | `src/app/(dashboard)/admin/settings/page.tsx` | Global settings & feature flags |
| `Shared/profile.php` | `src/app/(dashboard)/shared/profile/page.tsx` | Profile edit, avatar upload to Supabase Storage |
| `Shared/report.php` | `src/app/(dashboard)/shared/reports/page.tsx` | Issue submission form |

---

## 5. Claude Skills Workflow & Conflict Prevention

The Next.js repository (`hksps`) uses the **PHINMA AI Builder** skill chain:

```
/grill-with-docs ➔ /pitch ➔ /to-spec ➔ /to-tickets ➔ /implement ➔ /code-review
```

### Why There Is Zero Conflict:
1. **Separation of Directories**:
   - `HKSPS` (PHP repo, read-only reference).
   - `hksps` (Next.js repo, active development target).
2. **Harmonious Skills Integration**:
   - `/grill-with-docs`: Focuses on the core problem (e.g., student scholars tracking manual duty hours and applying for department slots). Generates `docs/PROBLEM.md`.
   - `/pitch`: Summarizes the 3-minute pitch into `docs/PITCH.md`.
   - `/to-spec`: Instructed by `CLAUDE.md` to read `context.md`. It directly incorporates the entities, tables, and roles from `context.md` into `docs/SPEC.md`.
   - `/to-tickets`: Breaks `docs/SPEC.md` into small, vertical-slice tickets under `docs/tickets/` (e.g. `01-live-on-vercel.md`, `02-supabase-connection.md`, `03-student-duty-browser.md`, `04-duty-timer.md`).
   - `/implement`: Reads each ticket and builds the features in `src/` step-by-step.
