# Gymora Frontend

A real React app (Vite + React Router) for the Gymora gym management system. It is
**not a visual mockup** — every screen calls your Express API over `fetch`:

- `Login` → `POST /api/v1/auth/login`
- session check on load → `GET /api/v1/auth/me`
- each dashboard → `GET /api/v1/{role}/dashboard`

The five roles from your `profiles.role` enum (`owner`, `admin`, `reception`, `trainer`,
`member`) each get their own route and dashboard component, gated by the role in the
logged-in profile — mirroring the `authenticate()` + `authorize(...)` pattern in your
backend middleware.

## 1. Point it at your backend

```bash
cp .env.example .env
```

Edit `.env` if your Express server isn't on the default:

```
VITE_API_URL=http://localhost:5000/api/v1
```

## 2. Install and run

```bash
npm install
npm run dev
```

This starts the frontend on `http://localhost:5173`. Make sure:

- Your Express backend is running (`npm run dev` in `backend/`).
- `FRONTEND_URL` in `backend/.env` is set to `http://localhost:5173` so CORS allows it.
- You've created at least one test user in Supabase Auth **and** a matching row in
  `profiles` with a matching `id`, per your backend setup guide.

## 3. Log in

Use one of your test accounts (e.g. `admin@gymora.test`). On success you're redirected
to `/owner`, `/admin`, `/reception`, `/trainer`, or `/member` based on `profile.role`.
The session (`access_token`, `user`, `profile`) is kept in `localStorage` and re-verified
against `GET /auth/me` on every page load — an expired or tampered token bounces you back
to `/login` instead of showing stale data.

## What each dashboard does right now

Every dashboard:

1. Calls `GET /api/v1/{role}/dashboard` with the stored bearer token.
2. If the response has a `dashboard: { ... }` object (like your sample
   `adminController.js`), renders each field as a stat block automatically — no
   hardcoding, so it adapts as you add fields.
3. Always shows the raw JSON response underneath, so you can see exactly what the
   backend sent while you're building out the real controller logic.
4. Lists the planned REST endpoints for that role (from your route plan) in a grid, so
   the sidebar reflects where the build is headed. Only "Dashboard" is a live route today
   — the rest are placeholders until you add the matching backend route and a page here.

If a role's route isn't implemented on the backend yet (e.g. you've only built
`admin/dashboard` so far), the other dashboards will show a clear "Couldn't load this
dashboard" panel with the HTTP status, instead of failing silently.

## Project layout

```
src/
├── api/client.js            fetch wrapper: auth header, error handling, base URL
├── context/AuthContext.jsx  session state, login/logout, /auth/me check on load
├── components/
│   ├── ProtectedRoute.jsx   redirects to /login or to the user's own role route
│   ├── Sidebar.jsx          per-role nav (from roleConfig.js)
│   └── DashboardShell.jsx   shared dashboard chrome + data fetch, used by all 5 pages
├── pages/
│   ├── Login.jsx
│   └── dashboards/
│       ├── OwnerDashboard.jsx
│       ├── AdminDashboard.jsx
│       ├── ReceptionDashboard.jsx
│       ├── TrainerDashboard.jsx
│       └── MemberDashboard.jsx
├── roleConfig.js            nav labels + planned endpoint list per role
└── styles/                  tokens.css (palette/type) + global.css (layout)
```

## Extending a dashboard

To turn a placeholder nav item (say, Admin → "Members") into a real page:

1. Add the backend route (`GET /api/v1/admin/members`) per your existing pattern.
2. Add a fetch call in a new `AdminMembers.jsx` page, similar to `DashboardShell.jsx`.
3. Add a route for it in `App.jsx` and wire the matching `Sidebar.jsx` button to
   navigate there instead of being disabled.

## Next: connect the recommendation engine

Once the five dashboards are pulling real data, the natural next step (per your own
build order) is wiring the trainer's "Recommendations" panel to your Python
recommendation engine output via `POST /trainer/recommendations/:id/approve`.
