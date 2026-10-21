# 03 — Authentication and Roles

**What to build:** Users can sign in using email and password via Supabase Auth. A profiles table stores role information (scholar, teacher, admin). The login page provides quick one-click buttons for pre-seeded demo accounts and redirects each user to their specific role dashboard.

**Blocked by:** 02

**Status:** todo

- [ ] Profiles table created linked to auth.users with role constraint (scholar, teacher, admin)
- [ ] Row Level Security restricts users to viewing their own profile and admins to viewing all
- [ ] Three demo accounts created in Auth and profiles (scholar@hksps.edu, teacher@hksps.edu, admin@hksps.edu)
- [ ] Login screen (/auth/login) features fast one-click demo credentials for easy judging
- [ ] Successful login redirects scholar to /dashboard/scholar, teacher to /dashboard/teacher, and admin to /dashboard/admin
- [ ] Navigation header displays logged-in user name, role badge, and working sign-out button
- [ ] Works on the live URL
