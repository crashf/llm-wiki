---
title: OAuth SSO Patterns
type: concept
tags: [oauth, sso, azure-ad, google, authentication, security]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

OAuth SSO Patterns for MSP client portals support multiple authentication providers — Microsoft Entra ID (Azure AD), Google Workspace, and email magic links. All flows validate user identity against the PSA (ConnectWise) as authorization gatekeeper, not the identity provider.

## Key Points

- **Multi-provider:** Support Microsoft 365, Google, and email — user chooses preferred method
- **PSA validation:** Identity provider proves identity; PSA proves authorization
- **Multi-tenant:** Azure AD configured for any tenant (common endpoint), not single-org
- **JWT sessions:** Stateless authentication with custom claims from PSA

## Provider Configuration

### Microsoft Entra ID (Azure AD)

```typescript
MicrosoftEntraID({
  clientId: process.env.AUTH_MICROSOFT_ENTRA_ID_ID,
  clientSecret: process.env.AUTH_MICROSOFT_ENTRA_ID_SECRET,
  issuer: process.env.AUTH_MICROSOFT_ENTRA_ID_ISSUER,
  authorization: { params: { scope: "openid email profile User.Read" } },
})
```

**Multi-tenant setup:**
- Issuer: `https://login.microsoftonline.com/common/v2.0`
- Register app in Azure Portal > App registrations ( multitenant)
- Enable ID tokens + Access tokens
- Redirect URI: `{PORTAL_URL}/api/auth/callback/microsoft-entra-id`

**Permissions needed:**
- `openid` — ID token
- `email` — user email
- `profile` — user name
- `User.Read` — get user profile (optional)

### Google Workspace

```typescript
Google({
  clientId: process.env.AUTH_GOOGLE_ID,
  clientSecret: process.env.AUTH_GOOGLE_SECRET,
})
```

**Setup:**
- Create OAuth client ID in Google Cloud Console
- Authorized redirect URI: `{PORTAL_URL}/api/auth/callback/google`
- Scopes requested by default: `email`, `profile`, `openid`

### Email Magic Link

```typescript
Credentials({
  id: "magic-link",
  name: "Email",
  credentials: {
    email: { label: "Email", type: "email" },
    token: { label: "Token", type: "text" },
  },
  async authorize(credentials) {
    // Validate token, return user
    return { id, name, email };
  },
})
```

**Flow:**
1. User enters email on login page
2. System generates signed token, sends via SMTP
3. User clicks link with token
4. Token validated, session created

## PSA Integration

The critical piece: after OAuth success, validate against PSA contact:

```typescript
async signIn({ user }) {
  if (!user.email) return false;
  const contact = await findContactByEmail(user.email);
  if (!contact) return "/login?error=not_registered";
  return true;
}

async jwt({ token, user }) {
  if (user?.email) {
    const contact = await findContactByEmail(user.email);
    if (contact) {
      token.contactId = contact.id;
      token.companyId = contact.company?.id;
      token.companyName = contact.company?.name;
    }
  }
  return token;
}
```

## Security Considerations

- **Multi-tenant CORS:** Restrict API origins in production
- **Session hardening:** Use `AUTH_SECRET` for JWT signing (32+ chars)
- **Rate limiting:** Apply to login endpoints
- **Security headers:** X-Frame-Options, CSP, Referrer-Policy via middleware

## Connections

- Related to [[client-portal]] — uses these OAuth providers
- Related to [[tech-method-portal]] — portal pattern using SSO
- Related to [[connectwise-manage]] — PSA for contact validation