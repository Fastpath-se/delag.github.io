# DELAG website — launch handoff (v2.5)

Single static file, no build step, no external requests, no cookies, no analytics.
CSS, JS, fonts, images and SVG are all inside `DELAG_launch_site_v2_5.html`.

## 0. Start here

The website is `DELAG_launch_site_v2_5.html` (rename it to `index.html` when deploying). Everything else in this package is either an asset to upload
(`launch-assets/`) or a tool for the team (`tools/`, `retired-solution/`). **Do not upload `tools/` or `retired-solution/` to the public server.**
Sections 1–2 are the launch steps; section 6 lists what still needs a decision or legal input.

## 1. Deploy

| File | Goes to |
|---|---|
| `DELAG_launch_site_v2_5.html` | `/index.html` |
| `launch-assets/og-image.png` | `/og-image.png` |
| `launch-assets/apple-touch-icon.png` | `/apple-touch-icon.png` |
| `launch-assets/favicon.svg`, `favicon-32.png` | `/` (optional; the HTML has an inline SVG favicon) |
| `launch-assets/robots.txt`, `sitemap.xml` | `/` |

**Assumption to check:** canonical URL, `og:url`, `og:image` and the sitemap use `https://delag.com/`.
If the live domain differs, search-and-replace it in the HTML, `robots.txt` and `sitemap.xml`.

## 2. Configure before going live

1. **Contact form.** `FORM_ENDPOINT` at the top of the `<script>` block is empty; while empty, the form opens the visitor's
   mail app (mailto). Set it to a form backend (Formspree, Netlify Forms, own API) to send directly. It POSTs JSON:
   `topic, name, company, email, goal, stack` (+ a hidden `website` honeypot; if filled, the submit is dropped).
   `topic` is `general` or `special-ops`.
2. **hello@delag.com** must exist and be monitored (form fallback, contact page, founder "Email" links).
3. Check `og-image.png` with a link preview (LinkedIn Post Inspector / Slack) after upload.

## 3. Structure (what changed in this round)

- **Solution page is retired.** Old `#solution` links redirect to `#home/results`. The page's markup and CSS are in
  `retired-solution/` for re-introduction later (instructions at the top of `solution-page.html`).
- **Home order:** Hero → *Less overhead. Measured.* (`id="results"`) → Where we come in → Built for enterprise (with the stack diagram)
  → What stays yours + Built around your environment → Where DELAG creates leverage → Trusted with the hard parts.
- **CTAs:** every general call to action reads **Get in touch**. The top nav has the button only (the "Contact" text link is gone;
  on mobile it appears as "Get in touch" inside the menu). Special Ops keeps **Describe your case / Describe the problem**.
- **Numbers CTA:** under the benchmark cards there is a short invitation ("These are reference measurements… Tell us where it needs to run…") with a Get in touch button.
- **Special Ops form variant:** `#contact/special-ops` opens the same form with Special Ops wording ("Describe the case",
  target-hardware prompt), an amber "Special Ops enquiry" eyebrow, subject line `DELAG Special Ops enquiry from …` and `topic: special-ops`.
  Any other route back to Contact resets it to the general form.
- **Custom hardware** is listed under *Built around your environment* (amber). No customer or partner names are used anywhere.

## 4. Editing the benchmark numbers

The numbers appear in several places per card (text, bar width, count-up target, delta chip), so don't hand-edit them.
Edit `DATA` in `tools/benchmarks.py`, run `python benchmarks.py > results.html`, and paste the output over the
`<section … id="results">` block. Speed-up, deltas, bar widths, the "µs" line and the count-up targets are all computed.
Cards are ordered by ascending speed-up on purpose (the layout is built as a crescendo: 1.1× → 4.3× → 286×).
Latency deltas ≥ 99.5% are shown with two decimals (−99.65%) because they would otherwise round to "−100%".
The SMOE DELAG latency bar has a 5 px minimum width so it stays visible (true width is ~0.35%).

