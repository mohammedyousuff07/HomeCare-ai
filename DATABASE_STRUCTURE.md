# HomeCare AI database structure

## Database conventions

- MongoDB is configured with `MONGODB_URI` and `MONGODB_DATABASE`; the default database name is `homecare_ai`.
- The API creates indexes during startup. It also inserts demo users and services when they do not already exist.
- Documents use MongoDB `ObjectId` values for `_id` and references. API responses serialize `_id` to a string `id`.
- Timestamps are JavaScript `Date` values. Booking schedules are stored as dates, and API code interprets them in UTC.
- MongoDB does not enforce foreign keys. API routes validate ownership and referenced IDs.

## Collections used by the application

### Accounts and profiles

#### `users`

One account per customer, professional, or admin.

Fields: `_id`, `name`, `email` (normalized lowercase), `phone`, `password_hash` (bcrypt), `role` (`customer`, `professional`, `admin`), `status` (`active` or `suspended`), `professional_status` (`pending`, `approved`, or `rejected` for professionals), `created_at`, `updated_at`.

Indexes: unique `email`.

#### `customer_profiles`

One preferences/profile record per customer, linked by `user_id`.

Fields: `user_id`, `phone`, `preferred_language`, `preferred_contact_method`, `created_at`, `updated_at`.

Indexes: unique `user_id`.

#### `professional_profiles`

One service profile per professional, linked by `user_id`.

Fields include `user_id`, `phone`, `specialty`, `experience_years`, `service_areas`, `bio`, `weekly_availability`, `verification_status`, optional `credential_document_type`, and timestamps. Name and email remain on the corresponding `users` document. Government ID numbers are not stored.

Indexes: unique `user_id`; `service_areas`.

#### `professional_documents`

Professional application and verification documents. Document bytes are encrypted in the database; API responses omit encrypted content and cryptographic metadata.

Fields include `user_id`, `document_type`, `file_name`, `mime_type`, `content_encrypted`, `encryption_iv`, `encryption_tag`, `review_status`, `reviewer_id`, `reviewed_at`, and timestamps.

Indexes: `user_id` + `review_status`; unique `user_id` + `document_type`.

### Services and bookings

#### `services`

Service catalogue used to create bookings.

Fields: `name`, `category`, `description`, `base_price`, `active`, `created_at` (plus MongoDB `_id`).

Indexes: unique `name`.

#### `service_categories`

Service category catalogue.

Fields: `name`, `description`, `display_order`, `active`, timestamps.

Indexes: unique `name`.

#### `bookings`

The central record linking a customer, service, and optional assigned professional.

Fields: `customer_id` → `users._id`, optional `professional_id` → `users._id`, `service_id` → `services._id`, `service_name` and `quoted_price` snapshots, `currency`, `property_id`, `room`, `address`, `scheduled_for`, `notes`, `status`, `customer_approved_at`, `final_price`, `created_at`, `updated_at`. Booking workflow additions: `problem_photos` (attachment references with file name and MIME type), `material_preference`, `material_procurement_type`, `estimated_arrival`, `estimated_duration_minutes`, `actual_start_time`, `actual_end_time`, and `cost_breakdown` (`serviceFee`, `materials[]`, `totalAmount`).

Booking statuses include `pending`, `confirmed`, `inspection_completed`, `estimate_approved`, `en_route`, `in_progress`, `completion_requested`, `completed`, and `cancelled`. `confirmed` remains supported for existing bookings created before estimates were added.

Indexes: `customer_id` + `scheduled_for`; `status`.

#### `booking_attachments`

Small customer-uploaded issue photos or short videos stored separately from booking list records. Each document includes `booking_id`, `uploaded_by`, `file_name`, `mime_type`, Base64 `data`, and `created_at`. Access is restricted to the customer, the assigned professional, or a professional with a pending offer. Current upload limits are three photos up to 1 MB each, or one MP4/WebM video up to 4 MB, with a 6 MB combined request cap.

#### `job_offers`

