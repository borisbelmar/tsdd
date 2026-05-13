# Case Study: Small Team — 2 to 3 Developers

> **Product:** `Reserva` — a SaaS appointment booking platform for small businesses.
> **Team:** 3 developers — Ana (frontend), Diego (backend), you (full-stack, coordinating).
> **Stakeholder:** Lucía, the product owner (also handles sales and support).
> **Stack:** Next.js (frontend), Go (backend API), PostgreSQL, Redis, Stripe.

## The Situation

Lucía wants to launch an MVP in 6 weeks. The core features:

1. Businesses can create an account and set up their booking page
2. Customers can view available time slots and book appointments
3. Businesses receive notifications for new bookings
4. Businesses can manage their schedule (working hours, time off, services)
5. Customers can cancel or reschedule bookings

Without coordination, Ana might build the booking UI assuming one data model while Diego builds a different one. Specs prevent this.

## What TSDD Looks Like Here

### Week 1: Proposal and Breakdown

You invoke `proposal-writer` with Lucía and the team. The proposal at `docs/proposal/reserva-mvp.md` defines:

- **Problem:** Small businesses (salons, consultants, tutors) lose bookings because they rely on phone calls or WhatsApp. They need a simple, self-serve booking page.
- **Solution:** A SaaS platform where businesses create a booking page, customers book online, and both sides get notifications.
- **Scope (MVP):** Account creation, booking page, schedule management, email notifications, Stripe for paid plans. Excludes: SMS notifications, team calendars, recurring bookings, integrations with Google Calendar.
- **Planned specs:**
  1. `001-business-onboarding.md` — signup, account setup, booking page creation
  2. `002-schedule-management.md` — working hours, services, time off
  3. `003-public-booking-page.md` — customer-facing booking UI
  4. `004-booking-engine.md` — slot calculation, conflict detection, booking creation
  5. `005-email-notifications.md` — confirmation and reminder emails
  6. `006-stripe-billing.md` — subscription plans and payment

The team reviews the proposal. Diego flags that specs 003 and 004 are tightly coupled — the booking page needs the booking engine's API. You sequence them: 004 first, then 003.

### Week 2: Writing Specs in Parallel

**You write spec 001 (business onboarding)** with `spec-writer`:

```markdown
# Spec 001: Business Onboarding

## Context
New businesses need to create an account, configure their profile,
and get a public booking page URL.

## Scope
- Signup with email + password (no OAuth for MVP)
- Email verification required before booking page goes live
- Profile setup: business name, description, timezone, logo
- Booking page URL: `reserva.app/{business-slug}`
- Default booking page shows "Coming soon" until schedule is configured

## Out of scope
- Custom domains
- Team member invitations
- Multi-location support

## Files to modify
- `app/signup/page.tsx` — signup form (Ana)
- `app/verify-email/page.tsx` — email verification (Ana)
- `app/[slug]/page.tsx` — public booking page stub (Ana)
- `internal/auth/signup.go` — signup handler (Diego)
- `internal/auth/verify.go` — email verification (Diego)
- `internal/db/schema/businesses.go` — business schema (Diego)
- `internal/db/queries/businesses.go` — business queries (Diego)

## Acceptance criteria
- [ ] User can sign up with email + password
- [ ] Verification email is sent with a working link
- [ ] Unverified accounts show "verify your email" message
- [ ] Verified accounts get a booking page at `/slug`
- [ ] Booking page shows "Coming soon" until schedule is configured
- [ ] Business can edit profile (name, description, timezone, logo)
- [ ] Slug is URL-safe and unique
```

**Diego writes spec 004 (booking engine)** with `spec-writer`:

```markdown
# Spec 004: Booking Engine

## Context
The core logic that calculates available time slots, detects conflicts,
and creates bookings. This is the backend service the public booking
page (spec 003) will consume.

## Scope
- Calculate available slots based on business schedule (spec 002)
- Detect and prevent double-booking
- Create booking with customer info, service, and time slot
- Cancel booking (soft delete, sends cancellation notification)
- Reschedule booking (cancel old + create new, atomic)
- API endpoints: GET /slots, POST /bookings, DELETE /bookings/{id}, PUT /bookings/{id}/reschedule

## Out of scope
- Waitlist for fully booked slots
- Buffer time between appointments
- Recurring bookings

## Files to create
- `internal/booking/engine.go` — slot calculation and conflict detection
- `internal/booking/engine_test.go` — comprehensive test coverage
- `internal/api/handlers/bookings.go` — API handlers
- `internal/db/schema/bookings.go` — booking schema
- `internal/db/queries/bookings.go` — booking queries

## Dependencies
- Requires spec 002 (schedule management) to be done — slot calculation needs schedule data

## Acceptance criteria
- [ ] GET /slots returns available 30-min slots for a given date
- [ ] Slots respect business working hours and time off
- [ ] POST /bookings creates a booking and marks slot as unavailable
- [ ] Double-booking the same slot returns 409 Conflict
- [ ] DELETE /bookings/{id} soft-deletes and frees the slot
- [ ] PUT /bookings/{id}/reschedule is atomic (old cancelled, new created)
- [ ] All operations are timezone-aware
```

