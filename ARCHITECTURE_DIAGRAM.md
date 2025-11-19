# GeoPrice Architecture Diagrams

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER BROWSER                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Next.js Frontend (React)                     │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐         │  │
│  │  │   Pages    │  │ Components │  │   Hooks    │         │  │
│  │  │ (SSR/CSR)  │  │   (UI)     │  │  (Logic)   │         │  │
│  │  └────────────┘  └────────────┘  └────────────┘         │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────────────────────┬─────────────────────────────────────┘
                            │ HTTPS
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Vercel Edge Network                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         Express.js Backend (Serverless)                   │  │
│  │  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐         │  │
│  │  │ Routes │→ │Control │→ │Service │→ │  Repo  │         │  │
│  │  │        │  │  lers  │  │        │  │        │         │  │
│  │  └────────┘  └────────┘  └────────┘  └────────┘         │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────┬─────────────────────┬───────────────────────────────┘
            │                     │
            ▼                     ▼
┌───────────────────┐   ┌───────────────────┐
│  MongoDB Atlas    │   │   External APIs   │
│  ┌─────────────┐  │   │  ┌─────────────┐ │
│  │  Products   │  │   │  │   Stripe    │ │
│  │   Orders    │  │   │  │  Exchange   │ │
│  └─────────────┘  │   │  │    Rate     │ │
└───────────────────┘   │  └─────────────┘ │
                        └───────────────────┘
```

## 🔄 Request Flow Diagram

### 1. Initial Page Load (SSR)

```
User visits site
      │
      ▼
┌─────────────────┐
│ Vercel Edge     │ Reads IP → Detects Country (e.g., "IN")
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Next.js Server  │ Fetches products from backend
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Backend API     │ Queries MongoDB
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ MongoDB         │ Returns products
└────────┬────────┘
         │
         ▼
HTML with products sent to browser (SEO-friendly, fast)
```

### 2. Client-Side Price Fetch

```
Page loads in browser
      │
      ▼
React hydrates (becomes interactive)
      │
      ▼
useLocalizedPrices hook runs
      │
      ▼
POST /api/price { country: "IN" }
      │
      ▼
┌─────────────────────────┐
│ Price Controller        │
│ 1. Validate country     │
│ 2. Map IN → INR         │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ Currency Service        │
│ 1. Check cache (15 min) │
│ 2. If miss, fetch rates │
│ 3. Convert USD → INR    │
└────────┬────────────────┘
         │
         ▼
Returns: [
  { productId: "123", localizedPrice: 8299.99, currency: "INR" }
]
      │
      ▼
Frontend stores in Map
      │
      ▼
UI displays: ₹8,299.99
```

### 3. Checkout Flow

```
User clicks "Buy Now"
      │
      ▼
POST /api/create-checkout-session
{ productId: "123", currency: "INR", country: "IN" }
      │
      ▼
┌─────────────────────────┐
│ Checkout Controller     │
│ 1. Validate input       │
│ 2. Get product          │
│ 3. Convert price        │
└────────┬────────────────┘
         │
         ▼
┌─────────────────────────┐
│ Payment Service         │
│ Creates Stripe session  │
└────────┬────────────────┘
         │
         ▼
Returns: { sessionUrl: "https://checkout.stripe.com/..." }
      │
      ▼
window.location.href = sessionUrl
      │
      ▼
User redirected to Stripe
      │
      ▼
User completes payment
      │
      ▼
Stripe sends webhook to /api/webhook
      │
      ▼
┌─────────────────────────┐
│ Webhook Controller      │
│ 1. Verify signature     │
│ 2. Extract session data │
│ 3. Create order         │
└────────┬────────────────┘
         │
         ▼
Order saved in MongoDB
      │
      ▼
