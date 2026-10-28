# Service Marketplace MVP — Technical Implementation Plan

**Version:** 1.1 — consolidated working baseline
**Date:** 28 October 2026
**Status:** Business decisions 1–59 consolidated; decision 60 intentionally remains open
**Planning scope:** This document is a build specification and implementation plan. It does not mean the application has been implemented or that legal/privacy approval has been obtained.

## Document control and decision labels

| Label | Meaning |
|---|---|
| **[APPROVED]** | A product/business decision adopted by the owner through the decision process up to decision 59. |
| **[DERIVED]** | A technical consequence required to enforce approved behavior; it does not change the product rule. |
| **[TECH-PROPOSAL]** | A recommended engineering choice consistent with the agreed stack, subject to implementation review. |
| **[OPEN-60]** | A deliberate unresolved product decision: how an accepted price is handled when inspection reveals extra work or a price change. |
| **[LEGAL-GATE]** | Must be reviewed by qualified Egyptian counsel/privacy specialists and relevant vendors before launch with real personal data. |

### Source of truth

This version merges the uploaded **Service Marketplace MVP — Technical Implementation Plan** with the owner-approved business decisions numbered 1–59. If an older technical draft, the PRD, a backlog ticket, or a diagram conflicts with an **[APPROVED]** rule in this document, update that older artifact rather than silently reviving its behavior. Sections about provider choice, costs, regional hosting, and legal interpretation remain proposals or gates unless explicitly approved.

## Contents

1. Executive summary and MVP scope
2. Consolidated product baseline
3. Architecture and technology decisions
4. System modules and trust boundaries
5. Domain model and database constraints
6. Business workflows and state machines
7. API contract plan
8. Authentication, permissions, and account recovery
9. Notifications, background jobs, and deadlines
10. Security, privacy, files, retention, and backups
11. Implementation roadmap
12. Development backlog
13. Testing and acceptance strategy
14. Deployment, observability, and operations
15. Risks, gates, and remaining decisions
16. Definition of Ready / Done
17. Decision traceability (1–59)

---

# 1. Executive summary and MVP scope

The MVP is a free, bilingual, responsive website connecting customers with approved workers (“صنايعية”) in **Cairo and Giza, Egypt**. A customer posts an ordinary service request with a specialty, area, description/address, date, and one or more service periods. Eligible workers send priced offers with a proposed start time and estimated duration. The customer compares offers and accepts one; acceptance creates the booking immediately if the server confirms all eligibility, capacity, and time-conflict rules atomically.

The product has three operational surfaces in one website: customer experience, worker experience, and a protected admin dashboard. Arabic is RTL and English is LTR. Desktop, tablet, and mobile web are in MVP scope. Native iOS/Android apps, urgent requests, in-platform payments, commission collection, chat, GPS/map-based discovery, and worker ratings of customers are out of scope.

## 1.1 Agreed technology baseline

| Area | Baseline |
|---|---|
| Web framework | Next.js App Router, React, TypeScript, Tailwind CSS |
| Application shape | Modular monolith; keep domain rules in server-side modules instead of scattering them through components/routes |
| Database | PostgreSQL |
| ORM/data access | Prisma for normal models/queries; reviewed SQL migrations or raw SQL where PostgreSQL-specific constraints and locking are required |
| Authentication | Auth.js Credentials flow backed by Prisma; account is selected by normalized phone number **and role** |
| Session | JWT session with a maximum lifetime of 24 hours; no `sessionVersion` database check on every request |
| Background processing | `pg-boss` with a separate, long-running worker process; durable jobs and reconciliation sweeps |
| Hosting | DigitalOcean Droplet with Docker Compose for web and worker; production PostgreSQL is a separate managed service |
| Private files | Local filesystem on the application host, outside the public web root; the web and worker use a shared private path and access checks |
| SMS | Twilio is the preferred candidate for OTP delivery in Egypt, subject to sender registration, carrier testing, cost/contract review, and legal review |

These choices capture the technical direction already selected. They do not remove the need to test the chosen database constraints, hosting topology, SMS deliverability, and file-backup plan before launch.

## 1.2 Critical engineering principle

**The frontend may explain a rule, but only the server and database may enforce it.** Every write affecting an offer, booking, worker eligibility, deadline, account state, private file, complaint, or admin action must be authorized and validated on the server. The acceptance transaction must remain correct under concurrent requests and retries.

---

# 2. Consolidated product baseline

## 2.1 User types and account model

- There are two public account roles: **Customer** and **Worker**. Every account has exactly one role.
- The same verified phone number may be used for one customer account and one worker account. They remain separate accounts with separate passwords, profiles, role-specific state, and sessions.
- Sign-in asks the user to choose the role and authenticates the normalized phone number plus that role. Account existence must not be leaked by role-selection or error messages.
- Password reset/recovery applies only to the selected role account unless a separately authorized process explicitly says otherwise.
- Admin accounts are separate privileged accounts, created by a technical administrator outside the public registration flow. Admin sign-in requires password plus SMS OTP.

## 2.2 Registration and worker approval

- Phone ownership is verified with OTP. Account recovery uses OTP, subject to rate limits.
- Worker application includes profile data, requested specialties, requested service areas, and an identity document (ID image or approved official alternative); work photos/profile details are optional where the UI supports them.
- Admin manually reviews the worker application and approves or rejects it. Rejection must include a user-facing reason; the worker can edit and resubmit.
- Admin manages the permitted specialty catalogue and approves each worker's specialties and service areas.
- A worker may offer only services within approved specialties and areas. While an edit to specialties or areas is pending review, only the old approved values remain active and public. New values are not usable until approved. Rejections leave the old approved values active.
- Low-risk profile edits may apply immediately. Legal name, identity documents, specialties, and areas require admin review. Phone-number changes require verification. Sensitive changes and review decisions are audited.
- Temporarily suspended, permanently suspended, and deleted workers must be hidden from the public worker directory immediately.

## 2.3 Service requests

- MVP launches in Cairo and Giza and supports ordinary requests only. Urgent requests are deferred.
- One request represents one specialty, one service area, one date, a description and address details; service-request photos are optional.
- The customer selects one or more service periods. Supported custom periods are 1–12 hours, must start before they end, remain within the same local date, must not overlap another period in the same request, and must not be expired or already started. Use `Africa/Cairo` for local validation.
- Request lifetime is 48 hours from creation, even for requests whose selected service date is later. A request also closes when its last selected period ends if it still has no confirmed booking.
- Before any offer is accepted, the customer may edit the request. Editing invalidates all pending offers, notifies affected workers, and permits new offers. A request may not be edited after acceptance/booking.
- After a confirmed booking is cancelled, the request enters **Awaiting Customer Decision**. It cannot be edited while awaiting the choice. The customer may reopen it only if the original 48-hour lifetime is still valid and at least one selected period is still in the future; otherwise it closes. Reopening never resets the original lifetime.
- When a request reopens, the prior offers eligible for reconfirmation are offers closed by another offer being accepted and the offer associated with a customer-cancelled booking; invalidated and withdrawn offers are not eligible. An eligible worker may explicitly reconfirm the original price and appointment, or submit a revised offer, within up to 2 hours. The window ends sooner if the original 48-hour request lifetime expires or no future period remains. The worker who cancelled the last confirmed booking for that request may not submit or reconfirm an offer on it. All reconfirmations/new offers re-check worker eligibility, time conflicts, and capacity.

## 2.4 Offers, appointment times, and booking acceptance

- A worker may have **one active offer per request**. That offer may be edited while the request is open and it has not been accepted. Edits notify the customer and preserve offer-revision history. No duplicate parallel offers for the same worker/request.
- Offers are shown newest first and include price, proposed start time, estimated duration, worker rating, and relevant profile details so the customer can compare manually.
- The worker supplies a proposed start time and estimated service duration. Duration is 15 minutes to 12 hours in 15-minute increments. The system calculates `end_at = start_at + duration`; the complete half-open interval `[start_at, end_at)` must fit inside one customer-selected service period.
- Overlapping offers may exist before confirmation. At acceptance, the system atomically rechecks request/offer validity, worker approval/status, approved specialty and area, deadline, worker time conflicts, and the maximum of seven active confirmed jobs. The first valid transaction to commit wins.
- Accepting an offer confirms the booking immediately. A second worker approval is not required.
- The worker may send offers to many requests, but may have at most **7 active confirmed bookings in total at one time** (not 7 per day). The accepted booking occupies its full actual start-to-end interval.
- Any overlap with another confirmed interval for the same worker is a conflict. Use PostgreSQL half-open time ranges `[start,end)` so one booking may end exactly when the next starts. This is separate from the seven-booking cap.

