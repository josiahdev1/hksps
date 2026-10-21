# 06 — Scholar Ledger and Compliance

**What to build:** The scholar dashboard calculates and renders their real-time progress toward their term quota (progress bar, credited hours, remaining hours) backed by an immutable ledger of all verified credits. The admin dashboard displays a compliance overview of all scholars (complete, on track, behind) and the mutation audit trail.

**Blocked by:** 05

**Status:** todo

- [ ] Academic terms and scholarship records tables created with RLS
- [ ] Database view v_scholar_hours calculates rendered hours, remaining hours, and completion percentage without storing totals
- [ ] Scholar dashboard displays progress bar, completion percentage, remaining hours, and a full chronological ledger of credits
- [ ] Row Level Security strictly prevents scholars from viewing other scholars' records or modifying their own ledger rows
- [ ] Admin dashboard displays the compliance roster showing each scholar's status (complete, on track, behind)
- [ ] Admin can view the system audit trail recording key application and hour-logging actions
- [ ] Works on the live URL