User redirected to /success
```

## 📊 Data Flow Layers

```
┌──────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                     │
│  (React Components, UI, User Interactions)               │
│  - ProductCard.tsx                                       │
│  - CountrySelector.tsx                                   │
│  - Header.tsx                                            │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                      │
│  (Business Logic, State Management)                      │
│  - useCountry hook                                       │
│  - useLocalizedPrices hook                               │
│  - API Client                                            │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│                      API LAYER                            │
│  (HTTP Endpoints, Request/Response)                      │
│  - Routes                                                │
│  - Controllers                                           │
│  - Middleware                                            │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│                    BUSINESS LOGIC LAYER                   │
│  (Core Functionality, Domain Logic)                      │
│  - Product Service                                       │
│  - Currency Service                                      │
│  - Payment Service                                       │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│                    DATA ACCESS LAYER                      │
│  (Database Operations)                                   │
│  - Product Repository                                    │
│  - Order Repository                                      │
└────────────────────────┬─────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────┐
│                    DATABASE LAYER                         │
│  (Data Storage)                                          │
│  - MongoDB (Products, Orders)                            │
└──────────────────────────────────────────────────────────┘
```

## 🔐 Security Flow

```
┌─────────────────────────────────────────────────────────┐
│                    Request Arrives                       │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  CORS Middleware: Check origin                          │
│  ✓ Allowed: Continue                                    │
│  ✗ Blocked: Return 403                                  │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Request Logger: Log request details                    │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Webhook Route: Verify Stripe signature                 │
│  (Only for /api/webhook)                                │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Body Parser: Parse JSON                                │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Controller: Validate input                             │
│  - Required fields                                      │
│  - Data types                                           │
│  - Business rules                                       │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Mongoose: Schema validation                            │
│  - Type checking                                        │
│  - Constraints (min, max, unique)                       │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│  Error Handler: Catch and format errors                 │
│  - Hide sensitive info in production                    │
│  - Log errors                                           │
│  - Return user-friendly message                         │
└─────────────────────────────────────────────────────────┘
```

## 🎯 Component Hierarchy

```
App
│
├── Layout (Root)
│   ├── Header
│   │   └── CountrySelector
│   │       └── Select (Radix UI)
│   │
│   └── Page (Server Component)
│       └── HomePageClient (Client Component)
│           ├── useCountry hook
│           ├── useLocalizedPrices hook
│           │
│           ├── Alert (Error display)
│           │
│           └── ProductCard (for each product)
│               ├── Image (Next.js optimized)
│               ├── Card (Shadcn UI)
│               ├── Skeleton (Loading state)
│               └── Button (Buy Now)
│
├── /success (Success page)
└── /cancel (Cancel page)
```

## 💾 Database Schema

```
┌─────────────────────────────────────────┐
│            Products Collection           │
├─────────────────────────────────────────┤
│ _id: ObjectId                           │
│ name: String (required, trimmed)        │
│ description: String (required)          │
│ basePrice: Number (required, min: 0)    │
│ sku: String (required, unique, indexed) │
│ images: [String] (required, min: 1)     │
│ createdAt: Date (auto)                  │
│ updatedAt: Date (auto)                  │
└─────────────────────────────────────────┘
                    │
                    │ Referenced by
                    ▼
┌─────────────────────────────────────────┐
│             Orders Collection            │
├─────────────────────────────────────────┤
│ _id: ObjectId                           │
│ productId: ObjectId → Products          │
│ amount: Number (required, min: 0)       │
│ currency: String (USD|INR|GBP)          │
│ stripeSessionId: String (unique, index) │
│ status: String (pending|paid|failed)    │
│ customerCountry: String (required)      │
│ createdAt: Date (auto)                  │
│ updatedAt: Date (auto)                  │
└─────────────────────────────────────────┘
```

## 🚀 Deployment Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    GitHub Repository                     │
│  (Source Code)                                          │
└────────────────────────┬────────────────────────────────┘
                         │ git push
                         ▼
┌─────────────────────────────────────────────────────────┐
│                    Vercel Platform                       │
│  ┌───────────────────────────────────────────────────┐ │
│  │  1. Detect changes                                │ │
│  │  2. Install dependencies                          │ │
│  │  3. Build frontend (Next.js)                      │ │
│  │  4. Build backend (TypeScript → JavaScript)       │ │
│  │  5. Deploy to edge network                        │ │
│  └───────────────────────────────────────────────────┘ │
└────────────────────────┬────────────────────────────────┘
                         │
         ┌───────────────┴───────────────┐
         │                               │
         ▼                               ▼
┌──────────────────┐          ┌──────────────────┐
│  Frontend        │          │  Backend         │
│  (Static + SSR)  │          │  (Serverless)    │
│                  │          │                  │
│  Deployed to:    │          │  Deployed to:    │
│  Edge Network    │          │  Edge Functions  │
│  (Global CDN)    │          │  (Auto-scale)    │
└──────────────────┘          └──────────────────┘
```

This visual guide should help you understand how all the pieces fit together!