## 2.5 Booking cancellation and rescheduling

- Either party may cancel a confirmed booking only **before the agreed service period starts**. Cancellation requires a reason; choosing “Other” requires an explanation. Notify the other party and record the event in the audit log. Cancellation alone has no automatic penalty; repeated patterns may be reviewed by an admin.
- After the service period starts, do not use the ordinary cancellation flow. Use the incident/no-show/complaint pathway as applicable.
- Either party may request a change to the confirmed appointment. There is one pending change request per booking. The other party must explicitly approve it. The response deadline is the earlier of 12 hours after the request or the original service-period start.
- A proposed new time does not reserve capacity while approval is pending. The original appointment stays confirmed. At approval, the system re-checks availability and conflicts atomically. If unavailable, reject the change and keep the original appointment.
- Booking phone/contact details are visible to the two parties only while a booking is confirmed and hidden after it is cancelled or completed.

## 2.6 Completion, complaints, no-show reports, and ratings

- When the worker marks a booking complete, the customer has 48 hours to confirm or report a problem. If the customer remains silent, the booking automatically becomes `Completed` at the deadline. Completion remains independent from reports: filing a complaint or no-show/service-not-performed report does not stop the applicable automatic-completion timer.
- If the worker never marks completion, the booking automatically becomes `Completed` 48 hours after the agreed service window ends, including when a complaint or no-show/service-not-performed report has been filed.
- **Completion is independent from complaint/no-show reporting.** The automatic-completion timer still runs even if a no-show or service-not-performed report has been filed. Do not block the timer solely because a report is open.
- A customer may submit a no-show/service-not-performed report within 48 hours after the service period ends. A report requires a written description and incident time; supporting files/screenshots/call logs are optional. Do not add dedicated “I have arrived” or mandatory contact-attempt event logs.
- A reported no-show enters `No-show Under Review` and must be explicitly decided by an admin; there is no automatic closure merely because time passes. If the booking has not yet auto-completed, it remains under review and consumes a worker slot until the admin decision. When the review is decided before auto-completion, close the booking and release the slot for every review outcome; do not mark it Completed or allow a rating from that review path.
- If auto-completion has already happened before the admin confirms a no-show, preserve the `Completed` status; log the confirmed violation separately in the admin/incident audit history. Completed records remain counted as completed in reports even if a no-show is later confirmed. A no-show report must **permanently disable rating for that booking regardless of the review outcome**.
- The ordinary complaint categories are: service quality; unfinished service; agreed price mismatch; damage; inappropriate behavior; no-show; other. Description/category are required; evidence is optional. A complaint is independent from completion and rating except for the specific permanent rating block above.
- The worker has 24 hours to respond to a complaint. After the deadline, admin may continue investigation; non-response alone is neither proof of a violation nor an automatic penalty.
- When additional information is requested, the requested party has 7 days to respond. No response leads to `Inconclusive` with no automatic penalty. Admin may extend the deadline with a documented reason. The 3-day resolution target starts only once all required information is complete.
- Complaint outcomes are `Confirmed`, `Not Substantiated`, or `Inconclusive`. Notify both parties of the outcome, but do not disclose private evidence or internal investigation notes. Any warning/suspension is a separate audited admin action.
- A customer has 14 days from completion to leave one final review. Notify once at completion; do not send a review reminder. Once submitted, the customer cannot edit/delete it. Admin may hide/remove policy-violating content, with an audit record. An ordinary open complaint does not itself block a review; a no-show/service-not-performed report does.

## 2.7 Suspension, deletion, and duplicate-account evasion

- Admin may issue a formal warning or a temporary suspension for 1, 3, 7, 14, or 30 days, or a permanent suspension, with a recorded reason. Admin can lift a temporary suspension early with a reason. A temporary suspension does not extend an existing one silently; escalation to permanent suspension is a distinct audited action.
- Suspension restricts new activity according to role while keeping permitted management of existing bookings. A suspended worker is hidden from directory/search and is not eligible for new offers. A permanently suspended worker account is prevented from returning through a replacement worker account during a 6-month keyed-HMAC identity-fingerprint retention window.
- Permanent suspension applies to the worker account/role only. A customer account is not automatically suspended just because its linked worker account is permanently suspended.
- Account deletion requires the account-deletion flow and may be blocked while there are active confirmed bookings or unresolved complaints requiring the account's participation. On approved deletion, immediately disable access and hide the public profile. Delete identity documents/private files once they are no longer legitimately needed; retain only minimum legally required booking, complaint, and audit data in restricted or pseudonymized form for a legally approved period.
- The identity fingerprint is a **keyed HMAC**, not a plain hash, derived from the normalized identity number, with the secret kept separately from ordinary database data. Keep it for six months from permanent suspension, including during deletion requested inside that interval, then purge it. This design requires a legal/privacy gate before real-data launch.

## 2.8 Notifications and interaction model

- Routine and time-sensitive notifications are in-app only. SMS is for OTP/verification/recovery and admin MFA only. The business accepts that users may respond late because they may not open the website in time; deadlines still must be enforced reliably.
- Normal notifications are retained for 6 months and automatically purged after that. Opening the notification list marks notifications displayed in that list as read. Audit, complaint, and legally retained records follow their own retention rules and are not purged as ordinary notifications.

---

# 3. Architecture and technology decisions

## 3.1 Application boundaries

Use a **modular monolith**, with one codebase and one clear server-side domain layer. The web app and background worker are separate runtime processes built from the same repository and share domain types/services and database access. Do not introduce microservices for MVP.

### Recommended repository structure (proposal)

```text
apps/
  web/                  # Next.js App Router: public, customer, worker, admin UI + server endpoints
  worker/               # pg-boss worker process and deadline/reconciliation jobs
packages/
  domain/               # business rules, policies, state-transition services
  db/                   # Prisma schema/client, SQL migrations, transactions
  validation/           # shared input schemas and API error codes
  i18n/                 # translation dictionaries and notification templates
  config/               # typed environment configuration
  test-utils/           # fixtures and database test helpers
```

This is a recommended monorepo layout, not a required exact folder structure. Preserve the boundaries even if the repository starts smaller.

## 3.2 Database and Prisma

- PostgreSQL is the source of truth.
- Prisma handles ordinary CRUD, relations, and typed queries.
- Use reviewed SQL migrations for database-specific features Prisma schema syntax cannot fully express, especially the exclusion constraint for overlapping confirmed bookings, supporting extensions/indexes, and any lock-oriented query.
- Keep transaction boundaries in domain services, not in React components or arbitrary route handlers.
- Every state transition should update the entity, append the audit event, and create its in-app notification/outbox record in the same database transaction where possible.
- Use optimistic version fields for edit-sensitive records and idempotency keys for retry-prone state-changing API calls.

## 3.3 Authentication and sessions

Use Auth.js with a custom Credentials flow backed by Prisma. Use normalized phone + selected role for account lookup. Hash passwords with Argon2id or the approved strong password-hashing implementation selected at build time. Configure JWT session maximum age to 24 hours. Do not query `sessionVersion` on every request. Instead, every protected server operation must check the current account state, role, ownership, worker approval/suspension status where relevant, and the requested entity's current state. A JWT can remain cryptographically valid until expiry, so account restrictions must still be enforced server-side on protected operations.

## 3.4 Job processing

Use `pg-boss` with a separately running worker process. Jobs must be durable, retry-safe, idempotent, and observable. Schedule jobs for known deadlines where appropriate and run reconciliation scans for records whose deadline is due. The scheduler is not a substitute for lazy validation: a request or offer must be rejected as expired even if the deadline job is delayed. Monitor queue lag, retries, poison jobs, and overdue records.

## 3.5 Private file storage

Store files on a private local filesystem outside the public directory. Web and worker containers must share the same configured private volume/path in the chosen Docker Compose topology. Do not expose a file by constructing a public URL. Every read/download passes through an authenticated, authorized server endpoint. Backups must capture both database and private files and be restorable as a consistent-enough service state.

## 3.6 Deployment

Use Docker Compose on a DigitalOcean Droplet for the web process and long-running worker process, with a separate managed PostgreSQL service for production. Keep file storage private on the host and ensure the worker can access it. Separate development, test, and production configuration. Run the app with a non-root user, restrictive firewall, HTTPS/reverse proxy, encrypted secrets, and automated security updates. Exact region, storage of backups, vendor data-processing arrangements, and transfer of identity documents are subject to the legal/privacy gate.

---

# 4. System modules and trust boundaries

