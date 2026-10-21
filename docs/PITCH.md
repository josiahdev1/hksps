# Hawak Kamay (HKSPS) — 3-Minute Pitch

## Spoken Script (~370 words / ~3 minutes)

**[Neil — The Person — 0:00 to 0:45]**
Last semester, during final exam week, I saw my friend and fellow Hawak Kamay scholar running between department buildings with a damp paper slip in his hand. He was trying to track down a professor who had already left for the day. He needed two more signatures to prove he had completed his thirty required service hours. If he didn't hand that slip to the scholarship office by 5:00 PM, his tuition discount for the next semester was gone.

**[Josiah — Today — 0:45 to 1:15]**
Right now, hundreds of scholars rely on paper slips and messy group chats to find duties and record their time. Slips get crumpled, coffee spills on them, and faculty forget to sign. Worst of all, students have no way to verify their total approved hours until finals week, when it's already too late to fix a missing credit.

**[Neil — Our Fix — 1:15 to 2:00]**
We are building Hawak Kamay Portal, a mobile web app that connects scholars directly to faculty duties and keeps an instant, tamper-proof record of their service hours. A student opens the app on their phone, sees open duty slots across campus, and taps to apply. Once the duty is done, the faculty member confirms the hours with two taps, and the student's dashboard immediately updates their progress bar and remaining hours.

**[Josiah — The Demo Moment — 2:00 to 2:30]**
In Session 3, we will show this live on a phone in three minutes. We will pick an open two-hour library duty, apply as a student, sign in as the faculty member to approve the hours, and watch the student's dashboard jump to seventy-five percent completion. Then, we will prove that not even the student can tamper with or edit that verified ledger.

**[Neil — How We'll Know — 2:30 to 3:00]**
Before Session 3, we will put this live Vercel app into the hands of two actual Hawak Kamay scholars and one faculty coordinator on our campus. We will ask them to post, claim, and sign off on a real campus duty. If they can complete that whole cycle in under two minutes without asking us for help, we know it works.

---

## Pressure-Test: The 3 Hardest Questions

### 1. Faculty Adoption (Attacking our weakest point)
**Question**: "Why would a busy professor bother logging into a website to type in hours when they can just scribble their signature on a student's paper slip?"
**Honest Answer**: Faculty hate the paper system too because every finals week, desperate students knock on their doors asking them to re-sign lost slips from two months ago. In our app, confirming hours takes five seconds right from their phone as soon as the task finishes. Still, faculty adoption is our highest hurdle. That's why we designed the teacher interface to require zero training: one simple screen, pick the applicant, enter start and end time, and tap confirm.

### 2. Why Not a Google Sheet?
**Question**: "Why build custom software instead of setting up a shared Google Sheet or Google Form?"
**Honest Answer**: A shared spreadsheet with hundreds of students breaks down quickly. Anyone can accidentally delete rows, alter hours, or view other students' private phone numbers and student IDs. Furthermore, a spreadsheet cannot enforce slot limits: if a lab duty has two openings, forty students will paste their names in. Our database enforces strict slot limits and Row Level Security so scholars can only see and touch their own records.

### 3. Human Error & Disputes
**Question**: "What happens when a faculty member enters the wrong student or logs four hours instead of two?"
**Honest Answer**: We use an append-only accounting ledger. No one—not even an administrator—can silently overwrite or delete an hour record. Instead, the faculty member submits a reversal entry with an explanation, which subtracts the incorrect time, and logs the correct entry. This ensures the scholarship office always has a transparent, dispute-proof audit trail.

