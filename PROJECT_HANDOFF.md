# HomeCare AI — Project Handoff for Another AI

This document is a source-grounded guide to the current repository. Use it as context before proposing or making changes. The code is authoritative when it differs from this summary or from an older project document.

## 1. Product overview

**HomeCare AI** is a home-services and home-maintenance web application. It brings homeowners, service professionals, and administrators into one workflow: customers describe a repair and upload photos, professionals inspect the request and send a timed, itemized estimate, customers approve it, and both sides follow the job through completion and payment confirmation. The application also provides a product catalog for common repair materials and tools to manage a home’s service history, maintenance reminders, and warranties.

- **Primary market/context:** home services in India; prices are displayed in INR (`₹`).
- **Customer users:** people arranging home repairs and keeping home records.
- **Professional users:** plumbers, electricians, AC/HVAC technicians, cleaners, and other home-service providers.
- **Admin users:** staff who approve professionals, manage the marketplace, resolve complaints, and maintain platform records.
- **Product promise:** organize the service lifecycle and its records, with clear estimates and catalog-listed material prices.

The marketing name contains “AI,” but that does not mean every customer workflow calls a live AI model. The customer “AI assessment” screen is currently a guided entry point to report a problem. A separate optional LLM integration exists for admin-reviewed code-reference suggestions; it is not the same as an AI repair diagnosis.

## 2. Technology and architecture

| Layer | Current implementation |
| --- | --- |
| Front end | React 19, JavaScript ES modules, Vite 8, React Router 7 |
| Styling | Custom CSS/design tokens plus Tailwind CSS 4 Vite plugin; responsive layouts; light/dark theme |
| UI icons | `lucide-react` |
| HTTP | Axios client in `src/services/api.js` |
| API | Node.js and Express 5 (`server/index.js`) |
| Database | MongoDB with the official `mongodb` driver (`server/db.js`); this project does **not** use Mongoose schemas |
| Authentication | Bearer token stored in browser local storage; API validates signed HMAC tokens and checks user role/status |
| Shared data/helpers | `shared/` (catalog seed data and knowledge matching) |

The front end and API live in one repository and share the root package manifest. The API normally listens on port `5001`; Vite listens on `5173`. `vite.config.js` proxies `/api` to port `5001`. `VITE_API_URL`, when present in `.env`, takes precedence over that proxy because the Axios client uses it directly. The current local `.env` has been configured to use port `5004` because another older server occupied `5001`; this local file is private and must not be copied into source control or shared with credentials. `.env.example` contains safe placeholders and the normal `5001` defaults.

### Important files

- `src/main.jsx` — initializes React, global styles, auth provider, and visual effects.
- `src/App.jsx` — route map and role-protected page composition.
- `src/context/AuthContext.jsx`, `src/context/useAuth.js` — session, roles, theme, sign-in/out.
- `src/components/ProtectedRoute.jsx` — client-side route guard; the server must still enforce access.
- `src/services/api.js` — Axios base URL and automatic bearer-token header.
- `src/layouts/` — shared navigation/layout wrappers.
- `src/pages/` — screens grouped by customer, professional, and admin roles.
- `src/components/` — shared navigation, cards, status badges, messages, payments, and booking UI.
- `src/design-system.css`, `src/index.css`, `src/App.css`, `src/professional.css`, and page CSS — visual system and screen styles.
- `server/index.js` — API routes, role enforcement, validation, database seeding, and startup.
- `server/db.js` — MongoDB client and connection helpers.
- `shared/storeCatalog.js` — current starter products and listed prices; seeded into MongoDB during API startup.
- `shared/knowledgeMatching.js` — matching and metadata helpers for the admin code knowledge feature.
- `DATABASE_STRUCTURE.md` — collection/index/relationship detail.
- `PROJECT_STRUCTURE.md` — directory guide.
- `knowledge-base/homecare-truth-table.json` and `scripts/evaluate-project.mjs` — project evaluation requirements and evaluation pipeline.

## 3. Roles, login, and access rules

There are three account roles: `customer`, `professional`, and `admin`.