| Module | Responsibilities | Key boundaries |
|---|---|---|
| Identity & Accounts | Customer/worker accounts, phone verification, passwords, selected-role login, account recovery | Same phone can map to separate role accounts; always authenticate account + role |
| Worker Onboarding | Application fields, identity-document upload, approval queue, resubmission, pending edits | No public or offer eligibility before approval; old approved specialty/area values remain active during re-review |
| Catalog & Areas | Admin-managed specialties, certificate requirements where applicable, service-area hierarchy and bilingual names | Only active admin-approved values are selectable and eligible |
| Worker Discovery | Directory, filters, approved specialties/areas, rating summary, optional work photos | Hide suspended/deleted workers immediately; never return phone, ID, private address, or private files |
| Service Requests | Request creation/editing, period validation, matching, expiry, reopen/close | Request lifetime remains original across reopen; edit invalidates pending offers |
| Offers | Create/edit one offer per worker/request, price/duration/time validation, revision history, reconfirmation | Offer eligibility checked both on save and again on acceptance |
| Booking & Scheduling | Atomic acceptance, cap 7, conflicts, contact visibility, cancellation, change requests | PostgreSQL transaction + exclusion constraint + worker-level capacity lock |
| Completion & Reviews | Completion deadlines, auto-completion, rating windows, review moderation | No-show/service-not-performed report blocks rating for that booking permanently |
| Complaints & Incidents | Complaint submission, response deadlines, extra-info requests, no-show review, outcomes | Private evidence/internal notes are not exposed to the other party |
| Notifications | In-app inbox, read state, template rendering in user's language, 6-month purge | Insert with state change; opening list marks visible notifications read |
| Admin & Audit | Verification, catalog, incident/complaint review, warning/suspension, account governance | Reason required for consequential actions; append-only audit events |
| Background Worker | Expiry jobs, auto-completion, cleanup, notification delivery to inbox, reconciliation | All jobs idempotent; business action validated again in the database |
| Files | Type/size/count validation, random names, protected download, retention deletion | Private outside public root; authorized access only |

## 4.1 Authorization pattern

Authorization is not just a role check. Each protected action validates:
1. authenticated account and role;
2. account is active enough for the requested action;
3. ownership/relationship to request, offer, booking, complaint, or file;
4. current state allows the action;
5. deadline and approved specialty/area constraints;
6. for admins, the permitted admin capability and a required reason where applicable.

Use deny-by-default policies. Never trust user IDs, role values, price, end time, booking status, or file ownership supplied by the browser without server validation.

---

# 5. Domain model and database constraints

The table list below is a starting domain model. Final column names should follow one naming convention and the Prisma schema. Sensitive details must be separated from public profile data.

## 5.1 Principal entities

| Entity | Important fields / relationships |
|---|---|
| `UserAccount` | id, role (`CUSTOMER`/`WORKER`/`ADMIN`), normalized phone, password hash, status, phone verified at, created/updated timestamps |
| `CustomerProfile` | account id, display/legal name and customer-specific profile fields |
| `WorkerProfile` | account id, public display fields, bio, experience, legal name and sensitive verification state in restricted fields; approved specialties/areas are separate relations |
| `WorkerApplication` | worker account, identity document reference, submitted form, status, reviewed by/at, rejection reason, resubmission metadata |
| `Specialty` | bilingual label, active flag, optional certificate requirement, admin audit fields |
| `WorkerSpecialty` | worker, specialty, approval status, approved timestamps; enforce uniqueness for worker/specialty |
| `ServiceArea` | bilingual name, parent/region relationship, active flag |
| `WorkerServiceArea` | worker, area, approval status, approved timestamps; enforce uniqueness for worker/area |
| `ServiceRequest` | customer, specialty, area, text address/landmark, description, service date, status, created/expires/closed timestamps, version |
| `RequestPeriod` | request, start/end instants, original local date/time-zone representation needed for display/validation, order; unique and non-overlap within request |
| `PrivateFile` | random storage key, category, detected MIME, bytes, uploader, associated object, created/deleted metadata; no public URL |
| `Offer` | request, worker, status, price in integer minor units, start/end, duration, submitted/updated timestamps, version; one active offer per worker/request |
| `OfferRevision` | immutable snapshot of changed price, interval, duration, actor, and timestamp |
| `Booking` | accepted request/offer snapshot, customer, worker, price, `tstzrange`/start-end, duration, status, confirmed/completed/cancelled timestamps, contact-visibility state |
| `BookingChangeRequest` | booking, requester, old/new time snapshot, status, response deadline, response actor/time, reason if rejected |
| `CompletionRecord` | booking, worker completion timestamp, automatic/manual completion source, 48h fallback deadline |
| `Review` | booking, customer, final rating/content, submitted timestamp, moderation state; one per booking maximum |
| `Complaint` | booking, reporter, category, description, submitted timestamp, worker response deadline/response, outcome, resolved by/at |
| `ComplaintEvidence` | complaint, private-file reference; visible only to authorized investigator and owner as policy allows |
| `InformationRequest` | complaint, requested party, question, created/due/responded timestamps, response content/status, extension reason |
| `NoShowReport` / `Incident` | booking, reporter/subject, written description, incident timestamp, optional evidence, review status, reviewer, outcome; independent of completion status |
| `Notification` | recipient account, type/template key, parameters, read timestamp, created/purge timestamps |
| `AuditEvent` | actor, action, entity type/id, reason, old/new state metadata, request ID, timestamp; append-only permissions |
| `IdentityFingerprint` | worker account, keyed HMAC value/key version, created and scheduled purge timestamps; restricted and never returned through public APIs |
| `OtpChallenge` | account/normalized phone context, purpose, code hash, expires timestamp, attempt count, resend-after, consumed timestamp |
| `IdempotencyRecord` | actor, key, route/action, request hash, stored outcome, expiry |
| `BackgroundJob` | Managed by pg-boss or job metadata table; business deadlines still reside on domain records |

The same phone number across Customer and Worker roles is represented through separate role accounts, not one account that can switch role. Ensure uniqueness is scoped to `(normalized_phone, role)` rather than phone alone.

## 5.2 Key database rules

1. **One active offer per worker/request.** Enforce with a unique constraint or partial unique index over active statuses; withdrawn/closed historic offers do not block future offers unless the business status rules say so.
2. **No overlapping confirmed booking intervals per worker.** Use PostgreSQL `tstzrange` and an exclusion constraint (for example `EXCLUDE USING gist (worker_id WITH =, confirmed_range WITH &&) WHERE (status = 'CONFIRMED')`) after validating the exact migration design and supporting operator class. Use half-open ranges `[start,end)`.
3. **Maximum seven active confirmed bookings.** An exclusion constraint does not enforce a count. Lock the worker row (or a dedicated worker-capacity row) inside the booking-acceptance transaction, count active confirmed bookings, reject if count is already 7, then create the booking. Every accept path must use the same locking order.
4. **Atomic offer acceptance.** Lock/check the request, offer, worker capacity and relevant booking data; re-check expiration, worker status/approval, specialty/area, interval and pending offer validity; create booking; update offer/request/competing offers; insert notifications and audit event; commit once.
5. **Request periods.** Validate that periods are on the same local service date, start before end, last 1–12 hours, do not overlap each other, and are unexpired/not started when a request is created or reopened. Enforce at service layer and use DB constraints wherever practical.
6. **Price and duration.** Store price as integer minor currency units; do not use floating-point amounts. The UI display currency and final minor-unit convention must be confirmed in config before payment-related work (payments are not in MVP).
7. **One review and one complaint policy.** Unique booking relation where rules require one final review per booking and one complaint per booking (if a separate additional complaint category is required later, change only with product approval). A no-show/service-not-performed report permanently sets a booking-level `rating_blocked_at` or equivalent flag.
8. **Audit integrity.** Admin actions and important state transitions must append audit events. Ordinary app roles cannot update/delete audit rows.
9. **Identity fingerprint.** Use keyed HMAC with key versioning; never store a plain hash of a national ID number for the anti-evasion function.
10. **Notifications purge.** Normal notification purge uses a 6-month cutoff. Audit/complaint/required records have separate retention policies and legal approval.

## 5.3 Time storage and interval conventions

- Persist instants as PostgreSQL `timestamptz` in UTC. Evaluate elapsed deadlines using actual timestamps, not assumed 24-hour calendar days.
- Use `Africa/Cairo` for local display, request-period validation, local date boundaries, and daylight-saving cases.
- Preserve enough local-date/time-zone input to reconstruct what the customer selected and explain it consistently after timezone rules change.
- Use half-open intervals `[start_at, end_at)` for bookings and periods. Back-to-back appointments where one ends exactly as the next starts are allowed.
- Store deadline instants (`expires_at`, `response_due_at`, etc.) rather than recomputing from mutable inputs without a recorded snapshot.

