

---

## [2026-04-18] ingest | Club Sixty Six — Squarespace Billing Migration COMPLETE

**Type:** Data Migration + Feature Deployment — COMPLETED ✅
**Impact:** Critical — 26 migrated members can now complete billing setup
**Reporter:** Olivia McKerrow (billing not working for Squarespace members)

### Problem Statement
Members transferred from Squarespace had database accounts but no Stripe billing:
- ❌ No auto-renewal capability
- ❌ "No billing account" errors when accessing `/account`
- ❌ Members couldn't manage or cancel subscriptions
- ❌ 44 users missing `stripeCustomerId` in database

### Investigation Findings

**Database Analysis:**
| Metric | Count |
|--------|-------|
| Total users | 64 |
| Without Stripe customer | 44 (69%) |
| Without Stripe subscription | 54 (84%) |
| Full billing setup | ~10 |

**Root Cause:** Bulk import script created user records with `membership: "ACTIVE"` but never created corresponding Stripe customers.

### Solution Implemented

**Phase 1: Bulk Customer Creation (26 ACTIVE users only)**
```typescript
// Excluded INACTIVE users — they don't need billing until reactivated
for (const user of activeUsers) {
  const existingCustomers = await stripe.customers.list({ email: user.email });
  if (existingCustomers.length > 0) {
    // Link existing customer
    await prisma.user.update({ 
      where: { id: user.id }, 
      data: { stripeCustomerId: existing.id } 
    });
  } else {
    // Create new Stripe customer
    const customer = await stripe.customers.create({
      email: user.email,
      name: user.name,
      metadata: { 
        userId: user.id, 
        source: "squarespace_migration" 
      },
    });
  }
}
```

**Results:**
- ✅ 26 new Stripe customers created
- ✅ 0 existing customers found (clean slate)
- ✅ 0 errors
- ✅ Rate limited: 100ms delay between API calls
- ✅ All linked to database with metadata tracking

**Phase 2: API Updates**

**`/api/stripe/portal`:**
- Returns `needsBillingSetup: true` for users with customer ID but no subscription
- Returns `hasCustomer: true/false` to differentiate states

**`/api/stripe/checkout`:**
- Modified trial eligibility check:
  ```typescript
  // No trial for migrated active members
  const isBrandNewUser = user.membership !== "ACTIVE" && !user.trialEndsAt;
  ```

**Phase 3: UI Updates**

**Account Page (`/account`):**
- Shows "Billing Setup Required" section for users with `stripeCustomerId` but no subscription
- "Complete Billing Setup" button reveals plan selection (Monthly $29 / Annual $228)
- Post-payment: shows "Manage Billing" button linking to Stripe Customer Portal

### Deployment
- ✅ Database backup: `clubsixty_backup_20260418_155446.sql.gz`
- ✅ Build: Successful (99 static pages)
- ✅ PM2: Online, serving updated code
- ✅ Test account: `wayne@ledrew.org` verified working

### Rollback Procedures
```bash
# Full database restore
zcat backups/clubsixty_backup_20260418_155446.sql.gz | psql ...

# Selective user rollback
cd /opt/clubsixty && npx ts-node scripts/stripe-customer-rollback.ts --email user@example.com
```

### Member Communication
26 members need to complete billing setup:
1. wayne.ledrew@tech-method.com
2. jasmine@ledrew.org
3. amberly.pye@icloud.com
4. elaineler.1107@gmail.com
5. [22 more...] (see full list in project docs)

**Email template drafted** with personalized setup instructions and direct account link.

### Key Lessons
- **Data migration ≠ account creation** — imported users need explicit Stripe onboarding
- **Trial logic matters** — migrated active members shouldn't get free trial
- **State visualization** — UI should clearly show "needs billing setup" vs "active subscription"
- **Metadata tracking** — Stripe customer `source: "squareness_migration"` enables future queries

### Related Concepts
- [Club Sixty Six Infrastructure](wiki/concepts/club-sixty-six.md) — PM2 deployment patterns
- [Stripe Automatic Tax](wiki/concepts/stripe-automatic-tax-checkout.md) — checkout session patterns

---
