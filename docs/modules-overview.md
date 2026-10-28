# Modules Overview

This document defines module responsibilities and boundaries, not implementation order. Verify the actual repository before assuming all modules exist.

## 1. Accounts and Authentication

**Responsibilities**
- Customer and worker registration and login.
- Phone ownership verification and recovery.
- Role-specific account identity and access control.

**Boundaries**
- Server-side authentication and authorization remain mandatory.
- A person needing both roles uses separate accounts.
- The same phone may be used across role-separated accounts only under the approved verification rules.

## 2. Worker Profiles and Verification

**Responsibilities**
- Worker profile data, specialties, service areas, and optional work samples.
- Identity document submission, admin review, rejection, and resubmission.
- Admin-managed specialty and area mappings.

**Boundaries**
- Identity documents are sensitive and must be protected.
- Only approved specialties may be offered.
- Worker approval status is controlled by the backend/admin workflow.

## 3. Service Requests

**Responsibilities**
- Customer request creation, editing, and closure.
- Request description, area, optional images, and required time period.
- Request lifecycle and post-no-show decision.

**Boundaries**
- Editing is permitted only before an offer is accepted, according to the approved rule.
- Urgent requests are outside the MVP.
- Validate request state transitions on the server.

## 4. Offers

**Responsibilities**
- Worker offer submission and customer comparison.
- Offer price and proposed time details.
- Offer acceptance and state transitions.

**Boundaries**
- Offers should match the customer's requested time period.
- Recheck worker availability when accepting an offer.
- Enforce the active-job cap and prevent confirmed schedule conflicts atomically.

## 5. Bookings and Scheduling

**Responsibilities**
- Confirmed booking details, schedule conflicts, time-change requests, and cancellation.
- Worker delay/no-show flow.
- Customer approval of changed appointment or price before additional work.

**Boundaries**
- Changes requiring the other party's approval must not apply prematurely.
- Concurrency must be considered for overlapping bookings and capacity checks.
- A cancellation reason and incident record are required where specified by business rules.

## 6. Reviews

**Responsibilities**
- Customer ratings and reviews associated with completed jobs.

**Boundaries**
- Enforce eligibility on the server.
- Prevent unauthorized or duplicate review behavior according to the final detailed rules.

## 7. Complaints and Administration

**Responsibilities**
- Complaint creation, categorization, evidence handling, investigation workflow, and outcome notification.
- Admin handling of worker verification, specialties, areas, and complaints.

**Boundaries**
- Restrict access to private evidence and internal investigation data.
- Notify both parties of outcomes without exposing protected internal details.

## 8. Notifications and Background Jobs

**Responsibilities**
- Asynchronous notifications and other approved background work.
- Retry handling and job status visibility.

**Boundaries**
- Use pg-boss and the approved worker architecture.
- Make retries safe where practical and minimize personal data in job payloads.

## 9. Shared Contracts

**Responsibilities**
- Cross-application request/response types and suitable Zod schemas.

**Boundaries**
- Avoid framework-specific coupling unless explicitly approved.
- Keep API contracts and frontend usage aligned.
