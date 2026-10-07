---
name: checkout-optimizer
description: >
  Audit and improve how a web product turns interested users into paying customers: when and where
  the upgrade offer or paywall appears, how the paywall or upgrade popup is built, the Stripe Checkout
  session behind the buy button, and the funnel tracking from "offer shown" to "paid". Produces a
  ranked list of changes with reasons, puts business and legal choices (trials, reference prices,
  public user counts) in front of the user, then builds what they approve. Use this whenever someone
  wants more purchases, upgrades or trial starts, says nobody buys or conversion is low, asks to
  optimize, review or rework a paywall, pricing modal, upgrade popup, plan picker, Stripe checkout or
  "Kaufweg", shares paywall research (a video, case studies, Superwall/RevenueCat/Mobbin examples), or
  writes things like "checkout optimieren", "Paywall verbessern", "mehr Käufe", "warum kauft keiner",
  even when the word "checkout" never appears.
---

# Checkout optimizer

In a freemium web product the money is made on a short path: the user meets an offer, opens it,
picks a plan, gets through the Stripe page and pays. This skill improves that path in three places:

1. **The offer**: when it appears, what it says, how it is built (paywall, upgrade popup, pricing modal).
2. **The Stripe Checkout page**: the session the buy button creates.
3. **The measurement**: events from "offer shown" to "paid", so the next change is less of a guess.

Work in two phases: analyse and propose, then build what the user approves. Business and legal
choices (a trial, struck-through prices, a guarantee, a public user count) belong to the user. Put
them in front of the user with a recommendation instead of deciding silently, because they carry
revenue and legal consequences you can't see from the code.

## Phase 1: Understand the product and its numbers

Judge nothing before you know what is there. Read the code; don't assume a stack.

- **Where money changes hands.** Search for `checkout.sessions.create`, `stripe`, `price_`,
  `subscription`, `paywall`, `upgrade`, `pricing`, `premium`, `pro`, `unlock`, `plan`. Find the route
  that creates the session, the webhook that grants access (`checkout.session.completed`,
  `invoice.paid`, `customer.subscription.*`) and where access is stored.
- **Every place the offer appears.** Buttons, popups, locked features, pricing page, settings, emails
  with a buy link. For each: what opens it, what it shows, which plan is preselected, the copy.
- **The plans.** Prices, intervals, one-time vs recurring, trials, guarantees, refunds, the default
  plan. Read the real values from code or config, not from marketing pages.
- **The value moment.** What does a user do right before they would think "I want more of this"?
  Finished a read, exported a file, got a result, hit a limit, reached for a premium surface. This is
  the most important thing to understand, and it is different in every product.
- **What is measured today.** Analytics calls (`trackEvent`, `gtag`, `posthog`, `plausible`,
  `mixpanel`), admin dashboards, purchase tables.
- **The numbers**, where they can be read safely: signups per day, active users, offer views,
  checkouts started, purchases. Read-only queries are fine; never write to production data. If there
  are no numbers, say so and estimate the volume from what you can see. Volume decides how the user
  can learn from a change (see "Low volume").
