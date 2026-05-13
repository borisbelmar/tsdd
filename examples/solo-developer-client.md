# Case Study: Solo Developer — Client with Backlog

> **Product:** AutoParts Store — an online store + booking system for an auto parts shop.
> **Developer:** You, freelancer.
> **Client:** Roberto, owns a mechanical parts shop. Wants to sell online and let customers book installation appointments.
> **Stack:** Next.js, PostgreSQL, Stripe, Tailwind.

## The Situation

Roberto approached you with a list of things he wants:

> "I need people to buy parts online, book installation appointments, and get me a notification when someone buys something."

You translate this into a structured workflow. Without specs, this becomes scope creep. With specs, you have a contract.

## What TSDD Looks Like Here

### Step 1: Proposal

You invoke `proposal-writer` to structure Roberto's request. You work through it together:

- **Problem:** Roberto loses sales because customers can't check inventory or buy outside business hours. Installation bookings happen over WhatsApp and get lost.
- **Solution:** E-commerce site with product catalog, cart, checkout, and a booking system for installations.
- **Scope:** MVP includes catalog, cart, Stripe checkout, booking calendar, email notifications. Excludes: loyalty program, wholesale pricing, multi-location inventory.
- **Planned specs:**
  1. `001-product-catalog.md` — browse, search, filter products
  2. `002-shopping-cart.md` — add, remove, update quantities
  3. `003-checkout-stripe.md` — payment integration
  4. `004-booking-system.md` — installation appointment scheduling
  5. `005-email-notifications.md` — order and booking confirmations

The proposal is saved at `docs/proposal/auto-parts-store-mvp.md`. You share it with Roberto. He confirms the scope. This is your alignment.

### Step 2: Write Specs

You invoke `spec-writer` for each planned spec. Let's look at one:

**Spec 001: Product Catalog**

```markdown
# Spec 001: Product Catalog

## Context
Customers need to browse available auto parts, search by name or SKU,
and filter by category, vehicle model, and price range.

## Scope
- Product listing page with pagination (20 per page)
- Search by product name or SKU
- Filter by: category, vehicle model, price range
- Product detail page: name, description, price, stock status, images
- Admin: add/edit/delete products via `/admin/products`
- Products stored in PostgreSQL `products` table

## Out of scope
- Product reviews or ratings
- Related products recommendations
- Bulk import/export (CSV)
- Multi-currency support

## Files to create
- `app/products/page.tsx` — listing page
- `app/products/[slug]/page.tsx` — detail page
- `app/admin/products/page.tsx` — admin CRUD
- `lib/db/schema/products.ts` — product schema
- `lib/db/queries/products.ts` — product queries

## Acceptance criteria
- [ ] Customer can view paginated product list
- [ ] Search returns results matching name or SKU (case-insensitive)
- [ ] Filters combine correctly (category + price range)
- [ ] Product detail shows all fields including stock status
- [ ] Out-of-stock products show "Out of stock" badge
- [ ] Admin can add a new product with image upload
- [ ] Admin can edit existing product details
- [ ] Admin can delete a product (soft delete)
```

You share this spec with Roberto. He says: "Can customers see if a part fits their car?" You add to the spec:

> "Vehicle compatibility section on product detail page — shows which vehicle models the part fits."

You update the acceptance criteria. Roberto confirms. The spec is ready.

### Step 3: Implement

You invoke `spec-implement` for spec 001. It reads the spec, reads your existing codebase, plans the changes, and executes. You review the output, run the dev server, verify the acceptance criteria.

When you're satisfied, you commit:

```
feat(catalog): add product listing, detail, search, filter, and admin CRUD

Spec: docs/specs/001-product-catalog.md
```

### Step 4: The Scope Creep Moment

Roberto messages you: "Can we add a shopping cart before checkout?"

You check the spec index — spec 002 is the shopping cart, already planned. You say: "Yes, it's already in the plan. I'll do it after the catalog."

This is TSDD protecting you. Without the spec index, this conversation becomes "I thought that was included." With it, you point to the roadmap.

### Step 5: The Unexpected Bug

During spec 003 (checkout), you discover Stripe webhooks fail because the dev environment doesn't have a public URL. You need ngrok or a tunnel.

You don't silently fix it. You update spec 003's technical notes:

```markdown
## Technical notes
- Stripe webhooks require a public URL. Use ngrok for local development.
- Add `STRIPE_WEBHOOK_SECRET` to `.env.example`.
```

The spec reflects reality.

### Step 6: The Booking Complication

Spec 004 (booking system) gets interesting. Roberto says: "I don't want bookings to be created if no mechanic is available."

This wasn't in the original proposal. You check — the proposal said "booking calendar" but didn't specify mechanic availability. You write a new spec:

```markdown
# Spec 006: Mechanic Availability

## Context
Bookings should only be available when a mechanic has an open slot.
Roberto has 2 mechanics, each works 8am-6pm, Monday-Saturday.

## Scope
- Mechanic profiles in database (name, working hours, max bookings per day)
- Booking calendar only shows available slots
- Slots are 1-hour blocks
- When a mechanic reaches max bookings for a day, that day shows as unavailable
- Admin can set mechanic schedules and time off

## Out of scope
- Automatic mechanic assignment (manual selection by customer)
- Partial-day schedules
- Overtime or extended hours

## Files to create
- `lib/db/schema/mechanics.ts` — mechanic schema
- `lib/db/schema/availability.ts` — availability rules
- `lib/booking/availability.ts` — slot calculation logic
- `app/booking/availability-checker.tsx` — available slots UI

## Acceptance criteria
- [ ] Calendar shows only slots with available mechanics
- [ ] Booking a slot reduces available count
- [ ] When count reaches 0, slot becomes unavailable
- [ ] Admin can set mechanic time off (blocks specific dates)
- [ ] Admin can change mechanic working hours
```

Roberto confirms. You implement. The spec index now has 6 specs instead of 5 — and that's fine. The proposal was a starting point, not a prison.

## Your Spec Index

```markdown
# Specs Index

| ID   | Title                  | State     | Layer     |
|------|------------------------|-----------|-----------|
| 001  | Product catalog        | done      | Frontend  |
| 002  | Shopping cart          | done      | Frontend  |
| 003  | Checkout with Stripe   | in-progress| Backend  |
| 004  | Booking system         | draft     | Full-stack|
| 005  | Email notifications    | draft     | Backend   |
| 006  | Mechanic availability  | draft     | Full-stack|
```

You share this with Roberto. He can see what's done, what's in progress, and what's next. No status meetings needed.

## What You Actually Use

| Stage | When |
|---|---|
| `proposal-writer` | At project start, and when scope expands significantly |
| `spec-writer` | For every feature, before implementation |
| `spec-implement` | After spec sign-off |
| `agents-writer` | Once, to set up AGENTS.md with client communication rules |
| `tsdd-init` | Once, at project start |

Your specs are your contract with Roberto. If it's not in a spec, it's not in the work. If Roberto wants something new, it gets a spec. The spec index is your shared progress tracker.
