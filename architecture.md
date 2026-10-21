# HKSPS Master Architecture & System Design Specification
## 1,000,000 Users Scale • Vercel Native • Hosted Supabase • RA 10173 DPA Compliant

> **Required Reading Order**:
> 1. `CLAUDE.md` — Workshop rules, security baseline, and skill workflow.
> 2. `context.md` — Domain logic, legacy PHP (`HKSPS`) parity mapping, and PostgreSQL schema.
> 3. `architecture.md` (This document) — 1M-user system design, 60 engineering skills, concurrency burst shield, and team ownership.
>
> **100% Self-Contained Project Notice**:
> This repository (`hksps`) is fully self-contained. All legacy PHP workflows, tables, and rules are already captured in `context.md`.
> Developers and AI agents must focus exclusively on this Next.js + Supabase repository. No external PHP folder is required or should be searched for.

---

## 1. Engineering Team Ownership & Workstreams

| Team Member | Engineering Role | Core Responsibilities & Workstream |
|---|---|---|
| **Neil** | **Lead UI/UX & Frontend Engineer** | • Design tokens, semantic themes, responsive layouts (375px mobile to 1440px desktop).<br>• Accessible UI primitives in `src/components/ui/` (`Button`, `Card`, `Input`, `Label`, `Textarea`).<br>• Client interactions: instant button lock via `useActionState`, skeleton loaders, modal dialogues.<br>• Clean visual hierarchy for Student, Teacher, and Admin dashboards. |
| **Josiah** | **Lead Backend & Systems Architect** | • Database modeling, PostgreSQL schema, RLS policies in `supabase/sql/`.<br>• Next.js Server Actions in `src/server/` with strict **Zod** schema validations.<br>• Tamper-proof server-side duty stopwatch timer & hour calculation engine.<br>• Supabase Auth SSR session handling, Supabase Storage buckets, and Realtime channels. |

---

## 2. Part 1: 1M Users System Design (Vercel Native + Hosted Supabase)

Classic system-design components and the Vercel + Hosted Supabase native primitives that run each one:

```
[Users: Students, Teachers, Admins, Visitors]
                      │
                      ▼ (1. HTTPS Request)
     ┌──────────────────────────────────┐
     │      Vercel Edge Network         │ <── (Public Cache Hit / CDN)
     └──────────────────────────────────┘
                      │
                      ▼ (2. TLS 1.3 Secure API Gateway)
     ┌──────────────────────────────────┐
     │   Vercel Firewall / Edge WAF     │ <── (3. Rate Limiter / Flood Filter - HTTP 429)
     └──────────────────────────────────┘
                      │
                      ▼ (4. Serverless Load Balancing)
     ┌──────────────────────────────────┐
     │   Next.js 16 App Serverless      │ <── (5. Zod Validation, Auth & Session Guard)
     │          (Edge/Node)             │
     └──────────────────────────────────┘
                      │
                      ▼ (HTTPS PostgREST / Port 6543)
     ┌──────────────────────────────────┐
     │  Supavisor Connection Pooler     │ <── (6. Shared Transaction Connection Pool)
     └──────────────────────────────────┘
                      │
                      ▼
     ┌──────────────────────────────────┐
     │   Hosted PostgreSQL 15+ & RLS    │ <── (ACID, Atomic Row Locks, Triggers)
     ├──────────────────────────────────┤
     │  • Supabase Auth (HttpOnly)      │
     │  • Supabase Storage (Private)    │
     │  • Supabase Realtime (WAL CDC)   │
     │  • pg_cron Scheduled Tasks       │
     └──────────────────────────────────┘
```

### ★ How One Request Travels (Steps 1 to 6)

1. **Step 1: Public Cache Hit (CDN)**
   User opens the portal; public landing pages (`/`), department badges, and static marketing assets come instantly from the **Vercel Global Edge Network** without hitting origin compute.
2. **Step 2: Secure API Gateway**
   Dynamic & authenticated requests travel via TLS 1.3 HTTPS to the Next.js App Router gateway with server-side cookie verification.
