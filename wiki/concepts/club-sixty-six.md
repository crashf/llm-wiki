# Club Sixty Six

Subscription fitness platform for [Olivia McKerrow](https://clubsixty.ca), built on Club Sixty Six.

## Overview
- **URL:** https://clubsixty.ca
- **VM:** 170.205.18.121 (Ubuntu, OpenClaw managed)
- **SSH:** `~/.ssh/id_clubsixty`, user: `adrianna`
- **Stack:** Next.js (standalone), PostgreSQL, PM2, Nginx

## Infrastructure

### VM Access
| Detail | Value |
|---|---|
| Host | 170.205.18.121 |
| SSH | `ssh -i ~/.ssh/id_clubsixty adrianna@170.205.18.121` |
| adrianna password | `C66_Secure_Vm_2026!` |
| clubsixty password | `C66_Secure_Vm_2026!` |

### Deployment
- **Path:** `/opt/clubsixty`
- **Process manager:** PM2 as `clubsixty` user — always use `sudo -u clubsixty pm2 <cmd>`
- **Restart:** `sudo -u clubsixty pm2 restart clubsixty`
- **Deploy command:** `cd /opt/clubsixty && rm -rf .next && npm run build && cp -r public .next/standalone/ && cp -r .next/static .next/standalone/.next/ && sudo -u clubsixty pm2 restart clubsixty`

### Environment
- **Config:** `/opt/clubsixty/.env`
- **Secrets stored:** `.secrets/stripe-*`, `.secrets/resend-*`, `.secrets/cpanel-clubsixty-*`, `.secrets/squarespace-*`
- **Database:** PostgreSQL, user=`clubsixty`, db=`clubsixty`, pw=`C66_db_2026_Secure!`, connect via `-h 127.0.0.1`

## Integrations

### Stripe
- **Account:** Olivia McKerrow (live mode)
- **Stripe Secret Key:** `[REDACTED]` (standard, permanent)
- **Publishable Key:** `pk_live_51O6wgGDBjdJd16kq1jinoHzK7aouDkSK2NdT85KWXlBeg7VLF6CkBIJTVZRrJtP0kHsp5fzSB8jpltOBsiQQygCB00W57jqB1y`
- **Products:** Annual (`price_1THKruDBjdJd16kqutZsVGEZ`), Monthly (`price_1THKrtDBjdJd16kq7OpRzLmI`)
- **Webhook:** Active at Stripe

> ⚠️ **Stripe restricted key issue:** Restricted keys (`rk_live_`) in Olivia's dashboard can have **forced 7-day expiration**. Always use standard `[REDACTED]` keys — they are permanent and avoid rotation cycles.

### Email
- **Transactional:** Resend (join/olivia/info/support @clubsixty.ca)
- **Inbound:** cPanel mail → `joinclubsixty@gmail.com`
- **DNS:** cPanel on tech-method.com (not clubsixty.ca:2083)

### Domain
- Registrar: GoDaddy
- DNS: cPanel on tech-method.com
- Domain: clubsixty.ca

## Incident History

### 2026-04-10: "Failed to start checkout" on membership renewal
**Issue:** Users with `FREE_TRIAL` status received "Failed to start checkout" error when attempting to renew.

**Root causes:**
1. Missing `customer_update: { address: 'auto' }` for automatic tax (requires billing address)
2. Stale `stripeCustomerId` values from old test environment

**Fix:** Updated checkout route to collect address and validate customer IDs before use.

**Full details:** [INCIDENT-2026-04-10-checkout-fix.md](../../projects/ClubSixtySix/INCIDENT-2026-04-10-checkout-fix.md)  
**Pattern:** [[stripe-automatic-tax-checkout]] — general pattern for this issue

## Troubleshooting

### "Something went wrong" during checkout
**Symptoms:** User sees generic error on checkout page, StripeAuthenticationError in server logs.

**Cause:** Expired Stripe API key in `.env`.

**Fix:**
1. Get fresh `[REDACTED]` key from Stripe Dashboard → Developers → API Keys → Create secret key
2. `ssh -i ~/.ssh/id_clubsixty adrianna@170.205.18.121`
3. `echo 'C66_Secure_Vm_2026!' | sudo -S sed -i 's/^STRIPE_SECRET_KEY=.*/STRIPE_SECRET_KEY=<NEW_KEY>/' /opt/clubsixty/.env`
4. `sudo -u clubsixty pm2 restart clubsixty`

### Server Action Errors
**"Failed to find Server Action"** — caused by browser caching an old deployment. Clear browser cache or do a fresh deploy.

## Related
- [[OpenClaw Bot Debugging]] — same SSH patterns
- [[openclaw-fleet-management]] — OpenClaw fleet context
