---

## [2026-04-19] ingest | Summer 2026 RV Camping Trip — Complete Route Planning

**Type:** Travel Planning + Logistics — COMPLETED ✅
**Impact:** Personal — 91-day cross-country RV trip fully planned and documented
**Reporter:** Wayne LeDrew

### Trip Overview

**91-day RV adventure** across 4 Canadian provinces with Jasmine:
- **Start:** May 28, 2026 (Kitchener, ON)
- **End:** August 26, 2026 (Hamilton, ON)
- **Distance:** ~8,683 km
- **Vehicle:** Diesel truck + 30ft travel trailer
- **Total stops:** 21 campgrounds + 7 travel stops

### Route Summary

**Outbound:** Kitchener → Sault Ste Marie → Terrace Bay → Dryden → Manitoba → Saskatchewan → Alberta
**Prairie Loop:** Regina → Cypress Hills → Waterton → Bow Valley → Banff → Lake Louise → William Switzer → Miquelon Lake → Vermilion → Saskatchewan Landing → Manitoba
**Return:** Asessippi → Grand Beach → Quetico → Marathon → Sault Ste Marie → Piotr's Cottage → Hamilton

### Reservations Confirmed (10 bookings)

| Park | Dates | Reservation # |
|------|-------|---------------|
| Cypress Hills PP | Jun 3-7 | 2-1353345 |
| Crooked Creek Campground | Jun 7-14 | CRO-26-1082 |
| Bow Valley PP | Jun 14-18 | 2-1357950 |
| Tunnel Mountain Village II | Jun 18-26 | INPC25-15795597B1 |
| Lake Louise Hard-Sided | Jun 26-Jul 4 | INPC25-15803320B2 |
| William Switzer PP | Jul 4-10 | 2-1378073 |
| Miquelon Lake PP | Jul 10-17 | 2-1383600 |
| Vermilion PP | Jul 17-24 | INMB26-83429B1 |
| Saskatchewan Landing PP | Jul 24-31 | 2-2044209 |
| Asessippi PP | Jul 31-Aug 4 | INMB26-289278B1 |
| Grand Beach PP | Aug 4-7 | INMB26-83429B1 |
| Quetico PP | Aug 7-14 | INOP25-18658994B1 |

**All reservations confirmed** — RVlife export imported and cross-checked.

### Fuel Planning — Diesel Route

**50 exact fuel stops** mapped from Kitchener to Hamilton:
- **Critical stops:** Wawa, Terrace Bay, Pincher Creek, Canmore (avoid Banff prices!), Saskatoon, Atikokan
- **Budget:** ~$3,000-3,500 CAD for ~2,042L diesel
- **Longest legs:** SK Landing → Asessippi (586km), Grand Beach → Quetico (636km)

**Price strategy by province:**
- Alberta: Cheapest (~$1.50-1.65/L) — fill big here
- Saskatchewan: Cheap (~$1.55-1.70/L)
- Manitoba: Moderate (~$1.60-1.75/L)
- Ontario: Most expensive (~$1.75-1.95/L)
- Banff/Lake Louise: AVOID (~$1.90-2.10/L)

### Key Stops

**Longest stays:**
1. Piotr's Cottage (Georgian Bay) — 10 nights (Aug 16-26)
2. Tunnel Mountain Village II (Banff) — 8 nights
3. Lake Louise Hard-Sided — 8 nights

**Walmart overnights:** 7 nights (Sault Ste, Dryden, Portage la Prairie x2, Regina)

### Deliverables Created

**Project folder:** `projects/camping-2026/`
- `README.md` — Project overview
- `itinerary.md` — Complete day-by-day schedule
- `reservations.md` — All booking reference numbers
- `diesel-stops.md` — Exact station names & addresses (50 stops)

**Memory files:**
- `memory/summer-2026-camping-trip.md` — Full itinerary
- `memory/summer-2026-camping-trip-fuel-plan.md` — Strategic fuel planning
- `memory/summer-2026-camping-trip-diesel-stops.md` — Exact stops

**Wiki:**
- New concept page: `camping-2026-trip.md` — Trip summary for reference
- Updated index with new concept entry

### Pre-Trip Checklist (Upcoming)

- [ ] Vehicle inspection (diesel truck + trailer)
- [ ] DEF level check
- [ ] Packing list from previous trips
- [ ] Offline maps downloaded
- [ ] Confirm reservation emails printed/saved

### Related

