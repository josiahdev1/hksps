# Hawak Kamay Scholarship & Student Portal System (HKSPS)

## Who
The Hawak Kamay college scholar who must fulfill mandatory service duty hours each semester to maintain their tuition scholarship.

## The struggle
Scholars struggle to discover available duty openings across academic departments and have no reliable way to verify their rendered hours. Paper slips get lost or damaged, faculty forget to submit physical signatures to the scholarship office, and scholars only discover at midterms or finals that their hours were never credited, putting their scholarship in immediate jeopardy.

## Today's workaround
Scholars rely on disorganized Messenger group chats to find openings, carry physical paper log sheets to get manual signatures, and repeatedly line up at the scholarship office to manually verify their accumulated hours. This costs hours of anxiety, wasted commute and queuing time, and frequent disputes over lost records.

## The MVP
A mobile-first web app where scholars instantly discover and apply for open faculty duties, and monitor a tamper-proof, real-time ledger of their approved hours and progress toward term compliance.

## How we'll know it worked
A 3-minute live Vercel demo showing:
1. A scholar browsing open duties and applying on mobile.
2. A faculty member approving the application and confirming 2 rendered duty hours.
3. The scholar's dashboard immediately updating to show credited hours, remaining balance, and completion percentage.
4. Tamper resistance verified: scholars cannot modify the ledger or see others' data (enforced by Supabase RLS).
Tested prior to Session 3 by at least two active HK scholars and one faculty member on their actual smartphones.

## Out of scope
- Real-time stopwatch / background clock-in timer (parked in favor of teacher-confirmed logging).
- QR code, geolocation, or physical check-in beacons.
- Scholarship renewal applications and eligibility tagging.
- PDF certificate generation and ledger CSV/Excel exports.
- Visual chart breakdowns and complex analytics.
- Email and SMS alerts (in-app notifications only).
- Roster bulk CSV import and multi-campus tenancy.

## Glossary
- **Scholar**: An enrolled student under the Hawak Kamay scholarship program with a required service hour quota for the academic term.
- **Duty**: A faculty-posted service task with a department, defined date, estimated hours, and a strict slot limit.
- **Application**: A scholar's formal request to render service for a specific duty, undergoing transition from `pending` to `approved` or `rejected`.
- **Rendered Hours**: Confirmed time served by an approved scholar, calculated by policy rules (minimum 30 minutes, 15-minute rounding, 16-hour cap) and stored in an append-only ledger (`duty_hours`).
- **Ledger**: The append-only, immutable record of credited duty hours and signed reversals.
- **Hour Policy**: The versioned system rule that defines rounding intervals, minimum duty duration, and maximum per-application caps.

