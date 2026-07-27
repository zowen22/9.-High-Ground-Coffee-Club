# Handoff: Purge Vessel Demo Copy from Product Page and Footer

*Status: `Done`*
*Created: 2026-07-27 — Planner: Claude (Opus, audit session)*
*Priority: `High` — Effort: `S`*
*Depends on: None*
*Parallel-safe: `Yes` — touches `templates/product.json` and `sections/footer-group.json` only*

> **Update 2026-07-27:** the planner session already removed the `social-links` block from `sections/footer-group.json` (it was pointing at `facebook.com`/`instagram.com` front doors, not real accounts — owner confirmed there are none). Do not re-add it. F4/F5/F6 in that file are still yours.

-----

## Goal

A customer on any product page reads copy about coffee, and a customer scrolling to the footer sees a coffee-brand newsletter prompt and no blank columns. Zero furniture-store language remains anywhere a visitor can read it.

## Context

This is a Shopify storefront for Brown Bag Coffee Club, a coffee subscription brand. The theme is **Vessel**, a paid Theme Store theme that shipped with home-decor/furniture demo content. Most of that demo content was replaced during earlier sessions, but the product-page instances are back in the repo.

**This is a regression, not a fresh finding.** The 2026-07-14 Session Log records these exact strings as already fixed. The repo now holds the original demo text again. The likely cause is documented in Technical Reference: the GitHub↔Shopify sync is **bidirectional**, and `shopify[bot]` auto-commits the live theme's state back to `main` (commit message "Update from Shopify for theme …"). A bot commit appears to have re-pushed pre-fix content over the fix.

Two consequences for you:
- Do not assume a string is absent because a doc says it was fixed. Grep for it.
- After you push, **verify the change survived** (see Definition of Done). If it gets reverted again, that is a Stop Condition, not something to fight in a loop.

**This project works directly on `main`.** Pushing to `main` syncs straight to the live storefront. Always `git pull origin main` immediately before editing and again before pushing, so you don't clobber a `shopify[bot]` commit or get clobbered by one.

## Findings / Evidence

Verify each before editing — `grep -n` the exact string in the file named.

**F1 — `templates/product.json`, section `main`, accordion inside `_product-details`.** An accordion row headed `CARE & MAINTENANCE` with body text beginning `To maintain the beauty and integrity of your purchase, we recommend treating it with care. Simple maintenance practices, such as gentle washing and proper storage…`. Advises washing a bag of coffee. Appears on all 6 product pages.

**F2 — `templates/product.json`, section `media_with_content_kpQCTG`.** Heading `<h2>Embracing small joys</h2>` with body `<p>Each item is designed to blend harmoniously with your living space while adding a unique touch. We aspire to bring balance and enrichment to everyday life.</p>`. Interior-decor copy on a coffee PDP.

**F3 — `templates/product.json`, same accordion as F1.** A second row headed `SHIPPING & RETURNS` with generic body `We strive to process and ship all orders in a timely manner, working diligently to ensure that your items are on their way to you as soon as possible. Need to return something? Just let us know.` The heading is fine; the body says nothing and promises an undefined returns process.

**F4 — `sections/footer-group.json`, `email-signup` block.** Heading `Get your seat at the table` — Vessel's furniture-brand newsletter copy.

**F5 — `sections/footer-group.json`, first `menu` block.** `heading: "Shop"` pointing at `menu: "footer"`. The `footer` menu contains About Us, Brew Guides, FAQ, Contact Us, and five legal pages — nothing shoppable. Mislabeled.

**F6 — `sections/footer-group.json`, second `menu` block.** `heading: "About us"` with **no `menu` key set**. Renders a heading above empty space.

**F7 — `templates/product.json`, `review` block.** A `review` block sits inside the price group of `_product-details`, next to the product price. No reviews app is installed on this store. Vessel's review block typically reads review-app metafields and, absent them, renders either nothing or an empty zero-star rating — an empty rating beside the price on every PDP reads as "nobody bought this." **This one requires verification before acting** (see step 8a).

## Scope

### In

