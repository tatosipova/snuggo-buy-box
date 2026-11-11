# Snuggo – mobile hero + buy box (Shopify section)

A single Online Store 2.0 section, `sections/snuggo-buy-box.liquid`, that replaces the first
screen and buy box of the Snuggo product page. Built for the TLA web developer task; it is
not the live theme's code and copies none of it.

**Live preview (development store, best on a phone):**
https://oafq6fu3800jpqvs-76101419101.shopifypreview.com/products/snuggo
(store password if asked: `diaghe`)

## What it implements (audit issue numbers in brackets)

| Area | Behaviour | Status (Built · Store-tested · Locally tested · Pending live QA) |
| --- | --- | --- |
| First screen (1) | One announcement line; 4:5 gallery capped at 52% of the screen height; headline with the product name and benefit ("Snuggo: their favourite toy, talking back"); one line saying what Snuggo is (a small device that clips onto the plush they already love); price with saving visible on load on a 390 × 844 phone. Optional small name line above the heading (off by default) | Built · Store-tested |
| Purchase choice (1) | One / Two Snuggos as radio buttons directly below; per-unit price on the 2-pack when `custom.units` is set; saving shown per option | Built · Store-tested |
| Sticky Add to cart (1) | Visible whenever the main button is out of view, including on load (main button below the fold); hidden while the main button is visible; desktop hidden | Built · Store-tested · testable design decision |
| Safety lines (2) | Up to 4 editable "trust point" blocks under the button, optional links | Built · Locally tested (copy limited to claims the page already makes) |
| Rating (3) | Stars and count from Shopify's standard `reviews.rating` / `reviews.rating_count` metafields; nothing shown if absent; optional real quote instead | Built · Locally tested (mock data); needs authentic reviews |
| Back-to-top (4) | Theme floating buttons are lifted above the sticky bar while it shows | Built · Pending integration QA; theme-wide fix proposed |
| Afterpay line (5) | "or 4 interest-free payments of …" | Built, **off by default** |
| Ongoing cost (9) | Optional "No subscription…" field, empty by default | Built, empty by default; needs confirmation |
| Fit note (6) | The page's own FAQ answer as one line under the choice | Built · Locally tested |
| Prices | Every price, including the saving in the announcement line (`[saving]` token), comes from variant data through `money` / `money_without_trailing_zeros`, so all amounts share the storefront's currency; removing compare-at prices ends the sale display everywhere. The announcement saving always uses the first (single-pack) variant, so it never shows the 2-pack saving next to the single price | Built · Store-tested |
| Countdown | One timer, shown only before `sale_end` **and** while the selected variant has a sale price (see handover note) | Built · Locally tested |
| Availability | Choosing an option updates both buttons: enabled with the add label, or disabled with the sold-out label | Built · Locally tested |
| Theme editor | Init returns a cleanup; `shopify:section:unload` clears the timer and observers, `shopify:section:load` re-mounts | Built · Locally tested (simulated events) |

## How it works

- **Add to cart is a standard Shopify product form** (`{% form 'product' %}`). The variant sent
  to the cart is one hidden input named `id`; the One / Two radios only update it. Theme and app
  scripts that read `[name="id"]` (AJAX cart drawers often do) therefore always get the chosen
  variant. Trade-off: with JavaScript disabled, Add to cart still works but adds the default
  option (One Snuggo).
  The section adds no tracking code of its own. Whether GA4, the Meta pixel and Conversions API
  record each add-to-cart correctly (once, with the right value and currency) depends on how the
  store's integrations are set up, so this is **pending integration QA** on the development store.
- **No currency formatting in JavaScript.** Each option carries its prices pre-formatted by
  Liquid in `data-` attributes (prices, saving, Afterpay amount, availability). The script copies
  those values into the main price, saving, Afterpay line, sticky bar and both buttons when the choice
  changes, and updates `?variant=` in the URL. The gallery counter uses an IntersectionObserver on the
  slides, so it doesn't depend on the gallery's width.
- **Performance.** No libraries. The gallery is CSS scroll-snap. The first image is eager
  with `fetchpriority="high"` and responsive `srcset`; other images are lazy; Shopify videos use
  `preload="none"` with a poster image.
- **Accessibility.** Options are a `fieldset` with a `legend`; native radios (keyboard and
  screen-reader friendly); visible focus outlines; price changes announced via `aria-live`;
  44 px+ touch targets; the sticky bar is removed from the tab order while hidden; respects
  `prefers-reduced-motion`.

## Handover note: sale timing

