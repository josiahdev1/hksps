# HKSPS: Hawak Kamay

**Project context for agents and contributors.** Read this before any task. Deeper detail lives in `DATA_MODEL.md` and `METHODOLOGY.md`; open them only when the task touches them.

## Purpose

Hawak Kamay (HKSPS) connects scholars who must render service hours with faculty who need help on classroom, administrative, and event tasks. Faculty post duties, scholars apply, faculty approve applicants and confirm the hours rendered, and the scholarship office tracks every scholar's progress against the term requirement.

## Roles

| Role | Abilities |
|---|---|
| `public` | Read open duties and the published hour methodology. |
| `scholar` | Browse and apply for duties, track own applications, view own ledger and progress. |
| `teacher` | Post duties, review applicants to own duties, approve or reject, log and reverse hours for own duties. |
| `admin` | Verify users, assign roles, manage terms and scholarship records, publish hour policies, view compliance and the audit log. |
| `pending` | Registered teacher or admin awaiting verification. Public access plus own profile. |
| `auditor` (Later) | Read-only aggregate compliance data and audit log. |

**Account rules**

- Scholars self-register with a student ID number and receive `scholar`.
- Teacher and admin sign-ups receive `pending` until an admin assigns the role. A staff role is effective only while `verified_at` is set.
- Only an admin changes `role`, `verified_at`, or `is_active`. Sign-up never takes a role from client data.
- Deactivated accounts keep their history and lose access.

## Workflow

1. Admin sets up the term, assigns teacher roles, and creates scholarship records with required hours.
2. Teacher posts a duty (`draft` → `open`).
3. Scholar applies once per duty.
4. Teacher approves up to the slot count, or rejects with notes.
5. Teacher logs rendered time against the approved application. The system applies the hour policy and writes a ledger credit.
6. Mistakes are corrected with a reversal entry, then a new credit.
7. Scholar dashboard shows rendered, required, remaining, percent complete, and as-of time.
8. Admin compliance view shows each scholar as complete, on track, or behind.
9. (Next) Scholar requests renewal; admin decides.

## Scope

- **MVP:** auth and verification · public job board · duty create, publish, close, cancel · apply and cancel · approve and reject with slot limits · hour logging and reversal · scholar dashboard and ledger · terms and scholarship records · admin compliance view · user and role management · audit log · in-app notifications · one seeded hour policy.
- **Next:** renewal and eligibility tags · hour policy screen · email notifications · PDF certificates and exports · charts · rate limiting · scheduled contact-number cleanup.
- **Later:** auditor role · QR or location check-in · SMS · multi-campus · roster import.

Build the MVP only unless a ticket says otherwise.

## Business rules

| ID | Rule |
|---|---|
| HK-01 | One application per scholar per duty (`UNIQUE (job_id, student_id)`). |
| HK-02 | Application transitions: `pending` → `approved`, `rejected`, `cancelled`; `approved` → `completed`, `cancelled`. The rest are final. Approval needs a free slot and records `decided_by` and `decided_at`. |
| HK-03 | Duty lifecycle: `draft` → `open` → `closed` or `cancelled`; `draft` → `cancelled`. Only open duties accept applications. |
| HK-04 | Hours are logged only against an approved application, by the posting teacher or an admin. |
| HK-05 | `duty_hours` is append-only. A correction is a reversal row (negative credited seconds, reason required, one reversal per credit), then a new credit if needed. |
| HK-06 | Remaining hours, percent complete, and completeness are derived in views. No stored counters. Every figure carries `as_of`. |
| HK-07 | The hour policy in force is applied when a row is written and its version is stored. New policies never change past rows. |
| HK-08 | An entry counts toward the scholarship record whose term contains `duty_date`. Terms never overlap. |
| HK-09 | Phone numbers need `consent_at`. Public views and JSON carry no personal data. Teachers see only applicants to their own duties; scholars see only their own data. |
| HK-10 | Every change to terms, scholarship records, duties, applications, hours, policies, and role or verification fields writes an audit row. Audit rows are immutable. |
| HK-11 | Only an admin sets `role`, `verified_at`, or `is_active`. |
| HK-12 | (Next) Renewal decisions need an admin and a timestamp. |

## Architecture

**Flow.** A Server Action validates input with Zod, calls a database function with the user's session, the function checks role, ownership, and state, writes the rows, and an audit trigger records the change. The public job board is statically rendered and revalidated when a duty is published, closed, or cancelled. Dashboards render per request.

**Principles**

- Derive, do not store.
- The server is the authority: RLS enforces every read and write. Hiding a button is not authorization.
- Append-only history for hours and audit entries.
- Every hour total carries an as-of time and traces to a ledger row, verifier, and policy version.

**Security rules**

- RLS is enabled on every table. A table without policies is not shipped.
- Views use `security_invoker = true`. Only `v_public_jobs` is granted to `anon`, and it has no personal columns.
- `SECURITY DEFINER` functions set a fixed `search_path`, check role first, revoke `EXECUTE` from `public`, and grant it to `authenticated`.
- Revoke default table privileges from `anon` and `authenticated`, then grant per table. No `UPDATE` or `DELETE` on the ledger or audit log for any client role.
- Profile self-update is limited by column grant to first name, last name, phone, and consent.
- The service role key is server-only and never reaches a browser. Commit environment variable names only.

## Stack and conventions

Next.js App Router · TypeScript (strict) · Tailwind CSS · shadcn/ui · Supabase (Postgres, Auth, RLS) · Zod and React Hook Form · Vercel (every push to main deploys) · GitHub.

- Server Components by default; Client Components only for interaction.
- Generated Supabase types are the only source of row types.
- Every mutation input has a Zod schema shared with its form.
- State changes go through database functions, never direct client table writes.
- Migrations are numbered SQL files in `supabase/migrations`. Never edit an applied migration; add a new one.
- Each function and policy ships with its SQL tests in the same ticket.
- Mobile-first from 375px, WCAG AA contrast, light and dark themes.

## Repository layout

```
hawak-kamay/
├── AGENTS.md
├── middleware.ts
├── docs/            CONTEXT.md  DATA_MODEL.md  METHODOLOGY.md  (SPEC.md, PITCH.md, LATER.md)
├── tickets/         T01 to T14
├── supabase/
│   ├── migrations/  numbered SQL
│   ├── seed.sql     labeled fictional data
│   └── tests/rls_and_views.sql
└── src/
    ├── app/         (public)  (auth)  (dashboard)/{scholar,teacher,admin}  api/
    ├── components/  ui  charts  status  forms
    └── lib/         supabase/  permissions.ts  validators.ts  hours.ts  notify.ts
```

## Reference docs

| Open | When the task involves |
|---|---|
| `DATA_MODEL.md` | Tables, columns, constraints, migrations, RLS policies, views, the access matrix |
| `METHODOLOGY.md` | Hour logging, rounding, caps, reversals, completion math |