3. **Step 3: Rate Limiting & Edge WAF Protection**
   Vercel Firewall and Edge WAF rate limiters filter IP flood bursts, DDoS attacks, and unauthorized bots (returning `HTTP 429 Too Many Requests`).
4. **Step 4: Serverless Load Balancing**
   Vercel Edge request routing distributes traffic seamlessly across available Next.js Serverless Function compute instances.
5. **Step 5: Application Compute, Auth Guard & Zod**
   Server Action validates HttpOnly session cookies via `@supabase/ssr`, executes strict **Zod** schema parsing, prevents invalid state transitions, and opens a pooled database transaction.
6. **Step 6: Connection Pool & RLS Isolation**
   **Supavisor** shares pooled database connections (Port 6543). PostgreSQL executes the query with **Row Level Security (RLS)** returning only authorized rows strictly per tenant role (`student`, `teacher`, `admin`).

---

## 3. Part 2: 5-Tier Canonical Architecture & Roles

### 3.1 The 4 Canonical Roles (PHINMA UI Adaptation)

1. **Public Visitor / Prospective Scholar**:
   - Landing page (`/`), scholarship eligibility guidelines, list of university departments (`CITE`, `CEA`, `CMA`, `CAHS`, etc.).
   - Privacy Notice under **RA 10173 (Philippine Data Privacy Act)** and Terms of Service.
   - Zero Student PII exposed to public visitors.
2. **Student Scholar (`/student/*`)**:
   - Browse open scholarship duties with live slot counters.
   - Submit applications with cover letters (prevents duplicate applications via unique constraint).
   - Tamper-proof duty tracking: Clock in/out with server-calculated hours.
   - History of rendered duty hours, progress bar towards required semester hours (e.g. 50 / 90 hours).
   - Report submission for unsafe tasks or duty issues.
3. **Teacher / Facilitator (`/teacher/*`)**:
   - Post, edit, close, and manage duty slots (with schedule, location, deadline, max applicants cap).
   - Review applicants: Approve or reject student applications.
   - Real-time monitor of student facilitators currently clocked in.
   - Verify completed duty sessions and rendered hours.
4. **Admin / Scholarship Coordinator (`/admin/*`)**:
   - Institutional governance & department management.
   - User directory: student ID verification, teacher assignment, role management.
   - Resolution and triage of reported duty incidents.
   - Global audit activity logs and hours export.
   - Feature flags (`reports_enabled`, `job_applications_enabled`).

---

## 4. Part 3: High-Concurrency Burst Shield (Thousands of Simultaneous Clicks)

During peak scholarship application deadlines or duty clock-ins, thousands of students click simultaneously. The system uses a **6-Layer Concurrency Burst Shield**:

```
[Layer 1: Client Lock]       Button disables immediately via useActionState + Optimistic UI
             │
[Layer 2: Edge Guard]        Vercel WAF IP Rate Limiter blocks spam (HTTP 429)
             │
[Layer 3: Compute Shield]    Server Action parses payload via strict Zod schema + Idempotency check
             │
[Layer 4: Pool Queue]        Supavisor Connection Pooler absorbs connection spikes (Port 6543)
             │
[Layer 5: Database Lock]     PostgreSQL SELECT FOR UPDATE locks row atomically; caps max_applicants
             │
[Layer 6: Realtime Delta]    Supabase Realtime emits sub-50ms delta to listeners; zero manual refresh
```

1. **Layer 1: Client Instant Lock (Neil)**: Form buttons immediately disable upon submission using React 19 `useActionState` and optimistic indicators, preventing double-clicks.
2. **Layer 2: Edge Guard**: Vercel Edge Firewall drops anomalous traffic bursts and bot floods.
3. **Layer 3: Compute Shield (Josiah)**: Server Actions validate user authentication, verify roles, and validate inputs via Zod.
4. **Layer 4: Supavisor Queue**: Supavisor manages transaction pooling over port 6543, avoiding PostgreSQL connection exhaustion.
5. **Layer 5: Atomic Database Row Lock**: PostgreSQL handles slot decrements atomically using `SELECT ... FOR UPDATE` or SQL constraints (`check (current_applicants <= max_applicants)`).
6. **Layer 6: Realtime Delta Broadcast**: Once an application or duty status updates, Supabase Realtime broadcasts changes without requiring full-page re-fetching.

