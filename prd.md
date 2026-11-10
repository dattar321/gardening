this is a prd plan

# Product Requirements Document (PRD)

## Gardening Services Website

**Version:** 1.0 (Draft) **Product:** On-demand gardening & lawn care service website **Platform:** Responsive web (mobile-first, fully functional on mobile and PC)

---

## 1. Overview & Objective

Build a service-business website that allows customers to get instant quotes, book gardening services online, and request manual quotes by uploading images. The website is the primary conversion engine for the business — every page and flow should drive toward a quote or a booking.

The site blends two models reviewed during discovery:

- **Gardener Co** — instant online quotes, transparent pricing, online booking (on-demand model)
- **Jim's Mowing** — broad service catalogue, trust signals, quote-based enquiries (service-network model)

The site will support **both** an instant self-serve quote/booking flow **and** a manual image-quote request flow.

---

## 2. Services Offered

### 2.1 Core Services (instant quote + online booking)

1. Lawn Mowing & Edging
2. Weeding & Pruning
3. Garden Tidy-Up & Green Waste Removal
4. Fertilizing & Mulching
5. Gutter Cleaning
6. Pressure Washing
7. Turf Laying

### 2.2 Quote-Only Service

8. **Tree Planting / Landscaping ("plant trees and stuff")** — no fixed price; routes to the manual image-quote request form only. Site visit or image-based quote.

> **Note:** Every service has a **different cost model** (mowing ≠ hedging ≠ gutter cleaning). Pricing is not uniform and must be managed per service.

---

## 3. Quote Engine (Core Feature)

Three quote pathways, all reachable from a prominent "Get a Quote" CTA on the homepage.

### 3.1 Basic Quote (default)

Inputs:

- Service selection
- Suburb / postcode (validated against service area)
- Garden/property size (small / medium / large / acreage)

Output: **Instant starting price**, displayed before any further input. This is the primary conversion driver.

### 3.2 Advanced Quote (optional precision layer)

Unlocked via an "Advanced options" toggle under the basic quote. Adds:

- **Frequency** — one-off / weekly / fortnightly / monthly (recurring frequency applies a discount)
- **Condition** — maintained / overgrown / heavy weeds / debris present
- **Access** — side gate, steep slope, parking restrictions, rear access
- **Add-ons** — green waste removal, mulching, edging, extra hedges, etc.

Each input applies a **multiplier or fixed add-on** to the base price. Result is a refined, more accurate quote.

### 3.3 Manual Image Quote

Form fields:

- Image upload (one or more photos of the garden/job)
- Name
- Phone
- Email
- Service required
- Free-text notes

Behaviour:

- Submission saved to admin panel
- Notification sent to admin (see §7)
- Customer shown a confirmation with an expected response time
- Admin replies manually with a quote

Used for: tree planting, large/ambiguous jobs, and any user who prefers not to self-quote.

### 3.4 Pricing Engine Rules

- **Every service has its own pricing table** — base rate + multipliers per size tier + frequency discount + add-on prices.
- Pricing table is **fully editable from the admin panel** (no code changes required to update prices).
- Admin can create/edit/disable services, size tiers, multipliers, and add-ons.
- **AI image-based auto-quoting** is a **future enhancement**, not in scope for v1. The image upload and quote-request data model will be built now so this data is stored and structured for later AI use.

---

## 4. Booking & Payments

### 4.1 Book Online Now

- E-commerce-style checkout for the core services.
- User selects date + time slot from a calendar.
- Supports **one-off** and **recurring** bookings (weekly / fortnightly / monthly).
- **Upfront payment via Stripe.**

### 4.2 Stripe

- Stripe integration to be built.
- Includes an **admin toggle to switch payments on/off** ("pay later" mode) so the site can launch before payment is finalized, then enable Stripe when ready.
- **Live Stripe keys and account setup will be revised / finalised later.**

### 4.3 Scheduling / Calendar

- Bookings must integrate with **Google Calendar**.
- Options considered:
  - **Direct Google Calendar API** — free, most control, bookings write directly to calendar and free/busy is checked. *(Recommended.)*
  - **Cal.com** — open-source scheduler with free tier and Google Calendar sync.
  - **Calendly** — easiest, but paid API and vendor lock-in.