The team reviews both specs. Ana notes that spec 001's booking page stub needs to know the business slug format — you add it to the spec: "Slug format: lowercase, alphanumeric, hyphens allowed, 3-30 characters."

Specs are saved. Index updated.

### Week 3-4: Implementation

**Diego implements spec 004** (booking engine). He invokes `spec-implement`. While implementing, he discovers that the slot calculation needs to account for the service duration, not just fixed 30-minute blocks. The spec assumed 30-min slots, but services can be 30, 60, or 90 minutes.

He doesn't silently change it. He updates the spec:

```markdown
## Technical notes
- Updated: slots are service-duration-aware, not fixed 30-min blocks.
  A 60-min service shows 60-min slots, a 90-min service shows 90-min slots.
  This was discovered during implementation — the original spec assumed
  fixed 30-min slots which didn't match the service model from spec 002.
```

He commits:

```
feat(booking): implement slot calculation, conflict detection, and CRUD

- Service-duration-aware slot calculation (not fixed 30-min blocks)
- Atomic reschedule (cancel old + create new in single transaction)
- Timezone-aware slot generation
- Comprehensive test coverage for edge cases

Spec: docs/specs/004-booking-engine.md
```

**Ana implements spec 003** (public booking page) using Diego's API. She invokes `spec-implement` and sees that the API returns slots in a slightly different format than the spec described. She checks the actual API response, updates her implementation to match reality, and updates spec 003's technical notes to reflect the actual API contract.

**You implement spec 001** (business onboarding) in parallel. No conflicts — the specs are well-bounded.

### Week 5: The Integration Moment

Spec 005 (email notifications) touches code from multiple specs. You invoke `spec-writer` and make sure the spec lists all the files it will touch:

```markdown
# Spec 005: Email Notifications

## Context
Send confirmation emails for bookings, cancellations, and reminders
24 hours before appointments.

## Files to modify
- `internal/email/templates/booking-confirmed.html` — new template
- `internal/email/templates/booking-cancelled.html` — new template
- `internal/email/templates/booking-reminder.html` — new template
- `internal/email/sender.go` — new email service
- `internal/booking/engine.go` — hook into booking creation (from spec 004)
- `internal/api/handlers/bookings.go` — trigger cancellation email (from spec 004)
- `internal/scheduler/reminders.go` — new cron job for 24h reminders

## Acceptance criteria
- [ ] Booking confirmation email sent immediately after booking
- [ ] Cancellation email sent when booking is cancelled
- [ ] Reminder email sent 24 hours before appointment
- [ ] Emails include business name, service, date, time, and booking link
- [ ] Failed email sends don't break the booking flow (async, best-effort)
```

Diego implements it. The spec references files from his spec 004 work — no surprises because the spec listed them upfront.

### Week 6: Final Specs and Launch

Spec 006 (Stripe billing) is implemented. The spec index shows all specs done:

```markdown
# Specs Index

| ID   | Title                  | State | Layer      | Owner  |
|------|------------------------|-------|------------|--------|
| 001  | Business onboarding    | done  | Full-stack | You    |
| 002  | Schedule management    | done  | Full-stack | Ana    |
| 003  | Public booking page    | done  | Frontend   | Ana    |
| 004  | Booking engine         | done  | Backend    | Diego  |
| 005  | Email notifications    | done  | Backend    | Diego  |
| 006  | Stripe billing         | done  | Backend    | You    |
```

Lucía can see the entire MVP is complete. No "is it done yet?" conversations. The spec index says it all.

## What You Actually Use

| Stage | When |
|---|---|
| `proposal-writer` | At project start, to align the team and Lucía |
| `spec-writer` | Each spec owner writes their specs, team reviews |
| `spec-implement` | After spec review, each developer implements their assigned specs |
| `agents-writer` | Once, to set up AGENTS.md with team conventions |
| `tsdd-init` | Once, at project start |

The spec index replaces standup updates. If everyone can read it, they know who's doing what and what's blocking what.
