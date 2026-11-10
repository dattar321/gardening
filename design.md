# Gardening Website — System Design & Development Plan



Architecture confirmed

&#x20;Version 1.0 · Technical design



Your confirmed architecture:

| Component     | Technology              |
| ------------- | ----------------------- |
| Frontend      | Next.js + JavaScript    |
| Styling       | Tailwind CSS            |
| Backend       | FastAPI + Python        |
| Database      | PostgreSQL              |
| ORM           | SQLAlchemy              |
| Migrations    | Alembic                 |
| Image storage | External object storage |
| Calendar      | Google Calendar API     |
| Payment       | Stripe                  |
| Deployment    | Hostinger VPS           |
| Containers    | Docker Compose          |
| Reverse proxy | Caddy                   |
| Email         | SMTP provider           |

I recommend a modular monolithic architecture.

This means one application with clearly separated modules rather than multiple independent backend services. It is easier to develop, deploy, debug and maintain, and is sufficient for this business.

# 1. Overall system architecture

Users

Customers · Administrator

Next.js

Public website · Quote interface · Booking · Admin dashboard

FastAPI

REST API · Authentication · Business logic · Integrations

PostgreSQL

Application data

Object Storage

Customer images

External integrations

Stripe

Google Calendar

Email provider

Google Reviews

One additional component: a lightweight background worker.

It handles tasks such as sending emails, processing booking notifications and retrying failed integrations. It can share the same FastAPI codebase and database.

Important architecture decision: PostgreSQL will be the primary source of truth. Google Calendar will synchronize bookings and availability, not replace the database.

# 2. Frontend design

Use Next.js with JavaScript, not TypeScript initially if you prefer to keep the codebase entirely JavaScript.

Recommended frontend structure:

```
frontend/
├── app/
│   ├── (public)/
│   │   ├── page.js
│   │   ├── services/
│   │   ├── about/
│   │   ├── contact/
│   │   └── legal/
│   │
│   ├── quote/
│   │   ├── page.js
│   │   └── manual/
│   │
│   ├── booking/
│   │   ├── page.js
│   │   ├── checkout/
│   │   └── confirmation/
│   │
│   └── admin/
│       ├── login/
│       ├── dashboard/
│       ├── bookings/
│       ├── quotes/
│       ├── services/
│       ├── pricing/
│       └── settings/
│
├── components/
│   ├── layout/
│   ├── services/
│   ├── quote/
│   ├── booking/
│   └── admin/
│
├── lib/
│   ├── api.js
│   └── utils.js
│
└── public/
```

### Frontend performance strategy

- Server-render public pages where appropriate.
- Static generation for service pages.
- Client-side JavaScript only where interaction is needed.
- Lazy-load images.
- Use optimized WebP/AVIF images.
- Avoid heavy UI libraries.
- Keep the admin dashboard separate from public page bundles.

# 3. Backend design

Divide FastAPI into independent business modules.

```
backend/
├── app/
│   ├── main.py
│   │
│   ├── core/
│   │   ├── config.py
│   │   ├── security.py
│   │   └── database.py
│   │
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
│   │
│   ├── integrations/
│   │   ├── google_calendar.py
│   │   ├── stripe.py
│   │   ├── storage.py
│   │   └── email.py
│   │
│   └── worker/
│       └── tasks.py
│
├── migrations/
├── tests/
├── requirements.txt
└── Dockerfile
```

Each module should contain its own:

- Models
- Schemas
- Routes
- Services
- Business logic

For example, the pricing module calculates quotes. The booking module handles appointments. The payment module handles Stripe.

This separation makes future modifications easier.

# 4. Database design

Use PostgreSQL with relational tables.

I recommend the following structure.

## 4.1 Main tables