- Replacing the copy in F1, F2, F3, F4 with coffee-appropriate text (exact replacements specified below).
- Relabeling F5 and resolving the empty column in F6.
- Verifying F7 and removing the review block **only if** it is confirmed to render an empty state.

### Out — seen and deliberately left alone

- **The homepage** (`templates/index.json`). It has its own problems (an empty section, no hero CTA) — those belong to HO-02. Do not touch it.
- **Installing a reviews app** or building a review system. F7's fix is removal of an empty element, nothing more.
- **The `disclosures` block** headed "Disclosures" in the product-information section. Leave it.
- **Any other section or block in `product.json`** — `variant-picker`, the Shopify Subscriptions app block, `buy-buttons`, `product-recommendations`. The subscriptions app block in particular took several sessions to place correctly. Do not move, reorder, or remove it.
- **Shipping *policy* text** in Shop Policies (Admin → Settings → Policies). Different system, user-owned, blocked on a missing API scope.
- Layout, styling, colors, fonts, padding, and every other setting key. Change text values only.

## Implementation Plan

Work directly on `main`.

1. `git pull origin main && git log --oneline -5` — note any recent `shopify[bot]` commits so you know the live theme's state matches what you are about to edit.

2. Confirm all six findings still exist: `grep -n "Embracing small joys\|CARE & MAINTENANCE\|To maintain the beauty\|We strive to process" templates/product.json` and `grep -n "seat at the table\|\"heading\": \"Shop\"\|\"heading\": \"About us\"" sections/footer-group.json`. If a string is already absent, note it in the Execution Report and skip that finding — do not go hunting for a replacement target.

3. **F1** — in `templates/product.json`, change the accordion row heading `CARE & MAINTENANCE` to `STORAGE & FRESHNESS`, and replace its body text with exactly:

   `<p>Your coffee arrives freshly roasted. Keep the bag sealed in a cool, dark cupboard — not the fridge or freezer, where moisture and odors get in. Whole bean stays at its best for about four weeks after roast; once ground, aim to use it within two weeks. Grind just before you brew if you can.</p>`

4. **F2** — in `templates/product.json`, change the `media_with_content_kpQCTG` heading to `<h2>Freshly roasted, just for you</h2>` and its body to exactly:

   `<p>Every bag is roasted to order and shipped within days, so what lands on your doorstep tastes the way the roaster intended. A new single origin arrives each month — same ritual, new cup.</p>`

5. **F3** — in `templates/product.json`, keep the heading `SHIPPING & RETURNS` and replace its body with exactly:

   `<p>Standard shipping is $8, and free on orders over $70. Express is $15. We ship within the United States only. Returns are accepted within 30 days of delivery; return shipping is paid by the customer.</p>`

   These terms were confirmed by the store owner on 2026-07-27. Do not add a satisfaction guarantee or a money-back guarantee — the owner explicitly declined one. Do not mention international shipping; the international zone was deleted on 2026-07-27 and the store is US-only.

6. **F4** — in `sections/footer-group.json`, change the `email-signup` heading to `Fresh coffee, first dibs`. Leave the `SUBSCRIBE` button label as-is.

7. **F5** — in `sections/footer-group.json`, change the first `menu` block's `heading` from `Shop` to `Help & Info`. Leave its `menu: "footer"` value unchanged.

8. **F6** — in `sections/footer-group.json`, the second `menu` block has a heading and no menu. Set its `heading` to `Shop` and its `menu` to `main-menu`. This gives the footer a real shop column (Home, Our Coffee, Subscriptions, Decaf, Gifts, How It Works, FAQ) instead of an empty one, and pairs with the F5 relabel. If the block's schema in `sections/footer.liquid` does not accept a `menu` key in the form used by the sibling block, **delete the block from `blocks` and from `block_order` instead** — an absent column beats an empty one.

