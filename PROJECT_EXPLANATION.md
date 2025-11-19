# GeoPrice Project - Complete Explanation

## 📋 Table of Contents
1. [Project Overview](#project-overview)
2. [Technology Stack Explained](#technology-stack-explained)
3. [Backend Architecture](#backend-architecture)
4. [Frontend Architecture](#frontend-architecture)
5. [Data Flow](#data-flow)
6. [File-by-File Explanation](#file-by-file-explanation)

---

## 🎯 Project Overview

**GeoPrice** is a full-stack e-commerce platform that automatically adjusts product prices based on the user's geographic location. 

### Core Functionality:
1. **Automatic Location Detection**: Detects user's country using Vercel's geolocation
2. **Real-time Currency Conversion**: Converts prices using live exchange rates
3. **Secure Payments**: Integrates with Stripe for payment processing
4. **Responsive UI**: Modern, mobile-friendly interface built with Next.js

### Business Problem It Solves:
- **For Customers**: See prices in familiar currency, no mental conversion needed
- **For Business**: Increase conversion rates by reducing friction in international sales
- **For Operations**: Automated pricing eliminates manual price management

---

## 🛠 Technology Stack Explained

### Why MERN Stack?

**M - MongoDB**
- **Why**: Flexible schema for products (can add fields without migrations)
- **How**: Stores products and orders with Mongoose ODM
- **Alternative**: PostgreSQL (more rigid, better for complex relationships)

**E - Express.js**
- **Why**: Lightweight, flexible, industry-standard Node.js framework
- **How**: Handles API routes, middleware, and request/response cycle
- **Alternative**: Fastify (faster), NestJS (more opinionated)

**R - React (via Next.js)**
- **Why**: Component-based UI, huge ecosystem, excellent developer experience
- **How**: Next.js adds server-side rendering, routing, and optimization
- **Alternative**: Vue.js, Angular, Svelte

**N - Node.js**
- **Why**: JavaScript everywhere (same language frontend/backend)
- **How**: Runtime for Express server, handles async I/O efficiently
- **Alternative**: Python (Django/Flask), Java (Spring Boot)

### Additional Technologies:

**TypeScript**
- **Why**: Type safety catches errors before runtime
- **How**: Used in both frontend and backend for better code quality
- **Benefit**: Better IDE support, easier refactoring, self-documenting code

**Tailwind CSS**
- **Why**: Utility-first CSS, rapid development, consistent design
- **How**: Classes directly in JSX for styling
- **Alternative**: CSS Modules, Styled Components, SASS

**Stripe**
- **Why**: Industry-leading payment processor, handles security/compliance
- **How**: Creates checkout sessions, processes payments, sends webhooks
- **Alternative**: PayPal, Square, Razorpay

**Vercel**
- **Why**: Serverless deployment, automatic scaling, edge network
- **How**: Hosts both frontend and backend as serverless functions
- **Alternative**: AWS Lambda, Netlify, Railway

---

## 🏗 Backend Architecture

### Folder Structure Explained:

```
geoprice-backend/
├── api/                    # Vercel serverless entry point
│   └── index.ts           # Main handler for all API requests
├── src/
│   ├── config/            # Configuration files
│   ├── controllers/       # Request handlers (business logic entry)
│   ├── middleware/        # Express middleware (logging, errors)
│   ├── models/            # Mongoose schemas (database structure)
│   ├── repositories/      # Data access layer (database queries)
│   ├── routes/            # API route definitions
│   ├── services/          # Business logic (core functionality)
│   ├── types/             # TypeScript type definitions
│   └── utils/             # Helper functions
```

### Architecture Pattern: **Layered Architecture**


**Flow**: Request → Route → Controller → Service → Repository → Database

**Why this pattern?**
- **Separation of Concerns**: Each layer has one responsibility
- **Testability**: Can mock each layer independently
- **Maintainability**: Changes in one layer don't affect others
- **Scalability**: Easy to add new features without breaking existing code

### Layer Breakdown:

#### 1. **Routes** (`src/routes/`)
**Purpose**: Define API endpoints and map them to controllers

**Example** (`price.routes.ts`):
```typescript
priceRouter.get('/rates', (req, res, next) => {
  priceController.getRates(req, res, next);
});
```

**Why**: Separates URL structure from business logic

#### 2. **Controllers** (`src/controllers/`)
**Purpose**: Handle HTTP requests, validate input, call services, format responses

**Example** (`Price.controller.ts`):
```typescript
async getLocalizedPrices(req, res, next) {
  const { country } = req.body;
  if (!country) throw new ValidationError('Country required');
  
  const currency = this.currencyService.getCurrencyForCountry(country);
  const products = await this.productService.getAllProducts();
  // ... convert prices and respond
}
```

**Why**: Keeps HTTP concerns separate from business logic

#### 3. **Services** (`src/services/`)
**Purpose**: Core business logic, orchestrate operations, handle complex workflows

**Example** (`Currency.service.ts`):
- Fetches exchange rates from external API
- Implements 15-minute caching to reduce API calls
- Converts prices between currencies
- Maps countries to currencies

**Why**: Reusable business logic independent of HTTP layer

#### 4. **Repositories** (`src/repositories/`)
**Purpose**: Database access layer, abstract Mongoose operations

**Example** (`Product.repository.ts`):
```typescript
async findAll(): Promise<IProduct[]> {
  return await ProductModel.find();
}
```

**Why**: If you switch from MongoDB to PostgreSQL, only change repositories


#### 5. **Models** (`src/models/`)
**Purpose**: Define database schema, validation rules, relationships

**Example** (`Product.model.ts`):
```typescript
const ProductSchema = new Schema({
  name: { type: String, required: true, trim: true },
  basePrice: { type: Number, required: true, min: 0 },
  sku: { type: String, required: true, unique: true },
  images: { type: [String], required: true }
}, { timestamps: true });
```

**Why**: 
- **Validation**: Ensures data integrity at database level
- **Timestamps**: Automatically adds createdAt/updatedAt
- **Indexing**: `unique: true` on SKU creates database index

#### 6. **Middleware** (`src/middleware/`)
**Purpose**: Functions that run before/after route handlers

**Types**:
- `requestLogger.ts`: Logs all incoming requests
- `errorHandler.ts`: Catches and formats all errors
- CORS: Validates request origins

**Why**: Cross-cutting concerns applied to all routes

---

## 🎨 Frontend Architecture

### Folder Structure Explained:

```
geoprice-frontend/
├── app/                    # Next.js 14 App Router
│   ├── page.tsx           # Server Component (SSR)
│   ├── HomePageClient.tsx # Client Component (interactive)
│   ├── layout.tsx         # Root layout wrapper
│   └── globals.css        # Global styles
├── components/            # Reusable UI components
│   ├── ui/               # Shadcn UI components
│   ├── ProductCard.tsx   # Product display card
│   ├── Header.tsx        # Site header with country selector
│   └── CountrySelector.tsx # Dropdown for country selection
├── hooks/                 # Custom React hooks
│   ├── useCountry.ts     # Manages country state
│   └── useLocalizedPrices.ts # Fetches prices
├── lib/                   # Utility libraries
│   ├── api-client.ts     # Backend API communication
│   ├── currency-formatter.ts # Format prices
│   └── errors.ts         # Custom error classes
├── types/                 # TypeScript interfaces
└── constants/             # Configuration constants
```

### Next.js 14 App Router Explained:

**Server Components vs Client Components**


**Server Component** (`app/page.tsx`):
```typescript
export default async function Home() {
  const headersList = await headers();
  const detectedCountry = headersList.get('x-vercel-ip-country');
  const products = await apiClient.getProducts(); // Fetch on server
  
  return <HomePageClient initialCountry={detectedCountry} initialProducts={products} />;
}
```

**Why Server Component?**
- Runs on server, not sent to browser (smaller bundle)
- Can access headers, databases, file system
- Better SEO (HTML includes content)
- Faster initial load (data fetched before page sent)

**Client Component** (`app/HomePageClient.tsx`):
```typescript
"use client" // This directive makes it a client component

export function HomePageClient({ initialCountry, initialProducts }) {
  const [country, setCountry] = useState(initialCountry);
  // ... interactive logic
}
```

**Why Client Component?**
- Needs interactivity (useState, useEffect, event handlers)
- Runs in browser
- Can update without full page reload

### Custom Hooks Explained:

#### `useCountry` Hook
**Purpose**: Manage country selection and derive currency

```typescript
const { country, currency, setCountry } = useCountry('US');
```

**How it works**:
1. Accepts initial country from server
2. Stores country in state
3. Automatically maps country → currency
4. Provides setter to change country

**Why a hook?**
- Reusable logic across components
- Encapsulates country/currency relationship
- Clean API for components

#### `useLocalizedPrices` Hook
**Purpose**: Fetch and cache localized prices

```typescript
const { prices, isLoading, error, refetch } = useLocalizedPrices(products, country);
```

**How it works**:
1. Calls API when country changes
2. Stores prices in Map (productId → price)
3. Manages loading/error states
4. Provides refetch function

**Why a Map?**
- O(1) lookup time (fast)
- Better than array.find() which is O(n)


### API Client Pattern:

**Purpose**: Centralize all backend communication

```typescript
class ApiClient {
  private pendingRequests: Map<string, Promise<any>>;
  
  async getProducts(): Promise<Product[]> {
    return this.request('/api/products', { method: 'GET' });
  }
}
```

**Features**:
1. **Request Deduplication**: Prevents duplicate concurrent requests
2. **Error Handling**: Consistent error formatting
3. **Type Safety**: TypeScript ensures correct request/response types
4. **Centralized Config**: Base URL in one place

**Why this pattern?**
- DRY (Don't Repeat Yourself)
- Easy to add authentication later
- Easy to mock for testing
- Consistent error handling

---

## 🔄 Data Flow

### Complete User Journey:

#### 1. **User Visits Site**
```
Browser → Vercel Edge → Next.js Server Component
```

**What happens**:
- Vercel reads user's IP address
- Sets `x-vercel-ip-country` header (e.g., "IN" for India)
- Next.js server component reads header
- Validates country is supported, falls back to "US" if not

#### 2. **Initial Page Load (SSR)**
```
Next.js Server → Backend API → MongoDB → Response → HTML sent to browser
```

**What happens**:
- Server fetches products from backend API
- Backend queries MongoDB for all products
- Products sent to client component as props
- Browser receives HTML with products already rendered

**Why SSR?**
- Faster perceived load time
- Better SEO (Google sees content)
- Works without JavaScript

#### 3. **Client Hydration**
```
Browser downloads JS → React "hydrates" → Interactive
```

**What happens**:
- React attaches event listeners to server-rendered HTML
- Client component becomes interactive
- Immediately fetches localized prices for detected country


#### 4. **Fetching Localized Prices**
```
useLocalizedPrices hook → API Client → Backend → Currency Service → Exchange Rate API
```

**Detailed flow**:
1. Hook calls `apiClient.getLocalizedPrices('IN')`
2. API client sends POST to `/api/price` with `{ country: 'IN' }`
3. Backend controller validates country
4. Currency service maps 'IN' → 'INR'
5. Currency service checks cache (15-minute TTL)
6. If cache miss, fetches rates from ExchangeRate-API
7. Converts each product's USD price to INR
8. Returns array of products with localized prices
9. Hook stores in Map for fast lookup
10. Component re-renders with new prices

**Why this flow?**
- **Caching**: Reduces API calls (saves money, faster response)
- **Separation**: Currency logic separate from product logic
- **Flexibility**: Easy to add new currencies

#### 5. **User Changes Country**
```
User clicks dropdown → setCountry('GB') → useLocalizedPrices refetches → UI updates
```

**What happens**:
1. CountrySelector calls `onCountryChange('GB')`
2. `useCountry` hook updates state
3. `useLocalizedPrices` detects country change (useEffect dependency)
4. Automatically refetches prices for new country
5. UI shows loading skeleton
6. Prices update when fetch completes

**Why automatic refetch?**
- Better UX (no manual refresh button)
- React's useEffect handles dependencies
- Declarative programming (describe what, not how)

#### 6. **User Clicks "Buy Now"**
```
Button click → createCheckoutSession → Stripe API → Redirect to Stripe Checkout
```

**Detailed flow**:
1. User clicks "Buy Now" on ProductCard
2. `handleBuyNow(productId)` called
3. API client sends POST to `/api/create-checkout-session`
   - Body: `{ productId, currency: 'GBP', country: 'GB' }`
4. Backend validates product exists
5. Converts price to GBP
6. Creates Stripe checkout session with:
   - Product details
   - Price in GBP
   - Success/cancel URLs
7. Returns `{ sessionId, sessionUrl }`
8. Frontend redirects: `window.location.href = sessionUrl`
9. User completes payment on Stripe's secure page


#### 7. **Payment Completion (Webhook)**
```
Stripe → Webhook → Backend → Create Order → MongoDB
```

**Detailed flow**:
1. User completes payment on Stripe
2. Stripe sends webhook to `/api/webhook`
3. Backend verifies webhook signature (security)
4. Extracts session data (amount, currency, product)
5. Creates order in MongoDB with status 'paid'
6. Returns 200 OK to Stripe

**Why webhooks?**
- **Reliability**: Stripe retries if your server is down
- **Security**: User can't fake payment completion
- **Async**: Payment processing doesn't block user
- **Audit Trail**: All payments logged in database

---

## 📁 File-by-File Explanation

### Backend Files:

#### `api/index.ts` - Serverless Entry Point
**Purpose**: Vercel serverless function handler

```typescript
async function handler(req: VercelRequest, res: VercelResponse) {
  if (mongoose.connection.readyState !== 1) {
    await connectDatabase();
  }
  return app(req, res);
}
```

**Why this file?**
- Vercel looks for `api/index.ts` as entry point
- Checks database connection before handling request
- Reuses connection across requests (serverless optimization)

**Key Concept**: Serverless functions are stateless, so we check connection state

---

#### `src/app.ts` - Express Application Setup
**Purpose**: Configure Express app with middleware and routes

**Key sections**:

1. **CORS Configuration**:
```typescript
const corsOptions = {
  origin: (origin, callback) => {
    if (!origin || origin.includes('.vercel.app')) {
      callback(null, true);
    }
  },
  credentials: true
};
```

**Why**: 
- Allows frontend to call backend from different domain
- Blocks unauthorized domains
- Allows Vercel preview deployments

2. **Middleware Order**:
```typescript
app.use(cors());
app.use(requestLogger);
app.use('/api', webhookRouter);  // BEFORE express.json()
app.use(express.json());
app.use('/api', productRouter);
app.use(errorHandler);
```

**Why this order?**
- CORS first (reject unauthorized requests early)
- Webhook before json parser (needs raw body)
- Error handler last (catches all errors)


---

#### `src/config/database.ts` - MongoDB Connection
**Purpose**: Connect to MongoDB Atlas

```typescript
export const connectDatabase = async (): Promise<void> => {
  await mongoose.connect(config.MONGODB_URI);
  console.log('MongoDB connected successfully');
};
```

**Why Mongoose?**
- Schema validation
- Middleware (pre/post hooks)
- Query building
- Type safety with TypeScript

---

#### `src/models/Product.model.ts` - Product Schema
**Purpose**: Define product structure and validation

**Key features**:
```typescript
const ProductSchema = new Schema({
  name: { type: String, required: true, trim: true },
  basePrice: { type: Number, required: true, min: 0 },
  sku: { type: String, required: true, unique: true },
  images: { 
    type: [String], 
    validate: {
      validator: (v) => v && v.length > 0,
      message: 'Product must have at least one image'
    }
  }
}, { timestamps: true });
```

**Validation explained**:
- `required: true`: Field must exist
- `trim: true`: Removes whitespace
- `min: 0`: Price can't be negative
- `unique: true`: Creates index, prevents duplicates
- `timestamps: true`: Auto-adds createdAt/updatedAt
- Custom validator: Ensures at least one image

---

#### `src/models/Order.model.ts` - Order Schema
**Purpose**: Store completed orders

**Key features**:
```typescript
productId: { type: Schema.Types.ObjectId, ref: 'Product' }
```

**Why ObjectId reference?**
- Links to Product collection
- Can populate: `Order.find().populate('productId')`
- Maintains referential integrity

```typescript
stripeSessionId: { type: String, unique: true, index: true }
```

**Why indexed?**
- Webhook looks up order by session ID
- Index makes lookup O(log n) instead of O(n)
- Unique prevents duplicate orders


---

#### `src/services/Currency.service.ts` - Currency Conversion
**Purpose**: Handle all currency-related operations

**Key features**:

1. **Caching**:
```typescript
private cache: CacheEntry | null = null;
private readonly CACHE_DURATION = 15 * 60 * 1000; // 15 minutes

private isCacheValid(): boolean {
  if (!this.cache) return false;
  const cacheAge = Date.now() - this.cache.timestamp;
  return cacheAge < this.CACHE_DURATION;
}
```

**Why cache?**
- Exchange rates don't change every second
- Reduces API calls (ExchangeRate-API has limits)
- Faster response time
- Saves money (API costs)

2. **Price Conversion**:
```typescript
async convertPrice(amount, fromCurrency, toCurrency) {
  if (fromCurrency === toCurrency) return amount;
  
  const rates = await this.getExchangeRates(fromCurrency, [toCurrency]);
  const convertedAmount = amount * rates[toCurrency];
  return Math.round(convertedAmount * 100) / 100; // Round to 2 decimals
}
```

**Why round?**
- Avoid floating point errors (0.1 + 0.2 = 0.30000000000000004)
- Display-friendly prices

3. **Fallback Strategy**:
```typescript
catch (error) {
  if (this.cache) {
    logger.warn('Using expired cache as fallback');
    return this.cache.rates;
  }
  throw new ExternalServiceError('Exchange Rate Service');
}
```

**Why fallback?**
- Better UX (show slightly stale data vs error)
- Resilience (works even if API is down)

---

#### `src/services/Payment.service.ts` - Stripe Integration
**Purpose**: Handle Stripe checkout and webhooks

**Key methods**:

1. **Create Checkout Session**:
```typescript
async createCheckoutSession(product, currency, country) {
  const session = await this.stripeClient.checkout.sessions.create({
    payment_method_types: ['card'],
    line_items: [{
      price_data: {
        currency: currency.toLowerCase(),
        product_data: { name: product.name },
        unit_amount: Math.round(product.basePrice * 100) // Stripe uses cents
      },
      quantity: 1
    }],
    mode: 'payment',
    success_url: `${frontendUrl}/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${frontendUrl}/cancel`
  });
}
```

**Why multiply by 100?**
- Stripe expects amounts in smallest currency unit (cents for USD)
- $10.00 = 1000 cents


2. **Verify Webhook**:
```typescript
verifyWebhookSignature(payload, signature) {
  return this.stripeClient.webhooks.constructEvent(
    payload,
    signature,
    config.STRIPE_WEBHOOK_SECRET
  );
}
```

**Why verify?**
- Prevents fake payment notifications
- Ensures webhook came from Stripe
- Uses HMAC signature with shared secret

---

#### `src/controllers/Price.controller.ts` - Price Endpoints
**Purpose**: Handle price-related API requests

**Key method**:
```typescript
async getLocalizedPrices(req, res, next) {
  const { country } = req.body;
  
  const currency = this.currencyService.getCurrencyForCountry(country);
  const products = await this.productService.getAllProducts();
  
  const localizedProducts = await Promise.all(
    products.map(async (product) => {
      const localizedPrice = await this.currencyService.convertPrice(
        product.basePrice, 'USD', currency
      );
      return { ...product.toObject(), localizedPrice, currency };
    })
  );
  
  res.json(successResponse({ country, currency, products: localizedProducts }));
}
```

**Why Promise.all?**
- Converts all prices in parallel (faster)
- Alternative: `for` loop would be sequential (slower)

**Why .toObject()?**
- Mongoose documents have methods/getters
- .toObject() converts to plain JavaScript object
- Needed for JSON serialization

---

#### `src/middleware/errorHandler.ts` - Error Handling
**Purpose**: Catch and format all errors

```typescript
export const errorHandler = (err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';
  
  logger.error(message, err);
  
  res.status(statusCode).json({
    success: false,
    error: message,
    ...(config.NODE_ENV === 'development' && { stack: err.stack })
  });
};
```

**Why centralized?**
- DRY (don't repeat try-catch everywhere)
- Consistent error format
- Logging in one place
- Hide stack traces in production


---

### Frontend Files:

#### `app/page.tsx` - Server Component
**Purpose**: Initial page load with SSR

```typescript
export default async function Home() {
  const headersList = await headers();
  const detectedCountry = headersList.get('x-vercel-ip-country') || 'US';
  
  const country = isSupportedCountry(detectedCountry) 
    ? detectedCountry 
    : 'US';
  
  let products = [];
  try {
    products = await apiClient.getProducts();
  } catch (err) {
    console.error('Error fetching products:', err);
  }
  
  return <HomePageClient initialCountry={country} initialProducts={products} />;
}
```

**Key concepts**:
1. **async function**: Can await API calls on server
2. **headers()**: Next.js function to read request headers
3. **Validation**: Fallback to 'US' if country unsupported
4. **Error handling**: Graceful degradation (empty array vs crash)
5. **Props passing**: Server data → Client component

**Why this approach?**
- SEO: Products in HTML source
- Performance: Data fetched before page sent
- UX: Instant content, no loading spinner

---

#### `app/HomePageClient.tsx` - Client Component
**Purpose**: Interactive UI with state management

**Key sections**:

1. **State Management**:
```typescript
const { country, currency, setCountry } = useCountry(initialCountry);
const { prices, isLoading, error } = useLocalizedPrices(initialProducts, country);
const [checkoutError, setCheckoutError] = useState(null);
```

**Why multiple state sources?**
- `useCountry`: Manages country selection
- `useLocalizedPrices`: Fetches prices (separate concern)
- `checkoutError`: Local to this component

2. **Checkout Handler**:
```typescript
const handleBuyNow = async (productId) => {
  setCheckoutError(null);
  try {
    const session = await apiClient.createCheckoutSession(productId, currency, country);
    window.location.href = session.sessionUrl;
  } catch (error) {
    setCheckoutError(error.message);
  }
};
```

**Why window.location.href?**
- Full page redirect to Stripe (external site)
- Not a React Router navigation
- Stripe handles payment UI


3. **Responsive Grid**:
```typescript
<div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
  {initialProducts.map((product) => (
    <ProductCard key={product._id} product={product} />
  ))}
</div>
```

**Tailwind classes explained**:
- `grid`: CSS Grid layout
- `grid-cols-1`: 1 column on mobile
- `sm:grid-cols-2`: 2 columns on small screens (640px+)
- `lg:grid-cols-3`: 3 columns on large screens (1024px+)
- `gap-6`: 1.5rem spacing between items

---

#### `components/ProductCard.tsx` - Product Display
**Purpose**: Display single product with price and buy button

**Key features**:

1. **Image Optimization**:
```typescript
<Image
  src={imageUrl}
  alt={product.name}
  fill
  sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 33vw"
  priority={priority}
/>
```

**Next.js Image explained**:
- `fill`: Fills parent container
- `sizes`: Tells browser which size to load (responsive)
- `priority`: Loads immediately (for above-fold images)
- Automatic optimization (WebP, lazy loading)

2. **Loading States**:
```typescript
{isLoadingPrice ? (
  <Skeleton className="h-8 w-24" />
) : localizedPrice !== null ? (
  <p>{CurrencyFormatter.format(localizedPrice, currency)}</p>
) : (
  <p>Price unavailable</p>
)}
```

**Why three states?**
- Loading: Show skeleton (better UX than blank)
- Success: Show price
- Error: Show fallback message

3. **Button States**:
```typescript
<Button
  disabled={isProcessing || isLoadingPrice || localizedPrice === null}
  onClick={handleBuyNow}
>
  {isProcessing ? 'Processing...' : 'Buy Now'}
</Button>
```

**Why disable?**
- Prevents double-clicks
- Can't buy without price
- Visual feedback during processing

---

#### `hooks/useCountry.ts` - Country Management
**Purpose**: Manage country state and derive currency

```typescript
export function useCountry(initialCountry) {
  const [country, setCountryState] = useState(initialCountry);
  const currency = getCountryCurrency(country);
  
  const setCountry = useCallback((newCountry) => {
    setCountryState(newCountry);
  }, []);
  
  return { country, currency, setCountry };
}
```

**Why useCallback?**
- Memoizes function (same reference across renders)
- Prevents unnecessary re-renders in child components
- Performance optimization


---

#### `hooks/useLocalizedPrices.ts` - Price Fetching
**Purpose**: Fetch and cache localized prices

**Key features**:

1. **Automatic Refetch**:
```typescript
useEffect(() => {
  fetchPrices();
}, [fetchPrices]); // Runs when fetchPrices changes
```

**When does fetchPrices change?**
- When `country` changes (useCallback dependency)
- When `products` changes

2. **Map Storage**:
```typescript
const pricesMap = new Map();
localizedPrices.forEach((priceData) => {
  pricesMap.set(priceData.productId, priceData.localizedPrice);
});
setPrices(pricesMap);
```

**Why Map vs Object?**
- Faster lookups: `prices.get(id)` vs `prices[id]`
- Better for dynamic keys
- Cleaner API

3. **Error Handling**:
```typescript
try {
  const localizedPrices = await apiClient.getLocalizedPrices(country);
  setPrices(pricesMap);
} catch (err) {
  setError(err.message);
} finally {
  setIsLoading(false); // Always runs
}
```

**Why finally?**
- Ensures loading state cleared even on error
- Prevents stuck loading spinners

---

#### `lib/api-client.ts` - API Communication
**Purpose**: Centralize all backend requests

**Key features**:

1. **Request Deduplication**:
```typescript
private pendingRequests: Map<string, Promise<any>>;

async request(endpoint, options) {
  const requestKey = `${options.method}:${endpoint}:${options.body}`;
  
  if (this.pendingRequests.has(requestKey)) {
    return this.pendingRequests.get(requestKey); // Return existing promise
  }
  
  const requestPromise = fetch(url, options);
  this.pendingRequests.set(requestKey, requestPromise);
  
  return requestPromise;
}
```

**Why deduplicate?**
- Prevents race conditions
- Saves bandwidth
- Reduces server load
- Example: User clicks country selector twice quickly

2. **Error Handling**:
```typescript
if (!response.ok || !data.success) {
  throw new ApiError(data.error, response.status, endpoint);
}
```

**Why custom errors?**
- Type-safe error handling
- Can catch specific error types
- Better error messages


---

#### `lib/currency-formatter.ts` - Price Formatting
**Purpose**: Format prices with correct currency symbols

```typescript
export class CurrencyFormatter {
  static format(amount: number, currency: string): string {
    return new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: currency,
      minimumFractionDigits: 2,
      maximumFractionDigits: 2,
    }).format(amount);
  }
}
```

**Intl.NumberFormat explained**:
- Browser API for internationalization
- Automatically adds currency symbol ($, £, ₹)
- Handles decimal separators (. vs ,)
- Respects currency rules (JPY has no decimals)

**Examples**:
- `format(99.99, 'USD')` → "$99.99"
- `format(99.99, 'GBP')` → "£99.99"
- `format(99.99, 'INR')` → "₹99.99"

---

#### `constants/countries.ts` - Configuration
**Purpose**: Define supported countries and currencies

```typescript
export const COUNTRIES = [
  { code: 'US', name: 'United States', currency: 'USD', flag: '🇺🇸' },
  { code: 'IN', name: 'India', currency: 'INR', flag: '🇮🇳' },
  { code: 'GB', name: 'United Kingdom', currency: 'GBP', flag: '🇬🇧' },
];

export function getCountryCurrency(countryCode: string): CurrencyCode {
  const country = COUNTRIES.find((c) => c.code === countryCode);
  return country?.currency || 'USD';
}
```

**Why centralize?**
- Single source of truth
- Easy to add new countries
- Type-safe with TypeScript
- Reusable across components

---

## 🔐 Security Considerations

### 1. **Environment Variables**
```
MONGODB_URI=mongodb+srv://...
STRIPE_SECRET_KEY=sk_live_...
```

**Why .env files?**
- Keeps secrets out of code
- Different values per environment
- Never commit to Git (.gitignore)

### 2. **CORS Protection**
```typescript
origin: (origin, callback) => {
  if (origin === frontendUrl || origin.includes('.vercel.app')) {
    callback(null, true);
  } else {
    callback(null, false);
  }
}
```

**What it prevents**:
- Unauthorized websites calling your API
- Cross-site request forgery (CSRF)

### 3. **Stripe Webhook Verification**
```typescript
const event = stripe.webhooks.constructEvent(
  payload,
  signature,
  webhookSecret
);
```

**What it prevents**:
- Fake payment notifications
- Attackers creating free orders


### 4. **Input Validation**
```typescript
if (!productId) {
  throw new ValidationError('Product ID is required');
}

if (!SUPPORTED_CURRENCIES.includes(currency)) {
  throw new ValidationError('Invalid currency');
}
```

**What it prevents**:
- SQL injection (not applicable with MongoDB, but good practice)
- Invalid data in database
- Server crashes from unexpected input

### 5. **Mongoose Schema Validation**
```typescript
basePrice: { type: Number, required: true, min: 0 }
```

**What it prevents**:
- Negative prices
- Missing required fields
- Wrong data types

---

## 🚀 Performance Optimizations

### 1. **Server-Side Rendering (SSR)**
- Products fetched on server
- HTML sent with content
- Faster perceived load time
- Better SEO

### 2. **Request Deduplication**
- Prevents duplicate API calls
- Saves bandwidth
- Reduces server load

### 3. **Currency Rate Caching**
- 15-minute cache
- Reduces external API calls
- Faster response times
- Saves money

### 4. **Database Indexing**
```typescript
stripeSessionId: { unique: true, index: true }
```
- O(log n) lookups instead of O(n)
- Faster webhook processing

### 5. **Image Optimization**
- Next.js Image component
- Automatic WebP conversion
- Lazy loading
- Responsive sizes

### 6. **Code Splitting**
- Next.js automatically splits code
- Only loads needed JavaScript
- Smaller initial bundle

### 7. **Map for Price Lookups**
```typescript
const prices = new Map();
prices.get(productId); // O(1) lookup
```
- Faster than array.find() which is O(n)

---

## 🧪 Testing Strategy

### Unit Tests
- Test individual functions
- Mock dependencies
- Example: Currency conversion logic

### Integration Tests
- Test API endpoints
- Use test database
- Example: Create checkout session

### E2E Tests
- Test complete user flows
- Use real browser
- Example: Full purchase flow

---

## 📦 Deployment

### Vercel Deployment Process:

1. **Push to Git**
```bash
git push origin main
```

2. **Vercel Auto-Deploys**
- Detects changes
- Builds frontend and backend
- Deploys to edge network

3. **Environment Variables**
- Set in Vercel dashboard
- Different per environment
- Encrypted at rest


### Serverless Architecture:

**Traditional Server**:
```
Server always running → Costs 24/7 → Fixed capacity
```

**Serverless**:
```
Function runs on request → Pay per use → Auto-scales
```

**Benefits**:
- No server management
- Automatic scaling
- Pay only for usage
- Global edge network

**Challenges**:
- Cold starts (first request slower)
- Stateless (can't store in memory)
- Connection pooling needed

---

## 🔄 Common Workflows

### Adding a New Product:

1. **Create product in MongoDB**:
```javascript
db.products.insertOne({
  name: "New Product",
  description: "Description",
  basePrice: 99.99,
  sku: "PROD-004",
  images: ["url"]
});
```

2. **Product automatically appears**:
- Backend fetches all products
- Frontend displays in grid
- Prices converted automatically

### Adding a New Currency:

1. **Update constants**:
```typescript
// constants/countries.ts
{ code: 'CA', name: 'Canada', currency: 'CAD', flag: '🇨🇦' }
```

2. **Update backend mapping**:
```typescript
// utils/currencyMapper.ts
case 'CA': return 'CAD';
```

3. **No other changes needed**:
- Currency service handles conversion
- Frontend displays automatically

### Handling Errors:

**Backend**:
```typescript
throw new ValidationError('Invalid input');
// → errorHandler catches
// → Returns JSON error
// → Logs to console
```

**Frontend**:
```typescript
try {
  await apiClient.getProducts();
} catch (error) {
  setError(error.message);
  // → Display error alert
  // → User sees friendly message
}
```

---

## 🎓 Key Concepts Summary

### 1. **Separation of Concerns**
- Each file/function has one job
- Easy to find and fix bugs
- Easy to add features

### 2. **DRY (Don't Repeat Yourself)**
- Reusable components
- Shared utilities
- Custom hooks

### 3. **Type Safety**
- TypeScript catches errors early
- Better IDE support
- Self-documenting code

### 4. **Error Handling**
- Try-catch everywhere
- Graceful degradation
- User-friendly messages

### 5. **Performance**
- Caching
- Indexing
- Code splitting
- Image optimization

### 6. **Security**
- Environment variables
- Input validation
- CORS protection
- Webhook verification


---

## 🤔 Common Interview Questions & Answers

### Q: Why use TypeScript instead of JavaScript?
**A**: Type safety catches errors at compile-time, better IDE support, self-documenting code, easier refactoring, and improved team collaboration.

### Q: Why separate frontend and backend?
**A**: Separation of concerns, can scale independently, different deployment strategies, can swap frontend/backend technologies, better security (API keys not exposed).

### Q: Why use Mongoose instead of native MongoDB driver?
**A**: Schema validation, middleware hooks, query building, type safety, cleaner API, and better developer experience.

### Q: Why cache exchange rates?
**A**: Rates don't change frequently, reduces API calls (saves money), faster response times, and provides fallback if API is down.

### Q: Why use Next.js instead of Create React App?
**A**: Server-side rendering (better SEO), automatic code splitting, built-in routing, image optimization, API routes, and better performance.

### Q: Why use serverless instead of traditional server?
**A**: No server management, automatic scaling, pay per use (cheaper for low traffic), global edge network, and zero downtime deployments.

### Q: How does the webhook ensure security?
**A**: Stripe signs webhooks with HMAC signature using shared secret. Backend verifies signature before processing. Prevents fake payment notifications.

### Q: What happens if the currency API is down?
**A**: Service uses cached rates (up to 15 minutes old) as fallback. If no cache, returns error with user-friendly message.

### Q: Why use Map instead of Object for prices?
**A**: Faster lookups (O(1)), better for dynamic keys, cleaner API, and can use any type as key (not just strings).

### Q: How does request deduplication work?
**A**: Stores pending promises in Map with unique key. If same request made while pending, returns existing promise instead of creating new fetch.

---

## 📚 Learning Resources

### To understand this project better, learn:

1. **JavaScript/TypeScript**
   - Async/await
   - Promises
   - ES6+ features
   - Type systems

2. **React**
   - Hooks (useState, useEffect, useCallback)
   - Component lifecycle
   - Props vs State
   - Context API

3. **Next.js**
   - App Router
   - Server vs Client Components
   - Data fetching
   - Routing

4. **Node.js/Express**
   - Middleware
   - Routing
   - Error handling
   - Async operations

5. **MongoDB/Mongoose**
   - Schema design
   - Queries
   - Indexing
   - Relationships

6. **REST APIs**
   - HTTP methods
   - Status codes
   - Request/response cycle
   - Authentication

7. **Git/GitHub**
   - Version control
   - Branching
   - Pull requests
   - Collaboration

---

## 🎯 Next Steps to Improve

### Features to Add:
1. User authentication (login/signup)
2. Shopping cart (multiple items)
3. Order history
4. Product reviews
5. Search and filters
6. Admin dashboard
7. Email notifications
8. Inventory management

### Technical Improvements:
1. Add unit tests
2. Add E2E tests
3. Implement Redis caching
4. Add rate limiting
5. Add monitoring (Sentry)
6. Add analytics
7. Optimize database queries
8. Add CDN for images

---

## 🙏 Conclusion

This project demonstrates a production-ready full-stack application with:
- Modern architecture patterns
- Best practices
- Security considerations
- Performance optimizations
- Scalable design

Understanding each part and why it's used will help you:
- Build similar projects
- Answer interview questions
- Make architectural decisions
- Debug issues effectively
- Extend functionality

Good luck with your interview! 🚀
