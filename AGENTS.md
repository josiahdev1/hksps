# AGENTS.md

Hawak Kamay (HKSPS): scholars apply for faculty-posted service duties; faculty confirm hours; the scholarship office tracks compliance. Next.js + TypeScript + Supabase.

## Read first

- Before any task, read `docs/CONTEXT.md`.
- Read `docs/DATA_MODEL.md` only for schema, migrations, RLS, or views.
- Read `docs/METHODOLOGY.md` only for hour logging or hour math.
- Work on one ticket from `tickets/` at a time.

## Hard rules

- Build the MVP only unless the ticket says otherwise.
- Never store derived totals. Remaining hours and completion come from views.
- `duty_hours` and `audit_logs` are append-only. Corrections are new rows.
- State changes go through database functions, never direct client table writes.
- Every table has RLS. Views use `security_invoker = true`.
- Never edit an applied migration. Add a new one.
- The service role key is server-only. Commit environment variable names only.
- Validate every input with Zod.
