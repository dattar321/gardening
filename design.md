# Gardening Website --- System Design & Development Plan

**Version:** 1.0\
**Status:** Draft\
**Product:** On-demand gardening and lawn-care services\
**Deployment target:** Hostinger VPS with Docker Compose

## 1. Confirmed technology stack

  Area                    Technology
  ----------------------- --------------------------------------------------
  Frontend                Next.js + JavaScript
  Styling                 Tailwind CSS
  Backend                 FastAPI + Python
  Database                PostgreSQL
  ORM                     SQLAlchemy
  Migrations              Alembic
  Image storage           External object storage (Cloudflare R2 proposed)
  Calendar                Google Calendar API
  Payments                Stripe
  Deployment              Hostinger VPS
  Containers              Docker Compose
  Reverse proxy / HTTPS   Caddy
  Email                   SMTP provider

## 2. Architecture approach

Use a **modular monolith**: one backend application divided into clear
business modules. This is simpler and cheaper to deploy than
microservices and is sufficient for the first version.

PostgreSQL is the primary source of truth. Google Calendar is an
integration for availability and event synchronization; it does not
replace the booking database.

``` mermaid
flowchart TD
    U[Customers and Admin] --> FE[Next.js frontend]
    FE --> API[FastAPI REST API]
    API --> DB[(PostgreSQL)]
    API --> OBJ[Object storage]
    API --> STRIPE[Stripe]
    API --> GC[Google Calendar API]
    API --> EMAIL[Email provider]
    API --> WORKER[Background worker]
    WORKER --> DB
    WORKER --> GC
    WORKER --> EMAIL
```

The background worker handles email delivery, calendar synchronization
retries and other tasks that should not delay an HTTP request. It can
share the FastAPI codebase and database.

## 3. Frontend structure

``` text
frontend/
├── app/
│   ├── (public)/
│   │   ├── page.js
│   │   ├── services/
│   │   ├── about/
│   │   ├── contact/
│   │   └── legal/
│   ├── quote/
│   │   └── manual/
│   ├── booking/
│   │   ├── checkout/
│   │   └── confirmation/
│   └── admin/
│       ├── login/
│       ├── dashboard/
│       ├── bookings/
│       ├── quotes/
│       ├── services/
│       ├── pricing/
│       └── settings/
├── components/
│   ├── layout/
│   ├── services/
│   ├── quote/
│   ├── booking/
│   └── admin/
├── lib/
│   ├── api.js
│   └── utils.js
└── public/
```

### Frontend performance

-   Generate public service pages statically where possible.
-   Use server rendering where content is dynamic.
-   Load client-side JavaScript only for interactive features.
-   Optimize and lazy-load images.
-   Avoid large UI libraries unless needed.
-   Keep admin code out of public page bundles.

## 4. Backend structure

``` text
backend/
├── app/
│   ├── main.py
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   └── database.py
│   ├── modules/
│   │   ├── auth/
│   │   ├── customers/
│   │   ├── services/
│   │   ├── pricing/
│   │   ├── quotes/
│   │   ├── bookings/
│   │   ├── calendar/
│   │   ├── payments/
│   │   ├── reviews/
│   │   ├── notifications/
│   │   └── settings/
│   ├── integrations/
│   │   ├── google_calendar.py
│   │   ├── stripe.py
│   │   ├── storage.py
│   │   └── email.py
│   └── worker/
│       └── tasks.py
├── migrations/
├── tests/
├── requirements.txt
└── Dockerfile
```

Each module should separate database models, Pydantic schemas, API
routes and business logic.

## 5. Database design

### Proposed tables

  Table                   Purpose
  ----------------------- ----------------------------------------
  `users`                 Admin accounts
  `customers`             Customer contact details
  `services`              Gardening services and active status
  `service_pricing`       Pricing configuration per service
  `size_tiers`            Small, medium, large and acreage tiers
  `pricing_modifiers`     Condition and access adjustments
  `addons`                Optional service add-ons
  `postcodes`             Supported service areas
  `quotes`                Instant quote records
  `quote_requests`        Manual quote requests
  `quote_images`          Uploaded image metadata
  `bookings`              Parent booking records
  `booking_items`         Services included in a booking
  `booking_occurrences`   Individual recurring appointments
  `payments`              Payment records
  `calendar_events`       Google Calendar event references
  `notifications`         Admin and customer notifications
  `reviews`               Review request records
  `settings`              Business configuration
  `background_jobs`       Pending/retryable background tasks

### Key relationships

``` mermaid
erDiagram
    CUSTOMERS ||--o{ QUOTES : requests
    CUSTOMERS ||--o{ QUOTE_REQUESTS : submits
    CUSTOMERS ||--o{ BOOKINGS : makes
    SERVICES ||--o{ SERVICE_PRICING : has
    SERVICES ||--o{ QUOTES : selected_for
    SERVICES ||--o{ BOOKING_ITEMS : included_in
    QUOTE_REQUESTS ||--o{ QUOTE_IMAGES : contains
    BOOKINGS ||--o{ BOOKING_ITEMS : contains
    BOOKINGS ||--o{ BOOKING_OCCURRENCES : schedules
    BOOKINGS ||--o{ PAYMENTS : has
    BOOKINGS ||--o{ CALENDAR_EVENTS : synchronizes
```

