# 05 — Teacher Review and Logging

**What to build:** A faculty member can log in to their dashboard, view all applicants for their duties, approve applicants up to the slot limit (or reject them), and record verified rendered service hours (date, start time, end time, notes) into an append-only ledger.

**Blocked by:** 04

**Status:** todo

- [ ] Teacher dashboard lists duties posted by the logged-in faculty member
- [ ] Teacher can view pending applicants for each duty and click Approve or Reject
- [ ] System prevents approving more applicants than the duty's available slot capacity
- [ ] Duty hours table created as an append-only ledger with no client UPDATE or DELETE privileges
- [ ] Teacher can open an hour-logging modal for an approved applicant, input date, start time, end time, and optional notes
- [ ] Server action validates time range (minimum 30 minutes) and records verified credited seconds
- [ ] Works on the live URL
