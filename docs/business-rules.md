# Business Rules

This document is the authoritative home for approved business behavior. Keep rule identifiers stable so implementation and tests can reference them. The summary below captures the currently available approved decisions; review prior project decisions with the user before filling any gaps or changing meaning.

## 1. Product Scope

- **BR-001:** The MVP supports ordinary service requests. Urgent requests are deferred.
- **BR-002:** The initial service area is Cairo and Giza, Egypt.
- **BR-003:** The MVP is a responsive website for desktop, tablet, and mobile. Native mobile applications are deferred.
- **BR-004:** The product supports Arabic and English; Arabic uses RTL and English uses LTR.

## 2. Accounts and Roles

- **BR-010:** The system has customer and worker account roles; each account has exactly one role.
- **BR-011:** A person who needs both roles must use separate accounts.
- **BR-012:** The same phone number may be used for separate customer and worker accounts, subject to phone ownership verification.
- **BR-013:** Login uses phone number, password, and the selected account type.
- **BR-014:** Phone ownership verification supports confirmation and account recovery.
- **BR-015:** Worker identity verification is required; a worker submits an ID image or an approved official alternative document.
- **BR-016:** Admin manually reviews worker identity submissions before worker account approval.
- **BR-017:** If a worker verification is rejected, the worker can edit details and resubmit.
- **BR-018:** Workers can edit their submitted profile data.

## 3. Worker Profiles and Administration

- **BR-020:** A worker profile includes personal information, offered services, and service area.
- **BR-021:** Work photos/details are optional and can help introduce a worker; work photos may also be supplied with an offer.
- **BR-022:** Workers may select multiple specialties, but only specialties approved/managed by admin are available for offering services.
- **BR-023:** Admin manages approved specialties.
- **BR-024:** Workers specify their service area; admin may map a submitted sub-area to a broader area.

## 4. Service Requests

- **BR-030:** Customers can browse workers and create service requests with a description and area.
- **BR-031:** Request images are optional in the MVP.
- **BR-032:** The customer selects a service time period; it is required.
- **BR-033:** Offers should match the customer's selected time period.
- **BR-034:** A request remains editable until the customer accepts an offer.
- **BR-035:** Following a worker no-show, the customer can decide whether to search again or close the request.

## 5. Offers, Scheduling, and Bookings

- **BR-040:** Workers submit offers containing a price and proposed appointment/time period.
- **BR-041:** Ordinary offers are shown newest to oldest and include price, proposed appointment, and worker rating so customers can compare.
- **BR-042:** Workers can submit offers to multiple requests, but may have at most seven accepted/active jobs.
- **BR-043:** Cancelling an accepted job frees an active-job slot; the customer is asked to provide a cancellation reason when applicable.
- **BR-044:** A worker may have multiple bookings only when their confirmed time intervals do not conflict.
- **BR-045:** Before confirming a booking, the system rechecks availability; the first valid confirmed booking reserves the time slot.
- **BR-046:** Conflicting acceptance must be rejected and communicated clearly.
- **BR-047:** A confirmed booking time change requested by either party applies only after the other party approves and availability is rechecked.
- **BR-048:** If a worker changes an accepted offer before arrival, such as the appointment time, customer approval is required.
- **BR-049:** If inspection changes the agreed price or additional work is proposed, customer approval is required before extra work proceeds.
- **BR-050:** Customer cancellation before service requires a reason, notifies the worker, and logs the incident. Cancellation alone does not automatically impose a penalty.
- **BR-051:** The worker may suggest a prepared time period or a custom range when needed.

## 6. No-Show and Delays

- **BR-060:** A worker is treated as a no-show only after the agreed arrival window ends and an additional 15-minute grace period passes, with an attempt to contact the worker.
- **BR-061:** A worker may request a delay and the customer may contact the worker.
- **BR-062:** If the worker does not arrive, the request can be offered/transferred to a nearby worker according to the approved flow.
- **BR-063:** Any rule treating a non-responsive customer as having completed a job must be clarified against the exact previously approved flow before implementation; do not infer this behavior from incomplete notes.

## 7. Reviews and Complaints

- **BR-070:** Ratings/reviews are associated with completed jobs.
- **BR-071:** Customers can file complaints using a category and required description; evidence files/images are optional.
- **BR-072:** Complaint categories include service quality, unfinished service, agreed-price mismatch, damage, inappropriate behavior, no-show, and other.
- **BR-073:** Both customer and worker are notified of the complaint outcome.
- **BR-074:** Internal investigation details and private evidence are not disclosed to the other party without an approved basis.

## 8. Deferred or Unresolved Decisions

- Urgent service requests are deferred to a future version.
- For any uncertain rule, contradictory note, or missing detail, consult the recorded project discussion and ask the user before implementation.
- Do not silently renumber or reinterpret existing approved business rules.
- Update `docs/requirements-traceability.md` when rules are added, changed, or implemented.
