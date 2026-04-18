---
title: Pund-IT Client Portal
type: summary
tags: [client-portal, nextjs, connectwise, sso, msp]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

The Pund-IT Client Portal is a Next.js 16 self-service portal enabling clients to view tickets, invoices, and make payments. Built with TypeScript, Tailwind CSS v4, and NextAuth.js v5 for authentication. Multi-tenant ready — supports Microsoft 365, Google Workspace, and magic-link email authentication, all validated against ConnectWise Manage as the source of truth.

## Key Points

- **Stack:** Next.js 16, TypeScript, Tailwind CSS v4, NextAuth.js v5
- **Authentication:** 3 providers — Microsoft Entra ID (Azure AD), Google Workspace, Magic Link Email
- **Source of Truth:** ConnectWise Manage REST API (contacts, tickets, invoices)
- **Multi-tenant:** Environment-driven branding and config
- **JWT Sessions:** Stateless auth, no database required

## Architecture

```
Client Browser
    ↓
Next.js App (portal.pund-it.ca)
    ↓
NextAuth.js (Azure AD / Google / Email)
    ↓
ConnectWise Manage REST API
    ├── Contacts (auth validation)
    ├── Tickets (CRUD + notes)
    └── Invoices (read + line items)
    ↓
Stripe / PayPal (Phase 2)
```

## Key Features (Phase 1 MVP)

- **Dashboard:** Open ticket count, high priority count, unpaid invoices, outstanding balance
- **Tickets:** List view, detail view, create tickets, reply to tickets
- **Invoices:** List view, detail view, PDF export, payment UI (Stripe/PayPal pending)
- **Responsive:** Mobile hamburger menu, stacked layouts

## Environment Variables

| Variable | Description |
|----------|-------------|
| `AUTH_SECRET` | JWT signing key |
| `AUTH_MICROSOFT_ENTRA_ID_ID` | Azure AD app client ID |
| `AUTH_GOOGLE_ID` | Google OAuth client ID |
| `CW_SITE` | ConnectWise API host |
| `CW_PUBLIC_KEY` | CW API public key |
| `CW_PRIVATE_KEY` | CW API private key |

## Project Structure

```
src/
├── app/
│   ├── (portal)/           # Protected portal pages
│   │   ├── dashboard/
│   │   ├── tickets/
│   │   └── invoices/
│   ├── api/               # API routes
│   │   ├── auth/          # NextAuth endpoints
│   │   ├── tickets/
│   │   └── invoices/
│   └── login/
├── components/
└── lib/
    ├── auth.ts            # NextAuth config
    └── connectwise.ts     # CW API client
```

## Connections

- Related to [[tech-method-portal]] — portal pattern this project implements
- Related to [[oauth-sso-patterns]] — authentication providers used
- Related to [[connectwise-manage]] — backend API source of truth