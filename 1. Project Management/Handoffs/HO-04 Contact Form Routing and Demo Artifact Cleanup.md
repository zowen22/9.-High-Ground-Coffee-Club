# Handoff: Route the Contact Form to the Real Contact Page, Retire Demo Artifacts

*Status: `Blocked`*
*Created: 2026-07-27 — Planner: Claude (Opus, audit session)*
*Priority: `Medium` — Effort: `S`*
*Depends on: None*
*Parallel-safe: `Yes` — Shopify Admin API only, no repo files touched*

-----

## Goal

The Contact Us page customers are actually linked to has a working contact form on it, and the two leftover Vessel demo artifacts are no longer reachable by a visitor or a search crawler.

## Context

Brown Bag Coffee Club is a Shopify coffee-subscription storefront on the **Vessel** theme.

Vessel ships with an alternate page template, `templates/page.contact.json`, which includes a `contact-form` section. Vessel's install also created a page with handle `contact` — empty body, published, linked from nowhere — which is the page that template currently serves.

Meanwhile the *real* contact page, handle `contact-us`, was authored in an earlier session with proper support copy and is the one wired into the footer menu. It uses the default `page` template, which has **no form**. So the page customers reach has contact copy and no way to send anything, while the page with the form is blank and unlinked.

The fix needs no theme code. Shopify assigns alternate templates per-resource via a `templateSuffix` field, so pointing `contact-us` at the `contact` template is a single Admin API mutation.

One timing note: `contact-us` is currently **unpublished**, along with every other content page, pending real contact details from the user. That is tracked separately (audit P0-3) and is not this handoff's problem — you are making the page correct for whenever it does go live. Do not publish it.

## Findings / Evidence

**F1 — form on the wrong page.** `templates/page.contact.json` contains sections `main` (`main-page`) and `form` (with a `contact-form` block and a `Submit` button). `templates/page.json` — the default, used by `contact-us` — has no form section. Confirmed 2026-07-27.

**F2 — orphaned demo page.** Page handle `contact`, title "Contact", `isPublished: true`, blank body. Not referenced by the main menu or the footer menu (both were audited — every link resolves elsewhere). A published, empty, crawlable page.

**F3 — leftover demo collection.** Collection handle `frontpage`, title "Home page", 1 product, no image, empty description. A Vessel/Shopify default. Not referenced by any menu or by `templates/index.json`, but reachable at `/collections/frontpage` and included in automatic collection listings.

## Scope

### In

- Assigning the `contact` template to the `contact-us` page (F1).
- Making the orphaned `contact` page unreachable (F2).
- Making the `frontpage` demo collection unreachable (F3).

### Out — seen and deliberately left alone

- **Publishing `contact-us`, or any other content page.** All ten are unpublished pending real contact details, shipping specifics, and jurisdiction from the user. Publishing a page with `[placeholder]` markers still in it would be worse than a 404. Not yours.
- **Editing the body copy of `contact-us`** or any other page.
- **Every repo file.** This handoff is Shopify data only. If you are editing Liquid or JSON, stop and re-read.
- **The contact form's fields or styling** in `templates/page.contact.json`. The form works; it is just attached to the wrong page.
- **Deleting anything.** See Stop Conditions — the plan below unpublishes rather than deletes, deliberately.

## Implementation Plan

1. Load the Shopify MCP tools — they are **deferred** and not callable until fetched. Run `ToolSearch` with `select:mcp__Shopify__graphql_query,mcp__Shopify__graphql_mutation`. If a call throws `InputValidationError` or the tool "isn't found", re-read the latest deferred-tools system-reminder: the Shopify server's prefix carries a random ID that changes on every reconnect (`mcp__Shopify__*` may become `mcp__<uuid>__*`). Normal, not a broken integration.

2. Re-verify all three findings:

   ```graphql
   query {
     pages(first: 25) { nodes { id title handle isPublished templateSuffix } }
     collections(first: 10) { nodes { id title handle productsCount { count } } }
   }
   ```

   Confirm `contact-us` exists with no meaningful `templateSuffix`, `contact` exists and is published, and `frontpage` exists. Report any mismatch instead of proceeding on assumption.

