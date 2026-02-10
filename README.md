# Smart Cafeteria System — Developer Documentation

> **Demand Forecasting & Token-Based Queue Optimization System**
> 
> A full-stack campus cafeteria management platform that integrates meal slot pre-booking, demand forecasting, real-time digital token management, Stripe payments, and walk-in food redistribution — all designed to reduce food waste and queue congestion.

---

## 📋 Table of Contents

- [About](#about)
- [Tech Stack](#tech-stack)
- [Repository Structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
  - [1. Clone the Repository](#1-clone-the-repository)
  - [2. Install Dependencies (Monorepo)](#2-install-dependencies-monorepo)
  - [3. Backend Setup](#3-backend-setup)
  - [4. Frontend Setup](#4-frontend-setup)
- [Environment Variables](#environment-variables)
  - [Backend `.env`](#backend-env)
  - [Frontend `.env`](#frontend-env)
- [Available Scripts](#available-scripts)
  - [Root (Monorepo)](#root-monorepo)
  - [Backend](#backend)
  - [Frontend](#frontend)
- [Architecture Overview](#architecture-overview)
  - [Backend Architecture](#backend-architecture)
  - [Frontend Architecture](#frontend-architecture)
- [API Reference](#api-reference)
  - [Auth Routes](#auth-routes-apiauthx)
  - [Booking Routes](#booking-routes-apibookingsx)
  - [Token Routes](#token-routes-apitokensx)
  - [Payment Routes](#payment-routes-apipaymentsx)
- [Stripe Integration](#stripe-integration)
- [Code Quality & Linting](#code-quality--linting)
- [Git Hooks](#git-hooks)
- [Languages](#languages)
- [License](#license)

---

## About

The **Smart Cafeteria System** is a Level 5 (L5) university project that optimizes campus dining through:

- **Meal Slot Pre-Booking** — students reserve time-slotted meals (breakfast, lunch, snack, dinner) in advance.
- **Demand Forecasting** — analyses booking data and demand patterns per slot.
- **Digital Token Queue** — generates unique tokens, assigns service counters, and streams real-time queue positions via Server-Sent Events (SSE).
- **Stripe Payments** — supports Payment Intents, Checkout Sessions for wallet recharge, webhook verification, and bill-splitting across group bookings.
- **No-Show & Leftover Handling** — detects no-shows, reassigns food to walk-in users, and transfers leftovers to subsequent slots.

User roles: `user` (student), `staff`, `admin`.

---

## Tech Stack

### Backend

| Category | Technology |
|----------|-----------|
| **Runtime** | Node.js (v18+) |
| **Language** | TypeScript 5 |
| **Framework** | Express.js 5 |
| **Database & Auth** | Supabase (PostgreSQL + Auth + JWT) |
| **Payments** | Stripe (Payment Intents, Checkout Sessions, Webhooks) |
| **Validation** | Zod 4 |
| **Package Manager** | pnpm 10 |

### Frontend

| Category | Technology |
|----------|-----------|
| **Framework** | Next.js 16.1.1 (App Router) |
| **Language** | TypeScript 5 |
| **UI Library** | React 19.2.3 |
| **Styling** | Tailwind CSS v4, tailwindcss-animate |
| **UI Components** | shadcn/ui (Radix UI primitives) |
| **State Management** | Zustand 5 (with persist middleware) |
| **Server State** | TanStack React Query 5 |
| **Forms** | React Hook Form 7 + Zod |
| **HTTP Client** | Axios (with axios-retry) |
| **Animations** | GSAP 3 |
| **Icons** | Lucide React |
| **Notifications** | React Hot Toast / Sonner |
| **Date Utilities** | date-fns 4 |

### Shared Tooling

| Tool | Purpose |
|------|---------|
| **pnpm Workspaces** | Monorepo management |
| **Biome** | Linting & formatting (replaces ESLint + Prettier for most checks) |
| **Husky** | Git hooks (pre-commit) |
| **lint-staged** | Run Biome on staged files before commit |

---

## Repository Structure

```
Smart-Cafeteria-System/
├── .github/
│   └── workflows/            # GitHub Actions CI/CD workflows
├── .husky/
│   └── pre-commit            # Runs `pnpm pre-commit` on staged files
├── backend/
│   ├── src/
│   │   ├── config/           # Supabase client configuration
│   │   ├── controllers/      # Express request handlers
│   │   ├── interfaces/       # Shared TypeScript types & interfaces
│   │   ├── middlewares/       # Auth middleware (JWT verification)
│   │   ├── routes/           # API route definitions
│   │   │   ├── authRoutes.ts
│   │   │   ├── bookingRoutes.ts
│   │   │   ├── tokenRoutes.ts
│   │   │   └── paymentRoutes.ts
│   │   ├── services/         # Business logic & Supabase data operations
│   │   ├── types/            # Express type augmentations
│   │   ├── utils/            # Helper functions (IST date utilities)
│   │   ├── validations/      # Zod request/schema validations
│   │   └── index.ts          # App entry point & Express setup
│   ├── .env                  # Environment variables (git-ignored)
│   ├── nodemon.json          # Nodemon configuration
│   ├── package.json
│   ├── tsconfig.json
│   └── README.md
├── frontend/
│   ├── examples/             # Reference implementations & patterns
│   │   ├── components/       # Example presentational components
│   │   ├── features/         # Example feature components with forms
│   │   ├── hooks/            # Example custom hooks (React Query)
│   │   ├── services/         # Example API service modules
│   │   ├── stores/           # Example Zustand stores
│   │   └── types/            # Example TypeScript type definitions
│   ├── public/               # Static assets (images, fonts)
│   ├── scripts/              # Init scripts (init.sh / init.ps1)
│   ├── src/
│   │   ├── app/              # Next.js App Router (pages & layouts)
│   │   ├── components/       # Reusable UI components
│   │   │   └── ui/           # shadcn/ui components
│   │   ├── data/             # Static menu data
│   │   ├── features/         # Feature-specific components
│   │   ├── hooks/            # Custom React hooks
│   │   ├── lib/              # Axios instance, API routes, utilities
│   │   ├── services/         # API service modules
│   │   ├── stores/           # Zustand state stores
│   │   ├── styles/           # Global CSS & component styles
│   │   └── types/            # TypeScript type definitions
│   ├── .env.example          # Environment variables template
│   ├── components.json       # shadcn/ui configuration
│   ├── next.config.ts
│   ├── tailwind.config.js
│   ├── tsconfig.json
│   └── package.json
├── design/                   # Design assets / mockups
├── biome.json                # Biome linter & formatter config (shared)
├── package.json              # Root workspace scripts
├── pnpm-lock.yaml
├── pnpm-workspace.yaml       # Workspace packages: ., backend, frontend
└── readme.md
```

---

## Prerequisites

Before you begin, ensure you have the following installed:

- **Node.js** >= 18.x  
- **pnpm** >= 10.x

> ⚠️ This project uses **pnpm** as the package manager. Using npm or yarn may cause dependency resolution issues.

### Installing pnpm

```bash
# Using npm
npm install -g pnpm

# Using Homebrew (macOS)
brew install pnpm

# Using corepack (recommended)
corepack enable
corepack prepare pnpm@latest --activate
```

### External Services

- **Supabase** account — [https://supabase.com/](https://supabase.com/)
- **Stripe** account (test mode) — [https://dashboard.stripe.com/register](https://dashboard.stripe.com/register)

---

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/AMRITH03/Smart-Cafeteria-System.git
cd Smart-Cafeteria-System
```

### 2. Install Dependencies (Monorepo)

From the **root** directory:

```bash
pnpm install
```

This installs dependencies for the root, `backend/`, and `frontend/` workspaces.

### 3. Backend Setup

```bash
cd backend
```

1. **Create `.env`** (see [Environment Variables — Backend](#backend-env) below).
2. **Run the development server:**
   ```bash
   pnpm dev
   ```
   The backend will be available at **http://localhost:3001**.

### 4. Frontend Setup

```bash
cd frontend
```

1. **Run the init script** to set up environment and dependencies:

   ```bash
   # macOS / Linux
   ./scripts/init.sh

   # Windows
   .\scripts\init.ps1
   ```

2. **Create `.env`** from the template (see [Environment Variables — Frontend](#frontend-env) below):
   ```bash
   cp .env.example .env
   ```

3. **Run the development server:**
   ```bash
   pnpm dev
   ```
   The frontend will be available at **http://localhost:3000**.

---

## Environment Variables

### Backend `.env`

Create `backend/.env` with the following:

```env
# Server Configuration
PORT=3001
NODE_ENV=development
FRONTEND_URL=http://localhost:3000

# Supabase Configuration
SUPABASE_URL=your_supabase_url
SUPABASE_KEY_ANON=your_supabase_anon_key
SUPABASE_SERVICE_ROLE=your_supabase_service_role_key

# Stripe Configuration (Test Mode)
STRIPE_SECRET_KEY=sk_test_your_stripe_secret_key
STRIPE_PUBLISHABLE_KEY=pk_test_your_stripe_publishable_key
STRIPE_WEBHOOK_SECRET=whsec_your_webhook_secret
```

### Frontend `.env`

Create `frontend/.env` from `frontend/.env.example`:

```env
# Backend API URL
NEXT_PUBLIC_BACKEND_URL="http://localhost:3001"

# Environment flag ("1" = production, "0" = test/development)
NEXT_PUBLIC_IS_PRODUCTION="0"

# Environment flag for build
NODE_ENV="development"
```

---

## Available Scripts

### Root (Monorepo)

| Command | Description |
|---------|-------------|
| `pnpm install` | Install all workspace dependencies |
| `pnpm build` | Build both frontend and backend |
| `pnpm lint` | Lint both frontend and backend |
| `pnpm pre-commit` | Run pre-commit checks on both workspaces |

### Backend

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start dev server with nodemon (hot-reload) |
| `pnpm build` | Compile TypeScript to `dist/` |
| `pnpm start` | Run compiled server from `dist/index.js` |
| `pnpm lint` | Run Biome linter |
| `pnpm check` | Run Biome check (lint + format) |
| `pnpm type-check` | Run TypeScript compiler check (`tsc --noEmit`) |
| `pnpm fix` | Auto-fix Biome issues |

### Frontend

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start Next.js dev server (Turbopack) |
| `pnpm dev:webpack` | Start Next.js dev server (Webpack, no Turbo) |
| `pnpm build` | Create production build |
| `pnpm start` | Start production server |
| `pnpm lint` | Run Biome linter |
| `pnpm check` | Run Biome check (lint + format) |
| `pnpm type-check` | Run TypeScript compiler check |
| `pnpm fix` | Auto-fix Biome issues |

---

## Architecture Overview

### Backend Architecture

The backend follows a **layered MVC architecture**:

```
Request → Route → Controller → Service → Supabase DB
                      ↑
               Middleware (Auth)
                      ↑
              Validation (Zod)
```

| Layer | Responsibility |
|-------|---------------|
| **Routes** | Define HTTP endpoints and wire up middleware/controllers |
| **Controllers** | Parse/validate requests, call services, send responses |
| **Services** | Business logic, Supabase queries, Stripe API calls |
| **Middlewares** | JWT authentication via Supabase (`requireAuth`) |
| **Validations** | Zod schemas for request body/query/param validation |
| **Interfaces** | Shared TypeScript types (`user.types.ts`, `booking.types.ts`, `token.types.ts`) |
| **Config** | Supabase client initialization |
| **Utils** | IST date/time utilities (IST = UTC+5:30) |

**Key backend design decisions:**
- Stripe webhook route is mounted **before** `express.json()` to receive the raw body for signature verification.
- All dates are processed in **IST (Indian Standard Time)** using custom utility functions.
- Express `Request` type is augmented with `user` and `token` fields.

### Frontend Architecture

The frontend follows a **feature-based architecture** with Next.js App Router:

#### State Management Strategy

| Type | Tool | Use Case |
|------|------|----------|
| **Server State** | TanStack React Query | API data fetching, caching, synchronization |
| **Client State** | Zustand | UI state, authentication, local preferences |
| **Form State** | React Hook Form | Form inputs, validation, submission |

#### API Layer

Centralized Axios instance (`src/lib/api.ts`) with:
- **Automatic retry** with exponential backoff for 429, 408, and timeout errors (3 retries)
- **Request interceptors** for authentication headers
- **Response interceptors** for global error handling & toast notifications
- **Credentials support** (`withCredentials: true`) for cookie-based auth
- **10s timeout** on all requests

#### Component Patterns

| Pattern | Location | Description |
|---------|----------|-------------|
| **Presentational Components** | `components/` | Pure UI, no business logic, prop-driven |
| **Feature Components** | `features/` | Business logic, hooks for data fetching |
| **UI Components** | `components/ui/` | shadcn/ui base components (Radix UI) |

#### Role-Based Routing

- `user` role → Landing page + booking flows
- `staff` role → Automatically redirected to `/staff` dashboard
- `useRole()` hook provides role flags (`isStaff`, `isUser`)

---

## API Reference

Base URL: `http://localhost:3001`

### Auth Routes (`/api/auth/*`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/auth/test` | ❌ | Health check |
| POST | `/api/auth/register` | ❌ | Register new user |
| POST | `/api/auth/signIn` | ❌ | Sign in |
| POST | `/api/auth/signOut` | ✅ | Sign out |
| POST | `/api/auth/forgot-password` | ❌ | Request password reset |
| POST | `/api/auth/reset-password` | ❌ | Reset password |
| GET | `/api/auth/profile` | ✅ | Get current user profile |
| PUT | `/api/auth/profile` | ✅ | Update profile |
| GET | `/api/auth/user/:userId` | ✅ | Get user by ID |

### Booking Routes (`/api/bookings/*`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/bookings/slots` | ❌ | Get available slots (`?date=&meal_type=`) |
| GET | `/api/bookings/slots/recommendations` | ❌ | Get slot recommendations (`?date=&group_size=`) |
| GET | `/api/bookings/slots/:slotId/menu` | ❌ | Get menu for a slot |
| GET | `/api/bookings/slots/:slotId` | ❌ | Get slot details |
| GET | `/api/bookings/demand-analysis` | ❌ | Demand analysis (`?date=`) |
| POST | `/api/bookings/menu/search` | ❌ | Search menu items |
| GET | `/api/bookings/users/search` | ✅ | Search users by email (group booking) |
| GET | `/api/bookings/my-bookings` | ✅ | Get user's bookings (`?status=`) |
| GET | `/api/bookings/payments` | ✅ | Get booking payments |
| POST | `/api/bookings` | ✅ | Create booking |
| GET | `/api/bookings/:bookingId` | ✅ | Get booking by ID |
| PUT | `/api/bookings/:bookingId` | ✅ | Update booking |
| DELETE | `/api/bookings/:bookingId` | ✅ | Cancel booking |

### Token Routes (`/api/tokens/*`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| GET | `/api/tokens/queue/status` | ❌ | Queue status (`?slot_id=&date=`) |
| GET | `/api/tokens/counters` | ❌ | Get service counters |
| GET | `/api/tokens/queue/live` | ❌ | SSE live queue updates (`?token_id=`) |
| GET | `/api/tokens/my-tokens` | ✅ | Get user's tokens (`?status=&date=`) |
| POST | `/api/tokens/generate` | ✅ | Generate token for booking |
| GET | `/api/tokens/booking/:bookingReference` | ✅ | Get token by booking ref |
| GET | `/api/tokens/:tokenId` | ✅ | Get token details |
| POST | `/api/tokens/:tokenId/activate` | ✅ | Activate token & assign counter |
| POST | `/api/tokens/meal-slot/:slotId/activate` | — | Activate all tokens for slot (staff) |
| POST | `/api/tokens/counters/:counterId/call-next` | — | Call next token (staff) |
| POST | `/api/tokens/:tokenId/mark-served` | — | Mark token served (staff) |
| POST | `/api/tokens/counters/:counterId/close` | — | Close counter & reassign tokens (staff) |
| POST | `/api/tokens/counters/:counterId/reopen` | — | Reopen counter (staff) |

### Payment Routes (`/api/payments/*`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/payments/stripe/webhook` | ❌ | Stripe webhook (raw body) |
| GET | `/api/payments/window/:bookingId` | ✅ | Get payment window status |
| POST | `/api/payments/window/:bookingId/extend` | ✅ | Extend payment window (admin) |
| POST | `/api/payments/wallet/contribute` | ✅ | Contribute to booking wallet (bill split) |
| GET | `/api/payments/wallet/:bookingId/contributions` | ✅ | Get wallet contributions |
| POST | `/api/payments/settle/:bookingId` | ✅ | Settle bill from wallet |
| POST | `/api/payments/stripe/create-intent` | ✅ | Create Stripe Payment Intent |
| POST | `/api/payments/stripe/confirm` | ✅ | Confirm Stripe payment |
| POST | `/api/payments/personal-wallet/create-session` | ✅ | Create wallet recharge session |
| GET | `/api/payments/personal-wallet/balance` | ✅ | Get wallet balance |
| GET | `/api/payments/personal-wallet/session-status/:sessionId` | ✅ | Get checkout session status |
| POST | `/api/payments/personal-wallet/confirm-recharge` | ✅ | Confirm wallet recharge |
| GET | `/api/payments/personal-wallet/transactions` | ✅ | Get wallet transactions |
| POST | `/api/payments/no-show/process/:slotId` | ✅ | Process no-shows (staff) |
| GET | `/api/payments/walk-in/available` | ❌ | Get walk-in meals |
| POST | `/api/payments/walk-in/assign` | ✅ | Assign walk-in meal (staff) |
| GET | `/api/payments/reports/no-show` | ✅ | No-show report |
| GET | `/api/payments/reports/unused-slots` | ✅ | Unused slots report |
| GET | `/api/payments/reports/unused-meals` | ✅ | Unused meals report |
| GET | `/api/payments/leftover/:slotId` | ✅ | Detect leftover food |
| POST | `/api/payments/leftover/transfer` | ✅ | Transfer leftover to next slot |
| POST | `/api/payments/process-expired` | ❌ | Process expired payments (cron) |

---

## Stripe Integration

### Getting Started with Stripe Test Mode

1. **Create a Stripe Account** at [dashboard.stripe.com/register](https://dashboard.stripe.com/register)
2. **Get API Keys** from [Developers > API Keys](https://dashboard.stripe.com/test/apikeys) (ensure **Test Mode** is toggled on)
3. **Set up Stripe CLI for local webhooks:**

   ```bash
   # macOS
   brew install stripe/stripe-cli/stripe

   # Windows (Scoop)
   scoop install stripe
   ```

   ```bash
   stripe login
   stripe listen --forward-to localhost:3001/api/payments/stripe/webhook
   ```

   Copy the `whsec_...` signing secret into your backend `.env`.

4. **Test Card Numbers:**

   | Card Number | Result |
   |-------------|--------|
   | `4242 4242 4242 4242` | Successful payment |
   | `4000 0000 0000 9995` | Declined payment |
   | `4000 0025 0000 3155` | Requires 3D Secure |

   Use any future expiry date and any 3-digit CVC.

### Webhook Events

The backend listens for:
- `payment_intent.succeeded`
- `payment_intent.payment_failed`

---

## Code Quality & Linting

The project uses **Biome** (v2.3.13) as a unified linter and formatter, configured in the root `biome.json`:

- **Formatter:** Tab indentation, 100 char line width, double quotes, trailing commas (ES5), always semicolons.
- **Linter:** Recommended rules enabled with specific overrides (see `biome.json`).
- **Files:** `**/*.js`, `**/*.ts`, `**/*.jsx`, `**/*.tsx`, `**/*.json`.

Run checks:

```bash
# Lint only
pnpm lint

# Full check (lint + format)
pnpm --filter ./backend run check
pnpm --filter ./frontend run check

# Auto-fix
pnpm --filter ./backend run fix
pnpm --filter ./frontend run fix

# Type check (TypeScript)
pnpm --filter ./backend run type-check
pnpm --filter ./frontend run type-check
```

---

## Git Hooks

**Husky** is configured with a `pre-commit` hook that runs:

```bash
pnpm pre-commit
```

This triggers **lint-staged** in both `backend/` and `frontend/`, which runs:

```bash
biome check --write --no-errors-on-unmatched
```

on all staged `*.{js,ts,jsx,tsx,json}` files — automatically fixing formatting and lint issues before commit.

---

## Languages

| Language | Percentage |
|----------|-----------|
| TypeScript | 94.9% |
| CSS | 4.9% |
| Other | 0.2% |

---

## License

ISC

---

**Happy Coding! 🚀**