---

# 6. Business workflows and state machines

The states listed here are technical names. The UI must present clear localized labels in Arabic and English. All transitions must be implemented in server-side domain services and covered by tests.

## 6.1 Service-request lifecycle

```text
OPEN_FOR_OFFERS
  ├─ edit before any accepted offer → OPEN_FOR_OFFERS (invalidate pending offers + notify workers)
  ├─ offer accepted → BOOKED
  ├─ 48h from creation reached → CLOSED_EXPIRED
  ├─ last selected period ends with no booking → CLOSED_EXPIRED
  └─ voluntary customer close (where enabled) → CLOSED_BY_CUSTOMER

BOOKED
  ├─ booking cancelled → AWAITING_CUSTOMER_DECISION
  └─ booking completed/closed → terminal/appropriate closed state

AWAITING_CUSTOMER_DECISION
  ├─ customer chooses reopen and original lifetime + future period valid → OPEN_FOR_OFFERS
  └─ customer chooses close, lifetime elapsed, or no future period → CLOSED
```

When reopening, keep the original creation/lifetime timestamp. Offers closed by another acceptance and the offer associated with a customer-cancelled booking may enter reconfirmation; invalidated and withdrawn offers may not. Eligible workers may reconfirm the original terms or submit a revised offer within a maximum 2-hour window, capped by request expiry and remaining future periods. The worker who cancelled the last confirmed booking is excluded for that request.

## 6.2 Offer lifecycle

```text
DRAFT (optional local-only UI state)
  → ACTIVE/PENDING
  → ACCEPTED (winning offer)
  → CLOSED_OTHER_OFFER_ACCEPTED
  → WITHDRAWN
  → EXPIRED
  → PENDING_RECONFIRMATION → ACTIVE/PENDING (after explicit reconfirmation)
```

Offer editing is allowed only while the request is open and no offer has been accepted. Editing creates an immutable revision, updates the active offer, and notifies the customer. A prior offer cannot become eligible merely because a client resubmits an old payload; the server revalidates all current rules.

## 6.3 Atomic booking acceptance

Accepting an offer is one database transaction. Recommended order:
1. Validate authentication and customer ownership.
2. Lock the request and target offer in a consistent lock order.
3. Verify request/offer still open and unexpired; confirm offer is for this request and has not been invalidated by request edits.
4. Verify worker account is approved, not suspended/deleted, and has the current approved specialty/area.
5. Lock worker capacity row; count active confirmed bookings; reject if 7 already exist.
6. Verify the proposed `[start,end)` fits wholly inside one customer period and does not overlap any other confirmed booking for this worker.
7. Create a `CONFIRMED` booking using immutable snapshots of the agreed price, start, duration, and end.
8. Mark the winning offer accepted; close competing offers; update request state; record the event and enqueue in-app notifications.
9. Commit. If a constraint/lock recheck fails, roll back the whole transaction and return a stable conflict error.

Handle simultaneous accepts with database locking/constraints, not UI-disabled buttons. Client retries must be idempotent.

## 6.4 Cancellation and rescheduling

Cancellation request is rejected once the service period has started. The UI requires a reason and requires explanatory text for “Other”. After cancellation, hide contact details, release capacity, notify the other party, write the audit event, and place the request in `AWAITING_CUSTOMER_DECISION` when the product flow calls for it.

A reschedule request stores the proposed time and response deadline. The existing interval remains the booking interval while pending; no provisional capacity reservation is made. At explicit approval, atomically validate the new period and conflicts. If it is no longer available, return a rejection result and leave the original booking unchanged. Automatic expiry occurs at the earlier of 12 hours after request creation and the original period start.

## 6.5 Completion and automatic-completion rules

At worker completion, save the event time and set a 48-hour customer response deadline. At deadline, run the automatic-completion transition independently of complaint or no-show/service-not-performed reporting. Filing a report does not stop or pause the timer; the complaint/incident workflow continues separately. If the worker never marked completion, auto-complete 48 hours after the service window ends, whether or not a report has been filed.

Decision 52 specifically requires that a filed no-show/service-not-performed report does **not** stop the automatic-completion timer. The race between a timer and admin review is therefore handled by transaction ordering:

- If the no-show review is decided **before** automatic completion, close the booking under the incident-review path for any review outcome, release the worker slot, do not mark it `Completed`, and do not permit a rating.
- If automatic completion has already committed, retain the `Completed` status even if the admin later confirms a no-show. Record the violation separately. All records still marked `Completed` count as completed in reporting, and the permanent rating block still applies to any booking with a no-show/service-not-performed report.

These outcomes must be tested with concurrent timer/admin transactions. Do not silently reverse a completed state after the fact.

## 6.6 Complaint and no-show review

Complaint categories: service quality; unfinished service; agreed price mismatch; damage; inappropriate behavior; no-show; other. Require category and description; allow optional evidence. Standard complaint deadlines are based on completion. The no-show/service-not-performed reporting window is 48 hours from the end of the service period, including where no completion exists.

The no-show report includes a written description and incident timestamp plus optional evidence. It enters `NO_SHOW_UNDER_REVIEW`. The admin must make an explicit outcome decision; time passing does not auto-close the review. While awaiting the decision (and before automatic completion has already occurred), the booking may continue consuming one of the worker's seven active slots. Admin review decision releases the slot under the rules above.

Worker response deadline is 24 hours. Missing this deadline allows the admin to proceed but does not itself establish a violation or cause an automatic penalty. Additional requested information is due within seven days unless an admin grants a documented extension. If the party does not respond, close as `INCONCLUSIVE` with no automatic penalty. The 3-day resolution target only starts when all required information is complete. The outcome is shared with the parties; private evidence and internal notes are not.

A no-show/service-not-performed report permanently blocks the rating for that booking, regardless of whether the admin confirms, dismisses, or marks the report inconclusive. This differs from an ordinary complaint, which does not automatically block a rating.

## 6.7 Review lifecycle

- Review eligibility begins at completion and ends 14 days later.
- One final review per booking. Customer cannot edit or delete after submission.
- Send one notification at completion only; no reminder notification.
- An ordinary complaint does not itself suppress or delay review eligibility.
- If any no-show/service-not-performed report was filed for the booking, rating remains disabled for that booking permanently.
- Admin content moderation must be scoped to policy violations and produce an audit record; moderation does not rewrite the submitted review history.

## 6.8 Worker profile changes and status

For low-risk edits, apply immediately. For legal name, identity documents, specialties, and service areas, create a pending change that must be reviewed; the current approved values remain public/eligible until the new values are approved. Phone change requires ownership verification. A worker whose account becomes temporarily or permanently suspended is hidden from discovery and cannot submit offers; the appropriate limited actions for existing bookings remain available according to policy.

## 6.9 Lost-phone recovery

Lost-phone recovery is not a self-serve phone swap. Open a formal recovery case; collect trustworthy identity evidence and role-appropriate supporting evidence; verify the new phone with OTP; record the case reason, evidence checklist, reviewer, and decision; change the phone only after manual approval; force a password reset; and audit the change. Deny the case if evidence is insufficient. Recovery of an admin's lost MFA phone is handled by a technical administrator through independent out-of-band identity verification, OTP to the new number, manual number update, and audit logging.

---

# 7. API contract plan

The UI can use Next.js Route Handlers or another internal REST layer, but endpoints must call the same domain services. Do not duplicate business logic in page handlers, server actions, background jobs, or worker-only code.

## 7.1 API conventions

- JSON request/response; TypeScript DTO/input schemas validated server-side.
- Return stable error codes and localized messages at the client. Do not expose raw database errors.
- Every protected endpoint uses server-side authentication, role, current-account-state, ownership, and entity-state checks.
- Use idempotency keys for retry-prone state transitions such as offer submission, accept, cancel, completion, complaint submission, and admin decisions.
- Use a version field or `If-Match`-style guard for offer/profile/request updates to prevent silent overwrite.
- Pagination must be bounded. Avoid returning ID documents, complaint evidence, internal notes, passwords, OTPs, HMAC keys, raw fingerprint values, or private contact fields by default.

## 7.2 Endpoint groups (illustrative)