### Database rules

1.  Store money as integer cents, not floating-point values.
2.  Store timestamps in UTC and configure the business timezone as
    `Australia/Perth`.
3.  Save a price snapshot with each quote and booking.
4.  Updating current prices must not change existing bookings.
5.  Store Stripe event IDs to prevent duplicate webhook processing.
6.  Store Google Calendar event IDs to prevent duplicate event creation.
7.  Disable services instead of deleting services with historical
    records.

## 6. Pricing engine

Every service has its own pricing configuration and may use a different
calculation method.

Examples:

-   Lawn mowing: property-size pricing.
-   Weeding: size and condition pricing.
-   Gutter cleaning: property and roof complexity.
-   Turf laying: area-based pricing.
-   Pressure washing: surface-area pricing.
-   Tree planting / landscaping: manual quote only.

A possible configurable calculation is:

`total = (base_price × size_factor × condition_factor × access_factor) - discount + addons`

This is a starting model, not a rule that every service must follow. The
pricing engine must support service-specific methods and fixed add-ons.

The admin must be able to edit prices and modifiers without code
changes. Seed prices are placeholders until the business supplies real
rates.

### Quote flow

``` mermaid
flowchart TD
    A[Select service] --> B[Enter postcode]
    B --> C{Postcode supported?}
    C -->|No| D[Show service-area message]
    C -->|Yes| E[Select property size]
    E --> F[Calculate starting price]
    F --> G{Advanced options?}
    G -->|Yes| H[Apply modifiers and add-ons]
    G -->|No| I[Show estimate]
    H --> I
    I --> J[Save quote]
    J --> K[Book or request manual quote]
```

## 7. Booking and Google Calendar

### Booking flow

``` mermaid
flowchart TD
    A[Select service] --> B[Select date]
    B --> C[Check availability]
    C --> D[Check Google Calendar free/busy]
    D --> E{Slot available?}
    E -->|No| F[Choose another slot]
    E -->|Yes| G[Create temporary database reservation]
    G --> H{Payments enabled?}
    H -->|Yes| I[Stripe Checkout]
    H -->|No| J[Pay later]
    I --> K{Payment successful?}
    K -->|No| L[Handle failure and reservation expiry]
    K -->|Yes| M[Confirm booking]
    J --> M
    M --> N[Sync Google Calendar event]
    N --> O[Send confirmation]
```

### Availability settings

Admin-configurable settings should include:

-   Working days and hours.
-   Service duration.
-   Travel buffer.
-   Minimum booking lead time.
-   Maximum jobs per day.
-   Blocked dates and holidays.

### Reliability rules

-   Check availability before displaying slots and recheck during
    booking.
-   Prevent overlapping bookings with database transactions and
    appropriate constraints.
-   Keep the local booking record if Google Calendar is unavailable.
-   Retry failed calendar synchronization in the background.
-   Create individual appointment occurrences for recurring bookings so
    one occurrence can be changed independently.

Google Calendar and PostgreSQL need a clear conflict policy. External
calendar events should be considered when checking availability, but
confirmed local bookings remain the application record.

## 8. Image uploads and object storage

Cloudflare R2 is a proposed S3-compatible storage option. Confirm
current pricing and account requirements before choosing it.

### Upload flow

1.  Browser requests a temporary upload URL from FastAPI.
2.  FastAPI validates the request and creates a signed URL.
3.  Browser uploads directly to object storage.
4.  Backend saves image metadata and object key in PostgreSQL.
5.  Admin views images using short-lived signed URLs.

### Upload safeguards

-   Allow only required formats such as JPEG, PNG and WebP.
-   Validate actual file content, not only file extensions.
-   Set configurable file-size and image-count limits.
-   Keep the bucket private.
-   Do not expose storage credentials in frontend code.
-   Consider malware scanning and image metadata removal.

Suggested initial limits: five images per request and 10 MB per image.
These are configurable defaults.

## 9. API design

Use REST endpoints under `/api/v1`.

  Method   Endpoint                   Purpose
  -------- -------------------------- --------------------------------
  GET      `/services`                List active services
  GET      `/services/{id}`           Service details
  POST     `/quotes/calculate`        Calculate estimate
  POST     `/quotes`                  Save instant quote
  POST     `/quote-requests`          Submit manual quote
  POST     `/uploads/presign`         Generate upload URL
  GET      `/availability`            Get available slots
  POST     `/bookings`                Create booking
  GET      `/bookings/{id}`           Booking details
  POST     `/payments/checkout`       Create Stripe Checkout session
  POST     `/webhooks/stripe`         Receive Stripe events
  GET      `/admin/bookings`          List/manage bookings
  GET      `/admin/quotes`            List/manage quote requests
  CRUD     `/admin/services`          Manage services
  CRUD     `/admin/pricing`           Manage pricing
  CRUD     `/admin/postcodes`         Manage service areas
  POST     `/admin/reviews/request`   Trigger review request