- Public pages are `/`, `/login`, and `/register`.
- Customer and professional roles may use `/store`; admins do not have a store route.
- A professional must have `professional_status === "approved"` before using the professional workspace. New professionals are routed to `/professional/application` until an admin approves them.
- Admin APIs are protected server-side by authentication and `requireAdmin` middleware.
- Professional APIs are protected by authentication and approved-professional middleware, with application endpoints available before approval.
- Customer APIs enforce customer role and booking/property ownership.
- The frontend route guard is for navigation only; it is not a substitute for API authorization.

Seed/demo accounts are documented in `README.md` (customer, professional, admin). The default demo password is `password123`. Treat these as local development fixtures only; do not use them as real deployment credentials.

Authentication tokens live in `localStorage` under `homecare-token`; a cached user profile is under `homecare-user`. Theme choice is saved separately as `homecare-theme`. Tokens expire after 12 hours. Configure stable secret values through environment variables for persistent sessions and document encryption; never paste the real `.env` into another AI/chat.

## 4. Pages and route map

### Public and shared

- `/` — marketing landing page; signed-in users are redirected to their role dashboard.
- `/login` — sign-in with account type selection and demo-account hints.
- `/register` — customer or professional account creation; professional registration collects specialty, years of experience, and service city.
- `/not-found` and the catch-all route — not-found screen.
- `/store` — authenticated store for customers and professionals.
- `/profile` — customer profile; `/admin/profile` is the admin profile.

### Customer routes

- `/dashboard` — customer home overview, saved homes, upcoming bookings, reminders, and recently completed work.
- `/my-home` — manage saved properties and home details.
- `/report-problem` — submit a problem/service request entry point.
- `/ai-assessment` — guided service request/assessment entry point; not a live diagnosis engine today.
- `/professionals` and `/professionals/:id` — browse approved professionals and view a profile.
- `/bookings` and `/bookings/:id` — create and track bookings; see estimates, approve material plans, and review completion.
- `/maintenance` — maintenance reminders/tasks.
- `/service-history` — completed-service records.
- `/home-passport` — home service record/passport view.
- `/warranty` — warranty information and claims.
- `/notifications` — in-app notifications.
- `/emergency` — emergency-service information/flow.

### Professional routes

- `/professional/application` — submit identity/credential documents and see review status.
- `/professional/dashboard` — workload and performance overview.
- `/professional/jobs` and `/professional/jobs/:id` — job offers, booking details, issue photos, estimate/material selection, messages, job progress, location, and payment confirmation.
- `/professional/profile` — professional profile.
- `/professional/availability` — availability schedule.
- `/professional/earnings` — earnings summary.
- `/professional/reviews` — customer feedback.
- `/professional/notifications` — professional notifications.

### Admin routes

- `/admin/dashboard` — platform overview.
- `/admin/users` — user management.
- `/admin/professionals` — professional review and approval.
- `/admin/bookings` — booking oversight and assignment.
- `/admin/categories` — service/category management.
- `/admin/complaints` — complaint investigation/resolution.
- `/admin/warranties` and `/admin/maintenance` — warranty and maintenance-rule management.
- `/admin/ai-settings` — AI-related settings/information.
- `/admin/analytics` — platform analytics.
- `/admin/audit-logs` — selected administrative activity.
- `/admin/evaluation-report` — generated project quality/evaluation report.
- `/admin/knowledge-base` — review and approve/reject code-reference entries.

## 5. Main workflows

### Customer booking

The booking screen is a three-stage wizard:

1. Select a service and room.
2. Enter a home/service address and preferred appointment time; attach issue photos or a short supported video.
3. Review the request, choose an initial material preference, and submit.

Material preference options are “let the professional advise,” “professional sources parts,” or “customer provides parts.” The customer can include optional notes. API validation checks role, ownership, service/property identifiers, supported room values, schedule, and media limits.

### Professional inspection and estimate

New work appears as a pending booking/offer. The professional reviews the customer request and attachments, then provides:

- future estimated arrival time (ETA),
- estimated work duration,
- labor/service fee,
- parts plan (none, in-app store, customer-provided, or pro-supplied),
- and, for in-app store materials, one or more catalog items and quantities.