| Group | Representative actions |
|---|---|
| Auth | Register customer/worker, verify phone OTP, sign in with selected role, request/reset password, resend OTP, sign out |
| Account recovery | Open lost-phone case, upload evidence, submit new-phone OTP, admin review/approve/reject |
| Worker | Get/edit profile, submit application, request sensitive profile changes, upload work photos, view application status |
| Discovery | List/search workers; filter by approved specialty and area; display public profile and review summary |
| Catalog | Read active specialties/areas; admin create/edit/disable catalog values and map sub-areas |
| Requests | Create/list/detail/edit/close/reopen request; manage time periods and optional photos |
| Offers | Create one offer per request, edit offer, withdraw, list offers, explicit reconfirmation |
| Booking | Accept offer, list/detail bookings, cancel with reason, request/approve/reject reschedule, report no-show |
| Completion/reviews | Mark complete, report problem, submit/view review, admin moderate review |
| Complaints | Submit complaint/evidence, worker response, answer information request, admin outcome/extension |
| Notifications | List notifications, open list/mark displayed items read, mark one read if needed |
| Admin | Worker review, catalog management, complaint/no-show queues, warnings/suspensions, deletion review, audit search |
| Internal worker | Process pg-boss jobs and reconciliation tasks; not exposed as public endpoints |

## 7.3 Error responses to standardize

Examples: `UNAUTHENTICATED`, `FORBIDDEN`, `ACCOUNT_SUSPENDED`, `WORKER_NOT_APPROVED`, `REQUEST_EXPIRED`, `OFFER_NOT_OPEN`, `OFFER_STALE`, `BOOKING_TIME_CONFLICT`, `WORKER_CAPACITY_REACHED`, `PERIOD_INVALID`, `RESCHEDULE_EXPIRED`, `CANCELLATION_WINDOW_CLOSED`, `OTP_EXPIRED`, `OTP_ATTEMPTS_EXCEEDED`, `RATE_LIMITED`, `VERSION_CONFLICT`, `FILE_NOT_AUTHORIZED`, and `VALIDATION_ERROR`. Map each code to a user-friendly Arabic/English message.

---

# 8. Authentication, permissions, and account recovery

## 8.1 Password rules

- Minimum password length: 12 characters.
- Allow passphrases; do not impose arbitrary character-composition rules.
- Block common/compromised passwords where feasible.
- Hash with Argon2id or a reviewed equivalent with an appropriate cost configuration.
- Use generic authentication errors so the endpoint does not reveal whether a phone/role account exists.
- Apply rate limits by phone/account and IP with progressive temporary delays rather than permanent account locks. Reset/recovery must be role-specific.

## 8.2 OTP policy

- Six-digit code.
- Valid for 3 minutes.
- Maximum 3 verification attempts per code.
- Resend cooldown: 90 seconds.
- Strict configurable limits by normalized phone and IP, including across resend/verification attempts.
- Invalidate on success or expiry; store a secure verifier/hash rather than reusable plaintext.
- Audit abuse/security events without logging the OTP value.
- Use the same base OTP constraints for admin SMS MFA, plus admin-specific rate limiting and audit.

## 8.3 JWT session implications

Auth.js JWT session lifetime is at most 24 hours. Do not implement a database `sessionVersion` lookup on every request. A valid JWT is not sufficient by itself: for every protected operation, check current account state and domain authorization in the database/service layer. Account restrictions can have delayed effect for purely token-derived UI state until token expiry, so public visibility and all protected writes must reflect the current database state.

## 8.4 Admin access

Admin accounts are provisioned by technical admin outside the dashboard. Admin login is password plus SMS OTP, candidate provider Twilio. Because SMS can be delayed and is exposed to SIM-swap risks, rate-limit and audit it and require a documented emergency-recovery runbook. A technical admin—not the account owner acting alone—handles lost admin MFA phone with out-of-band identity verification, OTP to new phone, manual number change, and audit entry.

---

# 9. Notifications, background jobs, and deadlines

## 9.1 Notification behavior

- Routine and time-sensitive business notifications are in-app only; no routine SMS/email/push.
- Notifications are stored as template key + parameters and rendered in the recipient's current language; Arabic uses RTL.
- When the notification list is opened, notifications included/displayed in that list are automatically marked read.
- Ordinary notification records are purged after six months. Audit logs, complaint records, and other governed data are not treated as ordinary notifications.
- Insert notification records with the state transition transaction where possible, or via a transactional outbox so a committed state change cannot lose its required notification.

## 9.2 Deadline inventory

| Event | Deadline/trigger | Required behavior |
|---|---|---|
| Request expiry | 48 hours after creation | Close even if selected service date is farther in the future |
| Last selected period ended | End of last period, if no confirmed booking | Close request |
| Reconfirmation window | Up to 2 hours after reopen | End earlier if original request lifetime expires or no future period remains |
| Cancellation | Before service period start only | Reject after start; use incident path instead |
| Reschedule response | Earlier of 12 hours after request or original period start | Expire pending change; original booking stays confirmed |
| Worker completion, customer silent | 48 hours after worker marks complete | Auto-complete if allowed by transactional ordering rules |
| Worker never marks completion | 48 hours after period end | Auto-complete independently of any filed report |
| No-show/service-not-performed report | Within 48 hours after service period end | Create admin review; does not suppress auto-completion timer |
| Complaint worker response | 24 hours after complaint | Admin may proceed if no response; no automatic penalty |
| Additional complaint info | 7 days after request, unless documented extension | Missing answer → Inconclusive; no automatic penalty |
| Complaint resolution target | 3 days after all required information is complete | Track operational target, not automatic guilt/closure |
| Review | Up to 14 days from completion | One final review; no reminder |
| OTP expiry | 3 minutes | Reject expired code |
| OTP resend | 90-second cooldown | Reject early resend |
| JWT session | At most 24 hours | Reauthentication after expiry |
| Normal notification retention | 6 months | Purge ordinary notifications |
| Identity fingerprint | 6 months from permanent worker suspension | Purge after retention period, including after deletion request within period |

## 9.3 Required scheduled/reconciliation jobs

Implement idempotent jobs for request expiry; request closure at last-period end; offer expiry; reconfirmation expiry; pending reschedule expiry; suspension end; auto-completion; complaint response and information deadlines; no-show review SLA visibility (not automatic closure); notification purge; identity fingerprint purge; deletion workflow; stale records and failed-job reconciliation.

Each job must:
- use a stable idempotency key or guarded state transition;
- re-read and lock the domain record before changing it;
- confirm the deadline is still applicable and not superseded;
- write the audit event and create required notifications;
- be safe to retry after partial failure;
- expose lag, retry count, last success, and error details without logging sensitive content.

---

# 10. Security, privacy, files, retention, and backups

## 10.1 Upload limits

| Purpose | Accepted types | Count/size |
|---|---|---|
| Service-request photos | JPEG, PNG, WebP | Up to 5 files, 5 MB each |
| Complaint evidence | JPEG, PNG, WebP | Up to 5 files, 5 MB each |
| Identity document | JPEG, PNG, PDF | Up to 10 MB per file |

Validate actual content, magic bytes and file size on the server; do not trust filename extensions or client MIME. Generate random storage names; never use the original filename as a path. Reject HTML, scripts and executable formats. Store outside the public directory. Authorize every download against role, ownership, and case/booking access. Consider image re-encoding and EXIF removal for user photos so hidden GPS metadata is not unnecessarily retained; document the behavior and test that it does not damage uploaded evidence.

## 10.2 Data access and audit

- Restrict national IDs, identity documents, complaint evidence, account-recovery evidence, and internal investigation notes to authorized reviewers and the relevant owner as the product policy permits.
- Never expose one party's private evidence or internal admin notes to the other party.
- Record important access to identity documents, evidence files, and admin decisions where proportionate and legally reviewed.
- Audit worker approval/rejection, sensitive profile changes, phone recovery, suspension/warning, complaint/no-show outcome, account deletion, file access where configured, and administrative changes to catalog and areas.
- Store secrets outside source control and container images. Rotate secrets with a documented procedure. The HMAC key must be separate from normal application data and include a key version field.

## 10.3 Account deletion and retention

- After approved deletion, deactivate access and remove the public profile immediately.
- Delete ID documents and private files once no legitimate operational/legal need remains.
- Keep only the minimum necessary booking, complaint, and audit information for the retention period approved by counsel; restrict access and pseudonymize where feasible.
- Retain the keyed identity fingerprint for six months after permanent worker suspension to deter replacement-account evasion, then purge it automatically.
- Treat identity documents, phone numbers, addresses, complaint evidence and HMAC fingerprints as high-risk data.

## 10.4 Backups and recovery

- Automate daily backups of the managed PostgreSQL database and server-local private files.
- Retain seven daily backups.
- Encrypt backups and store them off the Droplet, not on the same server/disk.
- Test restore and recovery monthly, including both database and private file storage, and document the recovery point and recovery time actually achieved.
- Before any production personal data is used, the backup destination, storage region, vendor terms, encryption, access, and retention must pass the legal/privacy gate.

