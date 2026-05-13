# Case Study: Large Team — 7 to 8 Developers

> **Product:** `MercadoLocal` — a multi-vendor marketplace for local artisans and food producers.
> **Team:** 8 developers across 3 areas — Frontend (3), Backend (3), Platform/Infra (2).
> **Stakeholders:** Product team (PM + designer), operations team, vendor relations.
> **Stack:** Next.js (frontend), Go microservices (backend), PostgreSQL, Redis, Kafka, AWS, Stripe Connect.

## The Situation

The company wants to launch a marketplace where local artisans and food producers can sell directly to consumers. Think Etsy meets farmers market.

The scope is large enough that without coordination, you'll get:
- Frontend building a checkout flow that the backend doesn't support
- Two backend devs modifying the same service with conflicting assumptions
- Platform team changing infrastructure that breaks running services
- Operations asking "when will X be ready?" with no clear answer

TSDD at this scale is not about ceremony — it's about making dependencies visible before anyone writes code.

## What TSDD Looks Like Here

### Phase 1: The Proposal (Week 1)

The product team presents the vision. You invoke `proposal-writer` with the full team and stakeholders.

The proposal at `docs/proposal/mercado-local-mvp.md` defines:

- **Problem:** Local producers have no good way to sell online. Marketplaces take 20-30% commissions. Social media sales are chaotic (DMs, manual payments, no inventory tracking).
- **Solution:** A marketplace with 8% commission, vendor-managed inventory, integrated payments via Stripe Connect, and delivery/pickup options.
- **Scope (MVP):** Vendor onboarding, product catalog, shopping cart (multi-vendor), checkout with split payments, order management, basic vendor dashboard, customer order tracking. Excludes: reviews/ratings, subscription boxes, delivery routing optimization, vendor analytics dashboard, promotional codes.
- **Planned specs (12 specs across 3 layers):**

```markdown
## Planned specs

### Vendor layer
1. `001-vendor-onboarding.md` — vendor signup, KYC, Stripe Connect onboarding
2. `002-vendor-dashboard.md` — manage products, inventory, orders
3. `003-product-management.md` — CRUD products, categories, images, stock

### Customer layer
4. `004-product-discovery.md` — browse, search, filter, categories
5. `005-shopping-cart.md` — multi-vendor cart, quantity management
6. `006-checkout-payments.md` — checkout flow, Stripe Connect split payments
7. `007-order-tracking.md` — customer order history and status tracking

### Platform layer
8. `008-order-service.md` — order creation, state machine, notifications
9. `009-payment-service.md` — Stripe Connect integration, webhooks, refunds
10. `010-notification-service.md` — email, push, in-app notifications
11. `011-search-service.md` — product search with Elasticsearch
12. `012-delivery-management.md` — pickup and delivery options, zones

### Dependencies
- 001 → 002, 003 (vendor must exist before dashboard or products)
- 003 → 004, 005 (products must exist before discovery or cart)
- 008 → 006, 007 (order service must exist before checkout or tracking)
- 009 → 006 (payment service must exist before checkout)
- 010 → 008 (notifications depend on order events)
- 011 → 004 (search powers discovery)
- 012 → 006 (delivery options needed at checkout)
```

The team reviews the proposal. The backend team flags that specs 008 (order service) and 009 (payment service) should be done in parallel since they're independent services. The frontend team notes that spec 005 (cart) needs the product API from spec 003.

You sequence the work into three waves:

- **Wave 1:** 001, 003, 008, 009, 011 (foundation services)
- **Wave 2:** 002, 004, 005, 010, 012 (consumer-facing features)
- **Wave 3:** 006, 007 (checkout and tracking — depend on waves 1 and 2)

### Phase 2: Spec Writing (Week 2)

Each spec gets an owner. They invoke `spec-writer` and produce specs. The team reviews for conflicts and clarity.

**Example: Spec 006 (Checkout & Payments)**

This is the most complex spec — it touches frontend, backend, and payment services.

```markdown
# Spec 006: Checkout & Split Payments

## Context
Customers need to checkout with items from multiple vendors. Stripe Connect
handles split payments: customer pays once, funds are distributed to each
vendor minus the platform commission (8%).

## Scope
- Checkout page: review cart, select delivery method, enter payment info
- Stripe PaymentIntent with transfer_data for each vendor
- Platform fee collected via application_fee_amount
- Order created only after successful payment (idempotent)
- Payment failure handling: clear error message, cart preserved
- Mobile-responsive checkout flow

## Out of scope
- Saved payment methods (MVP: always enter card)
- Installment payments
- Gift cards or promotional codes
- Invoice generation

## Files to modify
- `apps/web/app/checkout/page.tsx` — checkout page (Frontend: Ana)
- `apps/web/components/checkout/...` — checkout components (Frontend: Ana)
- `services/order/api/handlers/checkout.go` — checkout handler (Backend: Diego)
- `services/order/api/handlers/checkout_test.go` — checkout tests (Backend: Diego)
- `services/payment/api/handlers/stripe.go` — Stripe integration (Backend: Carlos)
- `services/payment/internal/stripe/connect.go` — Connect logic (Backend: Carlos)
- `packages/shared/types/order.ts` — order types (Platform: You)
- `packages/shared/types/payment.ts` — payment types (Platform: You)

## Dependencies
- Requires spec 005 (cart) — needs cart data
- Requires spec 008 (order service) — creates orders
- Requires spec 009 (payment service) — processes payments
- Requires spec 012 (delivery management) — delivery options

## Acceptance criteria
- [ ] Customer can complete checkout with items from multiple vendors
- [ ] Single payment charge covers all vendors + platform fee
- [ ] Each vendor receives their portion minus 8% commission
- [ ] Payment failure shows error and preserves cart contents
- [ ] Successful payment creates order with status "confirmed"
- [ ] Checkout is mobile-responsive (tested on 375px width)
- [ ] Idempotent: retrying a successful payment doesn't create duplicate orders
- [ ] Webhook handles Stripe async confirmation
```