The selected catalog product price is read and calculated by the server; the professional does not supply a separate markup price. The client requires a future ETA and at least one selected product when the in-app store plan is chosen. “Send estimate & accept” sends the estimate to the customer. The current screenshot-related fix makes a missing/invalid ETA focus the arrival field and show an inline error, instead of silently appearing to do nothing.

Professionals may choose items inside the estimate picker or browse the full store from a job. The full-store link carries the booking ID; “Select for this job” returns with the selected product ID so the job estimate automatically selects it. The current code is in `src/pages/professional/ProfessionalJobDetails.jsx` and `src/pages/Store.jsx`.

### Customer material approval and purchase

When an itemized estimate is ready, the customer can:

- approve the recommended products and estimate, or
- say they will provide their own parts.

If the customer chooses their own parts, the catalog-material lines are removed from the booking total; labor remains. If they approve recommended store products, the server snapshots those catalog items into a booking-linked store order at their listed prices. There is no product markup. Standalone customer cart orders are also supported and are marked pay-on-delivery.

### Job execution, completion, and payment

The practical status lifecycle is:

```text
pending
  → inspection_completed (professional accepts and sends estimate)
  → estimate_approved (customer approves estimate/material plan)
  → en_route (professional starts travel)
  → in_progress (professional arrives/starts work)
  → completion_requested (professional requests completion approval)
  → completed (customer approves completion)
```

`confirmed` remains supported for legacy bookings. Customers can cancel their own booking under allowed conditions. Professionals can move an accepted job through the allowed working statuses but cannot arbitrarily alter a customer booking.

The customer reports an offline payment method after completion; the professional confirms receipt. The current project has **no payment gateway** and does not collect card/bank data. Review creation is allowed after completion and confirmed payment.

### Multi-professional offers and decline handling

Open bookings may create one `job_offers` record per approved professional. The first eligible professional to accept is assigned; other pending offers expire. Declines can pass work to another professional or flag the booking for admin assignment if no candidate remains. Booking events and notifications record major transitions.

## 6. Store and product catalog

The current catalog lives in `shared/storeCatalog.js`, and `initDatabase()` upserts it into MongoDB’s `store_products` collection. Current categories include Plumbing, Electrical, AC & HVAC, and Home Cleaning. Products include taps, braided connectors, pipe fittings, bottle traps, valves, thread seal tape, switches, bulbs, sockets, MCBs, AC capacitors, insulated copper pair coil, AC brackets, coil cleaner, and microfiber cloths.

- Product fields used by the API/UI: `product_id`, `name`, `brand`, `category`, `price`, `sku`, `image`, `description`, `active`, timestamps.
- Product images are direct image URLs hosted by external retailer/supplier CDNs; the repo does not contain local original photo assets for these catalog items.
- Catalog prices are **starter listed prices**, not a live supplier feed. Confirm them against the actual supplier’s current tax-inclusive price before launch.
- The UI and backend use the catalog price as-is and calculate line totals as `price × quantity`; they do not add a product markup.
- `GET /api/store/products` requires authentication; customers can list and place standalone orders, professionals can browse and select products for a job.
- `POST /api/store/orders` is customer-only, validates up to 20 items and quantity bounds, and snapshots catalog pricing into `store_orders`.
- Product photos and prices are not a promise of current availability; there is no live stock/inventory integration.

## 7. API map (grouped by capability)

All paths below are prefixed with `/api`.

