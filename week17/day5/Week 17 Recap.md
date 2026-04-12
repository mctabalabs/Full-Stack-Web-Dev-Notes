# Week 17, Day 5: Week Recap

Three payment providers, one interface, smart routing, graceful degradation. This week's work is invisible to most customers but is the thing that lets a shop stay open when one provider is having a bad day.

---

## What You Built

1. **Stripe Checkout** integration with signed webhooks and `event.id` idempotency.
2. **Airtel Money** with token caching, initiate, callback, status query.
3. **Unified payments interface** -- one shape, three implementations, registry pattern.
4. **Deprecated legacy routes** kept working for backwards compatibility.
5. **Provider selection** by phone prefix and IP geolocation.
6. **Health checks** with Redis caching.
7. **Graceful degradation** when providers are down.
8. **Currency display** for international visitors.

---

## Self-Review Questions

**Stripe**
1. Why do we use Stripe Checkout (hosted) instead of Stripe Elements (embedded)?
2. What is `metadata.order_id` for on a Stripe session?
3. How do you verify a Stripe webhook signature, and what happens if the signature is wrong?
4. What is the `webhook_events` table for?

**Airtel**
5. Why is token caching crucial for Airtel?
6. How do Airtel status codes `TS`, `TF`, and `TIP` map to our `payment_status` column?
7. What makes Airtel's signature verification harder than Stripe's?

**Unified interface**
8. What three functions does every provider module expose?
9. How does the unified interface reduce the cost of adding a new provider?
10. Why are legacy M-Pesa routes still kept even after the refactor?

**Selection and fallback**
11. What does `pickDefaultProvider` use to decide a default?
12. What happens on the payment form when M-Pesa is down?
13. Why is the health check cached in Redis?
14. Why is the USD conversion display-only, not used for charging?

Target: 12/14 correct.

---

## Peer Coding Session

### Track A: Paystack as a fourth provider

Add Paystack (https://paystack.com). Same three functions. Same registry. Zero changes to `createOrder`. This is the ultimate test of your abstraction.

### Track B: Fee display per provider

Add a small fee line on the payment page: "M-Pesa charges KSh 0", "Stripe charges 3% + KSh 10". Let customers see the cost before picking. Bonus: charge them the fee (not just display it).

### Track C: A/B test the default

Log which provider each customer picked and which was recommended. Generate a simple report: what percentage override the default, which provider they switch to. This is how real e-commerce teams tune UX decisions.

### Track D: Refund testing

Each provider has a different refund API. Build three small scripts that refund a test transaction on each provider. Log the full round-trip to a file. Week 18 we build refunds properly; this session is scouting.

---

## Weekend Prep

This weekend is lighter -- Project 3 was last week, and Project 4 (Chama Savings) is Week 22. The weekend project this week is a payments harness: a small admin page where you can initiate test payments to each provider, see their responses in structured form, and verify the full round-trip works.

Details in `week17/weekend/project.md`.

Next week is the hardest week of Phase 3: webhook security, idempotency at every layer, refunds, and reconciliation. You will look at every line of payment code you have written and make it bullet-proof.
