# Handoff: Fix Double-Escaped HTML in Collection Descriptions

*Status: `Done`*
*Created: 2026-07-27 — Planner: Claude (Opus, audit session)*
*Priority: `High` — Effort: `S`*
*Depends on: None*
*Parallel-safe: `Yes` — no repo files touched at all; Shopify Admin API only*

-----

## Goal

The three real collection pages display their one-line description as a readable sentence instead of as literal HTML tags.

## Context

Brown Bag Coffee Club is a Shopify coffee-subscription storefront. Collection descriptions were written via the Shopify Admin API in an earlier session, and the HTML was escaped one time too many before being stored. The stored value contains the *entities* `&lt;p&gt;` rather than the *characters* `<p>`, so the storefront — which renders `descriptionHtml` as HTML — outputs the entities as visible text.

A customer landing on `/collections/decaf` today reads, literally:

```
<p>All the flavor, none of the caffeine.</p>
```

Product descriptions are **not** affected — those were checked and contain valid HTML. This is isolated to the three collections.

**No repo files change in this handoff.** All work is Shopify Admin API mutations through the Shopify MCP connector.

## Findings / Evidence

Confirmed 2026-07-27 via `collections { descriptionHtml }`. Stored values, verbatim:

| Collection | Handle | ID | Stored `descriptionHtml` |
|---|---|---|---|
| Decaf | `decaf` | `gid://shopify/Collection/693631516753` | `&lt;p&gt;All the flavor, none of the caffeine.&lt;/p&gt;` |
| Gifts | `gifts` | `gid://shopify/Collection/693631582289` | `&lt;p&gt;Give the gift of great coffee.&lt;/p&gt;` |
| Coffee Subscriptions | `coffee-subscriptions` | `gid://shopify/Collection/693781823569` | `&lt;p&gt;Rotating single-origin coffee, delivered on your schedule. Choose your format: bagged coffee, pods, espresso pods, or bulk.&lt;/p&gt;` |

A correctly-stored value looks like the products' — e.g. `<p>This month's origin: Ethiopia…</p>` — real angle brackets, with `&amp;` used only where an actual ampersand appears in the prose.

## Scope

### In

- Rewriting `descriptionHtml` on exactly those three collections so the markup is real HTML.

### Out — seen and deliberately left alone

- **The wording itself.** These sentences are approved copy. Re-escape them, do not rewrite, lengthen, or "improve" them.
- **Product descriptions.** Verified correct. Do not touch them.
- **The `frontpage` ("Home page") collection** — a leftover Vessel demo collection with 1 product. It is a real finding, and it belongs to HO-04. Leave it here.
- **Collection images, SEO fields, titles, handles, sort order, and product membership.** All verified correct in the audit.
- **Any repo file.** If you find yourself editing Liquid or JSON, you have misread this handoff — the bug is in stored store data, not in the theme.

## Implementation Plan

1. Load the Shopify MCP tools. They are **deferred** — they will not be callable until you fetch their schemas. Run `ToolSearch` with `select:mcp__Shopify__graphql_query,mcp__Shopify__graphql_mutation`. If the call fails with `InputValidationError` or the tool "isn't found", re-read the most recent deferred-tools system-reminder for the current tool names: the Shopify server's prefix carries a random ID that changes whenever it reconnects (`mcp__Shopify__*` may become `mcp__<uuid>__*`). This is normal, not a broken integration.

2. Re-verify the finding before changing anything:

   ```graphql
   query { collections(first: 10) { nodes { id handle descriptionHtml } } }
   ```

   Confirm the three handles above still return `&lt;`-escaped values. If any already contains real `<p>` tags, skip that one and note it in the Execution Report.

3. For each of the three collections, run `collectionUpdate` setting `descriptionHtml` to the unescaped form:

   - `decaf` → `<p>All the flavor, none of the caffeine.</p>`
   - `gifts` → `<p>Give the gift of great coffee.</p>`
   - `coffee-subscriptions` → `<p>Rotating single-origin coffee, delivered on your schedule. Choose your format: bagged coffee, pods, espresso pods, or bulk.</p>`

   Pass the value through GraphQL **variables**, not string-interpolated into the query body — that interpolation is the most likely way the original double-escape happened.

   Per the Shopify MCP server's documented workflow, run `validate_graphql_codeblocks` on the mutation before executing it, and check `userErrors` in every response.

4. Re-run the step-2 query as verification. Each value must now contain real `<` and `>` characters.

## Stop Conditions

Stop, set Status `Blocked`, record it, and ask the user if:

- **The mutation reports a missing scope or permission error.** This connector has hit real, permanent scope walls before (`write_legal_policies`, subscription-contract scopes). If `collectionUpdate` is blocked, that is a genuine limitation to surface, not something to retry around.
- **A description comes back still escaped after a successful mutation** — that means something in the write path is re-escaping, and a second attempt will not fix it.
- **The stored text differs from the table above.** Someone edited it in Admin since this audit. Report what you found rather than overwriting an intentional edit with older copy.
- **You cannot get the Shopify connector to respond** after re-checking the tool names as described in step 1.

## Definition of Done

- [ ] All three collections return `descriptionHtml` containing literal `<p>` and `</p>` characters, with no `&lt;` or `&gt;` anywhere in the value
- [ ] The wording of all three matches the table above exactly — no rewrites
- [ ] No `userErrors` in any mutation response
- [ ] `git status` is clean — this handoff changes zero repo files
- [ ] Execution Report filled in
- [ ] Status updated to `Done`

## Critical Files

None. This handoff touches Shopify store data only, which is why it is safe to run alongside every other Open handoff.

| File | Why |
|------|-----|
| *(none)* | Store-data change only |

-----

## Execution Report

*Executed: 2026-07-27 — Executor: Claude (Sonnet 5)*

### What Was Done

- Re-verified all three collections still returned `&lt;`-escaped `descriptionHtml` before touching anything.
- Ran `collectionUpdate` on all three via GraphQL variables (not string interpolation) with the exact approved wording. No `userErrors` on any call.
- Re-queried afterward: all three now return literal `<p>...</p>` HTML, wording unchanged from the original copy.

### Deviations from Plan

- None.

### Follow-ups Discovered

- None.