The spec is reviewed by Ana (frontend), Diego (order service), Carlos (payment service), and you (shared types). Everyone confirms their file assignments and scope. The spec is saved. Index updated.

### Phase 3: Implementation (Weeks 3-6)

**Wave 1 starts.** Five developers work in parallel on specs 001, 003, 008, 009, 011.

**The conflict moment:** Diego (spec 008, order service) and Carlos (spec 009, payment service) both need to define the `OrderPayment` type in `packages/shared/types/payment.ts`. They discover this when both specs list the same file.

Resolution: You (platform team) write a quick spec 013 for the shared types:

```markdown
# Spec 013: Shared Type Definitions

## Context
Multiple services need shared TypeScript types for orders and payments.
These live in packages/shared/types/ and are consumed by all services.

## Scope
- Define Order, OrderItem, OrderStatus types
- Define Payment, PaymentIntent, PaymentStatus types
- Define VendorPayout, Commission types
- All types are versioned and backward-compatible

## Files to create
- `packages/shared/types/order.ts`
- `packages/shared/types/payment.ts`
- `packages/shared/types/vendor.ts`

## Dependencies
- Must be done before spec 006 (checkout) and spec 008 (order service)
```

You implement it in a day. Diego and Carlos unblock. This is the spec index catching conflicts before they become merge conflicts.

**Wave 2 starts.** Specs 002, 004, 005, 010, 012. The frontend team builds the vendor dashboard and product discovery. The notification service sends its first test email.

**The scope creep moment:** Operations asks for a "pending orders" dashboard so they can manually intervene when something goes wrong. This wasn't in the proposal.

You check the spec index — no spec covers this. You write spec 014:

```markdown
# Spec 014: Operations Dashboard — Pending Orders

## Context
Operations team needs visibility into orders that are stuck or require
manual intervention (payment confirmed but order not created, vendor
didn't confirm within 24h, etc.).

## Scope
- Internal dashboard at `admin.mercadolocal.app/pending`
- Shows orders in "pending" state for > 1 hour
- Filter by vendor, date range, issue type
- Actions: manually confirm, cancel, re-notify vendor
- Access restricted to operations team (SSO)

## Out of scope
- Full order management (that's a future phase)
- Vendor-facing order management (spec 002 covers vendor side)
- Audit logging (future phase)

## Dependencies
- Requires spec 008 (order service) — reads order data
- Requires spec 010 (notification service) — can trigger re-notifications
```

Operations confirms scope. You add it to wave 2. The proposal is updated to reflect the new scope.

**Wave 3 starts.** Specs 006 (checkout) and 007 (order tracking). The most visible features. Ana leads the checkout frontend, Diego and Carlos coordinate the backend payment flow.

### Phase 4: Artifacts and Launch (Week 7)

All specs are done. The spec index:

```markdown
# Specs Index

| ID   | Title                      | State | Layer      | Owner    | Area     |
|------|----------------------------|-------|------------|----------|----------|
| 001  | Vendor onboarding          | done  | Full-stack | Marco    | Vendor   |
| 002  | Vendor dashboard           | done  | Frontend   | Ana      | Vendor   |
| 003  | Product management         | done  | Full-stack | Diego    | Vendor   |
| 004  | Product discovery          | done  | Frontend   | Sofia    | Customer |
| 005  | Shopping cart              | done  | Frontend   | Ana      | Customer |
| 006  | Checkout & split payments  | done  | Full-stack | Ana/Diego| Customer |
| 007  | Order tracking             | done  | Frontend   | Sofia    | Customer |
| 008  | Order service              | done  | Backend    | Diego    | Platform |
| 009  | Payment service            | done  | Backend    | Carlos   | Platform |
| 010  | Notification service       | done  | Backend    | Marco    | Platform |
| 011  | Search service             | done  | Backend    | Carlos   | Platform |
| 012  | Delivery management        | done  | Full-stack | Diego    | Platform |
| 013  | Shared type definitions    | done  | Platform   | You      | Platform |
| 014  | Ops dashboard — pending    | done  | Frontend   | Sofia    | Platform |
```

You generate artifacts:
- **ADR 001:** Why Stripe Connect over manual payouts (security, compliance, automation)
- **ADR 002:** Why Kafka for event streaming between services (reliability, replayability)
- **Architecture diagram:** Service topology and data flow
- **API contract:** OpenAPI spec for the public vendor API

The launch MR references all 14 specs and 4 artifacts. The product team can see exactly what was built.

## What You Actually Use

| Stage | When |
|---|---|
| `proposal-writer` | At project start, and when scope expands significantly |
| `spec-writer` | Each spec owner writes their specs, team reviews for conflicts |
| `spec-implement` | After spec review, each developer implements their assigned specs |
| `agents-writer` | Once, to set up AGENTS.md with team-wide conventions and strict boundaries |
| `tsdd-init` | Once, at project start |

At this scale, the spec index is your coordination tool. It replaces:
- "What are you working on?" → read the index
- "Is X done yet?" → check the state column
- "Can I start Y?" → check the dependencies column
- "Who owns Z?" → check the owner column

The specs themselves are the contracts that prevent conflicting implementations. The proposal is the alignment that keeps everyone building the same product.
