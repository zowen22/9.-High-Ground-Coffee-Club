# Audit: Customer-Facing Production Readiness

*Status: `Open`*
*Date: 2026-07-27 — Auditor: Claude (Opus, planner session)*
*Scope: what a customer sees and does — storefront reachability, content correctness, navigation, product/collection pages, cart, checkout-facing policy and shipping presentation.*
*Explicitly out of scope this pass: security, performance/Core Web Vitals, back-office/admin ergonomics, analytics/ads instrumentation, accessibility conformance testing.*

-----

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
| P0-4 | **Shop Policies are broken at checkout.** Only `PRIVACY_POLICY` exists and it is still Shopify's raw template with unrendered Liquid tags — a customer clicking "Privacy policy" in checkout reads `Last updated: {{ last_updated }}` and `please call {{ phone }} or email us at {{ email }}`. Refund, Terms of Service, Shipping, and Subscription policies do not exist at all. | `shop.shopPolicies` — single node, body contains literal `{{ last_updated }}`, `{{ shop_name }}`, `{{ phone }}`, `{{ email }}`, `{% if %}` blocks | @user (MCP lacks `write_legal_policies`) |

> Note on P0-4: this is distinct from the custom legal *Pages* (P0-3). Shop Policies are what render inside checkout and in the footer policy list. Both are currently broken, for different reasons.

## P1 — Visibly Broken or Wrong

| # | Finding | Evidence | Owner |
|---|---------|----------|-------|
| P1-1 | **Vessel furniture demo copy is live on every product page.** A visitor buying coffee reads an accordion titled "CARE & MAINTENANCE" advising "gentle washing and proper storage… refer to the care instructions included with each item," and a section headed "Embracing small joys" about items that "blend harmoniously with your living space." **This is a regression** — the Session Log records these as fixed on 2026-07-14; the repo currently holds the original demo text. Most likely reverted by a `shopify[bot]` bidirectional-sync commit. | `templates/product.json` — sections `main` (accordion rows) and `media_with_content_kpQCTG` | @claude — HO-01 |
| P1-2 | **Footer newsletter heading is demo copy**: "Get your seat at the table" — furniture-store voice, not coffee. | `sections/footer-group.json`, `email-signup` block | @claude — HO-01 |
| P1-3 | **Footer has an empty column and a mislabeled one.** Menu block #1 is labeled "Shop" but points at the `footer` menu (About Us, Brew Guides, FAQ, Contact, legal) — nothing shoppable. Menu block #2 is headed "About us" with no menu assigned, rendering a heading over blank space. | `sections/footer-group.json` — `menu` blocks | @claude — HO-01 |
| P1-4 | **Collection descriptions render as literal HTML.** All three real collections store double-escaped markup, so the page displays the characters `<p>All the flavor, none of the caffeine.</p>` instead of the sentence. | `collections.descriptionHtml` returns `&lt;p&gt;…&lt;/p&gt;` for decaf, gifts, coffee-subscriptions. Product descriptions are unaffected — correct HTML. | @claude — HO-03 |
| P1-5 | **Homepage has an empty section.** The `collection-list` section has `block_order: []` — no collection cards assigned. It occupies vertical space at the bottom of the homepage and shows nothing. | `templates/index.json` → `collection_list_bGtkjG` | @claude — HO-02 |
| P1-6 | **Duplicate "Standard" shipping rates at checkout.** Domestic zone has two active rates both named "Standard": $8.00 unconditional, and $0.00 conditioned on cart ≥ $70. Any order over $70 shows both, identically named, at different prices. The $8 rate has no upper-bound condition. | `deliveryProfiles` → Domestic zone `methodDefinitions` | @user (Admin → Shipping) |
| P1-7 | **The Contact Us page has no contact form.** The linked page (`/pages/contact-us`, in the footer menu) uses the default `page` template — text only. The contact form lives in `templates/page.contact.json`, which serves the orphaned, empty, published `/pages/contact` demo page nobody links to. Customers get contact copy with no way to submit anything. | `templates/page.contact.json` has the `contact-form` section; `templates/page.json` does not | @claude — HO-04 |

## P2 — Trust and Conversion Cost