## 10.5 Legal/privacy launch gate

Before real personal data is processed, obtain qualified Egyptian legal/privacy review covering: hosting and backup region; cross-border transfers; handling/retention of identity documents and complaints; consent and notice wording; deletion/retention requirements; breach response; vendor agreements; SMS sender registration and data use; and whether any permit or additional obligation applies. Until cleared, development/testing uses synthetic or appropriately anonymized data. This plan is not a legal opinion.

---

# 11. Implementation roadmap

Phases should be delivered as vertical slices with server rules, UI, tests, localization and operational support. Dates and estimates are provisional until team capacity is known.

| Phase | Scope | Exit criteria |
|---|---|---|
| P0 — Baseline freeze | Apply decisions 1–59; keep #60 open; confirm data fields, error codes, legal checklist and acceptance matrix | No unresolved ambiguity blocks the schema/transaction design; open items are assigned |
| P1 — Repository foundation | Next.js App Router, TypeScript, Tailwind, lint/typecheck/test, environment schema, i18n/RTL skeleton, Docker Compose | Fresh clone starts locally; CI checks pass; Arabic/English layout works |
| P2 — Database and domain core | Prisma schema, SQL migrations, audit pattern, test fixtures, worker lock/booking exclusion constraint, time helper | Real PostgreSQL integration tests prove conflicts and capacity behavior |
| P3 — Authentication | Auth.js Credentials, phone normalization, customer/worker role selection, password hash, OTP/rate limit, JWT 24h, recovery skeleton | Role-specific accounts work for same phone; generic errors; OTP tests pass |
| P4 — Admin/catalog foundation | Admin provision process, admin MFA, specialties/areas/period catalogue, audit views | Admin changes require authorization/reason and are audited |
| P5 — Worker onboarding and directory | Worker registration, private ID upload, application review/resubmission, approved profile, public discovery | Unapproved/suspended/deleted workers hidden; pending sensitive edits preserve old approved values |
| P6 — Requests and periods | Create/edit/close, optional photos, Cairo-time validation, 48h expiry, matching/filtering | Period and expiry edge tests pass; edit invalidates pending offers |
| P7 — Offers | One live offer per worker/request, price/start/duration, revision history, newest-first display, prior-offer reconfirmation | Offer interval fits one request period; revisions/notifications persist |
| P8 — Booking transaction | Acceptance, capacity 7, exclusion constraint, cancellation, rescheduling, contact visibility | Concurrency suite proves no overlapping active bookings and no eighth active booking |
| P9 — Completion and reviews | Worker completion, both auto-completion paths, 14-day final review, permanent rating block for reported no-show/service-not-performed | Race tests cover timer versus admin review; reporting semantics match decisions 41/52/54/55 |
| P10 — Complaints and incidents | Categories, optional evidence, 24h worker response, 7-day info request, 3-day target, admin outcomes, no-show queue | No silent admin auto-closure; privacy boundaries and deadlines pass tests |
| P11 — Account governance | Warnings, temporary/permanent suspension, sensitive edits, lost-phone recovery, deletion, HMAC purge | All administrative decisions are audited; suspended workers disappear from directory |
| P12 — Notifications and worker process | pg-boss worker, durable jobs, reconciliation, in-app notifications, read-on-open behavior, 6-month purge | Jobs retry safely, lag is observable, late jobs do not permit late actions |
| P13 — Security and launch readiness | Backup/restore, upload audit, penetration/security review, performance, accessibility/RTL, legal gate | Release checklist passes; personal-data/legal gates cleared |

### Parallel work

Once P2 has stabilized entity/status definitions and the domain service contracts, frontend UI, backend domain services, admin workflows, translations, and test fixtures can progress in parallel. Do not parallelize multiple implementations of the booking state machine or independently edit database lock ordering.

---

# 12. Development backlog

Each ticket should have an owner, dependencies, test cases, localization requirements, and observable completion criteria. The epics below organize implementation; they do not override the detailed rules above.

| Epic | Work items |
|---|---|
| E01 Platform foundation | Repo/workspaces, Next.js shell, Tailwind/logical CSS, Arabic/English i18n, env validation, lint/typecheck/test CI, Docker Compose |
| E02 Data and domain | Prisma schema, SQL migration conventions, typed domain errors, clock abstraction, audit events, idempotency helper, transactional notification outbox |
| E03 Authentication | Phone normalization, role-aware account model, Auth.js Credentials, Argon2id, OTP verify/resend/rate limits, JWT max-age 24h, password reset, lost-phone cases |
| E04 Admin and catalog | Admin provisioning, SMS MFA, specialty/area CRUD, bilingual fields, review queues, audited admin actions |
| E05 Worker onboarding | Application form, private ID upload, specialty/area selection, approve/reject/resubmit, gated profile edits, HMAC check and restricted matching workflow |
| E06 Worker directory | Public profile, search, approved filters, Arabic normalization, rating/count display, exclusion of suspended/deleted accounts |
| E07 Service requests | Create/edit, period builder/validation, optional photos, expiry, last-period closure, no-match experience, reopen/close decision |
| E08 Offers | One active offer, price/time/duration validation, revision history, new/updated offer notification, reconfirmation window |
| E09 Booking transaction | Acceptance transaction, conflict exclusion constraint, worker-capacity lock, max 7, contact privacy, cancellation reasons, booking changes |
| E10 Completion and reviews | Manual completion, 48h timeout, fallback auto-completion, 14-day review, moderation, permanent report rating block |
| E11 Complaints and no-show | Categories, file evidence, worker response deadline, information requests, extension, three outcomes, no-show review, report/complete race handling |
| E12 Notifications and background jobs | pg-boss process, deadline jobs, retries, reconciliation, mark displayed notifications read, 6-month notification cleanup |
| E13 Account governance | Warnings, suspensions, restricted existing-booking access, deletion/retention, HMAC expiry, admin lost-MFA recovery |
| E14 Security and operations | File validation, protected downloads, logs/metrics, backup database/files, tested restore, secret rotation, deployment hardening |
| E15 Quality and launch | Unit/integration/e2e tests, accessibility, responsive viewports, security review, legal/privacy checklist, runbooks |

## 12.1 Minimum acceptance checks for the highest-risk tickets

- Two concurrent offer-acceptance requests for the same worker and overlapping times cannot both commit.
- Eight concurrent attempts to accept offers for a worker with six active confirmed bookings cannot produce more than seven.
- Two back-to-back bookings are permitted; any real overlap is rejected.
- Request edit invalidates open offers and makes stale acceptance fail.
- A pending reschedule does not block another worker booking into the proposed new time; if the requested slot is no longer free when approved, the booking's original time remains.
- Request reopening does not reset the 48-hour lifetime and cannot offer on expired periods.
- No-show report before auto-completion: admin decision closes booking/releases slot and no rating is allowed. Auto-completion before admin confirmation: keep `Completed`, preserve completed reporting count, log violation separately, and keep rating blocked.
- Missing complaint response alone does not prove a violation or trigger an automatic penalty.
- Suspended/deleted worker profile disappears from the public directory without waiting for JWT expiry.
- Private identity/evidence file URLs cannot be fetched without the required authorization.

---

# 13. Testing and acceptance strategy

## 13.1 Test layers

1. **Unit tests:** period validation; `Africa/Cairo` conversions; deadline calculators; eligibility policies; transition guards; OTP attempts/cooldown; review eligibility; complaint deadline math.
2. **Database integration tests:** real PostgreSQL (not mocks) for Prisma migrations, exclusion constraint, worker-row locking, active-offer uniqueness, transaction rollbacks, idempotency, and audit/outbox behavior.
3. **API/domain integration tests:** authorization, state conflicts, cancellation reason enforcement, private-file access, account recovery, admin-only operations.
4. **End-to-end tests:** customer, worker and admin journeys in both Arabic and English across mobile, tablet and desktop viewports.
5. **Job tests:** pg-boss retries, duplicate delivery, worker restart, delayed job, expired JWT, deadline reconciliation, retention purge, and completion/no-show race.
6. **Security tests:** role separation for same phone, IDOR on requests/bookings/files, OTP brute-force/rate limits, generic login errors, upload MIME spoofing, privilege escalation, secret leakage and audit integrity.
7. **Operational tests:** backup restore for DB and file volume, disk-full behavior, worker downtime, pg-boss backlog and post-recovery reconciliation.

## 13.2 Critical business test cases

