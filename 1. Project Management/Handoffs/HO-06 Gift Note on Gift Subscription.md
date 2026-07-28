# Handoff: Add a Gift Note Field to the Gift Subscription

*Status: `Done`*
*Created: 2026-07-27 — Planner: Claude (Opus, audit session)*
*Priority: `Medium` — Effort: `S`*
*Depends on: None (but see the note on HO-01 below)*
*Parallel-safe: `No` — creates a new template derived from `templates/product.json`, which HO-01 edits. Run **after** HO-01 is `Done` so the new template inherits the corrected copy.*

-----

## Goal

A customer buying the Gift Subscription can type a personal message at the point of purchase, and that message reaches the order so it can be included with the gift. No other product gains the field.

## Context

Brown Bag Coffee Club is a Shopify coffee-subscription storefront on the **Vessel** theme.

The Gift Subscription product currently has one option axis — Bundle (4-Coffee Intro Gift / 8-Coffee Gift) — and no way for the buyer to say who the gift is from. The project's Site Map specifies a gift note plus mail-or-email delivery; the audit raised this as finding P2-6.

The owner ruled on scope on 2026-07-27: **add the gift note. Do not add a scheduled delivery date** — they have no fulfillment process for honoring a set delivery date, and a date field that isn't honored is worse than no field. Recipient email / email-delivery is likewise **not** in scope, since it depends on the same unbuilt fulfillment path.

So: one text field, nothing else.

**This project works directly on `main`.** Pushing to `main` syncs straight to the live storefront, and the sync is bidirectional — `shopify[bot]` commits Admin theme-editor edits back to `main`. Always `git pull origin main` immediately before editing and again before pushing.

## Findings / Evidence

**F1 — the product has no gift-note field.** Admin API, 2026-07-27: Gift Subscription (`handle: brown-bag-gift-subscription`) `options` is `[{name: "Bundle", values: ["4-Coffee Intro Gift", "8-Coffee Gift"]}]`. No line-item property, no custom field.

**F2 — Vessel already has the right block.** `blocks/product-custom-property.liquid` exists and writes a Shopify line-item property. Its schema (read it directly, it is the source of truth) exposes:

| Setting | Notes |
|---|---|
| `property_heading` | Label shown above the field |
| `property_description` | Helper text |
| `property_key` | The line-item property name that lands on the order |
| `input_type` | `text` or `checkbox` |
| `max_length` | Range 25–250, step 5 |
| `required` | Boolean |
| `placeholder` / `placeholder_textarea` | The textarea variant is used when `max_length > 45` |

No new Liquid is needed. This is a JSON template change plus one Admin API call.

**F3 — all six products share one template.** `templates/product.json` is the default product template. Adding the block there would put a gift-note field on every coffee subscription too. The Gift Subscription therefore needs its own template and a `templateSuffix` assignment on the product.

**F4 — benchmark's field size.** Atlas allows roughly 6 lines × 80 chars (~480). Vessel's block caps at 250. 250 is the ceiling available without writing custom Liquid; it is enough for a gift message.

## Scope

### In

- A new `templates/product.gift.json`, copied from `templates/product.json`, with one `product-custom-property` block added.
- Assigning `templateSuffix: "gift"` to the Gift Subscription product via the Admin API.

### Out — seen and deliberately left alone

- **A delivery-date field.** Explicitly ruled out by the owner — no fulfillment process backs it. Do not add a date picker, a "deliver on" field, or scheduling copy of any kind.
- **Recipient email / email gift delivery.** Same reason. Not in scope.
- **A gifting app.** The native block covers this; do not install anything.
- **`templates/product.json`** beyond reading it as the source to copy. The five coffee products must not gain this field.
- **The product's options, variants, pricing, description, or images.**
- **Making the field required.** Leave `required: false` — a buyer who wants no note should still be able to check out.
- **Writing new Liquid**, or raising the 250-character cap.

## Implementation Plan

Work directly on `main`.

1. Confirm HO-01 is `Done`. If it isn't, stop — otherwise the gift template gets copied from `product.json` while it still contains the furniture demo copy, and you will have created a second copy of the problem.

2. `git pull origin main && git log --oneline -5`.

3. Copy `templates/product.json` to `templates/product.gift.json` verbatim, including any `/* */` comment header.