- **What the pages around the checkout promise.** Terms, refund policy, FAQ, pricing and landing
  pages. After a pricing change they often still describe the old model ("no subscription", "30-day
  guarantee", "free forever"). A checkout that contradicts its own terms is a legal and trust problem
  that sits in front of every other improvement, so it goes high on the list.
- **What the user already decided.** Memory, CLAUDE.md, docs, ADRs, earlier conversations: "no
  discount codes", "no guarantee", "the lawyer approved reference prices". Don't re-propose something
  that was decided against unless you say why it's worth revisiting.

## Phase 2: Audit against what tends to work

`references/paywall-patterns.md` has the evidence and case studies behind each point; read it when
you need to justify or size a recommendation. `references/stripe-checkout.md` covers the Stripe side,
`references/funnel.md` the measurement. The short version, roughly in order of impact:

### 1. Timing beats design
A good paywall at the wrong moment still loses. Offer at the moment of felt need, with content that
matches the moment:
- Right after the user got the value (finished, exported, succeeded): "next time, do it without X",
  leading with the premium feature that removes X.
- When they reach for a premium surface or hit a limit: lead with the thing they just reached for.
- Never mid-task. Wait until the read, export or render has finished, and pause whatever is running.

Guard every automatic moment: once per page view, at most every few days, and only after real use (a
minimum of real time, not a jump to the end). The permanent upgrade button stays as the
always-available path; moments are additions to it.

### 2. Sell the outcome before the price
A paywall works better as a short flow than as one wall of plans. Step 1 shows the result (a live try
of the feature, a short video, a before/after); step 2 shows the plans. If the feature can be tried
live inside the panel, that beats any description. Step 1 carries one idea and one button.

### 3. Show what's missing
A small Free vs Paid table: three to five rows, mostly what only paid gets, plus one row the free plan
keeps, which reassures that nothing is taken away. Checkmarks and dashes, no paragraphs.

### 4. Lower the risk of saying yes
People hesitate because of risk ("will I forget to cancel?", "is this right for me?"), not only price.
- Under the buy button, one line of reassurance that is true for the selected plan: "No risk, cancel
  anytime" for monthly, "One payment, yours forever" for lifetime.
- The button names the action ("Start 7 free days", "Get Pro forever") and carries a right chevron.
  The evidence for each tweak alone is thin, but they are cheap and common among winning paywalls.
- A trial on the recurring plan, shown as a timeline: today free, reminder on day N, first charge on
  day M. A card-required trial filters out people who were never going to pay: fewer starts, many more
  paying customers. A card-free trial brings more starts, but in products where most users leave
  quickly it mostly delays the "no". Which one is a business decision for the user. If the timeline
  promises a reminder, the reminder has to be sent (see the Stripe reference).
- When most users leave quickly, or the user says a card-free trial won't work for them, recommend
  the card-required trial on the recurring plan for the first batch rather than "later". It is the
  risk reducer with the strongest evidence, and a cheap monthly price is not a substitute: the
  question in the buyer's head is "can I get out?", not "is it expensive?".
- Trials only for accounts that never had paid access. Stripe does not stop repeat trials by itself.
- When a push is wanted, a longer trial preserves trust better than a discount.

### 5. Keep the choice small
Two plans visible, the higher-value one (yearly or lifetime) first and preselected; anything else
behind "all plans". Prices formatted for the user's locale. Struck-through "was" prices only after a
legal OK (see Legal).

### 6. Honest proof
Real numbers from the database, rounded down ("500+ creators" when there are 529). No invented
ratings, testimonials, logos or "12 people bought today". If there are no reviews yet, propose a way
to collect them instead.

### 7. One nudge on the way out
When someone closes the plans with the expensive plan selected, offer the lighter option once
(monthly, or the trial). Not a discount, and once per visit at most.

### Recommend against, and say why
- Fake urgency: resetting countdowns, "only 2 left", spin-the-wheel discounts. They lift short-term
  numbers, teach users to close for a discount, erode trust, and in the EU they are unfair-practice
  risks.
- Discounts on close, except for real seasonal sales.
- Confusing trial toggles and pre-checked add-ons.
- Many small A/B tests at low traffic (below).
- Advertising features that are not shipped yet.

### Low volume
At a handful of signups a day, a split test on a button won't give a readable result for months.
Prefer one substantial change at a time, judged by comparing periods on the same funnel, and keep the
funnel running. If the user wants to change things now rather than measure first, that is a
legitimate call: build the funnel alongside the change, not as a gate in front of it.

Also check the reach: if only a small share of users ever gets to the value moment, the biggest lever
may sit before the paywall (activation). Say so when the numbers point there, in one short paragraph,
without letting it replace the paywall work the user asked for.

## Phase 3: Report and ask

Write the report in the user's language, in plain short sentences. Use this structure, with the
headings translated into the user's language:

```
**In short:** one or two sentences on where the biggest lever is.

**What already works**
- Only things that really work, each with why.

**What I would change, by impact**
1. **Title.** What changes and where (surface, file), why (evidence), effort (small/medium/large).
2. ...

**What I would not do**
- ...

**Decisions that are yours**
- The decision: the options, my recommendation, the business or legal angle.

Which points should I build? (Suggest a sensible first batch.)
```

Every point names the concrete surface and file. Every claim about the current state is something
you saw in the code or the data; mark guesses as guesses. Keep it readable in one sitting: at most
about seven changes, each in a few lines, the strongest first. Details the user can ask for later
(exact queries, copy for every locale, parameter lists) stay out of the first report.

Then stop and wait for the user's OK. Don't
start building before it, because the user may drop or reorder points and some depend on decisions
only the user can make.

## Phase 4: Build what was approved

- **Copy**: plain and short, in every locale the product supports. No em dashes in visible text: they
  read as machine-written; rewrite the sentence rather than swapping the character. Never promise
  something the product doesn't do (a reminder that isn't sent, a feature that isn't shipped).
- **Moment logic** (when to show, rate limits, eligibility) in a small pure module with unit tests, so
  it can't misfire silently. Keep UI state derived rather than synced through effects.
- **Stripe**: follow `references/stripe-checkout.md`. Test with test-mode keys only.
- **Funnel**: follow `references/funnel.md`. Every new step gets its event in the same change, with
  the surface and context as properties, so moments can be compared later.
- **The user's working rules** on commits and deploys come first: some deploy manually, some keep
  work in progress out of commits. Ask before deploying.

## Phase 5: Verify

Run the app and walk each path as a user would: every moment that opens the offer, both steps, the
plan switch, the exit nudge, and the button through to a Stripe test session with the right mode,
trial and branding. Run tests, types and lint.

Gotcha: a hidden or background browser tab throttles `requestAnimationFrame` to a crawl, so scroll-
or animation-driven triggers may never fire in an automated check. Move the state close to the
trigger (for example, scroll near the end) instead of waiting for it.

Report what you verified and what you could not (a real purchase, a real reminder email), and what
the user should check themselves.

## Legal: flag, don't decide

EU and German points to put in front of the user, who decides, ideally with a lawyer:
- **Reference prices.** Under the EU Omnibus rules a struck-through price has to be the lowest price
  actually charged in the 30 days before. A price that never existed is a risk.
- **Trials that become paid subscriptions.** Price, date of the first charge and how to cancel must be
  clear before the card is taken; cancelling must be as easy as signing up (in Germany the
  "Kündigungsbutton", § 312k BGB).
- **Digital content and the 14-day withdrawal right.** Immediate access needs the consumer's express
  consent and acknowledgment that they lose the right; collect both at checkout.
- **Tracking.** Open pixels and analytics can need consent.