| Table               | Purpose                           |
| ------------------- | --------------------------------- |
| users               | Admin accounts                    |
| customers           | Customer details                  |
| services            | Gardening services                |
| service_pricing     | Pricing configuration             |
| size_tiers          | Small, medium, large, acreage     |
| pricing_modifiers   | Condition and access adjustments  |
| addons              | Additional service prices         |
| postcodes           | Accepted service areas            |
| quotes              | Instant quote records             |
| quote_requests      | Manual image-quote requests       |
| quote_images        | Uploaded image metadata           |
| bookings            | Booking information               |
| booking_items       | Services included in booking      |
| booking_occurrences | Individual recurring appointments |
| payments            | Payment records                   |
| calendar_events     | Google Calendar event references  |
| notifications       | Admin/customer notifications      |
| reviews             | Review request records            |
| settings            | Business configuration            |
| background_jobs     | Pending integration tasks         |

## 4.2 Important relationships

\#chatgpt-mermaid-\_r_l7\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_l7\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_l7\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_l7\_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_l7\_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_l7\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_l7\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_l7\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_l7\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_l7\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_l7\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_l7\_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_l7\_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_l7\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_l7\_ p{margin:0;}#chatgpt-mermaid-\_r_l7\_ .entityBox{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_l7\_ .relationshipLabelBox{fill:rgb(48, 48, 48);opacity:0.7;background-color:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_l7\_ .relationshipLabelBox rect{opacity:0.5;}#chatgpt-mermaid-\_r_l7\_ .labelBkg{background-color:rgba(48, 48, 48, 0.5);}#chatgpt-mermaid-\_r_l7\_ .edgeLabel{background-color:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_l7\_ .edgeLabel .label rect{fill:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_l7\_ .edgeLabel .label text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_l7\_ .edgeLabel .label{fill:rgb(31, 78, 148);font-size:14px;}#chatgpt-mermaid-\_r_l7\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_l7\_ .edge-pattern-dashed{stroke-dasharray:8,8;}#chatgpt-mermaid-\_r_l7\_ .node rect,#chatgpt-mermaid-\_r_l7\_ .node circle,#chatgpt-mermaid-\_r_l7\_ .node ellipse,#chatgpt-mermaid-\_r_l7\_ .node polygon{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-\_r_l7\_ .relationshipLine{stroke:rgb(175, 175, 175);stroke-width:1px;fill:none;}#chatgpt-mermaid-\_r_l7\_ .marker{fill:none!important;stroke:rgb(175, 175, 175)!important;stroke-width:1;}#chatgpt-mermaid-\_r_l7\_ [data-look=neo].labelBkg{background-color:rgba(48, 48, 48, 0.5);}#chatgpt-mermaid-\_r_l7\_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_l7\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_l7\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_l7\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_l7\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_l7\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_l7\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_l7\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}CUSTOMERSQUOTESQUOTE_REQUESTSBOOKINGSSERVICESSERVICE_PRICINGBOOKING_ITEMSQUOTE_IMAGESBOOKING_OCCURRENCESPAYMENTSCALENDAR_EVENTSrequestssubmitsmakeshasselected_forincluded_inconverts_tocontainscontainsscheduleshassynchronizes

## 4.3 Important database rules

1. Store money as integer cents, not floating-point numbers.
2. Store booking dates in UTC and retain the business timezone (`Australia/Perth`).
3. Store the quoted price snapshot when a quote is generated.
4. Changing service prices must not change existing bookings.
5. Use unique Stripe event IDs to prevent duplicate payment processing.
6. Use unique calendar event IDs to prevent duplicate synchronization.
7. Use soft deletion or disabling for services that have historical bookings.

# 5. Pricing engine design

This is one of the most important parts of the system.

Different services must have independent pricing rules.

Example seed data (illustrative only):

| Service          | Base price | Size multiplier |
| ---------------- | ---------- | --------------- |
| Lawn mowing      | $50        | 1.0–2.5         |
| Weeding          | $40        | 1.0–2.0         |
| Gutter cleaning  | $90        | 1.0–2.0         |
| Pressure washing | $60        | 1.0–2.5         |
| Turf laying      | $100       | Area-based      |