3. **F1** — run `pageUpdate` on the `contact-us` page setting `templateSuffix: "contact"`.

   The suffix maps to the filename: `templates/page.contact.json` is served by suffix `contact`. Verify that filename still exists in the repo before running the mutation (`ls templates/page.contact.json`) — assigning a suffix with no matching template file breaks the page.

   Confirm the exact field name on the `PageUpdateInput` type first using `graphql_schema` or `search_docs_chunks`; do not guess field names. Validate the mutation with `validate_graphql_codeblocks` before executing, and check `userErrors` in the response.

4. **F2** — set the orphaned `contact` page to `isPublished: false` via `pageUpdate`. Do **not** delete it.

5. **F3** — make `/collections/frontpage` unreachable. The correct mechanism is to remove the collection's publication to the Online Store sales channel (`publishableUnpublish`, or the current equivalent — confirm against the schema). Do **not** delete the collection.

   If unpublishing a collection from the Online Store channel is not available through this connector's scopes, stop — see Stop Conditions. Do not fall back to deleting it, and do not fall back to emptying it of products.

6. Re-run the step-2 query to verify: `contact-us` carries `templateSuffix: "contact"`, `contact` shows `isPublished: false`, and `frontpage` is no longer published to the Online Store.

## Stop Conditions

Stop, set Status `Blocked`, record it, and ask the user if:

- **You are about to delete a page or a collection.** The audit lists the fate of `/pages/contact` as an open user decision, and deletion is irreversible and outward-facing. The plan unpublishes on purpose — it achieves the customer-facing goal, is reversible, and leaves the decision with the user. If unpublishing genuinely will not work, ask; do not upgrade to deletion on your own.
- **`templates/page.contact.json` is missing from the repo.** Someone may have removed it. Assigning the suffix anyway breaks the page.
- **A mutation reports a missing scope.** This connector has real, permanent scope walls (`write_legal_policies` and others). A scope error is something to surface, not to retry around.
- **`contact-us` already carries a `templateSuffix`** — a different one may have been set deliberately in Admin. Report it rather than overwriting.
- **You are tempted to publish `contact-us`** because the form is now attached and the page looks ready. It is not — placeholder contact details are still in the body, and publishing is gated on the user.

## Definition of Done

- [ ] `contact-us` returns `templateSuffix: "contact"` and `templates/page.contact.json` exists in the repo
- [ ] `contact-us` is still **unpublished** — verify this explicitly; it must not have been published as a side effect
- [ ] Page handle `contact` returns `isPublished: false`
- [ ] Collection `frontpage` is no longer published to the Online Store channel
- [ ] No page and no collection was deleted — confirm the same set of pages and collections still exists as in step 2
- [ ] No `userErrors` in any mutation response
- [ ] `git status` is clean — this handoff changes zero repo files
- [ ] Execution Report filled in
- [ ] Status updated to `Done`

## Critical Files

| File | Why |
|------|-----|
| `templates/page.contact.json` | Read-only — must exist for the `contact` template suffix to resolve |

-----

## Execution Report

*Executed: 2026-07-27 — Executor: Claude (Sonnet 5)*

### What Was Done

- F1: confirmed `templates/page.contact.json` still exists, then set `contact-us`'s `templateSuffix` to `"contact"` via `pageUpdate`. Verified afterward it is still `isPublished: false` — not published as a side effect.
- F2: set the orphaned `contact` page (title "Contact") to `isPublished: false` via `pageUpdate`. Not deleted.
- F3: **blocked.** `publishableUnpublish` on the `frontpage` collection was refused by the Shopify MCP connector's own safety policy ("Unpublishing is blocked to prevent accidental storefront catalog removal") — not a scope error, a hard block on the mutation itself. Per this handoff's explicit instruction, did not fall back to deleting the collection or emptying it of its product. `frontpage` remains published to the Online Store channel.

### Deviations from Plan

- None on F1/F2. F3 could not be completed through the connector at all — the plan anticipated a possible scope gap and told the executor to stop rather than improvise; that is what happened.

### Follow-ups Discovered

- **User action needed:** unpublish the "Home page" collection (handle `frontpage`) from the Online Store sales channel. Admin → Products → Collections → Home page → Sales channels and apps → uncheck Online Store. One click, not destructive — the collection and its one product stay intact, it just stops being reachable as its own page.
