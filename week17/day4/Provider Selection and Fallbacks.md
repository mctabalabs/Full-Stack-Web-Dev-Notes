# Week 17, Day 4: Provider Selection and Fallbacks

By the end of today, customers see the right payment method at the right time: Kenyan phones default to M-Pesa, international visitors see Stripe, Airtel users see Airtel. And when a provider goes down, the shop degrades gracefully instead of showing an error screen.

**Prior-week concepts you will use today:**
- The unified payments interface (Week 17, Day 3)
- The checkout payment form (Week 15, Day 2)
- IP geolocation (new today)

**Estimated time:** 2-3 hours

---

## Picking a Default Based on the Customer

You have three providers. The customer should rarely have to think about which one to pick. Give them the right default and let them change it.

### Phone number heuristic

```javascript
// app/lib/providerPicker.js
export function pickDefaultProvider({ phone, country }) {
  // Stripe for non-Kenyan customers
  if (country && country !== "KE") return "stripe";

  // Phone-based for Kenya
  if (phone?.startsWith("+254") || phone?.startsWith("254")) {
    const prefix = phone.replace("+", "").slice(3, 6);
    // Airtel prefixes: 0732-0739, 0750-0756, 0780-0789, etc.
    const airtelPrefixes = ["73", "75", "78"];
    const firstTwo = prefix.slice(0, 2);
    if (airtelPrefixes.includes(firstTwo)) return "airtel";
  }

  return "mpesa";
}
```

This is crude -- phone prefixes map to original carriers, and plenty of people have ported. It is a reasonable default, not a guarantee. Always let the customer override.

### Country detection

For the country, you can use the `x-forwarded-for` header with a free IP geolocation library. Install:

```bash
npm install geoip-lite
```

In a Server Component:

```javascript
import { headers } from "next/headers";
import geoip from "geoip-lite";

export default async function CheckoutPaymentPage() {
  const headersList = headers();
  const ip = headersList.get("x-forwarded-for")?.split(",")[0] || "";
  const geo = geoip.lookup(ip);
  const country = geo?.country || "KE";

  return <PaymentForm defaultCountry={country} />;
}
```

`geoip-lite` is a 35 MB database bundled into your deployment. Small price for working offline geolocation. For production-scale apps use a dedicated service like MaxMind or ipapi.co, but for a Kenyan shop the free library is plenty.

---

## The New Payment Form

Update the payment form to use the picker:

```jsx
// app/checkout/payment/PaymentForm.js -- key changes

const [method, setMethod] = useState(() =>
  pickDefaultProvider({ phone: info.phone, country: defaultCountry })
);

// Sort options to put the default first
const METHOD_OPTIONS = [
  { value: "mpesa", label: "M-Pesa", description: "Push prompt to your phone" },
  { value: "airtel", label: "Airtel Money", description: "USSD prompt to your phone" },
  { value: "stripe", label: "Card (Visa/Mastercard)", description: "Secure checkout with Stripe" },
  { value: "cod", label: "Cash on Delivery", description: "Pay our rider on arrival" },
];
```

Render the options with the recommended one highlighted:

```jsx
<fieldset className="space-y-2">
  <legend className="font-medium">Payment method</legend>
  {METHOD_OPTIONS.map((opt) => (
    <label key={opt.value} className="flex items-center gap-3 border rounded p-3 cursor-pointer">
      <input
        type="radio"
        name="method"
        value={opt.value}
        checked={method === opt.value}
        onChange={() => setMethod(opt.value)}
      />
      <div className="flex-1">
        <div className="font-medium">{opt.label}</div>
        <div className="text-xs text-gray-500">{opt.description}</div>
      </div>
      {opt.value === recommendedMethod && (
        <span className="text-xs bg-brand-accent text-white px-2 py-0.5 rounded">
          Recommended
        </span>
      )}
    </label>
  ))}
</fieldset>
```

Customers see their recommended method first with a badge, but can still click any option. This is the right balance between helpfulness and autonomy.

---

## Graceful Degradation

Payment providers go down. M-Pesa has maintenance windows. Airtel has API outages. Stripe once had a 90-minute global incident. A shop that shows "Payment failed, please try again later" during these times loses sales.

Better: detect the outage, hide the broken provider, and route customers to an alternative.

### Health checks

Add a simple health check to each provider:

```javascript
// server/services/payments/mpesaProvider.js
async function healthCheck() {
  try {
    // Cheap operation: get a token
    const token = await getDarajaToken();
    return { healthy: !!token };
  } catch {
    return { healthy: false };
  }
}

module.exports = { initiate, handleCallback, queryStatus, healthCheck };
```

