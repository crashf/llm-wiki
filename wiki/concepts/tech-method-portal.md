---
title: Tech-Method Portal Pattern
type: concept
tags: [portal, sso, multi-tenant, nextjs, msp, client-portal]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

The Tech-Method Portal Pattern is a self-service web portal for MSP clients, providing ticket management, invoice viewing, and payment capabilities. Built with Next.js and NextAuth.js, authenticating against a PSA system (ConnectWise) as the single source of truth. Multi-tenant ready with environment-driven configuration.

## Key Points

- **Authentication:** Multi-provider SSO (Microsoft Entra ID, Google, magic link), validated against PSA contacts
- **Authorization:** PSA contacts determine access — email must exist in PSA system
- **Data Source:** PSA (ConnectWise Manage) for contacts, tickets, and invoices
- **Multi-tenant:** Each MSP tenant has own ConnectWise instance, branding via environment variables
- **Stateless:** JWT sessions, no portal-specific database required

## Pattern Components

```
┌─────────────────────────────────────────────────┐
│                 Client Portal                   │
├─────────────────────────────────────────────────┤
│  Next.js App (SSR + API routes)                │
│  ├── NextAuth.js (multiple providers)        │
│  ├── Middleware (security headers + CORS)   │
│  └── API routes (ticket/invoice CRUD)      │
└─────────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────┐
│           PSA Integration                     │
├─────────────────────────────────────────────────┤
│  ConnectWise Manage REST API                 │
│  ├── /company/contacts (auth validation)  │
│  ├── /service/tickets (CRUD)              │
│  └── /finance/invoices (read)              │
└─────────────────────────────────────────────────┘
```

## Authentication Flow

1. User initiates sign-in (Microsoft 365, Google, or email)
2. NextAuth.js processes OAuth/magic-link flow
3. On success, callback includes user email
4. System queries PSA contact by email
5. If contact exists → create JWT with contactId, companyId, companyName
6. If contact missing → redirect to "not registered" error

## Implementation Notes

- **Multi-tenant Azure AD:** Use `https://login.microsoftonline.com/common/v2.0` issuer for any M365 tenant
- **JWT enrichment:** Populate custom claims (contactId, companyId) in NextAuth jwt callback
- **Session storage:** JWT strategy — tokens stored in browser cookie, no server-side session store
- **Rate limiting:** Use rate-limiter-flexible on API routes to prevent abuse
- **Security headers:** Add via Next.js middleware (X-Frame-Options, CSP, etc.)

## Connections

- Related to [[client-portal]] — Pund-IT's implementation of this pattern
- Related to [[oauth-sso-patterns]] — specific SSO provider configuration
- Related to [[connectwise-manage]] — PSA backend integration