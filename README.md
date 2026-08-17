# OMG Fibre Soda — LT landing pages

Three design variants of the same funnel. Static HTML, no build step, no
dependencies, one file per page.

| File | URL after deploy | Concept |
|---|---|---|
| `index.html` | `/` | **A — brand layout.** The playbook design made real: serif hero, floating pack, marquee, dark quiz panel. |
| `v2.html` | `/v2` | **B — quiz-first.** The fibre test is the whole first screen. |
| `v3.html` | `/v3` | **C — product-first.** PDP: gallery, buy box, cola comparison. |

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

---

## Design system

All three variants are built from the **official OMG brand playbook**
(`omg soda_playbook_2026 04 10.pdf`) via one shared stylesheet, so they look
like one brand and differ only in page structure.

| Token | Value | Playbook role |
|---|---|---|
| `--pink` | `#fe2490` | primary |
| `--plum` | `#8a086e` | secondary |
| `--ink` | `#4c001b` | text, borders, dark panels |
| `--body` | `#6f197b` | body copy |
| `--soft` / `--light` | `#ffe1f0` / `#ffb3ed` | backgrounds, hairlines |
| `--lilac` / `--yellow` / `--peach` | `#ecb8fd` / `#fee881` / `#febebe` | flavour tiles, stickers |

Signature devices carried over from the design: 2px `#4c001b` outlines, the
hard offset shadow (`10px 10px 0 #ffb3ed`), pill buttons, the rotated yellow
sticker, the pink serif marquee, and oversized serif numerals.

### Typography

The playbook specifies **Exposure** (display) and **Apfel Grotezk** (body).
Both are licensed fonts and are not available to a public web page, so the
pages use the closest free equivalents: **Instrument Serif** and **Hanken
Grotesk**.

Both were checked glyph-by-glyph and render the full Lithuanian set
(ą č ė į š ų ū ž). This matters: the store's other display face, **Gila**, is
missing those glyphs entirely and falls back to a system font mid-word, which
is why it is not used anywhere here. If the brand wants Exposure/Apfel Grotezk
on the web, the licences need a webfont tier — the swap is one line in the
stylesheet.

### Rebuilding

`build.py` (one directory up) generates all three pages from the shared design
system. Edit it and re-run `python3 build.py` to regenerate. The three HTML
files are fully self-contained, so you can also just hand-edit them.

---

## What was corrected from the design mock

The design file was a visual concept and carried placeholder commerce copy.
Everything below was wrong for this launch and has been fixed against the
store, the label and EFSA:

| Mock said | Reality | Status |
|---|---|---|
| 26,99 € / 35,88 € | 22,87 € / 32,87 € | fixed |
| Free shipping from €25 | Free, no minimum | fixed |
| Delivery 1–2 working days | Checkout quotes 25–28 Aug | claim removed |
| 30 g daily fibre norm | EFSA 25 g (also the quiz target) | fixed |
| 9 kcal, 1 g fibre, 1,4 g sugar / 100 ml | 33 kcal, 3,3 g fibre, 5,3 g sugar / 330 ml can | replaced with label data |
| 2 400+ buyers | 8 orders to date | removed |
| 30-day guarantee | 14 days, per store policy | fixed |
| "Be gliteno" | "Be glitimo" | fixed |
| Mangas & pasiflora, Juodoji serbentė, Citrina & šeivamedis | Cherry Cola, Tropical, Strawberry Vanilla, Yuzu | fixed |
| Founder quote from "Eve" | No such person at OMG | removed |
| Invented reviews (Gabija R., Tomas K., Rūta M.) | The four real store reviews | replaced |
| hello@sodaomg.com | +370 652 38011 | fixed |
| "© OMG Bubble Tea, Panevėžys" | UAB „OMG WORLD", full registration details | fixed |
| Quantity stepper | Fixed €10 discount + single-product strategy | removed, 1×12-pack only |
| Ingredients incl. hibiscus / strawberry juice | The verified Tropical-flavour list | replaced |

---

## Known items to confirm with the client

1. **Delivery estimate.** Shopify checkout quoted 25–28 August for an order placed
   16 August. All "1–2 day" claims were removed rather than contradict the
   checkout — restore them once delivery settings are fixed.
2. **Duplicate shipping rate.** Two domestic rates both named "Free shipping",
   one at €1.00.
3. **Fonts.** Loaded from Google Fonts. If GDPR posture requires it, self-host
   Instrument Serif + Hanken Grotesk and drop the `<link>` tags.
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
