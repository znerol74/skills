# Stripe Checkout: what to check and how to build it

The buy button usually creates a Checkout Session on the server and redirects to it. Everything
below is a parameter on that session or a webhook around it. Check the Stripe SDK version and the API
version the project pins before using a parameter; newer parameters fail on older API versions.

## Contents
1. Audit checklist
2. Trials
3. Branding when one Stripe account sells several products
4. Attribution: which moment led to the purchase
5. Webhooks
6. Example session (TypeScript)

## 1. Audit checklist

| Check | Why |
|---|---|
| `payment_method_types` is **not** hardcoded to `['card']` | Leaving it out lets the dashboard's dynamic payment methods show wallets (Apple Pay, Google Pay, Link) and local methods. A hardcoded card list hides them. |
| `customer` or `customer_email` is passed | Prefilled email, and with `customer` saved cards. One less field. |
| `client_reference_id` holds the user id | The webhook can grant access without guessing by email. |
| `locale` matches the app's language | Default is the browser's language; if the app has its own language setting, pass it. |
| `cancel_url` returns to where the user was, with the offer reopened | A cancel that lands on the homepage loses the person. |
| `success_url` shows what to do with the purchase right now | The first use of the paid feature is the best moment to confirm the value. |
| `allow_promotion_codes` only when codes are actually in use | A visible code field sends people off to search for codes. |
| Only the fields that are needed (`billing_address_collection`, `phone_number_collection`, `tax_id_collection`) | Every field costs completions. Tax may require the address; nothing else should. |
| `automatic_tax` matches the tax setup | It requires Stripe Tax to be configured. |
| `consent_collection.terms_of_service: 'required'` if terms must be accepted | Needs a terms URL in the dashboard's public business details. |
| `custom_text.submit.message` repeats the reassurance | E.g. "No charge until 14 Oct. Cancel anytime in Settings." Keep it true and short. |
| A trial, when offered, is configured as in section 2 | |
| Branding fits the product (section 3) | A checkout that looks like a different product looks like a scam. |
| The session carries attribution metadata (section 4) | Otherwise the funnel ends at "checkout started". |

`ui_mode: 'embedded'` (Checkout inside the page) is an option when leaving the site loses people;
it costs more integration work and there's no strong evidence it converts better. Low priority.

## 2. Trials

Card-required trial on a subscription:

```ts
mode: "subscription",
subscription_data: {
    trial_period_days: 7,
    metadata: { user_id, plan: "monthly", context },
},
// payment_method_collection defaults to "always" in subscription mode: the card is taken up front.
```

Card-free trial (more starts, many expire without paying):

```ts
payment_method_collection: "if_required",
subscription_data: {
    trial_period_days: 7,
    trial_settings: { end_behavior: { missing_payment_method: "cancel" } },
},
```

Rules that matter:
- **Eligibility is your job.** Offer the trial only to accounts that never had paid access, decided
  on the server when the session is created. Stripe will happily give the same person a second trial.
- **Trials are not sales.** A trial start creates a $0 checkout; count it separately from purchases
  everywhere (admin stats, analytics), or the numbers lie.
- **Access during the trial** is granted like a normal subscription, and revoked by the subscription
  webhook if it ends unpaid.
- **The reminder.** If the paywall timeline promises one, send it. `customer.subscription.trial_will_end`
  fires three days before the trial ends and is the natural trigger for your own email. Stripe's
  billing email settings also have a free-trial reminder option; check what the dashboard currently
  offers before building your own. Don't send both.
- **Settings page.** Show "Free until <date>, then <price>" and a direct way to cancel (the Stripe
  customer portal works). An account in a trial should never wonder whether it will be charged.
- **Copy.** Stripe's page already states the trial ("7 days free, then $4.99 per month"); don't
  contradict it in `custom_text`.

## 3. Branding when one Stripe account sells several products

Account branding (logo, colors) is global. When one account sells several products, newer API
versions accept per-session `branding_settings` (display name, icon or logo, background and button
colors, border style, font). Wrap it so an older API version or a refusal falls back to an unbranded
session instead of breaking the purchase:

```ts
async function createBranded(params, branding) {
    try {
        return await stripe.checkout.sessions.create({ ...params, branding_settings: branding });
    } catch (err) {
        if (isBrandingRefusal(err)) return stripe.checkout.sessions.create(params);
        throw err;
    }
}
```

Receipts, invoices and the customer portal keep the account's look; say so to the user.

## 4. Attribution: which moment led to the purchase

Pass the moment through the whole path:
- Client event `checkout_started` with `{ plan, trial, surface, context }`.
- The same `surface` / `context` in the session `metadata` (and `subscription_data.metadata` or
  `payment_intent_data.metadata`).
- The webhook writes the purchase with those fields. That row is the source of truth for "paid",
  not a client event, which can be blocked or lost on redirect.

## 5. Webhooks

| Event | Use |
|---|---|
| `checkout.session.completed` | Grant access; record the purchase (or the trial start) with attribution. |
| `invoice.paid` | Renewals; the first paid invoice after a trial is the trial converting. |
| `customer.subscription.trial_will_end` | Reminder email, three days before the trial ends. |
| `customer.subscription.updated` / `.deleted` | Cancel at period end, end of access. |
| `charge.refunded` | Revoke if the product promises that. |

Make handlers idempotent (the same event can arrive twice) and verify signatures.

## 6. Example session (TypeScript)

```ts
const lifetime = plan === "lifetime";
const trialDays = !lifetime && (await neverHadPaidAccess(userId)) ? 7 : 0;

const session = await createBranded(
    {
        mode: lifetime ? "payment" : "subscription",
        line_items: [{ price: lifetime ? PRICE_LIFETIME : PRICE_MONTHLY, quantity: 1 }],
        customer_email: email,
        client_reference_id: userId,
        locale,
        metadata: { user_id: userId, plan, context },
        ...(lifetime
            ? { payment_intent_data: { metadata: { user_id: userId, plan, context } } }
            : {
                  subscription_data: {
                      metadata: { user_id: userId, plan, context },
                      ...(trialDays ? { trial_period_days: trialDays } : {}),
                  },
              }),
        success_url: `${origin}/welcome-pro?session_id={CHECKOUT_SESSION_ID}`,
        cancel_url: `${origin}${returnPath}?offer=open`,
    },
    PRODUCT_BRANDING
);
```
