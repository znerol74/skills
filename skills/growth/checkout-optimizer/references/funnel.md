# Measuring the path from offer to purchase

The goal is one small table per moment that says where people drop: shown, opened, tried, saw the
plans, started checkout, paid. Nothing more elaborate is needed at low volume, and it is enough to
tell whether the last change helped.

## Events

Fire each on the client unless marked server. Every event carries `surface` (where: prompter, editor,
settings, email link) and `context` (which moment: after_finish, limit_hit, premium_click, default),
so moments can be compared.

| Event | When | Extra properties |
|---|---|---|
| `offer_shown` | The offer becomes visible (panel opens, or an inline card scrolls into view) | |
| `offer_demo_finished` | The live try or video in step 1 completed | |
| `offer_plans_viewed` | Step 2 (the plans) is shown | |
| `offer_exit_nudge_shown` | The one-time nudge on close appears | `from_plan` |
| `checkout_started` | The buy button was pressed and the session request sent | `plan`, `trial` |
| `purchase` (server) | Webhook recorded a paid purchase | `plan`, `amount` |
| `trial_started` (server) | Webhook recorded a $0 trial checkout | `plan` |
| `trial_converted` (server) | First paid invoice after a trial | `plan` |

Use the event names the project already has if it has some; consistency beats this table.

Purchases and trials come from the database the webhook writes, never from client events alone:
ad blockers drop analytics, and redirects lose events.

## Reading it

- Count **people**, not events: one person opening the panel five times is one.
- Exclude the operator's own and test accounts.
- Report counts next to percentages. At low volume "1 purchase to 2 purchases" is noise, not +100%.
- Compare **periods** (two weeks before vs two weeks after a change) on the same funnel, and change one
  substantial thing at a time. Note the date of each change next to the numbers.
- Trials sit in their own column until they convert or end.

## Where to show it

If the project has an admin page, add one table: rows are the steps, columns are the contexts, plus a
period selector. If it hasn't, a saved query the user can run is enough. Don't build an analytics
platform; the point is to read five numbers once a week.
