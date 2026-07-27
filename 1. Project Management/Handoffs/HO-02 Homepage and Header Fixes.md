# Handoff: Homepage Empty Section, Hero CTA, and Free-Shipping Messaging

*Status: `Open`*
*Created: 2026-07-27 — Planner: Claude (Opus, audit session)*
*Priority: `High` — Effort: `S`*
*Depends on: None*
*Parallel-safe: `Yes` — touches `templates/index.json` and `sections/header-group.json` only*

-----

## Goal

The homepage renders no empty sections, gives a first-time visitor an obvious way to start a subscription, and tells them free shipping exists at $70 — a real incentive currently mentioned nowhere on the site.

## Context

Brown Bag Coffee Club is a Shopify coffee-subscription storefront on the **Vessel** theme. The homepage (`templates/index.json`) was rebranded from Vessel's demo content in an earlier session, but the cleanup left one section configured with no content, and the hero was never given a call-to-action.

Separately: the store's delivery profile grants **free Standard shipping on domestic orders of $70 or more**. Nothing on the storefront says so. Average cart value sits well under that for a single bag, so surfacing the threshold is a direct nudge toward multi-bag orders.

Pushing to `main` syncs to the live storefront, and the sync is bidirectional — `shopify[bot]` commits Admin theme-editor edits back to `main`. Develop and push on `claude/website-production-audit-yv6dkg`, not `main`. Rebase on `origin/main` before starting if bot commits have landed.

## Findings / Evidence

**F1 — empty homepage section.** `templates/index.json`, section `collection_list_bGtkjG` (type `collection-list`, last in `order`). Its `block_order` is `[]`. It contains only the static `_collection-card` template block, so no cards render. The section reserves vertical space at the bottom of the homepage and displays nothing.

**F2 — hero has no CTA.** `templates/index.json`, section `hero_irjTFL`. Its only child block is a `text` block containing `<h1><em>A new origin</em><br/>every month</h1>`. There is no button, no subheading, no price anchor. The section's own `link` setting is `shopify://collections`, making the entire hero one large invisible click target — which is not discoverable as a CTA and, worse, points at the generic all-collections list rather than the subscriptions collection.

**F3 — free-shipping threshold invisible.** Verified in the store's `deliveryProfiles`: Domestic zone has a `$0.00` Standard rate conditioned on `TOTAL_PRICE >= 70`. `grep -ri "free shipping\|\$70" templates/ sections/` returns nothing.

**F4 — country and language selectors enabled.** `sections/header-group.json`, `header_section` settings: `show_country: true`, `show_language: true`. The store is single-currency USD. Whether these are justified depends on the store's Markets configuration, which this audit did not check.

## Scope

### In

- Resolving F1 by populating the collection-list section with the three real collections.
- Adding a visible CTA button to the hero (F2) and pointing it at the right destination.
- Adding free-shipping threshold messaging (F3) in exactly one place.
- Investigating F4 and reporting; changing it only if the finding is confirmed.

### Out — seen and deliberately left alone

- **Expanding the homepage with new content sections** — value props, how-it-works, guarantee, social proof. The audit flagged the homepage as thin (P2-2), but that is net-new content pending a scope decision from the user. Adding sections is explicitly not this handoff. Do not add press mentions, testimonials, or review counts under any circumstance — there are none, and inventing them is fabricating social proof.
- **`templates/product.json` and `sections/footer-group.json`** — HO-01 owns both. Editing them here will cause a merge conflict.
- **Hero imagery** — the hero uses stock placeholder photography. Known, user-owned, do not swap.
- **The two `product-list` sections** on the homepage. They correctly reference `coffee-subscriptions` and `gifts`. Leave them.
- **Theme colors, fonts, spacing, and section layout settings.**

## Implementation Plan

Work on branch `claude/website-production-audit-yv6dkg`.

1. `git fetch origin main && git log origin/main --oneline -5`. Rebase onto `origin/main` if `shopify[bot]` commits have landed since your branch point.

2. Confirm F1 and F2 still hold. Read `templates/index.json` and check `collection_list_bGtkjG` has `"block_order": []` and that `hero_irjTFL` has exactly one block.

