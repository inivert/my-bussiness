# SaaS Website Builder Platform Requirements

## Project Overview
Build a subscription-based website builder platform for small businesses with client portal and admin dashboard capabilities.

## Technology Stack

### Core Technologies
- Framework: Next.js 14+ with App Router
- Database: Supabase (Free Tier)
- Authentication: Clerk (Free Tier)
- File Storage: Uploadthing (Free Tier)
- Hosting: Vercel (Free Tier)
- Email: Resend (Free Tier)
- Payments: Stripe
- Client Websites: Astro 4+
- Styling: Tailwind CSS + shadcn/ui
- Type Safety: TypeScript strict mode

### Required Package Versions
```json
{
  "dependencies": {
    "next": "^14.0.0",
    "@clerk/nextjs": "^4.27.0",
    "@supabase/supabase-js": "^2.39.0",
    "uploadthing": "^6.0.0",
    "@stripe/stripe-js": "^2.2.0",
    "@tanstack/react-query": "^5.0.0",
    "zod": "^3.22.0",
    "tailwindcss": "^3.3.0",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.0.0",
    "tailwind-merge": "^2.0.0"
  }
}
```

## Project Structure
```
src/
├── app/
│   ├── (auth)/
│   │   ├── sign-in/
│   │   └── sign-up/
│   ├── (dashboard)/
│   │   ├── admin/
│   │   └── client/
│   ├── api/
│   └── page.tsx
├── components/
│   ├── ui/
│   ├── forms/
│   └── shared/
├── lib/
│   ├── utils.ts
│   └── constants.ts
└── types/
    └── index.ts
```

## Database Schema

```sql
-- Users (handled by Clerk)
-- Subscriptions
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id STRING REFERENCES auth.users(id),
  stripe_subscription_id STRING,
  plan_id STRING,
  status STRING,
  current_period_end TIMESTAMP,
  created_at TIMESTAMP DEFAULT now()
);

-- Websites
CREATE TABLE websites (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id STRING REFERENCES auth.users(id),
  subdomain STRING UNIQUE,
  custom_domain STRING,
  template_id STRING,
  settings JSONB,
  created_at TIMESTAMP DEFAULT now()
);

-- Tickets
CREATE TABLE tickets (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id STRING REFERENCES auth.users(id),
  website_id UUID REFERENCES websites(id),
  title STRING,
  description TEXT,
  status STRING,
  priority STRING,
  created_at TIMESTAMP DEFAULT now()
);

-- Ticket Attachments
CREATE TABLE ticket_attachments (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  ticket_id UUID REFERENCES tickets(id),
  file_url STRING,
  file_type STRING,
  created_at TIMESTAMP DEFAULT now()
);
```

## Environment Variables Structure
```env
# Development Environment (.env.development)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_
CLERK_SECRET_KEY=sk_test_
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard

SUPABASE_URL=your_supabase_url
SUPABASE_ANON_KEY=your_supabase_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key

STRIPE_SECRET_KEY=sk_test_
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_
STRIPE_WEBHOOK_SECRET=whsec_

UPLOADTHING_SECRET=sk_live_
UPLOADTHING_APP_ID=your_app_id

RESEND_API_KEY=re_test_

# Production Environment (.env.production)
# Same structure but with production values
# Remove all test keys and sample data
```

## Feature Implementation

### Subscription Plans
```typescript
export const SUBSCRIPTION_PLANS = {
  FREE: {
    name: 'Free',
    price: 0,
    features: ['1 Website', '5 Tickets/month', 'Basic Templates'],
    maxWebsites: 1,
    maxTickets: 5,
    stripePriceId: process.env.NODE_ENV === 'development' 
      ? 'price_test_id' 
      : 'price_live_id'
  },
  BASIC: {
    name: 'Basic',
    price: 29,
    features: ['3 Websites', '15 Tickets/month', 'Custom Domain'],
    maxWebsites: 3,
    maxTickets: 15,
    stripePriceId: process.env.NODE_ENV === 'development'
      ? 'price_test_id'
      : 'price_live_id'
  },
  PREMIUM: {
    name: 'Premium',
    price: 79,
    features: ['10 Websites', '50 Tickets/month', 'All Features'],
    maxWebsites: 10,
    maxTickets: 50,
    stripePriceId: process.env.NODE_ENV === 'development'
      ? 'price_test_id'
      : 'price_live_id'
  }
}
```

### Development vs Production Mode

Development Mode:
- Enable `NOT_READY_FOR_PRODUCTION` banner
- Use test API keys
- Include sample data
- Enable detailed error messages
- Set development database
- Mock certain API responses

Production Mode:
- Remove all test/sample data
- Use production API keys
- Disable detailed error messages
- Enable proper error tracking
- Use production database
- Implement rate limiting
- Enable security headers

### Security Implementation
```typescript
// middleware.ts
import { headers } from 'next/headers'
import { clerkMiddleware } from '@clerk/nextjs'

export default clerkMiddleware({
  publicRoutes: ['/api/webhook/stripe'],
  beforeAuth: (req) => {
    // Add security headers
    const requestHeaders = new Headers(req.headers)
    requestHeaders.set('X-Frame-Options', 'DENY')
    requestHeaders.set('X-Content-Type-Options', 'nosniff')
    requestHeaders.set('Referrer-Policy', 'strict-origin-when-cross-origin')
    requestHeaders.set(
      'Content-Security-Policy',
      "default-src 'self'; img-src 'self' https: data:; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';"
    )
  }
})
```

## Testing Requirements

### Test Data
```typescript
// lib/sample-data.ts
export const SAMPLE_WEBSITES = [
  {
    id: 'test-1',
    name: 'Sample Restaurant',
    template: 'restaurant-1',
    status: 'active'
  },
  // Add more sample data
]

// Remove this file in production
```

### Development Guards
```typescript
// lib/environment.ts
export const isDevelopment = process.env.NODE_ENV === 'development'

if (isDevelopment) {
  console.warn('Running in development mode - NOT READY FOR PRODUCTION')
}

// Components/NotReadyForProduction.tsx
export function NotReadyBanner() {
  if (process.env.NODE_ENV === 'production') return null
  
  return (
    <div className="bg-yellow-500 text-white p-2 text-center">
      Development Mode - NOT READY FOR PRODUCTION
    </div>
  )
}
```

## Client Website Templates (Astro)
```typescript
// Templates should be stored as separate Astro projects
// Each template should have:
- Base configuration
- Content schema
- Style variants
- Component library
- SEO defaults
```

## Deployment Strategy

### Development:
1. Local development using feature branches
2. Test deployment on Vercel preview URLs
3. Use development database
4. Enable detailed logging

### Production:
1. Main branch deployment only
2. Production database with backups
3. SSL certificates
4. CDN enabled
5. Rate limiting
6. Error tracking
7. Performance monitoring

## Additional Notes

1. Free Tier Limits:
- Monitor usage closely
- Implement graceful degradation
- Clear upgrade paths

2. Error Handling:
- Implement proper error boundaries
- User-friendly error messages
- Error logging system

3. Performance:
- Implement caching where possible
- Use edge functions for global deployment
- Optimize images and assets

4. Monitoring:
- Setup basic analytics
- Monitor API usage
- Track error rates

Would you like me to elaborate on any specific part of these requirements?