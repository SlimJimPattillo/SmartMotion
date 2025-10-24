# SmartMotion Architecture Guide

**For Students Learning Full-Stack Development**

This guide explains how the frontend and backend of SmartMotion are organized, how they communicate, and how data flows through the application.

---

## Table of Contents

1. [Big Picture Overview](#big-picture-overview)
2. [Frontend vs Backend: What's the Difference?](#frontend-vs-backend-whats-the-difference)
3. [Your App's Architecture](#your-apps-architecture)
4. [Data Flow Examples](#data-flow-examples)
5. [Key Concepts Explained](#key-concepts-explained)
6. [Common Patterns in Your Code](#common-patterns-in-your-code)

---

## Big Picture Overview

SmartMotion uses a **modern full-stack architecture** with Next.js, which combines frontend and backend in one framework. Here's the bird's eye view:

```
┌─────────────────────────────────────────────────────────────────────┐
│                           USER'S BROWSER                            │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │              FRONTEND (Client-Side React)                     │ │
│  │  • Pages (UI components)                                      │ │
│  │  • Forms, buttons, displays                                   │ │
│  │  • User interactions                                          │ │
│  └───────────────────────────────────────────────────────────────┘ │
└──────────────────────────┬──────────────────────────────────────────┘
                           │ HTTPS Requests
                           ↓
┌─────────────────────────────────────────────────────────────────────┐
│                    VERCEL (Cloud Hosting)                           │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │         NEXT.JS APP (Frontend + Backend Together)            │ │
│  │                                                               │ │
│  │  Frontend:                    Backend:                       │ │
│  │  • app/page.tsx              • app/api/*/route.ts            │ │
│  │  • app/dashboard/*           • middleware.ts                 │ │
│  │  • components/*              • lib/supabase/server.ts        │ │
│  │                                                               │ │
│  └───────────────────────────────────────────────────────────────┘ │
└──────┬──────────────────────────────────────┬───────────────────────┘
       │                                      │
       │ Database queries                     │ Payment processing
       ↓                                      ↓
┌─────────────────────┐              ┌─────────────────────┐
│   SUPABASE          │              │    STRIPE           │
│   (Backend Service) │              │   (Payment Service) │
│                     │              │                     │
│  • PostgreSQL DB    │              │  • Payment API      │
│  • Authentication   │              │  • Webhooks         │
│  • Real-time data   │              │  • Subscriptions    │
└─────────────────────┘              └─────────────────────┘
```

---

## Frontend vs Backend: What's the Difference?

### 🎨 Frontend (Client-Side)
**What it is:** Code that runs in the user's browser
**What it does:**
- Displays UI (pages, buttons, forms)
- Handles user interactions (clicks, typing)
- Shows data to the user

**In your app:**
- Pages: `app/page.tsx`, `app/dashboard/page.tsx`
- Components: `components/DashboardLayout.tsx`
- Marked with: `'use client'` directive

### ⚙️ Backend (Server-Side)
**What it is:** Code that runs on the server
**What it does:**
- Processes requests
- Talks to databases
- Handles authentication
- Protects sensitive data

**In your app:**
- API Routes: `app/api/create-checkout/route.ts`
- Server Components: Files without `'use client'`
- Middleware: `middleware.ts`
- Server utilities: `lib/supabase/server.ts`

---

## Your App's Architecture

### 1. Detailed Component Diagram

```
SmartMotion Application Structure
═══════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────┐
│                    CLIENT SIDE (Browser)                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  PUBLIC PAGES (Not authenticated)                          │
│  ┌─────────────────────────────────────────────┐          │
│  │  app/page.tsx                               │          │
│  │  • Landing page                             │          │
│  │  • Marketing content                        │          │
│  │  • Sign up button                           │          │
│  └─────────────────────────────────────────────┘          │
│                                                             │
│  AUTH PAGES                                                │
│  ┌─────────────────────────────────────────────┐          │
│  │  app/auth/signin/page.tsx                   │          │
│  │  app/auth/signup/page.tsx                   │          │
│  │  app/auth/reset-password/page.tsx           │          │
│  │                                              │          │
│  │  • Login forms                               │          │
│  │  • Creates Supabase client                   │          │
│  │  • Calls supabase.auth.signIn()              │          │
│  └─────────────────────────────────────────────┘          │
│                                                             │
│  PROTECTED PAGES (Requires login)                          │
│  ┌─────────────────────────────────────────────┐          │
│  │  app/dashboard/page.tsx                     │          │
│  │  app/dashboard/exercises/page.tsx           │          │
│  │  app/dashboard/progress/page.tsx            │          │
│  │  app/dashboard/quiz/page.tsx                │          │
│  │  app/dashboard/profile/page.tsx             │          │
│  │                                              │          │
│  │  • Fetches user data                         │          │
│  │  • Displays workouts, progress, quizzes      │          │
│  │  • Updates database via Supabase             │          │
│  └─────────────────────────────────────────────┘          │
│                                                             │
│  REUSABLE COMPONENTS                                       │
│  ┌─────────────────────────────────────────────┐          │
│  │  components/DashboardLayout.tsx             │          │
│  │  components/StreakCelebration.tsx           │          │
│  │                                              │          │
│  │  • Navigation, headers, footers              │          │
│  │  • Reused across multiple pages              │          │
│  └─────────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────┘
                           ↕
            HTTP Requests / API Calls
                           ↕
┌─────────────────────────────────────────────────────────────┐
│              SERVER SIDE (Vercel/Next.js)                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  MIDDLEWARE (Runs on every request)                        │
│  ┌─────────────────────────────────────────────┐          │
│  │  middleware.ts                              │          │
│  │                                              │          │
│  │  • Checks if user is authenticated           │          │
│  │  • Redirects to /auth/signin if not         │          │
│  │  • Protects /dashboard/* routes             │          │
│  │  • Runs BEFORE page loads                   │          │
│  └─────────────────────────────────────────────┘          │
│                                                             │
│  API ROUTES (Backend endpoints)                            │
│  ┌─────────────────────────────────────────────┐          │
│  │  app/api/create-checkout/route.ts           │          │
│  │  • POST /api/create-checkout                │          │
│  │  • Creates Stripe payment session            │          │
│  │  • Returns sessionId to client               │          │
│  │                                              │          │
│  │  app/api/webhook/route.ts                   │          │
│  │  • POST /api/webhook                        │          │
│  │  • Receives Stripe payment confirmations    │          │
│  │  • Updates database after payment            │          │
│  └─────────────────────────────────────────────┘          │
│                                                             │
│  SERVER UTILITIES                                          │
│  ┌─────────────────────────────────────────────┐          │
│  │  lib/supabase/server.ts                     │          │
│  │  • Creates Supabase client for server       │          │
│  │  • Uses cookies for authentication           │          │
│  │  • Used in API routes                        │          │
│  │                                              │          │
│  │  lib/supabase/client.ts                     │          │
│  │  • Creates Supabase client for browser      │          │
│  │  • Used in Client Components                 │          │
│  │                                              │          │
│  │  lib/stripe/client.ts                       │          │
│  │  • Stripe.js for browser                     │          │
│  └─────────────────────────────────────────────┘          │
└─────────────────────────────────────────────────────────────┘
                           ↕
                   External Services
                           ↕
┌─────────────────────────────────────────────────────────────┐
│                   EXTERNAL SERVICES                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌────────────────────┐        ┌────────────────────┐     │
│  │   SUPABASE         │        │      STRIPE        │     │
│  ├────────────────────┤        ├────────────────────┤     │
│  │ • PostgreSQL DB    │        │ • Payment API      │     │
│  │ • Auth (JWT)       │        │ • Checkout session │     │
│  │ • Real-time sync   │        │ • Webhooks         │     │
│  │                    │        │                    │     │
│  │ Tables:            │        │ Handles:           │     │
│  │  - profiles        │        │  - Card payments   │     │
│  │  - fitness_goals   │        │  - Donations       │     │
│  │  - exercises       │        │  - Subscriptions   │     │
│  │  - workout_logs    │        │                    │     │
│  │  - quizzes         │        │                    │     │
│  │  - streaks         │        │                    │     │
│  └────────────────────┘        └────────────────────┘     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Data Flow Examples

Let me show you **exactly** how data flows through your app with real examples from your code.

### Example 1: User Signs In

```
┌──────────────────────────────────────────────────────────────────┐
│ STEP 1: User enters email/password and clicks "Sign In"         │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 2: app/auth/signin/page.tsx (CLIENT SIDE)                  │
│                                                                  │
│  const handleSignIn = async (e) => {                            │
│    const supabase = createClient()  // Browser client           │
│    const { error } = await supabase.auth.signInWithPassword({   │
│      email,                                                      │
│      password                                                    │
│    })                                                            │
│  }                                                               │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 3: Request goes to SUPABASE                                │
│                                                                  │
│  • Supabase checks email/password                               │
│  • If valid, creates JWT token                                  │
│  • Sends token back to browser                                  │
│  • Browser stores token in cookies                              │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 4: User is redirected to /dashboard                        │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 5: middleware.ts runs BEFORE page loads                    │
│                                                                  │
│  • Checks cookie for auth token                                 │
│  • Validates token with Supabase                                │
│  • If valid: allows access to /dashboard                        │
│  • If invalid: redirects to /auth/signin                        │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 6: app/dashboard/page.tsx loads                            │
│                                                                  │
│  • Fetches user's workout data from Supabase                    │
│  • Displays dashboard UI                                        │
└──────────────────────────────────────────────────────────────────┘
```

### Example 2: User Makes a Donation (Stripe Payment)

```
┌──────────────────────────────────────────────────────────────────┐
│ STEP 1: User clicks "Donate $1" button                          │
│         Location: app/dashboard/profile/page.tsx                 │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 2: Frontend sends POST request                             │
│                                                                  │
│  const response = await fetch('/api/create-checkout', {         │
│    method: 'POST'                                                │
│  })                                                              │
│  const { sessionId } = await response.json()                    │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 3: Backend API route processes request                     │
│         Location: app/api/create-checkout/route.ts              │
│                                                                  │
│  export async function POST(req: NextRequest) {                 │
│    // 1. Get authenticated user                                 │
│    const supabase = await createServerClient()                  │
│    const { data: { user } } = await supabase.auth.getUser()    │
│                                                                  │
│    // 2. Create Stripe checkout session                         │
│    const session = await stripe.checkout.sessions.create({      │
│      line_items: [{ price_data: { unit_amount: 100 } }],       │
│      success_url: '/dashboard/profile?success=true',            │
│      metadata: { user_id: user.id }                             │
│    })                                                            │
│                                                                  │
│    // 3. Return sessionId to frontend                           │
│    return NextResponse.json({ sessionId: session.id })          │
│  }                                                               │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 4: Frontend redirects to Stripe checkout                   │
│                                                                  │
│  const stripe = await getStripe()                               │
│  await stripe.redirectToCheckout({ sessionId })                 │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 5: User enters payment info on Stripe's page               │
│         (Hosted by Stripe, not your app)                        │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 6: Stripe processes payment                                │
│         • User pays $1                                           │
│         • Stripe charges card                                    │
│         • Payment succeeds                                       │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 7: Stripe sends webhook to your app                        │
│         POST request to: /api/webhook                            │
│                                                                  │
│  Webhook payload:                                                │
│  {                                                               │
│    type: "checkout.session.completed",                          │
│    data: { metadata: { user_id: "123" } }                       │
│  }                                                               │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 8: Webhook handler processes payment                       │
│         Location: app/api/webhook/route.ts                      │
│                                                                  │
│  export async function POST(req: NextRequest) {                 │
│    // 1. Verify webhook signature (security)                    │
│    const event = stripe.webhooks.constructEvent(body, sig)      │
│                                                                  │
│    // 2. Handle completed payment                               │
│    if (event.type === 'checkout.session.completed') {           │
│      const userId = event.data.object.metadata.user_id          │
│      // Could update database here to mark donation             │
│    }                                                             │
│  }                                                               │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 9: User redirected back to your app                        │
│         URL: /dashboard/profile?success=true                     │
│         Shows: "Thank you for your donation!" message           │
└──────────────────────────────────────────────────────────────────┘
```

### Example 3: User Logs a Workout

```
┌──────────────────────────────────────────────────────────────────┐
│ STEP 1: User marks workout as complete                          │
│         Location: app/dashboard/page.tsx                         │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 2: Frontend creates Supabase client                        │
│                                                                  │
│  const supabase = createClient()  // Browser client             │
│                                                                  │
│  const { error } = await supabase                               │
│    .from('workout_logs')                                         │
│    .insert({                                                     │
│      user_id: user.id,                                           │
│      exercise_id: exerciseId,                                    │
│      completed_at: new Date(),                                   │
│      notes: "Felt great!"                                        │
│    })                                                            │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 3: Request goes to SUPABASE                                │
│                                                                  │
│  • Supabase receives INSERT request                             │
│  • Checks authentication (JWT from cookie)                      │
│  • Validates Row Level Security policies                        │
│  • Inserts row into workout_logs table                          │
│  • Returns success response                                     │
└──────────────────────────────────────────────────────────────────┘
                           │
                           ↓
┌──────────────────────────────────────────────────────────────────┐
│ STEP 4: Frontend updates UI                                     │
│                                                                  │
│  • Shows success message                                        │
│  • Updates workout count                                        │
│  • Checks and updates streak                                    │
│  • May show StreakCelebration component                         │
└──────────────────────────────────────────────────────────────────┘
```

---

## Key Concepts Explained

### 1. Client Components vs Server Components

#### Client Components (`'use client'`)

**File:** `app/auth/signin/page.tsx`

```typescript
'use client'  // ← This tells Next.js: "Run this in the browser"

import { useState } from 'react'  // ← Browser-only features

export default function SignInPage() {
  const [email, setEmail] = useState('')  // ← Interactive state

  // This runs in the USER'S BROWSER
  return (
    <input
      value={email}
      onChange={(e) => setEmail(e.target.value)}  // ← User interaction
    />
  )
}
```

**Why Client Component?**
- Needs `useState` for form inputs
- Handles user interactions (typing, clicking)
- Updates UI in real-time

#### Server Components (No `'use client'`)

**File:** `app/dashboard/education/page.tsx`

```typescript
// No 'use client' = Server Component

import { createServerClient } from '@/lib/supabase/server'

export default async function EducationPage() {
  // This runs on THE SERVER (Vercel)
  const supabase = await createServerClient()

  // Direct database access (secure!)
  const { data: articles } = await supabase
    .from('education_content')
    .select('*')

  // Returns pre-rendered HTML
  return <div>{articles.map(article => ...)}</div>
}
```

**Why Server Component?**
- Fetches data before page loads
- Faster for users (pre-rendered)
- More secure (database access hidden from browser)

### 2. API Routes (Your Backend Endpoints)

API routes are like "mini servers" inside your Next.js app.

**File:** `app/api/create-checkout/route.ts`

```typescript
// This is a BACKEND endpoint
// URL: https://your-app.com/api/create-checkout

export async function POST(req: NextRequest) {
  // 1. This code runs on the SERVER, not browser

  // 2. Can safely use secret API keys
  const stripe = new Stripe(process.env.STRIPE_SECRET_KEY)

  // 3. Processes the request
  const session = await stripe.checkout.sessions.create({...})

  // 4. Returns JSON response
  return NextResponse.json({ sessionId: session.id })
}
```

**How it's called from frontend:**

```typescript
// In a Client Component
const response = await fetch('/api/create-checkout', {
  method: 'POST'
})
const data = await response.json()
```

### 3. Middleware (The Gatekeeper)

**File:** `middleware.ts`

```typescript
export async function middleware(req: NextRequest) {
  // This runs BEFORE every page request

  // Check if user is logged in
  const { session } = await supabase.auth.getSession()

  // Protect /dashboard routes
  if (!session && req.nextUrl.pathname.startsWith('/dashboard')) {
    // Not logged in? Redirect to login page
    return NextResponse.redirect(new URL('/auth/signin', req.url))
  }

  // Logged in? Allow access
  return NextResponse.next()
}

// Only run on these routes
export const config = {
  matcher: ['/dashboard/:path*', '/auth/:path*']
}
```

**Visual Flow:**

```
User types URL → middleware checks → allowed/redirected → page loads
   ↓                    ↓                    ↓                ↓
/dashboard      Is user logged in?         YES           Show page
                       ↓
                      NO
                       ↓
              Redirect to /auth/signin
```

### 4. Supabase Client Types

You have **THREE** different Supabase clients for different use cases:

#### A. Browser Client (`lib/supabase/client.ts`)

```typescript
// Used in: Client Components ('use client')
// Runs in: User's browser
// Has access to: User's session via cookies

export const createClient = () =>
  createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL,      // ← Public (safe in browser)
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY  // ← Public (safe in browser)
  )
```

**Example usage:**
```typescript
'use client'

const supabase = createClient()
const { data } = await supabase.from('exercises').select('*')
```

#### B. Server Client (`lib/supabase/server.ts`)

```typescript
// Used in: Server Components, API routes
// Runs in: Server (Vercel)
// Has access to: Server-side cookies (more secure)

export const createServerClient = async () => {
  const cookieStore = await cookies()  // ← Server-only API

  return createClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY,
    {
      cookies: {
        getAll: () => cookieStore.getAll(),
        setAll: (cookies) => {...}
      }
    }
  )
}
```

**Example usage:**
```typescript
// In API route
const supabase = await createServerClient()
const { data: { user } } = await supabase.auth.getUser()
```

#### C. Admin Client (Webhook)

```typescript
// Used in: Webhooks where you need elevated permissions
// Runs in: Server (during webhook processing)
// Has access to: SERVICE_ROLE_KEY (admin access)

function getSupabaseAdmin() {
  return createClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL,
    process.env.SUPABASE_SERVICE_ROLE_KEY  // ← ADMIN KEY (very powerful!)
  )
}
```

**Why different clients?**
- **Security**: Admin key can bypass all security rules
- **Context**: Browser vs server have different capabilities
- **Cookies**: Browser and server access cookies differently

---

## Common Patterns in Your Code

### Pattern 1: Protected Page with Data Fetching

**File:** `app/dashboard/exercises/page.tsx`

```typescript
'use client'  // Client Component (needs useState, useEffect)

export const dynamic = 'force-dynamic'  // Don't pre-render (needs auth)

export default function ExercisesPage() {
  const [exercises, setExercises] = useState([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    async function fetchExercises() {
      const supabase = createClient()  // Browser client

      // Get current user's fitness goal
      const { data: { user } } = await supabase.auth.getUser()
      const { data: goal } = await supabase
        .from('fitness_goals')
        .select('goal_type')
        .eq('user_id', user.id)
        .single()

      // Get exercises for that goal
      const { data: exercises } = await supabase
        .from('exercises')
        .select('*')
        .eq('goal_type', goal.goal_type)

      setExercises(exercises)
      setLoading(false)
    }

    fetchExercises()
  }, [])

  if (loading) return <div>Loading...</div>

  return (
    <DashboardLayout>
      {exercises.map(exercise => (
        <ExerciseCard key={exercise.id} exercise={exercise} />
      ))}
    </DashboardLayout>
  )
}
```

**What's happening:**
1. Page loads (protected by middleware)
2. useEffect runs after first render
3. Fetches user's goal from database
4. Fetches exercises matching that goal
5. Updates UI with exercises

### Pattern 2: Form Submission with Error Handling

**File:** `app/auth/signup/page.tsx`

```typescript
const handleSignUp = async (e: React.FormEvent) => {
  e.preventDefault()  // Prevent page reload
  setError(null)      // Clear previous errors
  setLoading(true)    // Show loading state

  try {
    const supabase = createClient()

    // Attempt to create account
    const { error } = await supabase.auth.signUp({
      email,
      password,
    })

    if (error) throw error

    // Success! Redirect to onboarding
    router.push('/onboarding')

  } catch (error: any) {
    // Show error to user
    setError(error.message)
  } finally {
    // Always stop loading state
    setLoading(false)
  }
}
```

**Pattern breakdown:**
1. Prevent default form submission
2. Clear old errors
3. Show loading indicator
4. Try operation
5. Handle success or errors
6. Always cleanup (finally block)

### Pattern 3: Real-time Data with Subscriptions

If you wanted to add real-time updates (not in your current code):

```typescript
useEffect(() => {
  const supabase = createClient()

  // Subscribe to workout_logs changes
  const subscription = supabase
    .channel('workout_logs_changes')
    .on('postgres_changes',
      {
        event: 'INSERT',
        schema: 'public',
        table: 'workout_logs'
      },
      (payload) => {
        // When a new workout is logged, update UI
        setWorkouts(prev => [...prev, payload.new])
      }
    )
    .subscribe()

  // Cleanup on unmount
  return () => {
    subscription.unsubscribe()
  }
}, [])
```

---

## Summary: The Full Request Lifecycle

Let's trace ONE complete request from start to finish:

### Scenario: User views their dashboard

```
1. USER TYPES URL
   ↓
   User enters: https://smart-motion-uizl.vercel.app/dashboard

2. DNS LOOKUP
   ↓
   Browser finds Vercel's servers

3. REQUEST TO VERCEL
   ↓
   GET /dashboard
   Cookies: auth_token=abc123...

4. MIDDLEWARE RUNS (middleware.ts)
   ↓
   • Reads auth_token from cookies
   • Validates with Supabase
   • Token valid? Continue to next step
   • Token invalid? Redirect to /auth/signin

5. NEXT.JS ROUTING
   ↓
   • Matches /dashboard to app/dashboard/page.tsx
   • Checks: 'use client' directive present
   • Sends HTML shell to browser

6. BROWSER RECEIVES HTML
   ↓
   • Shows loading state
   • Downloads JavaScript bundle
   • React hydrates the page

7. CLIENT COMPONENT RUNS (app/dashboard/page.tsx)
   ↓
   • useEffect triggers
   • createClient() makes Supabase browser client
   • Fetches workout data from database
   • Database queries include auth token automatically

8. SUPABASE PROCESSES REQUEST
   ↓
   • Validates JWT token
   • Checks Row Level Security (RLS) policies
   • Returns only data user is allowed to see
   • Sends JSON response

9. FRONTEND UPDATES UI
   ↓
   • setState updates with workout data
   • React re-renders components
   • User sees their dashboard!

10. REAL-TIME LISTENING (if implemented)
    ↓
    • WebSocket connection to Supabase
    • Listens for any database changes
    • Auto-updates UI when data changes
```

---

## Key Takeaways for Students

1. **Next.js blends frontend and backend**
   - Same repo, same language (TypeScript)
   - Frontend: Pages and components
   - Backend: API routes and middleware

2. **Two types of components**
   - Client Components: Interactive, run in browser
   - Server Components: Data fetching, run on server

3. **Authentication flow**
   - User logs in → Supabase returns JWT
   - JWT stored in cookies
   - Middleware checks JWT on every request
   - Invalid JWT? Redirect to login

4. **Database access**
   - Browser uses public anon key
   - Server can use service role key for admin tasks
   - Row Level Security protects data

5. **Payment flow**
   - Frontend creates checkout session via API route
   - User pays on Stripe's site
   - Stripe sends webhook to confirm payment
   - Webhook updates your database

6. **Environment variables**
   - `NEXT_PUBLIC_*`: Safe for browser
   - Others: Server-only (secrets!)

---

## What to Study Next

1. **Next.js App Router**
   - https://nextjs.org/docs/app

2. **React Server Components**
   - https://react.dev/reference/rsc/server-components

3. **Supabase Authentication**
   - https://supabase.com/docs/guides/auth

4. **TypeScript Basics**
   - https://www.typescriptlang.org/docs/handbook/intro.html

5. **RESTful API Design**
   - Understand GET, POST, PUT, DELETE
   - HTTP status codes (200, 401, 404, 500)

---

**Questions to test your understanding:**

1. What's the difference between `lib/supabase/client.ts` and `lib/supabase/server.ts`?
2. Why do we need middleware.ts?
3. What happens if a user tries to access /dashboard without logging in?
4. Why can't we use `useState` in a Server Component?
5. When should you use an API route vs direct Supabase calls?

**Answers:**

1. `client.ts` is for browser (Client Components), `server.ts` is for server-side code (API routes, Server Components). They handle cookies differently.

2. Middleware protects routes by checking authentication before the page loads. It redirects unauthorized users.

3. Middleware catches them, checks their cookie, finds no valid session, and redirects to `/auth/signin`.

4. `useState` is a browser API that manages interactive state. Server Components render once on the server and send HTML—no interactivity.

5. Use API routes when you need to:
   - Use secret API keys (Stripe secret)
   - Process webhooks
   - Do complex server-side logic
   - Bypass Row Level Security (with service role key)

   Use direct Supabase calls when:
   - Simple CRUD operations
   - User is accessing their own data
   - RLS policies handle security

---

**Created for: Students learning full-stack development**
**App: SmartMotion**
**Date: October 24, 2025**
