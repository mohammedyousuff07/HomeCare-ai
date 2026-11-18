# HomeCare AI project file guide

This guide describes the existing folders. It does not move files or change how the project runs.

## Frontend — `src/`

The React application shown in the browser.

- `src/main.jsx` — starts the frontend application.
- `src/App.jsx` — application routes and page composition.
- `src/pages/` — screens grouped by role:
  - `src/pages/customer/` — customer dashboard, homes, bookings, maintenance, warranty, service history, complaints, notifications, professionals, and AI assessment.
  - `src/pages/professional/` — professional dashboard, applications, jobs, availability, earnings, profile, reviews, and notifications.
  - `src/pages/admin/` — admin dashboard and management screens for users, professionals, bookings, complaints, categories, warranties, maintenance, analytics, audit logs, AI settings, and the code knowledge base/evaluation report.
- `src/components/` — reusable interface pieces such as navigation, cards, booking/payment panels, messages, status badges, and route protection.
- `src/layouts/` — shared page layouts for customer, professional, and admin areas.
- `src/context/` — authentication state and hooks.
- `src/services/` — frontend API client used to communicate with the backend.
- `src/data/` — frontend data helpers and mock data.
- `src/assets/` — images and static assets imported by the frontend.
- `src/*.css` and CSS under `src/pages/` — application styling.

## Backend — `server/`

The Node.js and Express API, including MongoDB access.

- `server/index.js` — API routes, authentication/authorization, and application server.
- `server/db.js` — MongoDB connection and database helpers.

## Shared code — `shared/`

- `shared/knowledgeMatching.js` — matching and metadata helpers used by both the frontend and backend.

## Project configuration and support

- `package.json` and `package-lock.json` — scripts and dependency records for the combined frontend/backend project.
- `vite.config.js` — frontend development/build configuration and API proxy.
- `.env.example` — environment variable template. `.env` contains local settings and should remain private.
- `index.html` — frontend HTML entry point.
- `public/` — files served directly by the frontend, including icons and the published evaluation report.
- `scripts/evaluate-project.mjs` — project evaluation pipeline.
- `knowledge-base/homecare-truth-table.json` — evaluation rubric and expected checks.
- `evaluation/latest-metrics.json` — generated evaluation output.
- `DATABASE_STRUCTURE.md` — database collections and data relationships.
- `README.md` — setup and usage instructions.
- `output/` — generated presentation, diagrams, and report artifacts; these are project deliverables, not runtime source code.

## Simplified view

```text
HomeCare AI/
├── src/                  Frontend (React)
├── server/               Backend (Express + MongoDB)
├── shared/               Helpers used by both sides
├── public/               Static frontend files
├── scripts/              Development/evaluation tools
├── knowledge-base/       Evaluation reference data
├── evaluation/           Generated metrics
└── output/               Presentation and report deliverables
```

The project currently uses a single package setup, so the frontend and backend are separated by folders and responsibility while sharing root-level configuration.