Do the same for Airtel (try fetching a token) and Stripe (`stripe.accounts.retrieve()` with the account id -- works as long as Stripe's API is up).

### Aggregate health endpoint

```javascript
// server/routes/payments.routes.js
router.get("/health", asyncHandler(async (req, res) => {
  const payments = require("../services/payments");
  const methods = ["mpesa", "airtel", "stripe"];
  const results = await Promise.all(
    methods.map(async (m) => {
      const provider = payments.getProvider(m);
      const check = await provider.healthCheck();
      return [m, check.healthy];
    })
  );
  res.json(Object.fromEntries(results));
}));
```

Now the shop knows. Call it from a Server Component on the payment page:

```jsx
async function getHealthyProviders() {
  const res = await fetch(`${process.env.CRM_SERVER_URL}/api/payments/health`, {
    cache: "no-store",
  });
  return res.ok ? res.json() : { mpesa: true, airtel: true, stripe: true };
}

export default async function CheckoutPaymentPage() {
  const health = await getHealthyProviders();
  return <PaymentForm health={health} defaultCountry={/*...*/} />;
}
```

And in the form, hide or disable broken providers:

```jsx
{METHOD_OPTIONS.filter((opt) => opt.value === "cod" || health[opt.value]).map(...)}
```

COD always shows (it does not depend on an API). Digital providers show only when healthy. If M-Pesa is down, M-Pesa is invisible on the form; customers who would have defaulted to M-Pesa now default to Airtel or Stripe. Silent degradation.

### Admin visibility

Add a small status bar to the admin dashboard:

```jsx
<div className="flex gap-3 text-xs">
  <span className={health.mpesa ? "text-green-600" : "text-red-600"}>
    M-Pesa {health.mpesa ? "up" : "down"}
  </span>
  <span className={health.airtel ? "text-green-600" : "text-red-600"}>
    Airtel {health.airtel ? "up" : "down"}
  </span>
  <span className={health.stripe ? "text-green-600" : "text-red-600"}>
    Stripe {health.stripe ? "up" : "down"}
  </span>
</div>
```

The admin sees at a glance whether they should be chasing Safaricom support. In real life this detail has saved customers from three-hour "why can no one pay?" panics.

---

## Caching Health Results

Hitting the provider APIs on every page load is wasteful. Cache the health check in Redis for 30 seconds:

```javascript
const { getClient } = require("../config/redis");

router.get("/health", asyncHandler(async (req, res) => {
  const redis = await getClient();
  const cached = await redis.get("payments:health");
  if (cached) return res.json(JSON.parse(cached));

  const methods = ["mpesa", "airtel", "stripe"];
  const results = {};
  for (const m of methods) {
    const check = await payments.getProvider(m).healthCheck();
    results[m] = check.healthy;
  }

  await redis.set("payments:health", JSON.stringify(results), { EX: 30 });
  res.json(results);
}));
```

30 seconds is aggressive enough to catch most outages but lets the cache absorb most traffic. For really high traffic, consider a background job that updates the cache on a schedule -- Week 21 territory.

---

## Currency Display

Stripe is charging in KES (Kenyan Shillings), but a customer in Dubai may want to see USD. Convert at display time, not at payment time:

```javascript
async function convertKshToUsd(ksh) {
  // For the Marathon, hardcode a rate. In production, use a live FX API.
  const rate = 130; // KSh per USD (roughly)
  return (ksh / rate).toFixed(2);
}
```

On the product detail page:

```jsx
<p>KSh {(p.price_cents / 100).toLocaleString()}</p>
{country !== "KE" && (
  <p className="text-sm text-gray-500">
    About USD {convertKshToUsd(p.price_cents / 100)}
  </p>
)}
```

This is only a display hint. The actual charge is always in KES because that is what your bank account holds. Showing the USD estimate helps foreign customers decide whether a KSh 2,499 product is worth buying.

---

## Checkpoint

1. Kenyan phone in the info step -> payment page defaults to M-Pesa with a "Recommended" badge.
2. Phone starting `+25473...` -> defaults to Airtel.
3. International IP -> defaults to Stripe.
4. Killing M-Pesa (set consumer key to garbage and restart) -> health endpoint returns `mpesa: false` -> payment form hides the M-Pesa option.
5. Admin dashboard shows provider status at the top.
6. Health check result is cached in Redis for 30 seconds.
7. Checkout for a non-Kenyan visitor shows a USD estimate under the KSh price.

Commit:

```bash
git add .
git commit -m "feat: provider selection fallbacks and health checks"
```

---

## What You Learned

- Smart defaults beat blank radio groups.
- Health checks hide broken options; they do not show errors.
- Cached health endpoints protect the provider APIs from being hammered.
- Display currency is a UX hint, not a charge decision.

Tomorrow is the recap, and the weekend adds payments UX polish to Project 3. Week 18 digs into the security and correctness of the whole payments layer.