One offer per professional for an open, unassigned booking. Allows a new request to be offered to multiple approved professionals; the first acceptance assigns the booking.

Fields: `booking_id`, `professional_id`, `status` (`pending`, `accepted`, `declined`, or `expired`), `created_at`, `updated_at`.

Indexes: unique `booking_id` + `professional_id`; `professional_id` + `status` + `created_at`.

#### `booking_events`

Append-style booking activity history for creation, assignment, acceptance, decline, and status changes.

Fields: `booking_id`, `event_type`, `old_value`, `new_value`, optional `actor_user_id`, optional `assignment_source`, `created_at`.

Indexes: `booking_id` + `created_at`; `actor_user_id` + `created_at`.

#### `booking_messages`

Messages between the customer and assigned professional for a booking.

Fields: `booking_id`, `sender_id`, `recipient_id`, `sender_name`, `sender_role`, `message`, `created_at`.

Indexes: `booking_id` + `created_at`.

#### `store_products`

HomeCare Store catalog seeded with product name, brand, category, SKU, image path, description, and the exact listed `price`. Job material recommendations read the price from this catalog; professionals cannot enter a separate price or markup.

#### `store_orders`

Customer product orders and approved job-material orders. Stores customer/professional/booking references when relevant, a snapshot of product IDs, names, image paths, listed unit prices, quantities and line totals, delivery address, exact subtotal/total, order status, payment status, and timestamps. Standalone orders use pay-on-delivery; approved booking products are included in the booking’s service bill.

### Payments, reviews, and notifications

#### `payments`

One payment record is created when the customer approves completed work. This project does not connect to a payment gateway: the customer reports an off-app payment method and the professional confirms receipt.

Fields: `booking_id`, `customer_id`, `professional_id`, `amount`, `currency`, `status` (`due`, `cash_reported`, `method_reported`, `paid`), `payment_method`, `customer_reported_at`, `paid_at`, `confirmed_by`, optional `provider`, optional `provider_transaction_id`, and timestamps.

Indexes: `booking_id` + `created_at`; sparse unique `provider_transaction_id`.

#### `reviews`

Customer reviews of professionals, one review per booking. A review is allowed only after the booking is completed and payment is confirmed.

Fields: `booking_id`, `customer_id`, `professional_id`, `service_id`, `rating` (1–5), `comment`, `status` (for example `visible` or `hidden`), `created_at`.

Indexes: unique `booking_id`; `professional_id` + `created_at`.

There is no active professional-to-customer rating collection or endpoint.

#### `notifications`

Per-user in-app notifications for booking requests, booking changes, payments, reviews, complaints, and messages.

Fields: `user_id`, `type`, `title`, `message`, optional `booking_id`, optional `complaint_id`, `read_at`, `created_at`.

Indexes: `user_id` + `read_at` + `created_at`.

### Home records and maintenance

#### `properties`

Customer homes/locations.

Fields: `customer_id`, `name`, `property_type`, `address`, `rooms`, `assets`, `is_primary`, timestamps.

Indexes: `customer_id` + `is_primary`.

#### `maintenance_rules`

Reusable maintenance schedules.

Fields: `name`, `category`, `interval_days`, `priority` (`low`, `medium`, `high`), `description`, `active`, timestamps.

Indexes: unique `name`.

#### `maintenance_tasks`

Customer-specific reminders generated from rules.

Fields: `customer_id`, `property_id`, `rule_id`, `due_at`, `status`, optional `completed_booking_id`, timestamps.

Indexes: `customer_id` + `due_at` + `status`; `property_id` + `due_at`.

#### `warranties`

Warranty issued for a completed booking.

Fields: `booking_id`, `customer_id`, `service_id`, optional `professional_id`, `starts_at`, `expires_at`, `coverage`, `created_at`. Active/expired state is derived from `expires_at`.

Indexes: unique `booking_id`; `expires_at`.

#### `warranty_claims`

Customer claims against a warranty.