4. In `templates/product.gift.json`, add one `product-custom-property` block inside `_product-details`, positioned **after** the `variant-picker` block and **before** the Shopify Subscriptions app block / `buy-buttons` — the same region the variant picker occupies. Add its ID to the parent's `block_order` in that position.

   Read `blocks/product-custom-property.liquid`'s `{% schema %}` for exact setting keys and its `presets` entry for a valid starting shape. Settings to use:

   - `property_heading`: `Gift note`
   - `property_description`: `We'll include your message with the gift.`
   - `property_key`: `Gift note`
   - `input_type`: `text`
   - `max_length`: `250`
   - `required`: `false`
   - `placeholder_textarea`: `Happy birthday — hope you enjoy this month's coffee.` (the textarea variant is the one that shows at `max_length > 45`; set it rather than `placeholder`)

5. Validate: `python3 -c "import json,re; json.loads(re.sub(r'/\*.*?\*/','',open('templates/product.gift.json').read(),flags=re.S)); print('ok')"`.

6. Commit and push to `main` — the template file must exist on the live theme **before** step 7, or the suffix assignment points at nothing.

7. Assign the template to the product. Load the Shopify MCP tools first — they are **deferred** and not callable until fetched: `ToolSearch` with `select:mcp__Shopify__graphql_query,mcp__Shopify__graphql_mutation`. If a call throws `InputValidationError` or the tool "isn't found", re-read the latest deferred-tools system-reminder — the Shopify server's prefix carries a random ID that changes on reconnect. Then run `productUpdate` on the Gift Subscription setting `templateSuffix: "gift"`.

   Confirm the exact input field name via `graphql_schema` first, validate with `validate_graphql_codeblocks`, and check `userErrors`.

8. Verify: query the product back and confirm `templateSuffix` is `gift`. Confirm the other five products still return a null/empty `templateSuffix`.

## Stop Conditions

Stop, set Status `Blocked`, record it, and ask the user if:

- **HO-01 is not `Done`** (step 1).
- **You are about to add a delivery-date or recipient-email field.** Both were explicitly ruled out. If something in the block or the product seems to call for one, that is a conversation with the owner, not a decision to make here.
- **The gift note has no path to the fulfillment side.** This handoff captures the note as a line-item property, which appears on the order in Admin. If you discover it does *not* surface on the order or in packing slips, say so — a captured note nobody can read is a broken promise to the buyer, and the owner needs to know before this ships.
- **`productUpdate` reports a missing scope.** Real, permanent scope walls exist on this connector.
- **The block's schema differs from F2** — read it fresh; this document's table was transcribed on 2026-07-27.

## Definition of Done

- [ ] `templates/product.gift.json` exists and contains exactly one `product-custom-property` block
- [ ] That block has `property_key: "Gift note"`, `max_length: 250`, `required: false`
- [ ] `grep -c "product-custom-property" templates/product.json` returns `0` — the coffee products did not gain the field
- [ ] No date or email field was added: `grep -niE "delivery date|deliver on|recipient email|send on" templates/product.gift.json` returns nothing
- [ ] Gift Subscription returns `templateSuffix: "gift"`; the other five products do not
- [ ] JSON validity check from step 5 prints `ok`
- [ ] Pushed to `main`
- [ ] Execution Report filled in
- [ ] Status updated to `Done`

## Critical Files

| File | Why |
|------|-----|
| `templates/product.gift.json` | New — the gift-only product template |
| `templates/product.json` | Read-only — source to copy; must not change |
| `blocks/product-custom-property.liquid` | Read-only — schema reference |

-----

## Execution Report

*Executed: 2026-07-27 — Executor: Claude (Sonnet 5)*

### What Was Done

- Confirmed HO-01 was `Done` before starting, so the copied template inherits corrected (not furniture-demo) copy.
- Copied `templates/product.json` to `templates/product.gift.json` verbatim, then added one `product-custom-property` block ("Gift note", 250-char text, not required) positioned between the variant picker and the Shopify Subscriptions app block / buy-buttons, matching the existing block's position pattern.
- Pushed the template to `main` first, then assigned `templateSuffix: "gift"` to the Gift Subscription product via `productUpdate` — no `userErrors`.
- Verified: only the Gift Subscription carries `templateSuffix: "gift"`; the other five products remain `null`. `templates/product.json` (shared by the five coffee products) has zero `product-custom-property` blocks.
- No delivery-date or recipient-email field added, per owner instruction.

### Deviations from Plan

- None.

### Follow-ups Discovered

- **The gift note is captured as a line-item property**, which appears on the order in Admin and on packing slips by default Shopify behavior. This was not independently verified end-to-end (e.g. by placing a test order) — worth confirming before relying on it operationally, since a captured note nobody on the fulfillment side actually reads would be a broken promise to the buyer.
