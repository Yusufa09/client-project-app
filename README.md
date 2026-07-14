# Activa — *Be active always*

A gym fitness-challenge web app. Gym members earn points for completing healthy-habit goals and compete in balanced teams; gym administrators run time-boxed competitions, define goals, and track standings — all scoped per gym so a single deployment serves many gyms.

---

## Features

### For members
- **Passwordless-feeling sign-in** — join a gym with a name, password, and gym code (or by scanning the gym's QR code, which pre-fills the code).
- **Auto-balanced teams** — new members are assigned to the smallest team, so teams stay even even when people join mid-competition.
- **Goals** — complete gym-defined challenges for points. Goals can require multiple completions (e.g. "go to the gym 3×"), reset daily or weekly, and be limited to a date window.
- **Personal goals** — set your own private goals for the competition, visible only to you.
- **Body scans** — when enabled by the gym, log body-fat / muscle-mass / weight over time, see first-vs-latest trends, and earn points for your first scan.
- **Live leaderboard** — real-time team standings with animated progress bars.
- **Dark mode** — toggle in the nav; preference is remembered. (The sign-in screen is always light.)

### For gym administrators
- **Self-service signup** — register a gym at `/admin/signup`; a unique gym code is generated automatically.
- **Competitions** — create time-boxed competitions with a set of teams. Only one competition runs at a time.
- **Goals management** — create standard goals with points, target counts, recurrence, and date windows.
- **Body-scan settings** — enable per competition, pick which metrics to track, set first-scan and team-winner point awards, and declare the winning team.
- **Teams & members** — view rosters, search members, and see per-team standings; a single gym-wide QR code lives on the Teams page.
- **Admin invites** — invite co-administrators by email (delivered via Resend).
- **History** — review past competitions and their final standings.

---

## Tech stack

| Area | Choice |
|------|--------|
| Framework | [Next.js 14](https://nextjs.org) (App Router) + TypeScript |
| Backend / DB | [Supabase](https://supabase.com) (PostgreSQL, Auth, real-time) |
| Styling | Tailwind CSS + [shadcn/ui](https://ui.shadcn.com) (Base UI primitives) |
| Animation | framer-motion, canvas-confetti |
| Auth | Supabase Auth (admins) · bcrypt password hashes (members) |
| Email | [Resend](https://resend.com) (admin invites) |
| Misc | qrcode.react, react-hook-form + zod |

---


## Project structure

```
app/
  page.tsx            # Member sign-in ("Activa")
  dashboard/          # Member home: goals, points, mini-leaderboard
  goals/              # Personal goals management
  body-scan/          # Body-scan logging & trends
  leaderboard/        # Full real-time team leaderboard
  join/[code]/        # QR-code gym-join landing page
  admin/              # Admin console (competitions, goals, teams,
                      #   body-scans, admins, history, signup)
  api/                # Route handlers (member + admin + body-scan APIs)
components/
  dashboard/  leaderboard/  join/  admin/  ui/   # Feature + shadcn components
  ThemeProvider.tsx   # Dark-mode context (forces light on pre-auth routes)
  MemberNav.tsx       # Member nav bar + theme toggle + sign-out
hooks/                # useMemberSession, useLeaderboard, useGoals, ...
lib/
  enrollment.ts       # Balanced team assignment + member-state resolution
  points.ts           # Period keys, team colors, totals, fitness tips
  admin-auth.ts       # Per-gym admin context
  supabase/           # Server/admin Supabase clients
supabase/             # SQL schema + migrations
middleware.ts         # Protects /admin/* routes
```

---

## How it works

- **Multi-gym by design.** Every competition carries a `gym_id`; teams, goals, and enrollments inherit it via their competition. Admins are scoped to one gym via `getAdminContext()`.
- **Members** authenticate with a bcrypt-hashed password; their session is a `device_token` stored in `localStorage`. A returning login matches globally by name + password.
- **Admins** authenticate via Supabase Auth (email/password). `middleware.ts` guards `/admin/*` except the public login, signup, and accept-invite pages.
- **One active competition per gym**, enforced at the API layer. Expired competitions are auto-ended lazily whenever competition data is read (no cron needed).
- **Points** are awarded by a database trigger on `goal_logs` (the single source of truth); durable team awards (e.g. body-scan winner) live in `teams.bonus_points`, so `teamTotal = total_points + bonus_points`.

---

*Built with Next.js and Supabase.*