---

## 5. Part 4: The 60 Applied Engineering Skills (7 Pillars)

### Pillar 1: System Design & Scalability (12 Skills)
1. 1M-User Capacity Planning
2. Horizontal Serverless Autoscaling (Vercel)
3. Stateless Next.js Compute (No local disk persistence)
4. Distributed Multi-Region Edge Caching
5. Incremental Static Regeneration (ISR) for public directories
6. Dynamic Edge Stale-While-Revalidate
7. Read/Write Query Separation
8. Supavisor Connection Pooling (Port 6543)
9. Non-Blocking Async Database Triggers
10. Graceful Degradation Fallbacks (Empty states, error boundaries)
11. Circuit Breakers on Storage & External calls
12. Multi-Tenant Role Isolation

### Pillar 2: Frontend & Next.js 16 (10 Skills) — *Lead: Neil*
13. Next.js App Router Architecture
14. React 19 Server Components (RSC)
15. Server Actions with `useActionState`
16. Optimistic UI State with `useOptimistic`
17. Zero-Bundle-Size Server Data Fetching
18. Streaming SSR & Suspense Boundaries
19. `shadcn/ui` Accessible Primitives (`Button`, `Card`, `Input`, `Label`, `Textarea`)
20. Tailwind CSS v4 Design Tokens (Responsive 375px to 1440px)
21. Image Optimization via Next/Image (WebP/AVIF)
22. Core Web Vitals Optimization (INP/LCP/CLS)

### Pillar 3: Database & PostgreSQL (10 Skills) — *Lead: Josiah*
23. B-Tree Composite Indexing (`department_id, status`, `user_id, duty_date`)
24. PostgreSQL Full-Text Search on Duty Titles and Descriptions
25. BRIN Indexing for Audit Timestamps (`activities.created_at`)
26. Atomic Transactions (`BEGIN / COMMIT`)
27. Row-Level Locking (`FOR UPDATE`) for duty slot claims
28. Atomic Stored Procedures & Triggers (`sync_applicant_count`, `handle_new_user`)
29. Numbered SQL Migrations (`supabase/sql/001-*.sql`, `002-*.sql`)
30. EXPLAIN ANALYZE Query Profiling
31. Foreign Key CASCADE Integrity
32. Generated UUIDv4 Primary Keys (`gen_random_uuid()`)

### Pillar 4: Security & RA 10173 DPA Shield (9 Skills)
33. OWASP Top 10 Full Hardening
34. IDOR / BOLA Prevention via PostgreSQL Row Level Security (RLS)
35. Next.js SameSite CSRF Shield
36. React 19 JSX Native XSS Defense
37. Parameterized Queries (PostgREST / Supabase JS)
38. TypeScript Strict Type Safety
39. HttpOnly Secure Session Cookies (`@supabase/ssr`)
40. Zod Strict Schema Sanitization on all mutations
41. Philippine Data Privacy Act (RA 10173) Compliance:
   - Student contact numbers and PII hidden from unapproved peers and public.
   - Incident report attachments stored in private buckets with signed URLs.

### Pillar 5: High-Concurrency Burst Shield (7 Skills)
42. Frontend Button Debounce & Instant Lock
43. Client Idempotency Key UUID Generation
44. Edge Middleware IP Rate Limiting (HTTP 429)
45. Database Unique Idempotency Constraint (`unique (job_id, student_id)`)
46. Selective Row Locking (`FOR UPDATE`) on slot capacity
47. Supabase Realtime CDC / Broadcast Channels
48. Zero-Downtime Peak Traffic Handling