| ID | Scenario | Expected result |
|---|---|---|
| T01 | Same phone has customer + worker accounts | Separate passwords/profiles/role state; login role selects correct account |
| T02 | Wrong role chosen or account absent | Generic error without leaking account existence |
| T03 | Invalid custom service period | Server rejects wrong date, reversed range, overlap, duration outside 1–12h, or started/expired period |
| T04 | Offer duration exceeds selected period | Reject even if start time is inside the period |
| T05 | Concurrent overlapping acceptance | At most one confirmed interval; loser gets stable conflict response |
| T06 | Capacity at 6, two concurrent acceptances | Exactly one may become the seventh; no eighth booking |
| T07 | End equals next start | Both bookings allowed |
| T08 | Request edited while offers exist | All pending offers invalidated; workers notified; stale acceptance rejected |
| T09 | Booking cancellation at/after period start | Before start: reason required and cancel; at/after start: ordinary cancellation rejected |
| T10 | Reschedule left pending past deadline | Change expires; original booking remains confirmed |
| T11 | Reschedule approved after slot conflict appears | Change rejected; original appointment persists |
| T12 | Worker marks complete; customer silent | Auto-complete after 48h; review window starts from completion rule as defined |
| T13 | Worker never marks complete; a report may or may not exist | Auto-complete 48h after service window ends, independently of reports |
| T14 | No-show reported before auto-completion | Timer still runs, but if admin review commits first it closes booking/releases capacity; no rating |
| T15 | Auto-completion commits before no-show confirmation | Keep Completed; completed reporting count remains; log violation separately; rating stays blocked |
| T16 | Ordinary complaint is open | Complaint proceeds independently; rating may be submitted within 14-day window unless no-show/service-not-performed report exists |
| T17 | Worker fails to respond within 24h | Admin can continue; no automatic penalty/violation finding |
| T18 | Additional info not provided within 7 days | Inconclusive, no automatic penalty, unless admin records extension |
| T19 | Notification list opened | Displayed notifications are marked read; ordinary notifications purge after 6 months |
| T20 | Sensitive profile edit pending | Old approved specialty/area/legal-name data stays approved/live; new change not active |
| T21 | Worker suspended | Immediately hidden from directory; no new offers; account status checked server-side |
| T22 | ID/evidence file requested without permission | Deny; no public URL or storage key exposure |
| T23 | OTP reused/expired/4th attempt | Reject; record security event without storing plaintext OTP |
| T24 | Lost-phone evidence insufficient | Recovery denied; phone remains unchanged |

## 13.3 Quality gates

- TypeScript strict mode; lint and formatting clean; no failing unit/integration/e2e tests on the release branch.
- Database migrations apply from an empty database and from the previous production schema in a staging clone.
- Arabic RTL/English LTR, keyboard navigation, responsive layout, and form validation tested on real browser engines.
- No critical/high security findings unresolved; secrets not committed; dependency audit reviewed.
- Restore test passed and backup copies exist off-host.
- Legal/privacy gate is signed off before real personal data is processed.

---

# 14. Deployment, observability, and operations

## 14.1 Environments and deployment

- **Local:** Docker Compose for PostgreSQL and app dependencies; synthetic seed data only.
- **CI/test:** isolated database, automated migrations, unit/integration tests, static checks, test files only.
- **Staging:** production-like Docker setup; synthetic or explicitly approved anonymized data; test SMS routing separated from production.
- **Production:** DigitalOcean Droplet with Docker Compose for Next.js web and worker; separate managed PostgreSQL; private persistent file volume; HTTPS reverse proxy; secrets supplied outside images/source.

Deploy an immutable image/tag; run migrations as an explicit release step; back up before destructive migrations; verify worker health and queue backlog after deploy; support rollback of application images. Database migrations that cannot be rolled back safely require a forward-fix plan.

## 14.2 Observability

Capture structured logs with correlation/request IDs and no sensitive payload dumps. Monitor at minimum:

- web and worker process health;
- PostgreSQL connectivity, connection pressure, slow queries and disk/backup status;
- pg-boss queue depth, job lag, retry/failure counts and oldest due deadline;
- auto-completion/request-expiry/suspension/notification purge job lag;
- OTP send/failure rates by provider and carrier where available, without logging codes;
- 4xx/5xx rates, booking conflict rejections, idempotency replays and upload rejections;
- storage disk usage and failed private-file access attempts;
- backup freshness and last successful restore drill.

Set alerts for expired-deadline jobs beyond the service objective, repeated OTP delivery errors, failed backups, unusual auth failures, worker process down, disk capacity risk and unexpected changes to admin privileges.

## 14.3 Backups and incident runbooks

Daily encrypted off-host backups of the managed DB and private files; retain seven daily backups; monthly restore drill. Document steps for SMS outage, worker downtime, leaked admin credential, ID/evidence exposure, lost-file volume, database restore, and suspected account takeover. Incident runbooks must specify who can act, how to preserve audit evidence, how to notify affected people where required, and how to obtain legal guidance.

---

# 15. Risks, gates, and remaining decisions

## 15.1 Risk register

| Risk | Mitigation | Gate/owner |
|---|---|---|
| Race condition causes overlapping jobs or an eighth booking | PostgreSQL exclusion constraint, worker capacity lock, concurrent integration tests | Engineering lead |
| Auto-completion/no-show ordering produces inconsistent status | Explicit transaction-order rule and race tests in §6.5 | Engineering + product owner |
| JWT remains usable until expiry after account state changes | Check current account state on every protected operation; hide suspended/deleted profile immediately | Engineering |
| SMS OTP delayed or undeliverable on Egyptian networks | Test Twilio candidate, sender registration, carrier delivery, cost and fallback | Product/operations/vendor |
| Local file volume lost or inaccessible to worker | Shared persistent volume, off-host encrypted file backups, restore test | Operations |
| ID document or complaint evidence exposed | Private storage, authorization checks, least privilege, access audit, security test | Security/privacy owner |
| HMAC retention may create legal/privacy obligations | Counsel review of purpose, input, key handling, access and 6-month retention | **[LEGAL-GATE]** |
| Cross-border storage/backups or vendor processing not approved | Legal review of regions, data flows, contracts, notice/consent and retention | **[LEGAL-GATE]** |
| In-app-only deadlines are missed by users | Explicitly accept late-response risk; reliable server deadlines and clear in-product countdowns; no silent deadline changes | Product owner |
| Question 60 price-change behavior not decided | Keep implementation behind a documented product decision; do not invent approval workflow | Product owner |
| Team capacity/estimates inaccurate | Re-estimate after foundation/database phase based on team and test velocity | Project owner |

## 15.2 Legal/privacy gate checklist

Before real-data launch, obtain written review/approval for:
- processing and storing ID/national-ID documents and the identity fingerprint;
- six-month HMAC retention after permanent worker suspension;
- complaint evidence and internal investigation data;
- hosting region, managed DB region, backup region and cross-border transfers;
- privacy notice, consent/legal basis, account deletion, mandatory records and retention schedule;
- provider contracts and data processing terms for hosting, database, backups and SMS;
- required registrations/permits/incident and breach procedures under applicable Egyptian law.

## 15.3 Explicit open business decision

**Decision 60 — price change after inspection / extra work — OPEN.** Do not implement any one of the following until the owner chooses:
- **A. Additional-work approval:** worker explains extra work and revised total; customer explicitly approves before additional work; if declined, original scope/price remain if feasible, otherwise support/complaint path.
- **B. Full renegotiation:** pause work until customer accepts a newly agreed total price; without agreement use support/complaint path.
- **C. No in-booking price change:** accepted price remains final; any extra work is outside the platform or requires a new request.

Until answered, the booking keeps an immutable snapshot of its accepted price. Any price-change endpoint, UI, notification, and related workflow is blocked from implementation. The ordinary complaint category “agreed price mismatch” remains available for disputes under the current product rules.

Other engineering details (exact error-code naming, typography, table/column names, selected Twilio account configuration, operational SLO values, and detailed deployment sizing) may be finalized by engineering as long as they do not change an approved business behavior or pass through the legal gate unresolved.

---

# 16. Definition of Ready / Definition of Done

## 16.1 Definition of Ready

A ticket is ready only when:
- its user role and business outcome are clear;
- its state transitions and deadline behavior are defined;
- API/input validation and authorization rules are described;
- data migration/index/transaction implications are known;
- Arabic/English labels, errors, and RTL/LTR behavior are identified;
- accessibility and responsive cases are included;
- acceptance tests cover normal, invalid, expired, concurrent, and unauthorized paths as relevant;
- any legal/privacy dependency is marked and assigned;
- the ticket does not depend on unresolved decision 60 unless it implements a separate unrelated capability.