Actual rates will be entered through the admin dashboard.

### Pricing calculation

A simple model:

\\[ P = (B \times S \times C \times A) - D + X \\]

Where:

- \\(B\\): Base price
- \\(S\\): Size multiplier
- \\(C\\): Condition multiplier
- \\(A\\): Access multiplier
- \\(D\\): Recurring discount
- \\(X\\): Fixed add-ons

However, the engine should also support service-specific calculation methods.

For example:

- Lawn mowing: property-size pricing.
- Gutter cleaning: roof/property complexity.
- Turf laying: square metres.
- Pressure washing: surface area.
- Tree planting: manual quote only.

Important: Never assume all services can use the same formula. Admin should select a pricing method per service.

### Quote workflow

\#chatgpt-mermaid-\_r_le\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_le\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_le\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_le\_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_le\_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_le\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_le\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_le\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_le\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_le\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_le\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_le\_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_le\_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_le\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_le\_ p{margin:0;}#chatgpt-mermaid-\_r_le\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_le\_ .cluster-label text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_le\_ .cluster-label span{color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_le\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_le\_ .label text,#chatgpt-mermaid-\_r_le\_ span{fill:rgb(237, 237, 237);color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_le\_ .node rect,#chatgpt-mermaid-\_r_le\_ .node circle,#chatgpt-mermaid-\_r_le\_ .node ellipse,#chatgpt-mermaid-\_r_le\_ .node polygon,#chatgpt-mermaid-\_r_le\_ .node path{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-\_r_le\_ .rough-node .label text,#chatgpt-mermaid-\_r_le\_ .node .label text,#chatgpt-mermaid-\_r_le\_ .image-shape .label,#chatgpt-mermaid-\_r_le\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_le\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_le\_ .rough-node .label,#chatgpt-mermaid-\_r_le\_ .node .label,#chatgpt-mermaid-\_r_le\_ .image-shape .label,#chatgpt-mermaid-\_r_le\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_le\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_le\_ .root .anchor path{fill:rgb(175, 175, 175)!important;stroke-width:0;stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_le\_ .arrowheadPath{fill:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_le\_ .edgePath .path{stroke:rgb(175, 175, 175);stroke-width:1px;}#chatgpt-mermaid-\_r_le\_ .flowchart-link{stroke:rgb(175, 175, 175);fill:none;}#chatgpt-mermaid-\_r_le\_ .edgeLabel{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-\_r_le\_ .edgeLabel p{background-color:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_le\_ .edgeLabel rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_le\_ .labelBkg{background-color:rgba(0, 0, 0, 0.5);}#chatgpt-mermaid-\_r_le\_ .cluster rect{fill:rgb(48, 48, 48);stroke:rgba(255, 255, 255, 0.15);stroke-width:1px;}#chatgpt-mermaid-\_r_le\_ .cluster text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_le\_ .cluster span{color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_le\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(48, 48, 48);border:1px solid rgba(255, 255, 255, 0.15);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_le\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_le\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_le\_ .icon-shape,#chatgpt-mermaid-\_r_le\_ .image-shape{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-\_r_le\_ .icon-shape p,#chatgpt-mermaid-\_r_le\_ .image-shape p{background-color:rgb(0, 0, 0);padding:2px;}#chatgpt-mermaid-\_r_le\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_le\_ .image-shape .label rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_le\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_le\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_le\_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_le\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_le\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_le\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_le\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_le\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_le\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_le\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_le\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_le\_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-\_r_le\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_le\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_le\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_le\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_le\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_le\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_le\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_le\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(153, 206, 255);}#chatgpt-mermaid-\_r_le\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(153, 206, 255);}#chatgpt-mermaid-\_r_le\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_le\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_le\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(0, 14, 26);stroke:rgb(26, 62, 95);stroke-width:1px;}#chatgpt-mermaid-\_r_le\_ .node rect,#chatgpt-mermaid-\_r_le\_ .node circle,#chatgpt-mermaid-\_r_le\_ .node ellipse,#chatgpt-mermaid-\_r_le\_ .node polygon,#chatgpt-mermaid-\_r_le\_ .node path{fill:rgb(0, 40, 77);stroke:rgba(255, 255, 255, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_le\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_le\_ .mermaid-decision .label-container{fill:rgb(0, 14, 26);stroke:rgb(26, 62, 95);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_le\_ .edgePaths .flowchart-link{stroke:rgb(175, 175, 175);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_le\_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_le\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Select serviceEnter postcodeArea supported?Show unavailable messageSelect property sizeCalculate starting priceAdvanced options?Apply modifiers and add-onsShow final estimateSave quoteBook or request manualquoteNoYesYesNo

