# Eve Sales Calculator — Build Spec

> **Project:** Eve Sales Calculator
>
> **Parent Brand:** Nano Energii
>
> **Primary Goal:** Build an interactive web-based sales configurator that allows prospects to design a custom Eve unit by selecting features, add-ons, and unit size — then see a real-time calculated purchase price.

---

## 1. Overview

The Eve Sales Calculator is a client-facing tool that works like a car configurator for Nano Energii's Eve nano-scale energy units. After initial contact with a prospect, the sales team sends them a link to this calculator so they can "design" their custom Eve model by ticking boxes for features and add-ons.

Each region (Fiji, Gove, South Africa, UAE) receives a branded "custom model" experience — e.g. "Eve Fiji Model", "Eve Gove Model" — while using the same underlying configurator engine.

**Use case:**
- Prospect receives link after initial sales contact
- They select their region/model name
- They choose a unit size (3 smaller models, not the 640)
- They tick features and add-ons from Brad's spreadsheet
- Calculator shows real-time price at current rates
- Prospect can submit their configuration as an enquiry

---

## 2. Features

### Core
- Interactive feature/add-on selector with tick boxes
- Unit size selection (3 sizes from Brad's spreadsheet)
- Real-time price calculation as options change
- Region-specific model naming (Eve Fiji Model, Eve Gove Model, Eve SA Model, Eve UAE Model)
- Mobile-friendly responsive design
- Summary view with itemised breakdown

### Secondary
- PDF/print-friendly quote summary
- "Send my configuration" — email capture + enquiry submission
- Shareable configuration links (URL state encoding)
- Admin-editable pricing data (JSON config file, no database required for v1)
- Currency display per region (FJD, AUD, ZAR, AED, USD fallback)

### Nice-to-Have (Phase 2+)
- Animated unit visualisation showing selected features
- Comparison mode (side-by-side unit sizes)
- Sales team dashboard for submitted configurations
- Integration with CRM for lead capture

---

## 3. Tech Stack

Standard Shadow Software stack:

```
Framework:    React 18 + TypeScript + Vite
Styling:      Tailwind CSS + Shadcn/ui (Radix primitives) + Lucide icons
State:        React state + URL search params (no external state library needed)
Forms:        React Hook Form + Zod (enquiry submission form)
Routing:      React Router (region routes)
Data:         Static JSON config (pricing/features) — no backend for v1
Deployment:   Vercel / Replit / static hosting (pure SPA)
```

> **Note:** No backend or database required for Phase 1. Pricing data lives in a TypeScript config file. Enquiry submissions POST to a webhook (Make/Zapier/n8n or direct email via Resend).

---

## 4. Data Model

### Unit Sizes

Three unit sizes sourced from Brad's spreadsheet (not the 640):

```typescript
interface UnitSize {
  id: string;              // e.g. "eve-120", "eve-240", "eve-360"
  name: string;            // Display name
  basePrice: number;       // Base price in USD
  description: string;     // Short description
  specs: {
    capacity: string;      // e.g. "120 kWh"
    dimensions: string;
    weight: string;
    outputPower: string;
  };
}
```

### Features & Add-ons

```typescript
interface Feature {
  id: string;
  category: FeatureCategory;
  name: string;
  description: string;
  price: number;           // Additional cost in USD
  compatibleSizes: string[]; // Which unit sizes support this
  isDefault: boolean;      // Included in base price
  mutuallyExclusive?: string[]; // Feature IDs that can't be selected with this
}

type FeatureCategory =
  | "power-management"
  | "connectivity"
  | "monitoring"
  | "physical"
  | "warranty-support"
  | "installation";
```

### Region Configuration

```typescript
interface RegionConfig {
  id: string;              // e.g. "fiji", "gove", "south-africa", "uae"
  modelName: string;       // e.g. "Eve Fiji Model"
  currency: string;        // e.g. "FJD", "AUD", "ZAR", "AED"
  currencySymbol: string;  // e.g. "$", "R", "د.إ"
  exchangeRate: number;    // USD to local currency
  taxRate: number;         // Local tax percentage
  taxLabel: string;        // e.g. "GST", "VAT"
  shippingEstimate?: number;
  contactEmail: string;
  contactPhone?: string;
}
```

### Configuration State

```typescript
interface CalculatorState {
  region: string;
  unitSize: string;
  selectedFeatures: string[];  // Feature IDs
  quantity: number;
}

interface PriceBreakdown {
  basePrice: number;
  featuresTotal: number;
  subtotal: number;
  tax: number;
  shipping: number;
  total: number;
  currency: string;
  items: { name: string; price: number }[];
}
```

---

## 5. UI/UX

### Layout (Single Page App, stepped flow)

1. **Region Selection** — Card grid: Fiji, Gove, South Africa, UAE. Each shows flag/icon + model name. Sets branding context for rest of flow.

2. **Unit Size Selection** — 3 cards showing each Eve model size with specs, base price, and "Select" CTA. Visual size comparison.

3. **Feature Configurator** — Main interaction screen:
   - Left/top: categorised feature list with checkboxes
   - Right/bottom (sticky on mobile): live price summary panel
   - Categories as collapsible accordion sections
   - Incompatible features greyed out with tooltip explanation
   - Default/included features shown as pre-ticked and labelled "Included"

4. **Summary & Quote** — Full itemised breakdown:
   - Selected unit + size
   - All selected features with individual prices
   - Subtotal, tax, shipping estimate, total
   - "Send My Configuration" button → enquiry form
   - "Download PDF" / "Print Quote" button
   - "Start Over" / "Modify" button

5. **Enquiry Form** (modal or inline):
   - Name, company, email, phone
   - Message/notes field
   - Submits configuration + contact details to webhook
   - Confirmation screen with next steps

### Design Principles
- **Mobile-first** — prospects may be on phone in remote locations (Fiji, Gove)
- **Fast** — no loading spinners, instant price updates
- **Clear** — no ambiguity about what's included vs extra
- **Branded** — Nano Energii colours and logo, region-specific model badge
- **Trustworthy** — clean, professional, not cheap-looking

### Branding
- Primary: Nano Energii brand colours (green energy palette)
- Accent: region-specific subtle colour variation
- Font: Inter (body), plus a display font for headings
- Logo: Nano Energii / Eve product logo in header
- Dark mode support (optional, Phase 2)

---

## 6. Project Structure

```
/
├── src/
│   ├── App.tsx
│   ├── main.tsx
│   ├── index.css
│   ├── components/
│   │   ├── ui/                        # Shadcn/ui components
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   ├── Footer.tsx
│   │   │   └── PriceSummaryPanel.tsx  # Sticky price sidebar/footer
│   │   ├── region/
│   │   │   ├── RegionSelector.tsx
│   │   │   └── RegionCard.tsx
│   │   ├── configurator/
│   │   │   ├── UnitSizeSelector.tsx
│   │   │   ├── UnitSizeCard.tsx
│   │   │   ├── FeatureConfigurator.tsx
│   │   │   ├── FeatureCategory.tsx
│   │   │   ├── FeatureItem.tsx
│   │   │   └── IncompatibilityTooltip.tsx
│   │   ├── summary/
│   │   │   ├── QuoteSummary.tsx
│   │   │   ├── LineItem.tsx
│   │   │   └── PdfExport.tsx
│   │   └── enquiry/
│   │       ├── EnquiryForm.tsx
│   │       └── Confirmation.tsx
│   ├── config/
│   │   ├── units.ts                   # Unit size definitions + base prices
│   │   ├── features.ts               # All features/add-ons with pricing
│   │   ├── regions.ts                # Region configs + exchange rates
│   │   └── index.ts                  # Unified config export
│   ├── hooks/
│   │   ├── useCalculator.ts          # Main calculator state + logic
│   │   ├── usePriceBreakdown.ts      # Derived price computation
│   │   └── useRegion.ts             # Current region context
│   ├── lib/
│   │   ├── pricing.ts               # Price calculation utilities
│   │   ├── pdf.ts                   # PDF generation logic
│   │   ├── url-state.ts            # URL param encode/decode for sharing
│   │   └── utils.ts                # General utilities (cn, formatCurrency)
│   └── types/
│       └── index.ts                 # Shared TypeScript types
├── public/
│   ├── logo.svg
│   └── regions/                     # Region flag/badge assets
├── tailwind.config.ts
├── vite.config.ts
├── tsconfig.json
├── package.json
├── components.json                  # Shadcn/ui config
└── README.md
```

---

## 7. Deployment

### Phase 1 (MVP)
- Static SPA deployed to Vercel or Replit
- No backend — pricing config baked into bundle
- Enquiry form POSTs to webhook endpoint (Make/Zapier/n8n → email + CRM)
- Custom domain: TBD (e.g. `configure.nanoenergii.com` or `eve.nanoenergii.com`)

### Phase 2
- Add lightweight API for dynamic pricing updates
- Admin panel to edit features/pricing without code deploy
- Lead storage in database (Neon PostgreSQL + Drizzle)
- Email notifications via Resend on enquiry submission

### Hosting Requirements
- HTTPS (automatic via Vercel/Replit)
- Fast global CDN (prospects in Fiji, Gove, SA, UAE)
- No server-side rendering needed (pure client SPA)

---

## 8. Phases

### Phase 1 — MVP Calculator (Target: 1-2 days)
- [ ] Project scaffold (Vite + React + TS + Tailwind + Shadcn)
- [ ] Data config files with placeholder pricing (awaiting Brad's spreadsheet)
- [ ] Region selection screen
- [ ] Unit size selection screen
- [ ] Feature configurator with live price calculation
- [ ] Quote summary with itemised breakdown
- [ ] Mobile-responsive layout
- [ ] Enquiry form with webhook submission
- [ ] Deploy to Vercel/Replit

### Phase 2 — Polish & Integration
- [ ] PDF quote generation / print styling
- [ ] Shareable configuration URLs
- [ ] Currency conversion with live rates (or manual update)
- [ ] Nano Energii branding pass (colours, logo, fonts)
- [ ] Loading Brad's actual feature/pricing data
- [ ] Connect enquiry webhook to CRM

### Phase 3 — Advanced
- [ ] Admin pricing editor (no-code updates)
- [ ] Unit visualisation / 3D preview
- [ ] Comparison mode
- [ ] Analytics (which features are most popular per region)
- [ ] Sales team notification on high-value configs
- [ ] Multi-language support (if needed for regions)

---

## 9. Data Requirements (Pending)

The following data is needed from Brad's spreadsheet to populate the config:

- [ ] **Unit sizes:** names, specs, base prices for the 3 smaller models
- [ ] **Features/add-ons:** full list with categories, prices, compatibility per unit size
- [ ] **Region pricing:** exchange rates, tax rates, shipping estimates
- [ ] **Branding:** Nano Energii / Eve logo files, brand colour hex codes
- [ ] **Contact details:** per-region sales contact email/phone

Until this data arrives, the calculator will use realistic placeholder data that can be swapped in via the config files.

---

## 10. Notes

- This is a **sales tool**, not a public marketing site. It's sent directly to qualified prospects.
- Keep it fast and simple. No login required. No accounts.
- The configurator should feel premium — this is selling expensive energy infrastructure.
- URL state encoding means a salesperson can pre-configure a unit and send the link with options already selected.
- All pricing in USD as base, converted to local currency per region for display.