- **Auth/account:** `POST /auth/login`, `POST /auth/register`, `GET /auth/me`, `PATCH /account/profile`, `GET /health`.
- **Store:** `GET /store/products`, `GET /store/orders`, `POST /store/orders`.
- **Services and professionals:** `GET /services`, `GET /professionals`, `GET /professionals/:id`.
- **Customer home:** `GET/POST /customer/properties`, `PATCH /customer/properties/:id`, `GET/POST /customer/maintenance`, `PATCH /customer/maintenance/:id`, `GET /customer/overview`, `GET /dashboard/:userId`.
- **Bookings and collaboration:** `GET/POST /bookings`, `PATCH /bookings/:id`, `GET /bookings/:id/photos`, `GET/POST /bookings/:id/messages`, `POST /professional/jobs/:id/decision`.
- **Professional profile/jobs:** `GET /professional/me`, `PATCH /professional/profile`, `GET/PUT /professional/availability`, professional complaints/responses, job location sharing, job payment confirmation, professional notifications.
- **Professional application:** `GET /professional/application`, `POST /professional/application/documents`.
- **Payments and reviews:** `POST /payments/:bookingId/report`, `POST /professional/jobs/:id/payment/confirm`, `POST /reviews`.
- **Complaints:** customer `POST /complaints`; professional complaint list/response; admin list/update.
- **Notifications:** role-based notification lists and `PATCH /notifications/:id` read-state update.
- **Admin:** summary, users, professionals, bookings, services/categories, maintenance rules, warranties, analytics, audit logs, complaints, notifications, and code knowledge base/provisional review endpoints.

Read `server/index.js` for exact payloads, validation, permissions, and error messages. It is the source of truth; this grouped list is an orientation rather than a complete OpenAPI specification.

## 8. MongoDB data model at a glance

MongoDB ObjectIds are used for document IDs and references; response serialization exposes IDs as string `id` values. MongoDB does not enforce relational foreign keys, so the API validates ownership and references.

### Accounts and work

- `users`, `customer_profiles`, `professional_profiles`, `professional_documents`
- `services`, `service_categories`, `bookings`, `job_offers`
- `booking_attachments`, `booking_events`, `booking_messages`
- `store_products`, `store_orders`

### Payments and customer records

- `payments`, `reviews`, `notifications`
- `properties`, `maintenance_rules`, `maintenance_tasks`
- `warranties`, `warranty_claims`, `professional_locations`
- `complaints`, `audit_logs`

### Admin code-reference tools

- `ai_settings`, `ai_assessments`
- `code_knowledge_base`, `provisional_code_entries`

`DATABASE_STRUCTURE.md` documents collection fields, indexes, links, and data-handling rules. Some collections are foundations or are used only by specific optional/admin flows; their existence does not imply a full external integration.

### Important booking fields

Booking documents link `customer_id`, optional `professional_id`, `service_id`, optional `property_id`, and hold service/price snapshots. They also carry `room`, `address`, `scheduled_for`, `notes`, `status`, `problem_photos` references, material preferences/procurement type, ETA/duration, start/end times, `cost_breakdown`, final price, and timestamps.

The estimate breakdown shape is conceptually:

```js
{
  serviceFee: Number,
  materials: [{
    itemName: String,
    product_id: String, // present for in-app catalog items
    image: String,
    unit_price: Number,
    quantity: Number,
    cost: Number
  }],
  totalAmount: Number
}
```

The backend reconstructs catalog material costs from MongoDB prices at acceptance, rather than trusting a price submitted by the browser.

## 9. Security and data-handling notes

- Passwords are hashed with bcrypt; do not expose password hashes in API responses.
- Use `AUTH_TOKEN_SECRET` for stable HMAC token signing. If omitted, the server currently creates a random secret at startup, which invalidates existing sessions after a restart.
- Professional documents are encrypted using AES-256-GCM fields; configure a stable `DOCUMENT_ENCRYPTION_KEY` and do not expose encrypted payloads or cryptographic metadata through ordinary APIs.
- Never paste `.env`, MongoDB credentials, token secrets, document-encryption keys, or customer data into a public issue or another AI service.
- Customer issue media is stored as Base64 in `booking_attachments` with strict limits (up to three images at 1 MB each or one MP4/WebM video up to 4 MB; 6 MB combined request cap). Consider object storage before production-scale use.
- Live location is sensitive. The professional explicitly starts sharing, browser geolocation permission is required, and temporary location records use TTL expiry. Do not add background tracking.
- Payment is offline/manual; no card numbers or payment gateway tokens belong in this app as currently implemented.
- External catalog photo URLs depend on third-party hosts and can change or block hotlinking. For production, license/obtain supplier images and serve them from an approved stable asset store.

## 10. Current design language