Fields: `warranty_id`, `booking_id`, `customer_id`, `description`, `evidence_refs`, `status`, `resolution`, timestamps.

Indexes: `warranty_id` + `created_at`.

#### `professional_locations`

Temporary live location for an assigned professional during an in-progress booking.

Fields: `booking_id`, `professional_id`, `customer_id`, `latitude`, `longitude`, `updated_at`, `expires_at`.

Indexes: unique `booking_id`; TTL index on `expires_at`.

### Administration and AI records

#### `complaints`

Customer issues, optionally linked to a booking and assigned professional.

Fields include `customer_id`, optional `booking_id`, optional `professional_id`, `subject`, `description`, `status` (`open`, `investigating`, `resolved`), `outcome`, professional response fields, and timestamps.

Indexes: `created_at` descending.

#### `audit_logs`

Selected platform and admin actions.

Fields: `action`, `entity`, `entity_id`, `description`, `created_at` (and actor metadata where recorded by a route).

Indexes: `created_at` descending.

#### `ai_settings`

AI feature configuration.

Fields include `setting_key`, `enabled`, `provider`, `model`, `config_version`, `updated_by`, `updated_at`.

Indexes: unique `setting_key`.

#### `ai_assessments`

Saved AI assessment requests and results.

Fields include `user_id`, `input`, `category`, `result`, `confidence`, `model_version`, optional `booking_id`, `created_at`.

Indexes: `user_id` + `created_at`.

#### `code_knowledge_base`

Human-approved optimal code references used by the project comparison report.

Fields: `category`, `topic`, `normalized_topic`, `variant`, `description`, `tags`, `optimal_code`, `reference_source`, `source_path`, `metrics`, `status`, `approved_by`, `approved_at`, `created_at`, `updated_at`.

Indexes: `normalized_topic` + `status`; `tags` + `status`.

#### `provisional_code_entries`

Unseen code topics waiting for admin review. The system suggests metadata and tags, then uses an optional LLM candidate or falls back to the original project code. Candidates remain private to admins and are never used as optimal references until approved.

Fields: `category`, `topic`, `normalized_topic`, `variant`, `description`, `tags`, `source_path`, `user_code`, `candidate_optimal_code`, `candidate_source`, `generation_note`, `status` (`pending_review`, `approved`, `rejected`), `created_by`, `reviewed_by`, `reviewed_at`, `created_at`, `updated_at`.

Indexes: `source_path` + `status`; `created_at` descending.

## Relationship map

```text
users ──< customer_profiles
users ──< professional_profiles ──< professional_documents
users ──< bookings >── services
                  ├──< job_offers >── users (professionals)
                  ├──< booking_events
                  ├──< booking_messages
                  ├── payments
                  ├── reviews >── users (professional)
                  └── warranties ──< warranty_claims
users ──< properties ──< maintenance_tasks >── maintenance_rules
users ──< notifications
users ──< complaints
users ──< ai_assessments
services ──< reviews
services ──< warranties
bookings ── professional_locations (temporary, one per booking)
```

`──<` means one record can be referenced by many records. These references are stored as ObjectIds; API routes enforce their meaning and ownership.

## Seed data

On startup, the API inserts demo accounts if absent: customer `priya@homecare.ai`, professional `aarav@homecare.ai`, and admin `admin@homecare.ai`. The demo password is `password123`. It also seeds four services when absent: AC Preventive Service, Electrical Inspection, Plumbing Repair, and Home Deep Cleaning. Seed writes use `$setOnInsert`, so existing demo account values are not overwritten.

## Data handling rules

- Store password hashes only; never return `password_hash` from an API.
- Store timestamps as MongoDB dates.
- Save service name and quoted price snapshots on bookings so catalogue edits do not rewrite history.
- Do not store payment card details. Current payments are reported by customers and confirmed by professionals; only payment method and confirmation details are recorded.
- Protect professional verification documents; encrypted document content must not be returned in ordinary API responses.
- Validate ownership and role permissions in the API; frontend route guards alone are not authorization.