## 5. Responsive approach (so nothing has to be tuned by hand)

- **No `vh`/`dvh` anywhere.** Layout is driven by width only, so it doesn't jump when a mobile URL bar appears or the window height changes.
- **Every media block has an intrinsic ratio** (`aspect-ratio`): hero artwork (1579:541), card images (3:2, 16:9 on tablet/mobile),
  company image, contact image, step images; the diagrams and the Special Ops illustration are inline SVG with a `viewBox`. Nothing reflows as images load.
- **Fluid type and spacing** with `clamp()` (hero, numbers, spacing). Breakpoints: ≤1000px and ≤700px only.
- **Fonts are embedded** (Inter variable + JetBrains Mono, both SIL OFL 1.1), so text wraps identically on macOS, Windows, iOS and Android
  instead of falling back to different system fonts. The `→` arrow and the menu icon are SVG/CSS because those fonts don't include them.
- `text-size-adjust:100%` stops iOS from inflating text in landscape. The mobile h1 is capped so the longest word ("CONVERSATION.") never overflows on 320 px phones.
- Special Ops illustration on phones is a swipeable strip (labels stay readable) with a "Swipe to explore" hint.

Test tooling (Python + Playwright: `pip install playwright && playwright install chromium`):

- `tools/responsive-sweep.py file:///abs/path/index.html` — 30 viewport sizes (320–2560 px, portrait/landscape phones, tablets) × all pages;
  reports horizontal overflow, clipped text, overlaps, and hero metrics. Last run: 0 problems.
- `tools/site-check.py file:///abs/path/index.html` — 28 checks: links, routing, CTAs, section order, all benchmark figures, count-up,
  Special Ops form, fonts, mobile menu. Last run: 28/28.

## 6. Still missing (needs a decision or legal input)

- **Privacy policy.** The form collects name, company, email. Needs a policy page and a link near the form / in the footer. Not drafted; needs legal review.
- **Company details** (legal name, registration number, address) are commonly required on business sites in the EU. Check with legal what and where.
- **Cookies/analytics.** None today, so no cookie banner needed. Adding analytics changes that.
- **Benchmark methodology.** The footnote says "MacBook M5 Max, ONNX Runtime v1.30.0". Still to be stated: precision, batch size, median vs mean,
  peak vs RSS memory, number of runs. A technical reader will ask.

## 7. Known limitations

- **SEO: hash routing.** Pages are `#usecases`, `#specialops` etc.; search engines see one URL (`/`). Fine for launch. For per-page indexing,
  switch to real paths (`/use-cases`) with a server rewrite to `index.html`, `history.pushState`, and per-page `<title>`/description.
- **Hero and Special Ops artwork** are raster (1579 px / 1800 px wide) with crisp SVG labels on top. Slightly soft on retina; re-export the originals at 2× to fully fix.
- **Browsers.** Verified in Chromium only (see tools above). **Before launch, open the site once in Safari on an iPhone and in Firefox/Windows.** Not verified in Safari, Firefox or on real phones. Uses `mask`, `aspect-ratio`, `:has()` (focus ring only, degrades gracefully) and `backdrop-filter`.
- **File size** is ~1.4 MB (fonts ≈ 90 KB of that; the rest is image data).
- **Trademarks.** PyTorch, ONNX, TensorFlow, AMD, Arm and Intel appear as plain text with the trademark line; no third-party logos are used.

## 8. Where things live in the file

- `<style>` (first block): the whole stylesheet, sectioned, no `!important`. `<style id="assets">`: fonts and images as base64 (`--img-*`).
- Script (end of file): routing (`#page` or `#page/anchor`), Special Ops topic, founder toggles, form, count-up.
- Logo: inline SVG in `<nav>`. Diagrams: inline SVG with desktop + mobile variants (toggled by CSS).
- Fonts: Inter © The Inter Project Authors; JetBrains Mono © JetBrains s.r.o. — SIL Open Font License 1.1.