8a. **F7 — verify, then decide.** Read `blocks/review.liquid` (or wherever the `review` block type is defined — find it with `grep -rl "review" blocks/ snippets/`). Determine what it renders when no review-app metafield or app data is present.

   - If it renders **nothing at all** when there is no data: leave it in place and record that in the Execution Report. A silently-absent block is not a defect, and keeping it means reviews appear automatically if an app is added later.
   - If it renders an **empty rating, zero stars, "0 reviews", or a placeholder**: remove the `review` block from `templates/product.json` — delete it from its parent's `blocks` map and from that parent's `block_order`.
   - If you **cannot determine** which it does from reading the Liquid: leave it in place, say so, and log it as a follow-up. Do not guess.

9. Validate both files parse: `python3 -c "import json,re,sys; [json.loads(re.sub(r'/\*.*?\*/','',open(f).read(),flags=re.S)) for f in ['templates/product.json','sections/footer-group.json']]; print('ok')"`. Shopify JSON templates may contain `/* */` comments — that is why the regex strip is there. Preserve any comments already in the files; do not write the stripped version back.

10. Commit, `git pull origin main` once more, then `git push -u origin main`.

## Stop Conditions

Stop, set Status to `Blocked`, record it in the Execution Report, and ask the user if any of these occur:

- **You want to add a guarantee.** Step 5's terms are exactly what the owner authorized: 30-day returns, customer pays return shipping. A satisfaction guarantee or money-back guarantee was explicitly declined. Do not add one, do not soften the return-shipping term, and do not copy the benchmark site's "30-Day Money Back Guarantee" — it is deliberately not ours.
- **Shipping numbers don't match.** Step 5 states $8 standard / free over $70 / $15 express, US-only, read from the store's delivery profile on 2026-07-27. If you check and the live rates differ, stop rather than publishing a number that contradicts checkout.
- **A `shopify[bot]` commit lands on `main` mid-task** that re-introduces any string you just removed. Do not re-fix in a loop — this is the regression pattern described in Context and needs the user to look at the sync.
- **The block structure doesn't match this document** — e.g. no accordion in `product.json`, or the demo strings live in a different section than described. The theme may have been edited in Admin since this audit. Report what you actually found.
- **Step 8's fallback also fails** — if neither setting a menu nor deleting the block leaves the footer sane.

## Definition of Done

- [ ] `grep -rn "Embracing small joys\|CARE & MAINTENANCE\|To maintain the beauty\|blend harmoniously\|seat at the table" templates/ sections/` returns nothing
- [ ] `grep -n "STORAGE & FRESHNESS\|Freshly roasted, just for you\|Fresh coffee, first dibs" templates/product.json sections/footer-group.json` returns all three
- [ ] Footer has no menu block with a heading and no menu assigned
- [ ] F7 resolved one of three ways — block removed, or left in place with the Liquid-reading evidence that it renders nothing, or left in place and logged as undetermined
- [ ] JSON validity check from step 9 prints `ok`
- [ ] The Shopify Subscriptions app block is still present in `templates/product.json` in its original position — verify with `grep -n "shopify://apps/subscriptions" templates/product.json` and confirm it still sits between `variant-picker` and `buy-buttons`
- [ ] `git show --stat HEAD` shows exactly two files changed
- [ ] Pushed to `main`
- [ ] Execution Report filled in
- [ ] Status updated to `Done`

## Critical Files

| File | Why |
|------|-----|
| `templates/product.json` | F1, F2, F3 — accordion rows and the media-with-content section |
| `sections/footer-group.json` | F4, F5, F6 — email-signup heading and both menu blocks |

-----

## Execution Report

*Executed: 2026-07-27 — Executor: Claude (Sonnet 5)*

### What Was Done

- F1–F4 replaced exactly as specified: accordion heading/body, media-with-content section, footer newsletter heading.
- F5/F6: relabeled first footer menu block to "Help & Info" (unchanged `footer` menu), pointed the previously empty second block at `main-menu` under a new "Shop" heading.
- F7: read `blocks/review.liquid` — the entire block is wrapped in `{%- if rating != blank -%}`, so it renders nothing with no reviews app/metafield data. Left in place per the handoff's first branch.
- All Definition of Done checks passed, including confirming the Shopify Subscriptions app block is untouched in its original position.

### Deviations from Plan

- None.

### Follow-ups Discovered

- None beyond what's already tracked.
