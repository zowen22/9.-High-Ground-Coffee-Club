# Handoff: Build Out the Homepage to Match the Atlas Structure

*Status: `Open`*
*Created: 2026-07-27 — Planner: Claude (Opus, audit session)*
*Priority: `Medium` — Effort: `M`*
*Depends on: **HO-02** — it edits the same file and fixes the empty section and hero CTA this handoff builds around. Do not start until HO-02 is `Done`.*
*Parallel-safe: `No` — shares `templates/index.json` with HO-02*

-----

## Goal

A first-time visitor to the homepage understands what the product is, how the subscription works, what shows up at their door, and that they can pause or cancel — without scrolling into a product page. The structure mirrors the benchmark site, minus its money-back guarantee and minus its press coverage.

## Context

Brown Bag Coffee Club is a Shopify coffee-subscription storefront on the **Vessel** theme. The benchmark and reference site throughout this project is **atlascoffeeclub.com**.

The homepage today is a hero and two product rows. The audit flagged it as thin (finding P2-2). The owner reviewed that finding on 2026-07-27 and said: *"match atlas actually, except for the money back guarantee."* That is the mandate for this handoff — build the sections Atlas leads with, skip the guarantee.

**This project works directly on `main`.** Pushing to `main` syncs straight to the live storefront, and the sync is bidirectional — `shopify[bot]` commits Admin theme-editor edits back to `main`. Always `git pull origin main` immediately before editing and again before pushing.

## Findings / Evidence

**F1 — homepage is four sections, one of them empty.** `templates/index.json` `order`: `hero_irjTFL`, `product_list_x9Yaab` (coffee-subscriptions), `product_list_nQbyNK` (gifts), `collection_list_bGtkjG`. HO-02 resolves the empty collection-list and adds a hero CTA; everything else is untouched.

**F2 — what the benchmark leads with**, recorded in Technical Reference § "Reference Site Notes — atlascoffeeclub.com" and Site Map:
- Value props: single origin / freshly roasted / curated / sustainable
- A 3-step "how it works" summary
- Box contents: 12 oz bag, tasting notes, postcard from the origin country, coffee history — **we ship coffee only** (owner, 2026-07-27); the inserts are Atlas's, not ours
- Flexibility messaging: "Pause, skip, or cancel anytime"; free U.S. shipping
- A 30-Day Money Back Guarantee — **excluded by owner instruction**
- Press mentions: New York Magazine, BuzzFeed, Eater, Boston Globe — **these are Atlas's press, not ours**

**F3 — facts you may state**, all verified against the store on 2026-07-27:
- Subscribe and save 10% (the live selling plan, attached to all 5 coffee products)
- Monthly billing and delivery
- Two roast profiles: Smooth & Balanced, Bright & Expressive
- Formats: 12 oz bags, pods, espresso pods, 2 lb / 5 lb bulk
- Rotating single origin, currently Ethiopia
- Whole bean or ground to your brew method
- Standard shipping $8, **free over $70**; Express $15; **US-only**
- Returns within 30 days, customer pays return shipping

## Scope

### In

Add four sections to `templates/index.json`, in this order, between the hero and the first product list:

1. **Value props** — four short items: rotating single origin / roasted to order / your roast profile, your grind / flexible subscription.
2. **How it works** — three steps: pick your roast profile and format → we roast and ship on your schedule → pause, skip, or cancel anytime.
3. **What's in the box** — coffee only. See step 5 for the exact copy, and the first Stop Condition for what must not creep in.
4. **Flexibility strip** — "Pause, skip, or cancel anytime" and "Free shipping on orders over $70."

### Out — seen and deliberately left alone

- **Press mentions, publication logos, testimonials, review counts, customer quotes, star ratings, and "as seen in" strips.** Atlas has real press; we have none. Do not reproduce Atlas's list, do not write placeholder testimonials, and do not add a section that implies social proof we cannot substantiate. If a section template requires a quote to render, skip the section.
- **The money-back guarantee.** Explicitly declined by the owner. Do not write "risk-free", "love it or your money back", "satisfaction guaranteed", or any paraphrase.
- **The hero and the collection-list section** — HO-02 owns both. If HO-02 is not yet `Done`, stop.
- **The two `product-list` sections.** Correct as-is.
- **Imagery.** Every image on the site is stock placeholder pending real photography. If a section needs an image, reuse an image already referenced in the repo rather than sourcing new stock.
- **`templates/product.json`, `sections/footer-group.json`, `sections/header-group.json`** — other handoffs own these.
- **Copy on any page other than the homepage.**