# 6. Booking and calendar system

This module needs special attention because duplicate bookings and calendar conflicts can create real business problems.

### Booking workflow

\#chatgpt-mermaid-\_r_lq\_{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;fill:rgb(237, 237, 237);}@keyframes edge-animation-frame{from{stroke-dashoffset:0;}}@keyframes dash{to{stroke-dashoffset:0;}}#chatgpt-mermaid-\_r_lq\_ .edge-animation-slow{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 50s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_lq\_ .edge-animation-fast{stroke-dasharray:9,5!important;stroke-dashoffset:900;animation:dash 20s linear infinite;stroke-linecap:round;}#chatgpt-mermaid-\_r_lq\_ .error-icon{fill:rgb(48, 48, 48);}#chatgpt-mermaid-\_r_lq\_ .error-text{fill:rgb(237, 237, 237);stroke:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_lq\_ .edge-thickness-normal{stroke-width:1px;}#chatgpt-mermaid-\_r_lq\_ .edge-thickness-thick{stroke-width:3.5px;}#chatgpt-mermaid-\_r_lq\_ .edge-pattern-solid{stroke-dasharray:0;}#chatgpt-mermaid-\_r_lq\_ .edge-thickness-invisible{stroke-width:0;fill:none;}#chatgpt-mermaid-\_r_lq\_ .edge-pattern-dashed{stroke-dasharray:3;}#chatgpt-mermaid-\_r_lq\_ .edge-pattern-dotted{stroke-dasharray:2;}#chatgpt-mermaid-\_r_lq\_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_lq\_ .marker.cross{stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_lq\_ svg{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:16px;}#chatgpt-mermaid-\_r_lq\_ p{margin:0;}#chatgpt-mermaid-\_r_lq\_ .label{font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_lq\_ .cluster-label text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_lq\_ .cluster-label span{color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_lq\_ .cluster-label span p{background-color:transparent;}#chatgpt-mermaid-\_r_lq\_ .label text,#chatgpt-mermaid-\_r_lq\_ span{fill:rgb(237, 237, 237);color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_lq\_ .node rect,#chatgpt-mermaid-\_r_lq\_ .node circle,#chatgpt-mermaid-\_r_lq\_ .node ellipse,#chatgpt-mermaid-\_r_lq\_ .node polygon,#chatgpt-mermaid-\_r_lq\_ .node path{fill:rgb(9, 23, 44);stroke:rgb(31, 78, 148);stroke-width:1px;}#chatgpt-mermaid-\_r_lq\_ .rough-node .label text,#chatgpt-mermaid-\_r_lq\_ .node .label text,#chatgpt-mermaid-\_r_lq\_ .image-shape .label,#chatgpt-mermaid-\_r_lq\_ .icon-shape .label{text-anchor:middle;}#chatgpt-mermaid-\_r_lq\_ .node .katex path{fill:#000;stroke:#000;stroke-width:1px;}#chatgpt-mermaid-\_r_lq\_ .rough-node .label,#chatgpt-mermaid-\_r_lq\_ .node .label,#chatgpt-mermaid-\_r_lq\_ .image-shape .label,#chatgpt-mermaid-\_r_lq\_ .icon-shape .label{text-align:center;}#chatgpt-mermaid-\_r_lq\_ .node.clickable{cursor:pointer;}#chatgpt-mermaid-\_r_lq\_ .root .anchor path{fill:rgb(175, 175, 175)!important;stroke-width:0;stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_lq\_ .arrowheadPath{fill:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_lq\_ .edgePath .path{stroke:rgb(175, 175, 175);stroke-width:1px;}#chatgpt-mermaid-\_r_lq\_ .flowchart-link{stroke:rgb(175, 175, 175);fill:none;}#chatgpt-mermaid-\_r_lq\_ .edgeLabel{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-\_r_lq\_ .edgeLabel p{background-color:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_lq\_ .edgeLabel rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_lq\_ .labelBkg{background-color:rgba(0, 0, 0, 0.5);}#chatgpt-mermaid-\_r_lq\_ .cluster rect{fill:rgb(48, 48, 48);stroke:rgba(255, 255, 255, 0.15);stroke-width:1px;}#chatgpt-mermaid-\_r_lq\_ .cluster text{fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_lq\_ .cluster span{color:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_lq\_ div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";font-size:12px;background:rgb(48, 48, 48);border:1px solid rgba(255, 255, 255, 0.15);border-radius:2px;pointer-events:none;z-index:100;}#chatgpt-mermaid-\_r_lq\_ .flowchartTitleText{text-anchor:middle;font-size:18px;fill:rgb(237, 237, 237);}#chatgpt-mermaid-\_r_lq\_ rect.text{fill:none;stroke-width:0;}#chatgpt-mermaid-\_r_lq\_ .icon-shape,#chatgpt-mermaid-\_r_lq\_ .image-shape{background-color:rgb(0, 0, 0);text-align:center;}#chatgpt-mermaid-\_r_lq\_ .icon-shape p,#chatgpt-mermaid-\_r_lq\_ .image-shape p{background-color:rgb(0, 0, 0);padding:2px;}#chatgpt-mermaid-\_r_lq\_ .icon-shape .label rect,#chatgpt-mermaid-\_r_lq\_ .image-shape .label rect{opacity:0.5;background-color:rgb(0, 0, 0);fill:rgb(0, 0, 0);}#chatgpt-mermaid-\_r_lq\_ .label-icon{display:inline-block;height:1em;overflow:visible;vertical-align:-0.125em;}#chatgpt-mermaid-\_r_lq\_ .node .label-icon path{fill:currentColor;stroke:revert;stroke-width:revert;}#chatgpt-mermaid-\_r_lq\_ .node .neo-node{stroke:rgb(31, 78, 148);}#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].node rect,#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].cluster rect,#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].node polygon{stroke:url(#chatgpt-mermaid-\_r_lq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].swimlane.cluster rect{filter:none;}#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].node path{stroke:url(#chatgpt-mermaid-\_r_lq\_-gradient);stroke-width:1px;}#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].node .outer-path{filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].node .neo-line path{stroke:rgb(31, 78, 148);filter:none;}#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].node circle{stroke:url(#chatgpt-mermaid-\_r_lq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].node circle .state-start{fill:#000000;}#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].icon-shape .icon{fill:url(#chatgpt-mermaid-\_r_lq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_lq\_ [data-look="neo"].icon-shape .icon-neo path{stroke:url(#chatgpt-mermaid-\_r_lq\_-gradient);filter:drop-shadow( 1px 2px 2px rgba(185,185,185,1));}#chatgpt-mermaid-\_r_lq\_ .node text{font-size:14px;font-weight:600;letter-spacing:normal;fill:rgb(153, 206, 255);}#chatgpt-mermaid-\_r_lq\_ .edgeLabels text{font-size:13px;font-weight:600;letter-spacing:-0.08px;fill:rgb(153, 206, 255);}#chatgpt-mermaid-\_r_lq\_ .node tspan[font-weight="normal"],#chatgpt-mermaid-\_r_lq\_ .edgeLabels tspan[font-weight="normal"]{font-weight:600;}#chatgpt-mermaid-\_r_lq\_ .edgeLabel .label rect{opacity:1;rx:13px;ry:13px;fill:rgb(0, 14, 26);stroke:rgb(26, 62, 95);stroke-width:1px;}#chatgpt-mermaid-\_r_lq\_ .node rect,#chatgpt-mermaid-\_r_lq\_ .node circle,#chatgpt-mermaid-\_r_lq\_ .node ellipse,#chatgpt-mermaid-\_r_lq\_ .node polygon,#chatgpt-mermaid-\_r_lq\_ .node path{fill:rgb(0, 40, 77);stroke:rgba(255, 255, 255, 0.1);stroke-width:1px;}#chatgpt-mermaid-\_r_lq\_ .node rect{rx:16px;ry:16px;}#chatgpt-mermaid-\_r_lq\_ .mermaid-decision .label-container{fill:rgb(0, 14, 26);stroke:rgb(26, 62, 95);stroke-dasharray:2,2;}#chatgpt-mermaid-\_r_lq\_ .edgePaths .flowchart-link{stroke:rgb(175, 175, 175);stroke-width:1px;stroke-linecap:round;stroke-linejoin:round;}#chatgpt-mermaid-\_r_lq\_ .marker{fill:rgb(175, 175, 175);stroke:rgb(175, 175, 175);}#chatgpt-mermaid-\_r_lq\_ :root{--mermaid-font-family:-apple-system-body,ui-sans-serif,-apple-system,system-ui,"Segoe UI",Helvetica,"Apple Color Emoji",Arial,sans-serif,"Segoe UI Emoji","Segoe UI Symbol";}Customer selects serviceSelect dateCheck available time slotsGoogle Calendar free/busycheckSlot available?Choose another slotReserve slot in databasePayment enabled?Stripe checkoutPay later bookingPayment successful?Payment failed / releasereservationConfirm bookingCreate Google CalendareventSend confirmationNoYesYesNoNoYes

