# Stripe Automatic Tax in Checkout

Pattern for handling Stripe automatic tax with customer address requirements in Checkout sessions.

## The Problem

When enabling `automatic_tax: { enabled: true }` in Stripe Checkout, Stripe requires **either**:
1. A valid address already on the Customer object
2. Collection of address during checkout

Without this, the API returns:
```
customer_tax_location_invalid: Must have valid ShippingAddress or CustomerDetails address
```

## Solution

Add `customer_update` parameter to collect and save billing address:

```typescript
const checkoutSession = await stripe.checkout.sessions.create({
  customer: customerId,
  mode: "subscription",
  line_items: [{ price: priceId, quantity: 1 }],
  automatic_tax: { enabled: true },
  customer_update: { address: 'auto' },  // ← Collect & save address
  // ...
});
```

The `'auto'` setting tells Stripe to save the billing address entered during checkout to the Customer.

## Handling Stale Customer IDs

When customers have `stripeCustomerId` stored but the customer doesn't exist (e.g., from a different Stripe account or deleted):

```typescript
let customerId: string | null = user.stripeCustomerId;
if (customerId) {
  try {
    await stripe.customers.retrieve(customerId);
  } catch (err: any) {
    if (err?.code === 'resource_missing') {
      // Clear stale ID, create new customer
      customerId = null;
      await prisma.user.update({
        where: { id: userId },
        data: { stripeCustomerId: null },
      });
    } else {
      throw err;
    }
  }
}

if (!customerId) {
  // Create new customer
  const customer = await stripe.customers.create({
    email: user.email,
    name: user.name,
    metadata: { userId: user.id },
  });
  // Save and continue with new customer ID
}
```

## Key Error Codes

| Error Code | Cause | Fix |
|------------|-------|-----|
| `customer_tax_location_invalid` | No address for automatic tax | Add `customer_update: { address: 'auto' }` |
| `resource_missing` | Customer ID doesn't exist in Stripe account | Clear stale ID, recreate customer |

## References

- Stripe docs: [Automatic Tax in Checkout](https://docs.stripe.com/payments/checkout/tax-amounts)
- Incident reference: [Club Sixty Six checkout fix](../projects/ClubSixtySix/INCIDENT-2026-04-10-checkout-fix.md)

---
**tags:** stripe, checkout, tax, payments, patterns
