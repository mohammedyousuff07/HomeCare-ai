# HomeCare AI

HomeCare AI is a home-services booking app with a React frontend, Express API, and MongoDB database.

## Run locally

1. Install Node.js and MongoDB Community Server, or create a MongoDB Atlas database.
2. Copy `.env.example` to `.env` and set `MONGODB_URI` to your MongoDB connection string.
3. Install dependencies with `npm install`.
4. Start the API and frontend together with `npm run dev` (`npm run dev:full` is an alias). Use `npm run dev:frontend` only when the API is already running separately.

The API creates the required MongoDB collections and indexes on startup. Development mode inserts demo users, services, and a starter store catalog; demo users and starter products are not created in production. Catalog records are seeded with insert-only behavior, so admin-edited product details and prices are not overwritten on restart. The local MongoDB default is `mongodb://127.0.0.1:27017`, using the `homecare_ai` database.

## Demo accounts

Development only:
- Customer: `priya@homecare.ai` / `password123`
- Professional: `aarav@homecare.ai` / `password123`
- Admin: `admin@homecare.ai` / `password123`

Never use demo accounts or their passwords in production. Set `NODE_ENV=production`, a unique random `AUTH_TOKEN_SECRET` (at least 32 characters), a different random `DOCUMENT_ENCRYPTION_KEY` (at least 32 characters), and `CLIENT_URL` to a comma-separated allowlist of exact frontend origins. Production startup fails when those settings are missing or the secrets match. When moving existing encrypted professional documents to a new key, temporarily set `DOCUMENT_ENCRYPTION_KEY_PREVIOUS` to the old key; startup re-encrypts legacy documents before serving requests. The in-process login throttle is intended as a single-instance baseline; multi-instance deployments should put shared rate limiting at the gateway or use a shared limiter store.

## MongoDB database structure

See [DATABASE_STRUCTURE.md](./DATABASE_STRUCTURE.md) for the collection fields, relationships, indexes, and the distinction between collections used by current API features and foundations for planned features. API responses expose MongoDB `ObjectId` values as string `id` values. Passwords are stored as bcrypt hashes.

## API routes

- `GET /api/health`
- `POST /api/auth/register`
- `POST /api/auth/login`
- `GET /api/services`
- `GET /api/professionals`
- `GET /api/bookings?customerId=<user-id>`
- `POST /api/bookings`
- `PATCH /api/bookings/:id`
- `GET /api/dashboard/:userId`
- Admin service, professional, complaint, and summary routes under `/api/admin`

Set `MONGODB_URI`, `MONGODB_DATABASE`, `PORT`, and `CLIENT_URL` in `.env`. `CLIENT_URL` may contain comma-separated exact origins; local development defaults to localhost ports 5173 and 127.0.0.1:5173. Never commit `.env` or share an Atlas connection string containing credentials.

## Project evaluation

Run `npm run evaluate` to run the project-wide evaluation pipeline. It lints the source, creates a production build, checks role access and the main booking/payment/notification/data flows, compares those results with the weighted truth table, calculates a project score out of 100, and writes detailed metrics to `evaluation/latest-metrics.json`. The report also includes one JSON solution entry per JavaScript/JSX module, with category, topic, variant, LOC, function/class counts, lint/complexity/structure/logic scores, estimated maintainability, and a composite score.

The expected checks and metric scoring rubric are documented in `knowledge-base/homecare-truth-table.json`. The checks are requirement-based ideal outcomes for this project, rather than measurements taken from a separate reference implementation. Per-file complexity and maintainability are static estimates. The current pipeline checks project structure plus lint/build; it does not connect to MongoDB or prove live browser/API behavior. A reference implementation and a running test database would be needed for reference-code comparisons and integration testing.

Admins can open **Evaluation Report** to compare metrics and view the best fuzzy-matched approved code reference. **Knowledge Base** lets admins add unseen modules to a provisional queue, edit the suggested topic/description/tags, generate a code suggestion, and approve or reject it. Approved entries are saved to MongoDB. LLM generation is optional and server-side: set `CODE_LLM_ENDPOINT`, `CODE_LLM_API_KEY`, and `CODE_LLM_MODEL` in `.env`; otherwise the original module is the clearly labelled fallback. LLM output always requires admin review before it becomes an optimal reference.
