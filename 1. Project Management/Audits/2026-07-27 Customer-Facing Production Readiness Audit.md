# Audit: Customer-Facing Production Readiness

*Status: `Open`*
*Date: 2026-07-27 — Auditor: Claude (Opus, planner session)*
*Scope: what a customer sees and does — storefront reachability, content correctness, navigation, product/collection pages, cart, checkout-facing policy and shipping presentation.*
*Explicitly out of scope this pass: security, performance/Core Web Vitals, back-office/admin ergonomics, analytics/ads instrumentation, accessibility conformance testing.*

-----

## Corrections — 2026-07-27, same day

Three findings in the original pass were wrong. All three shared one cause: **customer-visible rendering was inferred from stored API values instead of being checked against rendered output.** Recorded here rather than silently edited, because it also calibrates how much the rest of this document should be trusted.

| Finding | Original claim | What's actually true |
|---|---|---|
| P0-4 (part) | Privacy policy displays raw Liquid merge tags at checkout | **Wrong.** Shopify renders policy Liquid at display time. The live policy shows real values. Verified by fetching the policy URL. The *other* half of P0-4 — Refund, Terms, Shipping, and Subscription policies not existing — still stands. |
| P1-6 | Two duplicate "Standard" domestic shipping rates shown at checkout | **Wrong.** There is one `DeliveryMethodDefinition` (`…722513`). The `$0` entry is the same definition surfaced with `?source=RateRangeCondition` — Shopify models "free over $70" as a rate-range condition on the Standard rate. Checkout shows one line, priced $8 or $0 by cart total. Nothing to de-duplicate. |
| P2-9 | International rates display as lowercase `usps` / `dhl_express` | **Wrong.** Those are internal identifiers; `carrierService.formattedName` returns "USPS (Discounted rates from Shopify Shipping)" and "DHL Express (Discounted rates from Shopify Shipping)". Moot regardless — the international zone was deleted. |

One finding was **understated**: P2-7 said the footer social block "renders empty." It was in fact configured with Vessel's demo defaults — five live icons linking to `facebook.com`, `instagram.com`, `youtube.com`, `tiktok.com`, and `twitter.com`, i.e. sending customers to those platforms' front doors. Fixed.

**New finding, P0-5**, surfaced while verifying the privacy policy: the rendered policy publishes the owner's **street address and phone number** (`1150 Overlook Dr, Alliance OH 44601`, `+1 330-614-2860`), auto-filled from Settings → Store details. The owner has since stated they do not want an address or phone associated with the store. This is live on a public URL now.

## Verdict

**Not production-ready.** Two hard blockers mean no customer can currently buy anything (storefront is password-protected; Shopify Payments is not enabled), and even if both were lifted today, a visitor would hit 404s on 10 of 16 navigation links and read furniture-store copy on every product page.

The commerce *plumbing* is in good shape — catalog, variants, weights, selling plans, SEO metadata, shipping zones, domain, SSL all check out. The gap is presentation and publish state.

Findings below are ordered by customer impact. Severity: **P0** = blocks purchase, **P1** = visibly broken/wrong to a visitor, **P2** = trust/conversion cost.

-----

## P0 — Blocks Purchase

| # | Finding | Evidence | Owner |
|---|---------|----------|-------|
| P0-1 | **Storefront is password-protected.** Every visitor gets the "Opening soon" splash. Nothing else in this audit is reachable by a customer until this is off. | `onlineStore.passwordProtection.enabled = true` (Admin API); `templates/password.json` renders `<h1>Opening soon</h1>` | @user (Admin → Online Store → Preferences) |
| P0-2 | **Shopify Payments not enabled.** Checkout cannot process payment. Carried over from WP2.2. | Known open item; requires real banking/tax info | @user |
| P0-3 | **All 10 content pages are unpublished.** 3 of 7 main-nav links (Our Coffee, How It Works, FAQ) and 8 of 10 footer links resolve to 404. Menus are correctly wired — the targets just aren't live. | `pages` query: `isPublished: false` on about-us, how-it-works, our-coffee, brew-guides, faqs, contact-us, terms-conditions, privacy-policy, cookie-policy, accessibility-statement | @user (fill placeholders) → @claude (publish) |
| P0-4 | **Four of five Shop Policies don't exist.** Only `PRIVACY_POLICY` is set (and it renders correctly — see Corrections). Refund, Terms of Service, Shipping, and Subscription policies are absent, so checkout links to nothing. Refund terms are now decided (30 days, customer pays return shipping, no guarantee) and need pasting in. | `shop.shopPolicies` returns a single node | @user (MCP lacks `write_legal_policies`) |
| P0-5 | **Owner's street address and phone are published** in the live privacy policy, auto-filled from store settings, against the owner's stated preference. | Rendered policy URL, verified 2026-07-27 | @user (Settings → Store details) |