- **Decision: pending** — final choice and availability rules (working hours, jobs per day, travel buffer, booking lead time) to be confirmed.

---

## 5. Public Pages & Content

### 5.1 Homepage

- Hero section with primary "Get a Quote" CTA
- Service overview (core services)
- **Trust & credibility module** — police-checked, fully insured, experienced gardeners
- Reviews section (Google Reviews)
- How it works (quote → book → service → review)
- Secondary CTA for manual image quote

### 5.2 Service Pages

- Individual landing page per service (supports SEO and clarity)
- Each service page includes its own instant-quote entry point

### 5.3 Service Area

- **Postcode/suburb field on the quote form** with validation against a list of accepted postcodes managed in admin.
- Purpose: block out-of-area leads.
- *(Full suburb coverage map is not required; only validation is in scope.)*

### 5.4 Reviews

- **Google Reviews pulled via API** into a dedicated reviews section/site-wide widget.

### 5.5 Admin-Triggered Review Requests

- After a job is marked complete in admin, a **review request** can be triggered to the customer.

### 5.6 Content Hub (Optional / secondary)

- Blog or "Garden Tips" section possible for SEO and retention — **not in v1 scope**, noted for future.

### 5.7 Standard Pages

- About
- Contact
- **Privacy Policy**
- **Terms & Conditions**
- **Cancellation / Refund Policy** (required for taking Stripe payments)

---

## 6. Admin Panel

**Scope: simple** (explicitly minimal).

Required functions:

- View/manage **bookings** (date, service, customer, status)
- View/manage **manual image-quote requests** (images, contact details, notes, status)
- Manage **pricing tables** per service (base rates, multipliers, add-ons)
- Manage **service area postcodes**
- **Job status tracking** (new → quoted → booked → completed → lost)
- **Trigger review requests** after job completion
- **Notifications** for new bookings and new quote requests

Notifications:

- In-admin notifications now.
- **Email** notifications for new quotes/bookings.
- **SMS via Twilio** — deferred; leave the integration hook in place for later.

---

## 7. Notifications & Communication

- New booking → admin notified (email + in-admin)
- New manual quote request → admin notified (email + in-admin)
- Customer → receives booking/quote confirmation
- Twilio SMS — **not in v1**, hook only

---

## 8. Technical Requirements

- **Fully responsive** — mobile-first design; all quote and booking flows must be seamless on phones.
- **Fast loading, SEO-friendly, accessible.**
- **Google Analytics** + **conversion tracking** on quote submissions and bookings.
- Integrations: **Stripe**, **Google Calendar API**, **Google Reviews API**, (future: Twilio).
- Admin CMS for pricing, content, bookings, quotes.

---

## 9. Deferred / Future Scope (explicitly out of v1)

1. **AI image-based auto-quoting** — data model and image upload built now; AI logic later.
2. **Twilio SMS** — hook only.
3. **Content/blog hub** — future.
4. **Suburb coverage map** — not required; postcode validation only.
5. **Final pricing numbers** — dummy/seed data at build; real rates entered in admin later.
6. **Brand assets** (logo, service photos, team photos) — placeholders at build; added before launch.
7. **Legal page copy** — placeholder; generated via a service like Termly/GetTerms before go-live.
8. **Stripe live account / keys** — revised and finalised later.

---

## 10. Launch Blockers (must be resolved before go-live)

- Real pricing entered into admin
- Brand assets (logo + service photos)
- Legal pages (Privacy, Terms, Cancellation/Refund)
- Stripe account live keys (or payments-left-off decision confirmed)
- Google Calendar integration choice + availability rules confirmed

---

## 11. Out of Scope for This PRD

- Native mobile apps
- Multi-language support
- Franchise/partner portal
- AI auto-quoting implementation







i'd make the site and deploy in hostinger. wanna use docker and for code js for frontend, and for backened maybe fastapi or any thing lightweight js.&#x20;



the website needs to be very fast and as less costly as possible.&#x20;



so first in very somple words, gives me planning, so then i can confirm and make details system design and work plannings