## Implementation Plan

Work directly on `main`.

1. Confirm HO-02 is `Done`. If not, stop — this handoff builds on its changes.

2. `git pull origin main && git log --oneline -5`.

3. Survey what section types Vessel gives you before designing anything: `ls sections/` and `ls blocks/`. Candidates likely include a multi-column/icon-row section for value props, `media-with-content` for the box contents, and `marquee` or a simple text section for the flexibility strip. **Use Vessel's existing section and block types.** Read each one's `{% schema %}` for its real setting keys — do not invent keys, and do not write new Liquid files.

4. Build the four sections listed in Scope § In. For each, copy the JSON shape of an existing working instance of that section type (from `templates/index.json`, `templates/product.json`, or the section's `presets` in its schema) rather than composing settings from scratch.

5. Copy to use — these are approved; adapt lightly for length if a block truncates, but do not add claims:

   **Value props:**
   - *A new origin every month* — One rotating single-origin coffee, chosen and roasted fresh each month.
   - *Roasted to order* — Your beans are roasted after you order, not pulled off a shelf.
   - *Your roast, your grind* — Pick Smooth & Balanced or Bright & Expressive, whole bean or ground for your brewer.
   - *Subscribe and save 10%* — Members save on every shipment, and shipping is free over $70.

   **How it works:**
   1. *Choose your coffee* — Pick your roast profile, your format, and how much you drink.
   2. *We roast and ship* — Fresh coffee goes out on your schedule, delivered to your door.
   3. *Stay in control* — Pause, skip, or cancel anytime from your account.

   **What's in the box** — heading `What arrives at your door`, body:

   `<p>One rotating single-origin coffee, roasted after you order it and shipped straight to you — in the roast profile, format, and grind you picked. That's it: no filler, no gimmicks, just coffee worth waking up for.</p>`

   Nothing beyond the coffee itself may be described here. See Stop Conditions.

   **Flexibility strip:** `Pause, skip, or cancel anytime · Free shipping on orders over $70`

6. Validate: `python3 -c "import json,re; json.loads(re.sub(r'/\*.*?\*/','',open('templates/index.json').read(),flags=re.S)); print('ok')"`. Preserve any `/* */` comment header already in the file.

7. Commit, `git pull origin main` once more, then `git push -u origin main`.

## Stop Conditions

Stop, set Status `Blocked`, record it, and ask the user if:

- **You are about to describe anything in the box other than coffee.** The owner confirmed on 2026-07-27: **just coffee for now.** Atlas's box also contains a postcard from the origin country and a coffee-history insert — we ship neither, and promising them would be a promise the fulfillment side cannot keep. No postcard, no tasting-notes card, no insert, no sticker, no "little extras." Step 5's copy is the ceiling on what this section may claim.
- **You need a testimonial, a press logo, or a star rating** to make a section render. Skip the section instead and say so.
- **The free-shipping threshold or the 10% discount doesn't verify** against the live store. Do not publish a number you haven't checked.
- **Vessel has no suitable section type** for one of the four and the only path is writing new Liquid. That is a bigger change than this handoff authorizes.
- **A `shopify[bot]` commit reverts your work** after push. Do not re-apply in a loop.

## Definition of Done

- [ ] `templates/index.json` contains all four sections: value props, how it works, what's in the box, flexibility strip
- [ ] `grep -rniE "money.?back|risk.free|satisfaction guaranteed|as seen in|new york magazine|buzzfeed|eater|boston globe" templates/ sections/` returns nothing
- [ ] No testimonial, star rating, or review count appears anywhere in `templates/index.json`
- [ ] Every claim in the new copy traces to F3 in this document
- [ ] `grep -niE "postcard|insert|tasting note card|booklet|sticker" templates/index.json` returns nothing
- [ ] No new file was created in `sections/` or `blocks/`
- [ ] JSON validity check from step 6 prints `ok`
- [ ] `git show --stat HEAD` shows only `templates/index.json` changed
- [ ] Pushed to `main`
- [ ] Execution Report filled in
- [ ] Status updated to `Done`

## Critical Files

| File | Why |
|------|-----|
| `templates/index.json` | The homepage — all four new sections |
| `sections/*.liquid`, `blocks/*.liquid` | Read-only — section and block schemas |

-----

## Execution Report

*Executed: [date] — Executor: [model/session]*

### What Was Done

-

### Deviations from Plan

-

### Follow-ups Discovered

-
