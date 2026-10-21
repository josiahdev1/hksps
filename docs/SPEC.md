# Spec: Hawak Kamay Scholarship & Student Portal System (HKSPS)

## Problem
Hawak Kamay college scholars have no central, transparent way to find open faculty duties and prove their rendered service hours. Paper slips are easily lost, coffee-stained, or left unsigned, and scholars only discover at finals week that their hours are missing, placing their tuition scholarship in immediate jeopardy.

## Solution
A responsive mobile-first web app where scholars easily discover and apply for open faculty duties, faculty approve applicants and verify rendered hours in seconds, and scholars track an audit-proof ledger showing exact remaining hours and term completion percentage.

## The user
The Hawak Kamay college scholar who must fulfill mandatory service duty hours each semester to keep their scholarship.

## User stories
1. As a scholar, I want to browse open faculty duties on my mobile phone, so that I can quickly find opportunities that fit my class schedule.
2. As a scholar, I want to apply for an open duty with an optional note, so that the faculty member knows I am available and qualified.
3. As a faculty member, I want to review applicants for my duty and approve them up to the slot limit, so that duties are staffed without overbooking.
4. As a faculty member, I want to log rendered hours for an approved scholar by picking the date, start time, and end time, so that their time is promptly credited.
5. As a scholar, I want to see my credited hours, remaining hours balance, and progress percentage on my dashboard, so that I always know where I stand before deadlines.
6. As a scholarship coordinator, I want to view a real-time compliance table of all scholars, so that I can see who is on track and who needs support.
7. As a scholar, I want my contact details and student ID to remain private from other students, so that my personal data is secure.

## Screens
1. `/` (Public Duty Board): Public duty listings with search and department filtering, showing available slots, estimated hours, and a direct Apply button.
2. `/auth/login`: Email and password login screen with one-click quick demo buttons for Scholar, Faculty, and Admin accounts.
3. `/dashboard/scholar`: Scholar dashboard with term progress bar, required vs. credited hours, application status list, and the verified append-only ledger.
4. `/dashboard/scholar/apply/[id]`: Duty application view to review duty expectations and submit an application with an optional note.
5. `/dashboard/teacher`: Faculty console to manage posted duties, review and approve/reject applicants, and open the quick hour-logging modal.
6. `/dashboard/admin`: Coordinator dashboard showing the live scholar compliance table (`complete`, `on track`, `behind`) and the system audit log.

## Data
1. `departments`: Academic departments (`id`, `code`, `name`). Anyone may read; only an admin may create or update.
2. `profiles`: User information linked to `auth.users` (`id`, `role`, `first_name`, `last_name`, `student_id_number`, `department_id`). Users may read their own profile; admins may read all profiles. Only an admin may change roles or verification status.
3. `academic_terms`: Semesters and academic years (`id`, `academic_year`, `semester`, `starts_on`, `ends_on`). Authenticated users may read; only an admin may create or update.
4. `scholarship_records`: Required service quota per scholar per term (`id`, `user_id`, `term_id`, `required_hours`, `status`). Scholars may read their own record; admins may read and create records. Hour totals are derived via views and never stored directly.
5. `job_postings`: Faculty duties (`id`, `title`, `description`, `department_id`, `posted_by`, `slots`, `estimated_hours`, `duty_date`, `status`). Anyone may read `open` duties; faculty may read, create, and update their own duties; admins may read and update all duties.
6. `applications`: Scholar applications (`id`, `job_id`, `student_id`, `cover_letter`, `status`, `applied_at`, `decided_at`, `decided_by`). Scholars may read their own applications; faculty may read applicants to their own duties; admins may read all. Status changes must pass through database functions enforcing slot caps.
7. `duty_hours`: Append-only hour ledger (`id`, `application_id`, `student_id`, `entry_type`, `duty_date`, `start_time`, `end_time`, `raw_seconds`, `credited_seconds`, `policy_version`, `verified_by`, `notes`, `created_at`). Scholars may read their own ledger rows; faculty may read rows for their duties; admins may read all. Rows are strictly immutable; client `UPDATE` and `DELETE` privileges are denied.
8. `audit_logs`: System mutation trail (`id`, `actor_id`, `action`, `entity`, `entity_id`, `before_state`, `after_state`, `created_at`). Only admins may read; writes occur strictly via database triggers.

## Login
Yes, Supabase Auth using email and password. Pre-seeded demo credentials will be available on the login page for rapid one-click testing:
- Scholar: `scholar@hksps.edu`
- Faculty: `teacher@hksps.edu`
- Admin: `admin@hksps.edu`

## Done for the demo
- [ ] Application deployed to a live Vercel URL connected to hosted Supabase.
- [ ] Public duty board displays active duties with slot counts.
- [ ] Scholar can log in and submit an application for an open duty.
- [ ] Faculty can log in, approve the applicant, and log rendered hours (e.g. 2 hours).
- [ ] Scholar dashboard reflects updated credited hours, remaining balance, and progress percentage.
- [ ] Database prevents overbooking beyond slot limits and blocks unauthorized ledger modifications via RLS.

## Out of scope
- Real-time in-browser stopwatch or GPS tracking (parked in favor of verified faculty logging).
- QR code check-in and location beacons.
- Scholarship renewal requests and eligibility tag workflows.
- PDF certificate generation and ledger CSV/Excel exports.
- Visual charts and complex analytics.
- Email and SMS delivery (in-app notifications only).
- Roster bulk CSV import and multi-campus tenancy.

