# OMG Fibre Soda — LT landing pages

Three design variants of the same funnel. Static HTML, no build step, no
dependencies, one file per page.

| File | URL after deploy | Concept |
|---|---|---|
| `index.html` | `/` | **A — brand-faithful.** The omgworld.com look, layout defects fixed. |
| `v2.html` | `/v2` | **B — quiz-first.** The fibre test *is* the landing page. |
| `v3.html` | `/v3` | **C — product-first.** PDP structure on a neutral canvas. |

All three share the same offer, the same Shopify checkout permalink, the same
pixel and the same quiz logic. They differ only in structure and visual system,
so a test between them measures layout, not copy.

---

## Before you deploy

Nothing. The Meta Pixel ID (`1031494262801665`) and the variant ID are already
set in each file's `CFG` block. The only thing that must change before the
campaign goes live is the domain — see **Domain** below.

---

## Deploy

Drop the folder on Vercel (or push to the repo Vercel is watching).

- Framework preset: **Other**
- Build command / output directory / install command: *(all empty — repo root is served)*

`vercel.json` sets `cleanUrls`, so `v2.html` is reachable at `/v2` and `v3.html`
at `/v3`.

---

## Domain — this part matters

| Type | Name | Value |
|---|---|---|
| CNAME | `go` | `cname.vercel-dns.com` |

**Do not run the live campaign on a `*.vercel.app` URL.** The Meta pixel writes
`_fbp` / `_fbc` as first-party cookies on the registrable domain. Serving from a
subdomain of `omgworld.com` carries those identifiers into the Shopify checkout;
serving from `vercel.app` does not, and purchases arrive unattributed. A
`vercel.app` URL is fine for the internal review only.

---

## How checkout works

Every CTA on every variant links to the same Shopify cart permalink:

```
https://omgworld.com/cart/57963807605067:1?discount=OMGLT10&locale=lt
```

This skips the English storefront entirely and drops the visitor into a
Lithuanian checkout with the €10 discount already applied. Each page also
forwards `fbclid`, `ttclid`, `gclid` and all `utm_*` parameters from the
incoming ad click onto that URL so Shopify's own pixel can stitch the session.

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
| `LP_CheckoutClick` | custom | Same click — separates LP intent from Shopify's own events |
| `LP_Scroll` | custom | 25 / 50 / 75 / 100% depth |

**Every event carries `lp_variant` (`A`, `B` or `C`).** That is what makes the
three-way comparison readable in Events Manager without three separate pixels.
Newsletter signups are also tagged `variant-A|B|C` in Shopify.

`InitiateCheckout` and `Purchase` are **not** fired here — Shopify's own Meta
integration owns those.

---

## Brand assets — where they come from

- **Logo:** the real store file, `omg_logo_black_3.png`, served from the Shopify CDN.
- **Product photography:** the Variety Pack's own media (12-pack shot, four-flavour
  shot, lifestyle shot) plus the four single-can shots, all hotlinked from the
  store CDN so imagery stays in sync.
- **UGC review cards:** the same four customers, photos and product tags as the
  "See what the community is saying" section on omgworld.com.
- **Footer:** mirrors the store footer — same four link columns, same pink
  (`#FFE1F0`) on plum (`#8A086E`), same social accounts — plus the company
  registration details.

### A note on the brand font

The store's display face **Gila is not used here on purpose.** It has no
Lithuanian diacritics: `ą`, `č`, `ė`, `į` and `ų` are missing from the file and
fall back to a system font *mid-word*, which looks broken in Lithuanian
headlines. These pages use **Montserrat**, which is already the store's body
face, so the pages stay on-brand without the glyph problem. If the brand wants
Gila in headlines, the font needs a Baltic character set added first.

---

## Known items to confirm with the client

1. **Delivery estimate.** Shopify checkout quoted 25–28 August for an order placed
   16 August. All "1–2 day" claims were removed rather than contradict the
   checkout — restore them once delivery settings are fixed.
2. **Duplicate shipping rate.** Two domestic rates both named "Free shipping",
   one at €1.00.
3. **Fonts.** Loaded from Google Fonts. If GDPR posture requires it, self-host
   Montserrat and drop the `<link>` tags.
4. **Reviews.** The four Lithuanian reviews are the client's own approved copy.
5. **Deposit (užstatas).** Partner stores charge €0,10 per can; the webshop does
   not. Worth confirming whether the same obligation applies to the online channel.

---

## Files

```
index.html    variant A — complete page, markup + CSS + JS
v2.html       variant B — complete page, self-contained
v3.html       variant C — complete page, self-contained
img/          can-in-hand shot with callouts (webp + jpg fallback)
vercel.json   caching + security headers + clean URLs
```
