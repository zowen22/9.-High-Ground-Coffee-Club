# Shop Policies — text to paste into Settings → Policies

*Created: 2026-07-27. Blocks audit finding P0-4.*

**Why this is a file and not something Claude did for you:** the Shopify MCP connector is missing the `write_legal_policies` scope, so `shopPolicyUpdate` is permanently blocked. These have to be pasted in by hand at **Settings → Policies**.

**Not legal advice.** These are drafts written to match the terms you confirmed on 2026-07-27 (US-only shipping, $8 standard / free over $70 / $15 express, 30-day returns with customer-paid return shipping, no satisfaction guarantee, monthly subscription with 10% member discount). Your attorney already reviewed the equivalent custom Pages; have them look at these too, since these are the ones that appear inside checkout.

**Privacy policy:** already set and rendering correctly. Don't touch it — except see the note at the bottom about your address and phone.

-----

## Refund policy

> We want you to enjoy your coffee. If something isn't right, you may request a return within **30 days** of delivery.
>
> To be eligible, the item must be unopened and in its original packaging. Return shipping is paid by the customer. Once we receive and inspect the return, we'll issue a refund to your original payment method within 5–10 business days.
>
> Coffee that has been opened cannot be returned for hygiene and food-safety reasons. If your order arrives damaged, or we sent the wrong item, contact us within 30 days and we'll make it right at no cost to you — don't ship it back until we've replied.
>
> To start a return, email us at customadditiveengineering@gmail.com with your order number.

## Shipping policy

> We currently ship within the **United States only**.
>
> - Standard shipping: **$8**, free on orders of **$70 or more**
> - Express shipping: **$15**
>
> Orders are roasted and dispatched as quickly as we can after they're placed. Once your order ships, delivery times depend on the carrier and your location.
>
> We are not responsible for delays caused by the carrier, incorrect addresses supplied at checkout, or packages marked delivered but reported missing. If a package is lost in transit, contact us and we'll help you open a claim with the carrier.
>
> Questions about a shipment: customadditiveengineering@gmail.com

## Terms of service

> By placing an order with Brown Bag Coffee Club, you agree to these terms.
>
> **Orders and pricing.** All prices are in U.S. dollars. We may change prices, products, and availability at any time. We reserve the right to refuse or cancel any order, including orders that appear fraudulent or that contain a pricing error.
>
> **Accuracy.** We work to keep product descriptions, tasting notes, and imagery accurate, but we do not warrant that all content is error-free. Coffee origin and tasting notes rotate and may change between shipments.
>
> **Your account.** You are responsible for keeping your account credentials secure and for activity that occurs under your account.
>
> **Limitation of liability.** To the fullest extent permitted by law, our liability for any claim relating to an order is limited to the amount you paid for that order.
>
> **Governing law.** These terms are governed by the laws of the State of Ohio, United States.
>
> **Contact.** customadditiveengineering@gmail.com

## Subscription policy

> **Billing.** Subscriptions bill **monthly**. Your first charge occurs when you place the order; each renewal is charged on the same date each month using your saved payment method. Subscribers save **10%** on every shipment.
>
> **Pause, skip, or cancel.** You can pause, skip a shipment, or cancel anytime from your account. Changes take effect for the next shipment, so make them **before** your renewal date — once an order has been charged and roasted, it can't be pulled back.
>
> **Changes to your plan.** Roast profile, format, and quantity can be changed from your account and apply to your next shipment.
>
> **Price changes.** If subscription pricing changes, we'll notify you before it takes effect and you may cancel before the next renewal.
>
> **Failed payments.** If a renewal payment fails, we'll retry. If it continues to fail, the subscription may be paused or cancelled.
>
> **Contact.** customadditiveengineering@gmail.com

-----

## One thing to fix before you paste

Your **Privacy policy is publicly published right now** with your street address and phone number in it:

> "please call **+1 330-614-2860** or email us at customadditiveengineering@gmail.com or contact us at **1150 Overlook Dr, Alliance OH 44601, United States**"

Shopify auto-fills those from **Settings → Store details** (currently `1150 Overlook Dr, Alliance, Ohio 44601` with phone `13306142860` on the shop's billing address). Change them there and the policy text updates itself — the merge tags re-render on the next view.

**Claude cannot do this for you.** Verified 2026-07-27 by enumerating the full Admin GraphQL `Mutation` type: the only shop-level mutations that exist are `shopLocaleEnable` / `shopLocaleDisable` / `shopLocaleUpdate`, `shopPolicyUpdate` (blocked — missing scope), and `shopResourceFeedbackCreate`. There is **no mutation for store details, business address, or store phone** — same permanent dead end as the domain settings. Admin UI only.

Two constraints worth knowing before you go in:

1. **You probably can't blank them.** Shopify requires a business address for a live store, and several jurisdictions require a contact address in a privacy policy. A **PO box** plus a forwarding or Google Voice number is the normal answer — replace rather than delete.
2. **There may be two addresses to change.** The billing address (shown above) is what the policy is pulling. Shopify also has a separate store/sender address in some setups. Check both while you're in there.