> Note on P0-4: this is distinct from the custom legal *Pages* (P0-3). Shop Policies are what render inside checkout and in the footer policy list. Both are currently broken, for different reasons.

## P1 — Visibly Broken or Wrong

| # | Finding | Evidence | Owner |
|---|---------|----------|-------|
| P1-1 | **Vessel furniture demo copy is live on every product page.** A visitor buying coffee reads an accordion titled "CARE & MAINTENANCE" advising "gentle washing and proper storage… refer to the care instructions included with each item," and a section headed "Embracing small joys" about items that "blend harmoniously with your living space." **This is a regression** — the Session Log records these as fixed on 2026-07-14; the repo currently holds the original demo text. Most likely reverted by a `shopify[bot]` bidirectional-sync commit. | `templates/product.json` — sections `main` (accordion rows) and `media_with_content_kpQCTG` | @claude — HO-01 |
| P1-2 | **Footer newsletter heading is demo copy**: "Get your seat at the table" — furniture-store voice, not coffee. | `sections/footer-group.json`, `email-signup` block | @claude — HO-01 |
| P1-3 | **Footer has an empty column and a mislabeled one.** Menu block #1 is labeled "Shop" but points at the `footer` menu (About Us, Brew Guides, FAQ, Contact, legal) — nothing shoppable. Menu block #2 is headed "About us" with no menu assigned, rendering a heading over blank space. | `sections/footer-group.json` — `menu` blocks | @claude — HO-01 |
| P1-4 | **Collection descriptions render as literal HTML.** All three real collections store double-escaped markup, so the page displays the characters `<p>All the flavor, none of the caffeine.</p>` instead of the sentence. | `collections.descriptionHtml` returns `&lt;p&gt;…&lt;/p&gt;` for decaf, gifts, coffee-subscriptions. Product descriptions are unaffected — correct HTML. | @claude — HO-03 |
| P1-5 | **Homepage has an empty section.** The `collection-list` section has `block_order: []` — no collection cards assigned. It occupies vertical space at the bottom of the homepage and shows nothing. | `templates/index.json` → `collection_list_bGtkjG` | @claude — HO-02 |
| ~~P1-6~~ | ~~Duplicate "Standard" shipping rates~~ — **withdrawn, see Corrections.** One rate with a free-over-$70 condition. Working as intended. | — | — |
| P1-7 | **The Contact Us page has no contact form.** The linked page (`/pages/contact-us`, in the footer menu) uses the default `page` template — text only. The contact form lives in `templates/page.contact.json`, which serves the orphaned, empty, published `/pages/contact` demo page nobody links to. Customers get contact copy with no way to submit anything. | `templates/page.contact.json` has the `contact-form` section; `templates/page.json` does not | @claude — HO-04 |

## P2 — Trust and Conversion Cost

| # | Finding | Evidence | Owner |
|---|---------|----------|-------|
| P2-1 | **No logo.** Header renders the shop name as plain text and the "Opening soon" splash has an empty logo block. **Favicon resolved 2026-07-27** — a flat brown-paper-bag icon was generated, uploaded to Shopify Files as `favicon-brown-bag.png`, and wired to the theme's `favicon` setting. Logo still needs a real brand asset. | `config/settings_data.json` | @user (logo asset) |
| P2-2 | **Homepage is thin.** Three usable sections: hero, two product rows. No value props, no how-it-works, no flexibility messaging. Owner decision 2026-07-27: match the Atlas structure, **minus** the money-back guarantee. Press mentions remain excluded — Atlas's coverage is not ours to claim. | `templates/index.json` — 4 sections, one of them empty (P1-5) | @claude — HO-05 |
| P2-3 | **Hero has no visible call-to-action.** The whole hero is a click target for `/collections`, but there is no button — the only text is the `<h1>`. No subhead, no price anchor, no "Start your subscription." | `templates/index.json` → `hero_irjTFL`, single `text` block | @claude — HO-02 |
| P2-4 | **Free-shipping threshold is invisible.** Free Standard shipping over $70 exists in the shipping config but is never mentioned — no announcement bar, no cart progress indicator, no PDP line. A live incentive nobody is told about. | `deliveryProfiles` vs. absence of any `$70`/"free shipping" string in `templates/` or `sections/` | @claude — HO-02 |
| P2-5 | **Store contact email is an unrelated business address**: `customadditiveengineering@gmail.com`. This is the address customers see on Shopify's transactional emails and the one that fills the `{{ email }}` merge tag in policies. | `shop.contactEmail` | @user |
| P2-6 | **Gift Subscription has no gift-note field.** A gift buyer cannot say who it's from. Owner decision 2026-07-27: add the gift note; **do not** add a delivery date or recipient email — no fulfillment process backs either. | `products` query — Gift Subscription `options` = `["Bundle"]` only | @claude — HO-06 |
| P2-7 | ~~**RESOLVED 2026-07-27.**~~ Footer social block was linking to `facebook.com`/`instagram.com`/etc. front doors (Vessel demo defaults), not rendering empty as originally reported. Owner confirmed no social accounts exist; block removed. | `sections/footer-group.json` | Done |
| P2-8 | **Product page shows a reviews block with no reviews app installed.** May render an empty/zero rating next to the price on every PDP — needs verification against the block's Liquid before removal. | `templates/product.json` → `review` block inside the price group | @claude — HO-01 |
| ~~P2-9~~ | ~~International rates display as raw carrier codes~~ — **withdrawn, see Corrections.** Moot regardless: international zone deleted 2026-07-27, store is US-only per owner. | — | — |
| P2-10 | **All imagery is stock placeholder.** Hero, all 6 products, all 3 collections use free-license Unsplash photography. Known and documented — restating because it is a launch gate, not a nicety. | Technical Reference; `templates/index.json` hero `image_1` | @user (photography) |
| P2-11 | **Leftover demo artifacts.** The Vessel `frontpage` ("Home page") collection still exists with 1 product; the empty `/pages/contact` page is published and indexable with a blank body. | `collections` query; `pages` query | @claude — HO-04 |
| P2-12 | **Country and language selectors are on** in the header on what is now confirmed a single-market, US-only store (one Market: "United States", primary; international shipping zone deleted). Turn both off. | `sections/header-group.json` → header settings | @claude — HO-02 |