## 16.2 Definition of Done

A ticket is done only when:
- domain rules live in a shared server-side module and are not duplicated inconsistently;
- authorization and server validation are implemented;
- audit/notification behavior is implemented for applicable transitions;
- database constraints and transactional behavior are covered by real PostgreSQL tests;
- unit/integration/e2e tests pass, including edge cases and idempotent retries;
- Arabic and English experiences are complete and reviewed at mobile/tablet/desktop sizes;
- errors are localized and do not leak internal details;
- logs/metrics are adequate without exposing sensitive data;
- docs, environment variables, migrations, and runbooks are updated;
- security/privacy requirements and applicable release gates are satisfied.

---

# 17. Decision traceability — business decisions 1–59

This index confirms that the agreed decisions have been carried into the implementation baseline. Detailed operative wording lives in the sections cited below.

| Decision | Approved behavior captured in this plan | Main section |
|---:|---|---|
| 1 | Worker completion starts 48h customer response window; silence auto-completes; complaints are independent | §2.6, §6.5 |
| 2 | No-show/service-not-performed report within 48h after service period end | §2.6, §6.6, §9.2 |
| 3 | Manual report with written description, incident time, optional evidence; no dedicated arrival/contact logs | §2.6, §6.6 |
| 4 | Routine/time-sensitive notification is in-app only; deadline still enforced | §2.8, §9 |
| 5 | Lost-phone recovery is manually reviewed with identity/role evidence; deny if insufficient | §2.2, §6.9, §8 |
| 6 | Next.js App Router, TypeScript, Tailwind, PostgreSQL, Prisma, modular monolith, pg-boss worker, Docker-capable hosting | §1.1, §3 |
| 7 | Auth.js Credentials custom flow backed by Prisma; role-aware phone login | §1.1, §3.3 |
| 8 | Twilio is preferred SMS candidate, subject to Egypt delivery/sender/cost/legal checks | §1.1, §8.4, §15 |
| 9 | Private local filesystem outside public root, shared with worker and backed up | §1.1, §3.5, §10.4 |
| 10 | DigitalOcean Droplet + Docker Compose, web/worker on same host | §1.1, §3.6, §14 |
| 11 | Separate managed PostgreSQL in production; verify cost/region/recovery before launch | §1.1, §3.6, §10.5 |
| 12 | Cancellation leads to Awaiting Customer Decision; reopen only within original lifetime and with a future period | §2.3, §6.1 |
| 13 | All requests expire 48h after creation even for future service dates | §2.3, §9.2 |
| 14 | Temporary/permanent suspended and deleted workers hidden immediately from public directory | §2.2, §2.7, §8.3 |
| 15 | Low-risk profile edits immediate; legal name/ID/specialties/areas reviewed; old approved values stay active; phone changes verified | §2.2, §6.8 |
| 16 | Review within 14 days; one notification at completion; no reminder | §2.6, §6.7, §9.2 |
| 17 | Keyed HMAC, six months from permanent suspension; key separate; legal review | §2.7, §5.1, §10.3 |
| 18 | Legal/privacy gate before real data; synthetic data until approved | §10.5, §15.2 |
| 19 | JWT max 24h; no per-request sessionVersion lookup; protected actions validate current state | §1.1, §3.3, §8.3 |
| 20 | Custom periods 1–12h, same date, non-overlap, valid future time, Africa/Cairo | §2.3, §5.3 |
| 21 | OTP 6 digits, 3 min, 3 attempts, 90s resend, strict phone/IP rate limits | §8.2 |
| 22 | Complaint worker-response deadline 24h; admin may proceed; no auto penalty | §2.6, §6.6, §9.2 |
| 23 | Deactivate/hide profile, delete files when no longer needed, retain minimum legally required data restricted/pseudonymized | §2.7, §10.3 |
| 24 | Request edit only before acceptance; invalidates pending offers and notifies workers | §2.3, §6.1 |
| 25 | Notifications retained 6 months, displayed items marked read on opening list; audit separate | §2.8, §9.1 |
| 26 | Capacity slot releases at Completed, Cancelled, or explicit admin release after review; audit action | §5.2, §6.5–6.6 |
| 27 | Worker who cancelled booking cannot offer/reconfirm on that same reopened request | §2.3, §6.1 |
| 28 | Auth.js JWT lifetime 24h and server-side state checks; no sessionVersion every request | §3.3, §8.3 |
| 29 | Lost-phone process, new-phone OTP, reason log, number change after approval, force password reset | §6.9 |
| 30 | Password min 12; allow passphrases; block common/compromised where feasible; progressive rate limits | §8.1 |
| 31 | No-show report manual with optional evidence; admin review; no auto penalty | §2.6, §6.6 |
| 32 | Upload type/count/size limits, true type validation, random names, private storage, authorized downloads | §10.1 |
| 33 | Reschedule deadline earlier of 12h and original period start; explicit approval | §2.5, §6.4, §9.2 |
| 34 | No slot hold for pending reschedule; at approval, recheck atomically; if unavailable original remains | §2.5, §6.4 |
| 35 | One final review, cannot edit/delete; admin may moderate with audit | §2.6, §6.7 |
| 36 | Ordinary complaint does not itself block review; no-show/service-not-performed report does | §2.6, §6.7 |
| 37 | Admin MFA uses password + SMS OTP; same OTP restrictions; audit and document risks | §8.2, §8.4 |
| 38 | Admin lost MFA phone requires technical admin/out-of-band verification/new phone OTP/manual update/audit | §8.4 |
| 39 | Worker may cancel only before service period start; required reason; no automatic penalty merely for cancellation | §2.5 |
| 40 | Customer may cancel only before service period start; required reason; no automatic penalty merely for cancellation | §2.5 |
| 41 | If no-show review is decided before auto-completion: close/release slot for all outcomes and do not mark Completed/rate | §2.6, §6.5 |
| 42 | Only offers closed by another acceptance or the customer-cancelled offer may be reconfirmed; invalidated/withdrawn offers excluded; original terms or revised offer within capped 2h; cancelling worker excluded | §2.3, §6.1 |
| 43 | If no worker completion, auto-complete 48h after service window ends; later decision 52 confirms reports do not stop the timer | §2.6, §6.5 |
| 44 | No automatic closure of no-show review; capacity can remain occupied until explicit admin decision | §2.6, §6.6 |
| 45 | Additional info due in 7d; no response -> Inconclusive/no penalty; extension documented; 3-day target starts after required info complete | §2.6, §6.6, §9.2 |
| 46 | Permanent suspension applies to worker role; customer account not automatically suspended; HMAC anti-evasion | §2.7, §10.3 |
| 47 | UTC instants, Africa/Cairo local time, elapsed deadlines from actual timestamps | §5.3 |
| 48 | Pending specialty/area edit uses old approved values only; new values not active until approved | §2.2, §6.8 |
| 49 | Separate customer/worker passwords, profiles, role/session; sign-in by phone + role | §2.1, §3.3, §8 |
| 50 | One active offer per worker/request; editable while open/unaccepted, notify customer, retain revisions | §2.4, §6.2 |
| 51 | Acceptance immediately confirms booking after atomic server/database checks; no second worker approval | §2.4, §6.3 |
| 52 | Auto-completion timer still runs even if a no-show/service-not-performed report is filed | §2.6, §6.5 |
| 53 | Any such report permanently disables rating for the booking regardless of outcome | §2.6, §6.7 |
| 54 | If auto-completion already occurred before no-show confirmation, preserve Completed and record violation separately | §2.6, §6.5 |
| 55 | Every Completed record counts in completion reports even if a no-show is later confirmed | §2.6, §6.5 |
| 56 | Daily DB + local file backups, seven daily copies, encrypted off-host; monthly restore test | §10.4, §14.3 |
| 57 | Any overlapping confirmed interval conflicts; `[start,end)` allows back-to-back; acceptance rechecks atomically | §2.4, §5.2–5.3 |
| 58 | Offer has start + estimated duration; system calculates end; interval fits within one selected period | §2.4, §5.2 |
| 59 | Duration range 15min–12h in 15min increments; system-computed end; reject if not fitting | §2.4, §5.2 |

---

## Final implementation instruction

Treat this version as the consolidated planning baseline for decisions 1–59. Before implementation of price adjustment/extra work, obtain the owner's answer to **Decision 60**. Do not invent a resolution for it in the UI, database behavior, API, or backlog. Before production launch with real personal data, complete the legal/privacy gate and verify the selected hosting, SMS, and backup setup.
