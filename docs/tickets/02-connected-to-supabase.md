# 02 — Connected to Supabase

**What to build:** The app connects to a hosted Supabase project. Environment variables are set both in Vercel and locally. The departments and job postings tables exist with Row Level Security enabled, and the public landing page fetches and renders open duty openings with available slot counts.

**Blocked by:** 01

**Status:** todo

- [ ] Supabase project created on Supabase.com
- [ ] NEXT_PUBLIC_SUPABASE_URL and NEXT_PUBLIC_SUPABASE_ANON_KEY configured in local environment and Vercel project settings
- [ ] Departments and job postings tables created with Row Level Security enabled
- [ ] Public RLS policy allows anonymous and authenticated users to read open duties
- [ ] Seed script adds at least three departments and five realistic open duty postings
- [ ] Landing page renders the live duty feed with title, department, estimated hours, and remaining slots
- [ ] Works on the live URL