The interface uses a calm, warm home-services visual system rather than a technical dashboard aesthetic:

- **Primary:** teal `#176B67`; hover `#105653`.
- **Accent:** warm terracotta `#D97859`.
- **Light background/surface:** warm ivory `#F5F2EA` / `#FFFEFA`.
- **Text/muted:** deep green-charcoal `#203432` / `#697975`.
- **Typography:** DM Sans for body and Fraunces for display headings (Google Fonts import in `src/design-system.css`).
- Rounded surfaces, soft shadows, responsive customer/pro/admin layouts, Lucide icons, and a persisted dark/light theme.
- `src/visual-effects.js` adds lightweight pointer-responsive background effects; avoid replacing it with a heavy animation dependency without need.

## 11. Development and verification

From the repository root:

1. Install Node.js and MongoDB Community Server, or provision MongoDB Atlas.
2. Create `.env` from `.env.example`; set `MONGODB_URI`, `MONGODB_DATABASE`, stable secrets, and a free API port. Do not commit `.env`.
3. Install dependencies with `npm install`.
4. Run front end and API together with `npm run dev`; front end defaults to `http://localhost:5173`, API to `http://localhost:5001` unless `.env` overrides them.
5. Run `npm run build` for the Vite production build; `npm run lint` for Oxlint; `npm run evaluate` for the broader evaluation pipeline.

The API creates indexes and seeds demo users, four common services, and the current store catalog on startup. A developer should verify `/api/health` and use a signed-in demo account to inspect protected endpoints. The repository evaluation pipeline checks source/lint/build and requirement-oriented structure; it does not prove MongoDB/browser end-to-end behavior.

## 12. Scope limits and unfinished integrations

Do not describe these as fully integrated unless new code proves otherwise:

- Live supplier catalog, current supplier prices, stock counts, shipment tracking, and refunds.
- Online card/UPI payment gateway (payments are reported and confirmed manually).
- AI diagnosis of photos, live AI-generated repair guidance, or guaranteed AI authenticity checks for documents. Document screening only checks known image metadata and remains inconclusive; admin review is still needed.
- Push/SMS/email delivery for in-app notifications unless an external provider is separately configured.
- Real-time GPS streaming when the professional leaves the job page or has not granted location permission.
- Cloud storage for issue photos/documents; media handling is currently database-based.

Optional server-side LLM configuration for admin code suggestions uses `CODE_LLM_ENDPOINT`, `CODE_LLM_API_KEY`, and `CODE_LLM_MODEL`. Suggestions remain provisional until an admin approves them; missing configuration falls back to the source module.

## 13. Instructions for an AI continuing work on this repo

1. Read this file, `README.md`, and the relevant source files before editing; inspect any `AGENTS.md` or local instructions first.
2. Preserve existing feature behavior, React Router paths, API routes, booking status semantics, request/response field names, form names, element IDs, and JavaScript hooks unless the user explicitly requests a functional change.
3. Keep frontend and backend validation aligned. Do not trust catalog price or role claims submitted by a browser; derive prices and permissions on the server.
4. Keep customer/professional/admin authorization checks on the API, not only in React.
5. Make the smallest focused change that resolves the user’s request, preserve the existing folder structure, and explain which files changed and what verification was run.
6. Do not run tests/build unless asked or needed to verify a requested fix; if run, report the actual result and known warnings.
7. Treat all `.env` values and any real user/document/booking data as private. Do not send messages, create purchases, accept real work, or transmit private project data to external services unless the user explicitly asks.

## 14. Quick mental model

```text
Customer: describe issue + attach photos → request appointment → review professional ETA/estimate
         → approve catalog parts or provide own → track job → approve completion → report payment/review

Professional: receive open offer → inspect issue media → choose catalog materials + quantity
             → provide future ETA, duration and labor → send estimate → travel/start/work
             → request completion → confirm offline payment

Admin: approve professionals → manage users/services/jobs/warranties/complaints
       → inspect analytics/audit records → review code-reference suggestions

React/Vite UI  ──Axios bearer token──> Express API  ──MongoDB driver──> MongoDB
```