## What Checked Out Clean

Worth recording so a later session doesn't re-audit it: all 6 products Active with images and SEO title/description; selling plans correctly attached to all 5 coffee products and correctly absent from the Gift product; variant option structure (Roast Profile × Quantity) matches the intended plan-builder UX; variant weights populated; product `descriptionHtml` is valid HTML with real tasting-note copy; main and footer menus resolve to real resources with no dead handles; domestic and international shipping zones exist and are active; custom domain live, primary, SSL enabled; 404 and cart templates have sensible copy and product recommendations.

## Owner Decisions — resolved 2026-07-27

| Question | Decision |
|---|---|
| Expand homepage to Site Map spec? | **Yes — match the Atlas structure, except the money-back guarantee.** Press mentions stay excluded; Atlas's coverage isn't ours to claim. → HO-05 |
| Gift note / delivery date on Gift product? | **Gift note yes, delivery date no** — no fulfillment process backs a set delivery date. Recipient email also out, same reason. → HO-06 |
| Fate of orphaned `/pages/contact`? | Unpublish, don't delete → HO-04 |
| Shipping rate naming? | Moot — the "duplicate" was a misread (see Corrections). Separately: **US-only shipping**, international zone deleted. |
| Support email? | Keep `customadditiveengineering@gmail.com` for now |
| Returns terms? | **30 days, customer pays return shipping, no satisfaction guarantee** |
| Social accounts? | None — footer block removed |
| Favicon? | Brown paper bag — generated and wired up |
| Photography? | Owner will supply later; stock placeholders stay for now |

### Still open

- **Logo asset** (P2-1) — needs real brand design, nothing to fabricate.
- **What actually goes in the box** — blocks the "what's in the box" homepage section in HO-05. Atlas ships a postcard from the origin country and a coffee-history insert; we have never committed to either. HO-05 will not invent contents.

## Handoffs Issued

| Handoff | Covers | Priority | Parallel-safe |
|---------|--------|----------|---------------|
| HO-01 — Purge Vessel demo copy | P1-1, P1-2, P1-3, P2-8 | High | Yes |
| HO-02 — Homepage and header fixes | P1-5, P2-3, P2-4, P2-12 | High | Yes |
| HO-03 — Fix collection description escaping | P1-4 | High | Yes |
| HO-04 — Contact form and demo-artifact cleanup | P1-7, P2-11 | Medium | Yes |
| HO-05 — Build out homepage to match Atlas | P2-2 | Medium | No — after HO-02 |
| HO-06 — Gift note on Gift Subscription | P2-6 | Medium | No — after HO-01 |

HO-01 through HO-04 are parallel-safe against each other: HO-01 owns `templates/product.json` + `sections/footer-group.json`, HO-02 owns `templates/index.json` + `sections/header-group.json`, and HO-03/HO-04 touch no repo files at all. HO-05 must wait for HO-02 (same file); HO-06 must wait for HO-01 (it copies `product.json`, which HO-01 fixes).

@user-owned items (P0-1, P0-2, P0-4, P0-5, P2-1, P2-5, P2-10) have no handoff — they require Admin actions or real-world assets Claude cannot supply. They are tracked in Work Packages.

**Resolved by the planner session on 2026-07-27, no handoff needed:** international shipping zone deleted (US-only), favicon generated and wired up, footer social block removed.