3. **F1 — populate the collection list.** Add three `_collection-card` block instances to `collection_list_bGtkjG`, referencing collection handles `coffee-subscriptions`, `gifts`, and `decaf`, in that order, and list their block IDs in `block_order`.

   Copy the shape of the existing static `_collection-card` block for settings — do not invent setting keys. To find the correct schema for a non-static collection card (what key holds the collection reference, and what the child blocks must look like), read `sections/collection-list.liquid` and `blocks/` for the collection-card block definition. The `product-list` sections in the same file use a `"collection": "coffee-subscriptions"` setting as a handle string — the collection card most likely follows the same convention, but confirm against the schema rather than assuming.

   If the schema turns out to require something this document doesn't anticipate, the acceptable fallback is to **delete the section entirely** — remove `collection_list_bGtkjG` from both `sections` and `order`. An absent section beats an empty one, and both product-list sections already surface the same collections. Note the fallback in your Execution Report.

4. **F2 — hero CTA.** Add a `button` block to `hero_irjTFL` alongside the existing text block, and add its ID to `block_order` after the text block. Use label `Start your subscription` and link `shopify://collections/coffee-subscriptions`.

   Read `sections/hero.liquid` (and the `button` block definition under `blocks/`) for the correct block type name and required settings. `templates/404.json` contains a working `button` block with `{"label": "Continue shopping", "link": "shopify://collections/all"}` — use it as the reference for structure.

   Also change the hero section's own `link` setting from `shopify://collections` to `shopify://collections/coffee-subscriptions` so the whole-hero click target agrees with the button.

5. **F3 — free-shipping message.** Add a short line under the hero heading, inside the hero's existing text block area, as a second `text` block: `<p>Free shipping on orders over $70.</p>`. Place it between the `<h1>` block and the button in `block_order`.

   One placement only. Do not also add an announcement bar, a cart progress bar, or PDP messaging — those are separate surfaces with their own considerations, and duplicating the claim in four places is how a stale promise ends up shipping after the threshold changes.

6. **F4 — investigate before changing.** Determine whether the store has more than one active Market. If it does not, set `show_country: false` and `show_language: false` in `sections/header-group.json`. If it does — or if you cannot determine it — **leave both settings alone** and report the finding. Do not guess.

7. Validate: `python3 -c "import json,re; [json.loads(re.sub(r'/\*.*?\*/','',open(f).read(),flags=re.S)) for f in ['templates/index.json','sections/header-group.json']]; print('ok')"`. These templates may contain `/* */` comments; preserve them, do not write back the stripped version.

8. Commit and push to `claude/website-production-audit-yv6dkg`.

## Stop Conditions

Stop, set Status `Blocked`, record it, and ask the user if:

- **The $70 threshold doesn't verify.** Step 5 publishes a shipping promise. Confirm against the store's live delivery profile (Domestic zone, the `$0.00` Standard rate's `TOTAL_PRICE >= 70` condition) before writing the line. If the number differs or the free rate is gone, do not publish a number.
- **The collection-card schema can't be satisfied and deleting the section also seems wrong** — e.g. the section turns out to be referenced elsewhere.
- **You are tempted to add a homepage section** that isn't F1/F2/F3. That impulse is correct in substance and out of scope here — log it under Follow-ups instead.
- **Markets configuration is ambiguous** for F4. Leaving the selectors on is the safe outcome; say so and move on rather than blocking the whole handoff.
- **A `shopify[bot]` commit reverts your changes** after push. Do not re-apply in a loop.

## Definition of Done

- [ ] `templates/index.json` has no section with an empty `block_order` (or the section was removed and the fallback is documented)
- [ ] The hero contains a button block labeled `Start your subscription` linking to `shopify://collections/coffee-subscriptions`
- [ ] `grep -n "Free shipping on orders over \$70" templates/index.json` returns exactly one match
- [ ] `grep -rn "Free shipping" templates/ sections/` returns exactly one match total — the claim appears in one place only
- [ ] Every collection handle referenced in `index.json` is one of `coffee-subscriptions`, `gifts`, `decaf` — verify none point at `all` or `frontpage`
- [ ] F4 either changed with justification, or left alone with the reason stated in the Execution Report
- [ ] JSON validity check from step 7 prints `ok`
- [ ] `git diff origin/main --stat` shows at most two files changed
- [ ] Pushed to `claude/website-production-audit-yv6dkg`
- [ ] Execution Report filled in
- [ ] Status updated to `Done`

## Critical Files

| File | Why |
|------|-----|
| `templates/index.json` | F1, F2, F3 — collection-list section and hero |
| `sections/header-group.json` | F4 — country/language selector settings (conditional) |
| `sections/hero.liquid`, `sections/collection-list.liquid`, `blocks/` | Read-only — block schema reference |
| `templates/404.json` | Read-only — working `button` block reference |

-----

## Execution Report

*Executed: [date] — Executor: [model/session]*

### What Was Done

-

### Deviations from Plan

-

### Follow-ups Discovered

-