All admin endpoints require authentication and authorization. Public
endpoints need validation and rate limiting.

## 10. Admin dashboard

### Navigation

-   Overview
-   Bookings
-   Manual quote requests
-   Customers
-   Services
-   Pricing configuration
-   Service areas
-   Calendar
-   Reviews
-   Settings

### Job statuses

Main workflow:

`NEW → QUOTED → BOOKED → COMPLETED`

Additional statuses:

-   `LOST`
-   `CANCELLED`
-   `PAYMENT_PENDING`

The dashboard should support search, filters and a detail page for each
booking or quote request.

## 11. Docker and VPS deployment

### Production containers

  Container    Responsibility
  ------------ -------------------------
  `frontend`   Next.js
  `backend`    FastAPI
  `worker`     Background tasks
  `db`         PostgreSQL
  `caddy`      HTTPS and reverse proxy

### Project root

``` text
gardening-website/
├── frontend/
├── backend/
├── docker/
│   └── caddy/
├── docs/
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
```

### Routing

``` text
yourdomain.com       -> Next.js
yourdomain.com/api/* -> FastAPI
```

### Production requirements

-   Store secrets in environment variables, not Git.
-   Enable HTTPS.
-   Persist PostgreSQL data in a Docker volume.
-   Back up the database off the VPS.
-   Configure container health checks and restart policies.
-   Configure log rotation.
-   Keep development and production environments separate.
-   Do not expose the PostgreSQL port publicly.

## 12. Security and reliability

  Area                Implementation
  ------------------- --------------------------------------
  Admin login         Secure password hashing
  Sessions            HttpOnly, Secure, SameSite cookies
  API                 Pydantic validation
  Database            SQLAlchemy and parameterized queries
  Uploads             File validation and private storage
  Payments            Verified Stripe webhooks
  Booking             Transactions and conflict prevention
  Spam                Rate limiting; CAPTCHA if needed
  Secrets             Environment variables
  Backups             Automated off-server backups
  Admin permissions   Role-based access control

Start with one administrator role. Add more roles only if the business
needs them.

## 13. Development milestones

  Milestone   Work                                 Priority
  ----------- ------------------------------------ ----------
  M1          Repository, Docker, VPS foundation   Critical
  M2          Database and backend foundation      Critical
  M3          Website UI and responsive pages      Critical
  M4          Services and pricing engine          Critical
  M5          Instant and manual quote systems     Critical
  M6          Admin dashboard                      Critical
  M7          Booking and Google Calendar          Critical
  M8          Stripe and email notifications       High
  M9          Reviews, analytics and legal pages   High
  M10         Testing, optimization and launch     Critical

Estimated development time: approximately nine weeks for one developer
working consistently. This is an estimate, not a fixed deadline.

## 14. Acceptance testing

  Feature         Acceptance criteria
  --------------- ----------------------------------------
  Website         Responsive on mobile and desktop
  Pricing         Correct service-specific calculations
  Postcodes       Unsupported areas rejected
  Manual quotes   Images and customer details saved
  Admin           Prices updated without code changes
  Booking         Overlapping bookings prevented
  Calendar        Confirmed bookings synchronized
  Stripe          Payments and webhook verification work
  Email           Customer and admin notifications work
  Security        Admin routes and uploads protected
  Performance     Pages and images optimized
  Backup          Database restore tested

## 15. Business decisions to confirm

Proposed defaults, not yet confirmed:

  Decision             Proposed default
  -------------------- ----------------------------------
  Currency             AUD
  Business timezone    Australia/Perth
  Booking duration     Configurable per service
  Payment              Full payment when enabled
  Pay later            Admin toggle
  Quote validity       14 days
  Image limit          5 per request
  Maximum image size   10 MB
  Customer accounts    Not required for v1
  Admin accounts       Login required
  Recurring bookings   Separate appointment occurrences
  Image storage        Cloudflare R2

## 16. First development sprint

-   [ ] Initialize Git repository.
-   [ ] Create Next.js frontend.
-   [ ] Create FastAPI backend.
-   [ ] Configure PostgreSQL and SQLAlchemy.
-   [ ] Configure Docker Compose.
-   [ ] Configure Caddy reverse proxy.
-   [ ] Run the application locally.
-   [ ] Verify frontend/backend/database communication.
-   [ ] Deploy initial version to Hostinger VPS.
-   [ ] Verify HTTPS and persistent database storage.

**Sprint completion:** The frontend, backend and database communicate
successfully, and the website is accessible through HTTPS on the domain.

## 17. Deferred from v1

-   AI image-based automatic quoting.
-   Twilio SMS integration.
-   Blog/content hub.
-   Suburb coverage map.
-   Native mobile applications.
-   Multi-language support.
-   Franchise or partner portal.
-   Multi-vendor functionality.