| # | Finding | Evidence | Owner |
|---|---------|----------|-------|
| P2-1 | **No logo and no favicon.** Header renders the shop name as plain text; the browser tab shows Shopify's default icon; the password/"Opening soon" splash has a logo block with nothing in it. Needs real brand assets. | `config/settings_data.json` — no `logo`/`favicon` keys, only `logo_height` | @user (assets) |
| P2-2 | **Homepage is thin.** Three usable sections: hero, two product rows. The Site Map specifies value props, a how-it-works summary, and social proof; the benchmark site leads with flexibility messaging ("pause, skip, cancel anytime"), a money-back guarantee, and free-shipping. None of that appears anywhere on the storefront. | `templates/index.json` — 4 sections, one of them empty (P1-5) | @claude — needs scope go-ahead, see Decisions Needed |
| P2-3 | **Hero has no visible call-to-action.** The whole hero is a click target for `/collections`, but there is no button — the only text is the `<h1>`. No subhead, no price anchor, no "Start your subscription." | `templates/index.json` → `hero_irjTFL`, single `text` block | @claude — HO-02 |
| P2-4 | **Free-shipping threshold is invisible.** Free Standard shipping over $70 exists in the shipping config but is never mentioned — no announcement bar, no cart progress indicator, no PDP line. A live incentive nobody is told about. | `deliveryProfiles` vs. absence of any `$70`/"free shipping" string in `templates/` or `sections/` | @claude — HO-02 |
| P2-5 | **Store contact email is an unrelated business address**: `customadditiveengineering@gmail.com`. This is the address customers see on Shopify's transactional emails and the one that fills the `{{ email }}` merge tag in policies. | `shop.contactEmail` | @user |
| P2-6 | **Gift Subscription is missing its gifting features.** The Site Map specifies a gift note plus mail-or-email delivery. The product has one option axis (Bundle: 4-coffee / 8-coffee) and no gift-message field, recipient email, or delivery-date input. A gift buyer cannot say who it's from. | `products` query — Gift Subscription `options` = `["Bundle"]` only | @claude — needs scope go-ahead |
| P2-7 | **Footer social links render empty.** A `social-links` block is present with no social accounts configured. | `sections/footer-group.json` → `utilities` → `social-links`; no `social_*` keys in settings | @user (accounts) → @claude |
| P2-8 | **Product page shows a reviews block with no reviews app installed.** May render an empty/zero rating next to the price on every PDP — needs verification against the block's Liquid before removal. | `templates/product.json` → `review` block inside the price group | @claude — HO-01 |
| P2-9 | **International shipping rates display as raw carrier codes** — `usps` and `dhl_express`, lowercase, as both name and description. Needs verification of how these actually present at checkout. | `deliveryProfiles` → International zone `methodDefinitions` | @user (verify + rename) |
| P2-10 | **All imagery is stock placeholder.** Hero, all 6 products, all 3 collections use free-license Unsplash photography. Known and documented — restating because it is a launch gate, not a nicety. | Technical Reference; `templates/index.json` hero `image_1` | @user (photography) |
| P2-11 | **Leftover demo artifacts.** The Vessel `frontpage` ("Home page") collection still exists with 1 product; the empty `/pages/contact` page is published and indexable with a blank body. | `collections` query; `pages` query | @claude — HO-04 |
| P2-12 | **Country and language selectors are on** in the header (`show_country`, `show_language`) on a single-currency USD store. Verify whether markets justify them; otherwise they add a decision the customer doesn't need. | `sections/header-group.json` → header settings | @claude — HO-02 |

## What Checked Out Clean

Worth recording so a later session doesn't re-audit it: all 6 products Active with images and SEO title/description; selling plans correctly attached to all 5 coffee products and correctly absent from the Gift product; variant option structure (Roast Profile × Quantity) matches the intended plan-builder UX; variant weights populated; product `descriptionHtml` is valid HTML with real tasting-note copy; main and footer menus resolve to real resources with no dead handles; domestic and international shipping zones exist and are active; custom domain live, primary, SSL enabled; 404 and cart templates have sensible copy and product recommendations.

## Decisions Needed From @user

1. **P2-2 homepage build-out** — expand the homepage to the Site Map spec (value props, how-it-works, guarantee, social proof)? This is net-new content, not a fix, so it isn't in any handoff yet. Press mentions in particular can't be fabricated — the benchmark's social proof has no equivalent here.
2. **P2-6 gift fields** — add gift note / recipient email / delivery date? Native Shopify needs either line-item properties in theme code or a gifting app. Affects HO scope.
3. **P2-11 orphan `/pages/contact`** — delete, or keep and repurpose as the contact form target? HO-04 assumes *repurpose the template, delete the page*; say otherwise and it stops.
4. **P1-6 shipping rate naming** — confirm the intended presentation: rename the $0 rate to "Free Standard Shipping (orders $70+)" and cap the $8 rate below $70, or keep both visible?

## Handoffs Issued

| Handoff | Covers | Priority | Parallel-safe |
|---------|--------|----------|---------------|
| HO-01 — Purge Vessel demo copy | P1-1, P1-2, P1-3, P2-8 | High | Yes |
| HO-02 — Homepage and header fixes | P1-5, P2-3, P2-4, P2-12 | High | Yes |
| HO-03 — Fix collection description escaping | P1-4 | High | Yes |
| HO-04 — Contact form and demo-artifact cleanup | P1-7, P2-11 | Medium | Yes |

All four are parallel-safe against each other: HO-01 owns `templates/product.json` + `sections/footer-group.json`, HO-02 owns `templates/index.json` + `sections/header-group.json`, and HO-03 and HO-04 touch no repo files at all (Shopify Admin API only).

@user-owned items (P0-1, P0-2, P0-4, P1-6, P2-1, P2-5, P2-7, P2-9, P2-10) have no handoff — they require Admin actions or real-world assets Claude cannot supply. They are tracked in Work Packages.