The countdown and the sale prices are deliberately linked but not driven by one source:
the countdown needs `sale_end` (a section setting) **and** a compare-at price on the selected
variant; the prices themselves only change when compare-at prices are edited in the product
admin. When the sale ends, the countdown and the announcement saving disappear on their own,
but **someone must still remove or update the compare-at prices** at 23:59 AEDT on 30 Nov
(manually or with a scheduled price change). Until then the page keeps showing the sale price
with no countdown.

## Tested on a Shopify development store (11 Nov 2026)

Shopify test theme; product set up like snuggo.com.au (One Snuggo $199 / compare-at $299,
Two Snuggos $349 / $598). Checked in iPhone Safari and Chrome's iPhone view, including
through an Australian VPN connection.

- First screen at iPhone size: name and benefit headline, what Snuggo is, price and saving, Add to cart.
- Switching One / Two updates price, saving and sticky bar.
- Cart: One Snuggo adds at $199, Two Snuggos at $349 (main button and sticky bar).
- Sticky bar shows while the main button is out of view and hides when it's visible.
- Fixed during store testing: the announcement saving opened as the 2-pack saving when the URL had
  `?variant=` for the 2-pack (now always the single-pack saving); the cart now reads one hidden
  `id` field (robust with theme cart drawers).
- Note: on preview links, Shopify's own preview bar can cover the sticky bar. Real visitors don't see it.

## Also tested locally (preview with mock data, Chrome mobile emulation)

- iPhone 390 × 844, Android 360 × 780, 320 px and 768 px widths: no script errors, no horizontal overflow.
- Choosing Two Snuggos updates price, compare-at, saving, Afterpay amount and sticky bar.
- Selected variant sold out → both buttons "Sold out" and disabled; choosing an available one re-enables both.
- No compare-at prices → announcement falls back, countdown hidden.
- Gallery counter correct after scrolling to slide 3.
- Simulated theme-editor `unload` / `load` events: cleanup runs, section re-mounts.
- Shopify Theme Check: 0 offenses.

Not yet tested: checkout, the live theme and its apps, other markets and currencies, tracking
events, performance on real traffic and screen readers. These are **pending QA on the live store**.

## Assumptions — check before using on the live store

1. **The 2-pack is a product variant.** The live page shows One at $199 and Two at $349, but
   from outside it's impossible to tell whether "Two" is a variant, a bundle product, a
   quantity discount or a cart script. This section assumes two variants of one product.
   If the live store uses something else, the option markup must change before launch.
2. **Variant metafields** `custom.tagline` (single-line text) and `custom.units` (integer, `2` on
   the 2-pack) exist; both are optional.
3. **Reviews** use Shopify's standard product rating metafields. Other review apps may need
   their own snippet in place of the rating block.
4. **Theme floating buttons** use `#scroll-to-top-button` or `.scroll-to-top`; other themes may
   need that selector changed.

## Acceptance criteria (checked = passed on the development store)

- [x] On a 390 × 844 phone, name, promise and price with saving are visible without scrolling.
- [x] Selecting Two Snuggos updates the main price, saving, Afterpay line (if on) and sticky bar.
- [x] Add to cart (main and sticky) adds the selected variant, quantity 1, at the right price in the cart.
- [ ] Checkout shows the same price and product (not checked).
- [ ] All prices on the section show one currency, on every storefront and market.
- [x] The sticky bar shows while the main button is out of view, hides while it is visible, and
      never covers another control (apart from Shopify's preview bar, preview links only).
- [ ] Add to cart works with JavaScript disabled (adds the default option; see trade-off above).
- [ ] GA4, Meta pixel and Conversions API each record one add-to-cart per add, with correct
      value and currency (see the test plan's tracking checks).
- [ ] Mobile LCP stays under 2.5 s; no layout shift when the sticky bar appears.
- [ ] Keyboard-only: options, button and links are reachable and visibly focused.
- [ ] The countdown hides after `sale_end`; delivery and Christmas cut-off copy is correct for
      the storefront's market.

## Setup on a store

1. Online Store → Themes → Edit code → `sections` → Add a new section → paste the file.
2. Product: two variants, "One Snuggo" ($199, compare-at $299) and "Two Snuggos"
   ($349, compare-at $598). Optional variant metafields as above. Drag the video to media position 2.
3. Theme editor → product template → Add section → **Snuggo buy box**. Hide the theme's default
   product information and related products sections and its announcement bars.
4. Media order used on the development store: the lifestyle photo first, the device-only photo
   second (one swipe away). Choosing the first photo is the brand's call and worth testing.
5. Everything else — heading, one-line promise, trust points, dates, colours, Afterpay, sticky bar —
   is in the section's settings.