### Calendar rules

The admin should be able to configure:

- Working days.
- Start and finish times.
- Appointment duration.
- Travel buffer.
- Minimum booking lead time.
- Maximum bookings per day.
- Blocked dates and holidays.

### Important implementation details

- Check Google Calendar availability before showing slots.
- Recheck availability when confirming a booking.
- Use a database transaction to prevent two customers booking the same slot.
- Keep a local booking record even if Google Calendar is temporarily unavailable.
- Retry failed calendar synchronization through the background worker.

For recurring bookings, create individual appointment occurrences linked to a parent booking. This allows one appointment to be rescheduled without changing the entire series.

# 7. Image upload and storage

External storage selected

I recommend evaluating Cloudflare R2.

[Exploring Cloudflare R2 Storage: A Cost-Effective Alternative to S3 - DEV Community](https://images.openai.com/static-rsc-4/SBh69Qrd6p40-bqIUa-c0bf5lMnBe2B2k2a5jrC1M4urE3iCx9OGSCnKwB28nE4W4Y1rs8HKB803cZJrvDRSmwZz6GjZKPaGN7HAfgmU2DputiySRD-WvuVeMAYCzu45p27-XuHhHnL7VaUjApdmOvUf4Iec4xyaVCI07NQW0nU?purpose=inline)

Cloudflare R2

S3-compatible object storage with no direct internet egress charges under its standard pricing model.

[Official website](https://www.cloudflare.com/developer-platform/products/r2/?utm_source=chatgpt.com)

### Upload process

1. Customer selects images.
2. Frontend requests a temporary upload URL from FastAPI.
3. Backend generates a signed upload URL.
4. Browser uploads directly to object storage.
5. Backend stores image metadata in PostgreSQL.
6. Admin can view the images through authorized access.

Do not send large image files through FastAPI unless necessary.

### Security rules

- Accept JPEG, PNG and WebP.
- Validate actual file types, not just extensions.
- Set file-size and image-count limits.
- Use private storage by default.
- Generate temporary signed URLs for viewing.
- Never expose storage credentials in frontend code.

# 8. API design

Use REST APIs with versioning:

`/api/v1/...`

Suggested endpoints:

| Method | Endpoint                 | Purpose                 |
| ------ | ------------------------ | ----------------------- |
| GET    | `/services`              | List active services    |
| GET    | `/services/{id}`         | Service details         |
| POST   | `/quotes/calculate`      | Calculate instant quote |
| POST   | `/quotes`                | Save quote              |
| POST   | `/quote-requests`        | Submit manual quote     |
| POST   | `/uploads/presign`       | Generate upload URL     |
| GET    | `/availability`          | Get available slots     |
| POST   | `/bookings`              | Create booking          |
| GET    | `/bookings/{id}`         | Booking details         |
| POST   | `/payments/checkout`     | Create Stripe checkout  |
| POST   | `/webhooks/stripe`       | Receive Stripe events   |
| GET    | `/admin/bookings`        | Manage bookings         |
| GET    | `/admin/quotes`          | Manage quote requests   |
| CRUD   | `/admin/services`        | Manage services         |
| CRUD   | `/admin/pricing`         | Manage pricing          |
| CRUD   | `/admin/postcodes`       | Manage service areas    |
| POST   | `/admin/reviews/request` | Trigger review request  |

Admin endpoints must require authentication and authorization.

# 9. Admin dashboard design

I recommend keeping the admin panel simple.

## Admin dashboard — proposed layout

Bookings

# 12

Illustrative data

New quotes

# 4

Illustrative data

Completed jobs

# 28

Illustrative data

Pending jobs

# 6

Illustrative data

Navigation

- Overview
- Bookings
- Manual quote requests
- Customers
- Services
- Pricing configuration
- Service areas
- Calendar
- Reviews
- Settings

### Job status workflow

Use these statuses:

`NEW → QUOTED → BOOKED → COMPLETED`

Additional statuses:

- LOST
- CANCELLED
- PAYMENT_PENDING

The dashboard should also provide search, filters and booking detail pages.

# 10. Docker and VPS deployment design

Use Docker Compose to manage the application.

### Production containers

| Container  | Purpose                 |
| ---------- | ----------------------- |
| `frontend` | Next.js                 |
| `backend`  | FastAPI API             |
| `worker`   | Background tasks        |
| `db`       | PostgreSQL              |
| `caddy`    | HTTPS and reverse proxy |

All containers run on one VPS initially.

### Request routing

```
yourdomain.com
       |
     Caddy
       |
       |------ /       -> Next.js
       |
       |------ /api/*  -> FastAPI
```

This gives you a single domain, simplifies browser security and avoids unnecessary cross-origin configuration.

### Suggested project root

```
gardening-website/
│
├── frontend/
├── backend/
├── docker/
│   └── caddy/
│
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
```

### Production requirements

- Environment variables for secrets.
- HTTPS enabled automatically.
- PostgreSQL data persisted in a Docker volume.
- Database backups stored outside the VPS.
- Container health checks and restart policies.
- Log rotation.
- Separate development and production environments.
- No public database port.

Do not put Stripe secret keys, Google credentials or storage credentials inside frontend code or Git.

# 11. Security and reliability

These are not optional, even for a small website.

| Area              | Implementation                               |
| ----------------- | -------------------------------------------- |
| Admin login       | Secure password hashing                      |
| Sessions          | HttpOnly, Secure, SameSite cookies           |
| API               | Input validation with Pydantic               |
| Database          | Parameterized queries through ORM            |
| Uploads           | File validation and private storage          |
| Payments          | Verified Stripe webhooks                     |
| Booking           | Database transaction and conflict prevention |
| Spam              | Rate limiting and CAPTCHA where appropriate  |
| Secrets           | Environment variables                        |
| Backups           | Automated off-server backups                 |
| Admin permissions | Role-based access control                    |

I recommend starting with one administrator role. More roles can be added if the business grows.

## 12. Development work plan

I would divide development into 10 milestones.

| Milestone | Development work                   | Priority |
| --------- | ---------------------------------- | -------- |
| M1        | Project setup, Docker, VPS         | Critical |
| M2        | Database and backend foundation    | Critical |
| M3        | Website UI and responsive pages    | Critical |
| M4        | Services and pricing engine        | Critical |
| M5        | Instant and manual quote systems   | Critical |
| M6        | Admin dashboard                    | Critical |
| M7        | Booking and Google Calendar        | Critical |
| M8        | Stripe and email notifications     | High     |
| M9        | Reviews, analytics and legal pages | High     |
| M10       | Testing, optimization and launch   | Critical |

### Suggested development schedule

Illustrative development allocation, assuming one developer working consistently. Not a fixed deadline.

0 weeks0.55 weeks1.1 weeks1.65 weeks2.2 weeksFoundationWebsite UIQuote engineAdmin panelBooking & integrationsTesting & launch

Estimated total: 9 weeks, allowing some overlap between development tasks. Integration testing and unexpected issues can extend this.

# 13. Testing and launch requirements

Before deployment, every major feature needs to pass acceptance testing.

| Feature       | Acceptance criteria                     |
| ------------- | --------------------------------------- |
| Website       | Responsive on mobile and desktop        |
| Pricing       | Correct service-specific calculations   |
| Postcodes     | Reject unsupported locations            |
| Manual quotes | Images and customer details saved       |
| Admin         | Can update pricing without code changes |
| Booking       | Cannot double-book a time slot          |
| Calendar      | Confirmed bookings synchronize          |
| Stripe        | Payment and webhook verification work   |
| Email         | Customer and admin notifications work   |
| Security      | Admin routes and uploads protected      |
| Performance   | Fast page loads and optimized images    |
| Backup        | Database can be restored successfully   |

# 14. Decisions still needed before implementation

The architecture is settled, but some business rules need confirmation.

| Question           | My recommendation                          |
| ------------------ | ------------------------------------------ |
| Currency           | AUD                                        |
| Business timezone  | Australia/Perth                            |
| Booking duration   | Configurable per service                   |
| Booking payment    | Full payment initially, configurable later |
| Pay later          | Admin toggle                               |
| Quote validity     | Configurable, e.g. 14 days                 |
| Image limit        | 5 images per request initially             |
| Maximum image size | 10 MB each                                 |
| Customer accounts  | No accounts required for v1                |
| Admin accounts     | Login required                             |
| Recurring bookings | Individual appointment occurrences         |
| Storage            | Cloudflare R2                              |
| Reviews            | Google Business Profile API                |

These are proposed defaults, not confirmed business requirements.

# 15. Recommended implementation order

My recommendation is to start coding in this order:

## First development sprint

Start here

1. Initialize Git repository.
2. Create Next.js frontend.
3. Create FastAPI backend.
4. Configure PostgreSQL and SQLAlchemy.
5. Create Docker Compose configuration.
6. Configure Caddy reverse proxy.
7. Run the complete application locally on macOS.
8. Deploy the first version to Hostinger VPS.

Sprint completion: The frontend, backend and database communicate successfully, and the website is accessible through HTTPS on your domain.

## 16. Final architecture summary

## Version 1 architecture

Architecture

Modular monolith

Frontend

Next.js / JavaScript

Backend

FastAPI / Python

Database

PostgreSQL

Infrastructure

Docker Compose / VPS

Image storage

Cloudflare R2

Estimated development

\~9 weeks

Not included in v1: AI quoting, SMS, blog, mobile app and multi-vendor functionality.

Next step: Finalize the business rules in section 14, then create the actual development blueprint: database SQLAlchemy models, API specifications, complete folder structure, Docker files and implementation tasks.

I can also review your implementation and update the development task list every week.

🛠️Review project progress weeklyTap to set up