- Concept page: [camping-2026-trip](./wiki/concepts/camping-2026-trip.md)
- Project: `projects/camping-2026/`
- Memory: `memory/summer-2026-camping-trip*.md`

---

## [2026-04-19] ingest | Anzen Egress + MikroWizard Features Integration Planning

**Type:** Architecture Planning + Documentation — COMPLETED ✅
**Impact:** Strategic — 6-week roadmap for operational capabilities in Anzen Egress portal
**Reporter:** Wayne LeDrew (CTO, Pund-IT)

### Work Completed

Reviewed [MikroWizard](https://github.com/MikroWizard/mikroman) (MikroTik management platform) for feature integration into Anzen Egress.

**Analysis of 7 operational features:**
| Feature | Decision | Rationale |
|---------|----------|-----------|
| Device Health Metrics | ✅ Add | CPU/Memory visibility per deployment for troubleshooting |
| Config Backup & Restore | ✅ Add | Rollback safety before upgrades |
| Firmware Management (Safe Updates) | ✅ Add | Security risk mitigation with auto-rollback |
| Syslog Viewer | ✅ Add | Per-deployment log aggregation |
| Scheduled Maintenance Tasks | ✅ Add | Automated daily backups, weekly checks |
| Diagnostics Terminal | ✅ Add | Safe command execution (ping, traceroute, etc.) |
| NOC Dashboard (fleet-wide) | ❌ Skip | Out of scope — use Grafana for 24/7 monitoring |
| Device Groups & Batch Ops | ❌ Skip | Egress is 1:1 customer:CPE, no bulk ops needed |
| RADIUS Server | ❌ Skip | Not applicable to egress tunnels |

### Deliverables Created

1. **MIKROWIZARD-INTEGRATION-MINIMAL.md** (13 KB)
   - 6-week implementation plan
   - 6 phases: Health → Syslog → Backups → Firmware → Tasks → Diagnostics
   - 8 new database tables
   - Scope: Per-deployment only (no fleet-wide features)

2. **MIKROWIZARD-FEATURES-PLAN.md** (28 KB)
   - Full 10-week comprehensive plan (for reference)
   - Complete schema, API endpoints, UI structure
   - 20+ tables, NOC dashboard, fleet operations

3. **TODO.md** (4.6 KB)
   - Task breakdown with checkboxes
   - API endpoint mapping
   - Database schema summary

4. **Wiki Updated**
   - New concept page: `anzen-egress-mikrowizard.md`
   - Added to wiki index

### Implementation Timeline

| Phase | Feature | Duration |
|-------|---------|----------|
| 1 | Device Health Metrics | Week 1 |
| 2 | Syslog Viewer | Week 1 |
| 3 | Config Backup & Restore | Week 2 |
| 4 | Firmware Management | Week 2 |
| 5 | Scheduled Tasks | Week 1 |
| 6 | Diagnostics Terminal | Week 1 |
| | Integration & Polish | Week 1 |
| **Total** | | **6 weeks** |

### Database Schema Summary

**New tables (8 total):**
- `device_health_metrics` — CPU/Memory snapshots
- `device_syslog` — Log aggregation
- `device_backups` — Config backups
- `firmware_repository` — Cached firmware versions
- `device_firmware_status` — Current vs available
- `firmware_upgrade_history` — Upgrade attempts
- `scheduled_tasks` — Recurring maintenance
- `task_executions` — Run history

### Git Operations

- **Anzen Egress Portal:** Pushed 3 new markdown files to `crashf/anzen-egress-portal`
- **LLM Wiki:** Synced concept page to `crashf/llm-wiki`
- **Live Server:** Copied TODO.md to punsvndocker02 (10.255.71.48)

### UI Changes

**Deployment Detail page gets 5 new tabs:**
```
[Overview] [Health] [Logs] [Backups] [Firmware] [Tasks] [Diagnostics] [History]
```

### Key Decisions

- **Keep scope minimal:** Added 6 features, rejected 4 (fleet-wide features)
- **Per-deployment only:** No global NOC dashboard, no bulk operations
- **Safe by default:** Firmware upgrades have automatic rollback on failure
- **6 weeks vs 10 weeks:** Saved 4 weeks by cutting fleet-wide features

### Related Concepts

- [Anzen Egress Infrastructure](wiki/concepts/anzen-egress-mikrowizard.md) — Full integration details
- [Anzen Egress GitHub](https://github.com/crashf/anzen-egress-portal) — Portal repo

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