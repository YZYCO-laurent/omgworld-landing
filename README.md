# OMG Fibre Soda — LT landing page

Single-page Lithuanian conversion funnel for the OMG Fibre Soda Variety Pack.
Static HTML, no build step, no dependencies. One request, no framework.

Live target: **https://go.omgworld.com**

---

## Before you deploy — one required edit

Open `index.html`, find the `CFG` block near the bottom (~line 780) and set the Meta Pixel ID:

```js
var CFG = {
  shop:      'https://omgworld.com',
  variantId: '57963807605067',   // Fibre Soda Variety Pack — 12 pack
  qty:       1,
  discount:  'OMGLT10',
  locale:    'lt',
  price:     22.87,
  currency:  'EUR',
  pixelId:   'REPLACE_WITH_PIXEL_ID',   // <-- set this
  ...
};
```

Until `pixelId` is set, the page works normally but fires no Meta events.

Everything else about the offer — price, discount code, variant, currency — is
configured in that same block. Changing the price on the page requires editing the
displayed strings too (search for `22,87`).

---

## Deploy

### Option A — GitHub + Vercel (recommended)

Gives you auto-deploy on push, a preview URL per commit for client review, and
one-click rollback.

```bash
git init
git add .
git commit -m "OMG Fibre Soda LT landing page"
git branch -M main
git remote add origin git@github.com:<org>/omg-lp-lt.git
git push -u origin main
```

Then in Vercel: **Add New → Project → Import** the repo.

- Framework preset: **Other**
- Build command: *(leave empty)*
- Output directory: *(leave empty — repo root is served)*
- Install command: *(leave empty)*

### Option B — Vercel CLI

```bash
npm i -g vercel
vercel --prod
```

### Option C — drag and drop

Zip this folder and drop it on the Vercel dashboard. Fine for a quick client
preview, but you lose version history.

---

## Domain — this part matters

Point **`go.omgworld.com`** at the deployment:

| Type  | Name | Value                 |
|-------|------|-----------------------|
| CNAME | `go` | `cname.vercel-dns.com` |

**Do not run the live campaign on a `*.vercel.app` URL.** The Meta pixel writes
`_fbp` / `_fbc` as first-party cookies on the registrable domain. Serving the page
from a subdomain of `omgworld.com` means those identifiers carry through to the
Shopify checkout; serving it from `vercel.app` means they do not, and purchases
arrive unattributed. A `vercel.app` URL is fine for the internal checkpoint review.

---

## How checkout works

Every CTA links to a Shopify cart permalink:

```
https://omgworld.com/cart/57963807605067:1?discount=OMGLT10&locale=lt
```

This skips the English storefront entirely and drops the visitor straight into a
Lithuanian Shopify checkout with the €10 discount already applied.

The page also forwards `fbclid`, `ttclid`, `gclid` and all `utm_*` parameters from
the incoming ad click onto that URL, so Shopify's own pixel can stitch the session.

Verified live on 16 Aug 2026: checkout renders in Lithuanian, total `22,87 €`,
shipping `NEMOKAMAI`, discount label `IŠ VISO SUTAUPYTA 10,00 €`.

---

## Tracking

Consent-gated — the pixel does not load until the visitor accepts. Events fired
before a decision are queued and replayed on accept, discarded on decline.

| Event | Type | Fires when |
|---|---|---|
| `PageView` | standard | Page load (post-consent) |
| `ViewContent` | standard | Page load |
| `LP_QuizStart` | custom | First quiz answer |
| `LP_QuizComplete` | custom | Result shown — carries `fibre_score`, `fibre_gap`, `bracket` |
| `Lead` | standard | Email submitted on the result screen |
| `AddToCart` | standard | Any CTA click, carries `cta_location` |
| `LP_CheckoutClick` | custom | Same click — used to separate LP intent from Shopify's own events |
| `LP_Scroll` | custom | 25 / 50 / 75 / 100% depth |

`InitiateCheckout` and `Purchase` are **not** fired here — Shopify's own Meta
integration owns those. Connect the pixel in the Shopify admin so those events
and CAPI work.

---

## Email capture

The quiz result form posts to Shopify's `/contact` newsletter endpoint through a
hidden iframe, tagging the customer `lp-fibre-quiz,lt-launch`. It does not navigate
away from the page.

Worth confirming on the first live submission that the customer record appears in
Shopify. If it does not, swap the form action for Klaviyo/Omnisend — the surrounding
markup and the `Lead` event do not change.

---

## Known items to confirm with the client

1. **Delivery estimate.** Shopify checkout quoted 25–28 August for an order placed
   16 August. All "1–2 day" claims were removed from the page rather than contradict
   the checkout — restore them once delivery settings are fixed.
2. **Duplicate shipping rate.** Two domestic rates both named "Free shipping", one at
   €1.00.
3. **Fonts.** Loaded from Google Fonts. If GDPR posture requires it, self-host
   Poppins and Inter and drop the `<link>` tags.
4. **Reviews.** The four Lithuanian reviews are the client's own approved copy from
   the campaign brief.

---

## Files

```
index.html        the entire page — markup, CSS, JS
img/benefits.*    can-in-hand shot with callouts (webp + jpg fallback)
vercel.json       caching + security headers
```

Product photography is hotlinked from the client's own Shopify CDN, which is fast
and keeps imagery in sync with the store.
