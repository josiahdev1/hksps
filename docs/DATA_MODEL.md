# HKSPS Data Model and Access Control

Reference for schema, migrations, RLS, and views. Business rules (HK-01 to HK-12) are in `CONTEXT.md`.

## Tables

| Table | Purpose | Phase |
|---|---|---|
| `departments` | Academic departments (name, code). | MVP |
| `courses` | Courses within a department. | MVP |
| `profiles` | One row per auth user. | MVP |
| `academic_terms` | Academic year and semester with start and end dates. | MVP |
| `scholarship_records` | One scholar's obligation for one term. | MVP |
| `job_postings` | A duty posted by a teacher. | MVP |
| `applications` | A scholar's application to a duty. | MVP |
| `hour_policies` | Versioned counting rules. | MVP (one seeded version) |
| `duty_hours` | Append-only ledger of credits and reversals. | MVP |
| `notifications` | In-app notifications. | MVP |
| `audit_logs` | Append-only mutation trail. | MVP |
| `renewal_requests` | Scholar renewal requests and admin decisions. | Next |
| `eligibility_tags` | Admin eligibility decisions with reasons. | Next |

## Key columns and constraints

**`profiles`.** `id` references `auth.users`. `role` is one of `scholar`, `teacher`, `admin`, `pending`; `requested_role` records what the user asked for. `first_name`, `last_name`, generated `display_name`, `student_id_number` (unique), `employee_id` (unique), `department_id`, `course_id`, `phone`, `consent_at`, `verified_at`, `is_active`. A scholar must have a student ID number. A phone number requires `consent_at`. The sign-up trigger copies names, student ID number, and requested role from sign-up data and sets the role to `scholar` or `pending` only.

**`academic_terms`.** `academic_year`, `semester` (`1st`, `2nd`, `summer`), `starts_on`, `ends_on`. Unique per year and semester; end after start; no overlapping date ranges.

**`scholarship_records`.** `user_id`, `term_id`, `required_hours` (greater than 0), `status` (`pending`, `active`, `completed`, `revoked`), `created_by`. Unique per scholar and term. Status records the office's decision; hour totals are never stored.

**`job_postings`.** `title` (5 to 150 characters), `description` (up to 3000), `department_id`, `posted_by`, `slots` (at least 1), `estimated_hours` (greater than 0), `duty_date`, `status` (`draft`, `open`, `closed`, `cancelled`), `created_at`, `published_at`, `closed_at`. An open duty has `published_at`.

**`applications`.** `job_id`, `student_id`, `cover_letter` (up to 1500), `status` (`pending`, `approved`, `rejected`, `completed`, `cancelled`), `applied_at`, `decided_at`, `decided_by`, `decision_notes`. Unique on `(job_id, student_id)`. An approved or rejected row has `decided_by` and `decided_at`.

**`hour_policies`.** `version` (primary key), `min_duty_minutes` (default 30), `rounding_minutes` (default 15), `cap_per_application_hours` (default 16), `description`, `effective_from`, `created_by`. The policy in force is the latest one whose `effective_from` has passed.

**`duty_hours`.** `application_id`, `student_id` (must match the application's scholar), `entry_type` (`credit`, `reversal`), `reverses_id` (required for a reversal, unique), `duty_date`, `start_time`, `end_time`, `raw_seconds`, `credited_seconds` (positive for a credit, the negative of the reversed credit for a reversal), `policy_version`, `verified_by`, `notes` (up to 500; required for a reversal), `created_at`. End is after start.

**`notifications`.** `user_id`, `type` (`application_received`, `application_decided`, `duty_hours_logged`, `system`), `title`, `message`, `link_url`, `is_read`, `created_at`.

**`audit_logs`.** `id`, `actor_id`, `actor_role`, `action`, `entity`, `entity_id`, `before_state`, `after_state`, `created_at`.

## Derived views

| View | Contents | Access |
|---|---|---|
| `v_public_jobs` | Open duties with department name, slots, estimated hours, date, and approved count. No personal data. | `anon`, `authenticated` |
| `v_job_applicant_counts` | Pending, approved, and rejected counts per duty. | Teacher (own duties), admin |
| `v_scholar_hours` | One row per scholarship record, including records with no hours: required hours, rendered seconds (sum of `credited_seconds` for entries whose `duty_date` falls in the term), remaining hours, completion percent, `is_complete`, `last_entry_at`, and `as_of`. | Scholar (own rows), admin |
| `v_admin_compliance` | `v_scholar_hours` joined with name, student ID number, department, and course. | Admin |

All views except `v_public_jobs` use `security_invoker = true`.

## Database functions

| Function | Caller | Effect |
|---|---|---|
| `app_role()` | Policies | Returns `scholar`, `teacher`, `admin`, or `pending`; downgrades staff without `verified_at` to `pending`. |
| `apply_for_job(job_id, cover_letter)` | Scholar | Inserts a `pending` application and notifies the teacher. |
| `cancel_application(application_id)` | Scholar (own) | Moves a pending or approved application to `cancelled`. |
| `decide_application(application_id, decision, notes)` | Teacher (own duty), admin | Approves within slots or rejects; records decider and time; notifies the scholar. |
| `log_duty_hours(application_id, duty_date, start_time, end_time, notes)` | Teacher (own duty), admin | Applies the policy in force and inserts a credit. |
| `reverse_duty_hours(duty_hours_id, reason)` | Teacher (own duty), admin | Inserts a reversal row. |
| `admin_set_role(user_id, role)` | Admin | Sets role and `verified_at`. |

Error codes: `42501` not permitted, `P0002` not found or not open, `23505` duplicate, `22023` invalid input or policy violation.

## Access matrix

| Resource | public | scholar | teacher | admin |
|---|---|---|---|---|
| `v_public_jobs` | Read | Read | Read | Read |
| `departments`, `courses`, `academic_terms`, `hour_policies` | Read | Read | Read | Read, create and update (policies: create only) |
| `profiles` | None | Own row: read; update names, phone, consent | Own row: same | Read all; set role, verification, active |
| `job_postings` | Open duties only | Open duties only | Own duties: all statuses, create, update. Others: open only | All |
| `applications` | None | Own: read; create and cancel through functions | Applicants to own duties: read; decide through function | Read all; decide |
| `duty_hours` | None | Own: read | Own duties: read; create through functions | Read all; create |
| `scholarship_records` | None | Own: read | None | All |
| `notifications` | None | Own | Own | Own |
| `audit_logs` | None | None | None | Read |

**Policy notes**

- Every table has explicit policies per role and operation. No policy means denial.
- Teacher policies check ownership through `job_postings.posted_by = auth.uid()` and require an effective teacher role.
- Clients have no direct `INSERT`, `UPDATE`, or `DELETE` on `applications` and `duty_hours`. The functions perform those writes after their own checks.
- `duty_hours` and `audit_logs` grant no `UPDATE` or `DELETE` to any client role, including admin.

