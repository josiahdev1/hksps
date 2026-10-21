# 04 — Scholar Application Flow

**What to build:** An authenticated scholar can view details of an open duty, click Apply to submit an application with an optional note, and see the submitted application listed with pending status on their scholar dashboard. Overbooking prevention ensures a scholar can only apply once per duty.

**Blocked by:** 03

**Status:** todo

- [ ] Applications table created with UNIQUE (job_id, student_id) constraint
- [ ] RLS policies allow scholars to view only their own applications and faculty to view applicants to their duties
- [ ] Scholar can click an open duty to view details and open the application form
- [ ] Submitting the form writes a pending application row and displays an instant success toast
- [ ] System blocks duplicate applications to the same duty with a clear user alert
- [ ] Scholar dashboard displays a list of active applications with their current statuses (pending, approved, rejected)
- [ ] Works on the live URL