### Pillar 6: Vercel DevOps & Cloud (6 Skills)
49. CI/CD Automated Pipelines (GitHub to Vercel on `main` push)
50. Automated Preview Deployments per Pull Request
51. Production Instant Deploy on Push
52. Environment Secret Vault Hygiene (`.env.local` never committed)
53. Global Edge Network & CDN Delivery
54. Automated SSL/HTTPS Certificate Renewal

### Pillar 7: Software Architecture & Clean Code (6 Skills)
55. SOLID Design Principles
56. Clean Architecture Module Boundaries (`src/app/`, `src/components/`, `src/server/`, `src/lib/`)
57. Domain-Driven Separation of Concerns
58. Self-Documenting Code Standards
59. 100% Legacy PHP Parity Discipline
60. Zero Technical Debt Migration Policy

---

## 6. Real-Time Engine (Hosted Supabase Realtime Only)

As required by the PHINMA AI Builder standard:
* **Zero custom Node.js WebSocket daemons** run on Vercel. Next.js runs completely serverless and stateless.
* All live updates (live attendance check-in, applicant approval notifications, duty slot updates) connect directly from client browsers to **Hosted Supabase Realtime Channels** (`postgres_changes` and `broadcast`).

---

## 7. Skill Execution & Team Alignment

When running skills in `.claude/skills/`:
1. `/grill-with-docs` — Formulate the core student scholar struggle (tracking manual duty hours and applying for department slots). Writes `docs/PROBLEM.md`.
2. `/pitch` — Script the 3-minute pitch for Session 1 in `docs/PITCH.md`.
3. `/to-spec` — Synthesize `docs/SPEC.md` using the exact entities, roles, and screens defined in `context.md` and `architecture.md`.
4. `/to-tickets` — Break the spec into vertical slice tickets in `docs/tickets/` with clear UI (Neil) and Backend (Josiah) ownership.
5. `/implement` — Implement ticket by ticket within the pinned stack.
6. `/code-review` — Validate security, RLS, and DPA compliance before demo.

---

## 8. Cross-Platform Team Shield (Windows vs Linux & Vercel Prevention)

Because teammates work across different operating systems (**Neil on Windows/macOS**, **Josiah on Linux**, and **Vercel on Linux**), we enforce these strict cross-platform rules:

### 8.1 Avoid Directory Collision on Windows/macOS
* **The Issue**: Windows (NTFS) and macOS (APFS) are **case-insensitive**. Having `HKSPS` (PHP) and `hksps` (Next.js) in the same parent directory will cause Windows to merge or refuse to clone them.
* **The Solution**: 
  - If a team member clones both repos locally, rename the old PHP folder:
    - Legacy PHP: `HKSPS-legacy` (or `hksps-php`)
    - Modern Next.js: `hksps`
  - Teammates working solely on UI/UX (Neil) only need to clone `https://github.com/josiahdev1/hksps.git` (`hksps`).

### 8.2 Strict Lowercase Casing for All Next.js Paths
* **The Issue (The Silent Vercel Build Killer)**: On Windows, `import { Button } from "@/components/UI/button"` works locally, but when pushed to GitHub and built on Vercel (Linux), Vercel crashes with `Module not found: Can't resolve '@/components/UI/button'`.
* **The Solution**:
  - All directory names inside `src/` **MUST BE 100% LOWERCASE**: `src/components/ui/`, `src/app/`, `src/lib/`, `src/server/`.
  - Always verify casing in TypeScript import statements matches exact filesystem casing.

### 8.3 Line Ending Normalization (`.gitattributes`)
* **The Issue**: Windows uses CRLF (`\r\n`), Linux uses LF (`\n`), causing phantom git diffs and ESLint linebreak errors.
* **The Solution**: Repository contains a root `.gitattributes` file enforcing `* text=auto eol=lf`.

### 8.4 Safe Shell Commands
* Avoid Linux-only shell pipelines (`grep`, `sed`, `awk`, `cat <<EOF`).
* Always provide commands that run identically in **PowerShell, Git Bash, and npm scripts** (e.g. `npm run dev`, `npm run build`).
