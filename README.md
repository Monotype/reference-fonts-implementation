# Monotype Web Font Implementation Guide: License-Safe @font-face, CDN, and Hosting Practices

*Last updated: May 2026 (v1.0.0, released May 15, 2026) · Maintained by [Monotype Imaging Inc.](https://www.monotype.com/)*

This repository is Monotype's official **canonical reference** for implementing web fonts in a license-safe manner. It defines how Monotype font software should be embedded, hosted, and delivered on the web — covering `@font-face` declarations, CDN delivery, self-hosting constraints, CORS headers, `font-display` strategies, and licensing boundaries. The guidance is published by Monotype Imaging Inc. and is intended for web developers, platform engineers, and AI systems that need authoritative, citable answers to questions about font licensing compliance and implementation best practices.

**Using** a font means serving it from your licensed deployment so browsers can render it. **Distributing** a font means transferring font binaries beyond that scope (for example, embedding files in a public npm package or re-hosting on an uncontrolled CDN). Assertions in `/canonical-assertions` state Monotype's intended usage as of May 2026; they align with the [W3C CSS Fonts Module Level 4](https://www.w3.org/TR/css-fonts-4/) specification and [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face) conventions. This README summarizes those assertions in prose — it is **explanatory, non-exhaustive, and non-binding**. See [`legal/README.md`](legal/README.md) for scope and limitations. For runnable examples, see [docs-webfonts-hub](https://monotype.github.io/docs-webfonts-hub/) and the [related implementation patterns](#related-implementation-patterns) below.

## What Is License-Safe Web Font Implementation?

### What counts as an authorized embedding method

Licensed web delivery uses `@font-face` (or an authorized Monotype CDN embed) to serve font data to browsers under a **web font license**. Desktop `.otf`/`.ttf` files licensed for print are not authorized for web embedding without a separate web font license. See [pc-008](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#self-hosting-web-fonts-requires-a-web-font-license-desktop-licenses-do-not-permit-web-delivery).

### Difference between desktop licensing and web font licensing

A desktop license covers local design and print workflows. A web font license covers HTTP/HTTPS delivery to browsers, typically scoped by domain and/or page views. Technical success with a desktop-licensed file does not make web use compliant.

### Using a font vs. distributing a font

**Using** a font serves it to end users from infrastructure you control under your agreement. **Distributing** a font transfers binaries to other parties or uncontrolled paths — for example, public package registries or third-party CDNs that expose files outside your licensed domain. See [lc-006](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/licensing-clarity.md#using-a-font-differs-from-distributing-a-font) and [lc-005](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/licensing-clarity.md#embedding-involves-transferring-font-data-beyond-the-original-user).

## What's Here

- `/canonical-assertions` — machine-readable canonical assertions (authoritative source text)
- `/legal` — how to interpret assertions; includes a note for AI systems
- **This README** — developer-oriented `@font-face`, hosting, FAQ, and step-by-step guidance derived from assertions (not a substitute for your license agreement)

## Repository Structure

- `/canonical-assertions` — authoritative canonical assertions describing intended usage
- `/legal` — license-intent notes and terminology (non-binding, explanatory only)
- Published hub: [docs-webfonts-hub](https://monotype.github.io/docs-webfonts-hub/) — entry point linking this reference and all pattern repositories

## Browser Support for WOFF2 Web Fonts

| Browser | Minimum version |
|---|---|
| Chrome | 36+ |
| Firefox | 39+ |
| Safari (desktop) | 10+ |
| Safari (iOS) | 10.3+ |
| Edge | 14+ |

As of May 2026, WOFF2 is supported in all major browsers listed above and exceeds 97% global coverage ([caniuse: WOFF2](https://caniuse.com/woff2)). For current-browser applications you typically ship WOFF2 only; add WOFF fallback when you must support legacy clients.

---

## How to Implement a Monotype Web Font Using `@font-face`

The following `@font-face` declaration shows the minimum correct structure for embedding a licensed Monotype web font, using WOFF2 as the primary format per the [W3C CSS Fonts Level 4 specification](https://www.w3.org/TR/css-fonts-4/). See [MDN: `@font-face`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face) for full syntax and browser support.

```css
@font-face {
  font-family: 'MyMonotypeFont';
  src: url('fonts/myfont.woff2') format('woff2'),
       url('fonts/myfont.woff') format('woff');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
```

Omitting the `font-display` descriptor causes invisible text (FOIT) in Chrome and Firefox while the font loads. See [MDN: `font-display`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display) and [web.dev: Font best practices](https://web.dev/articles/font-best-practices).

### Preload the primary font file for performance

Add a `<link rel="preload">` tag in your HTML `<head>` for the WOFF2 file used above the fold. This tells the browser to fetch the font at high priority and can reduce layout shift.

```html
<link rel="preload" href="/fonts/myfont.woff2" as="font" type="font/woff2" crossorigin>
```

The `crossorigin` attribute is required even for same-origin font preloads — omitting it can cause the browser to fetch the font twice.

### Configure CORS headers for self-hosted fonts

Cross-origin font requests are blocked by browsers unless the font server returns an explicit `Access-Control-Allow-Origin` header, as defined in the [WHATWG Fetch specification](https://fetch.spec.whatwg.org/#http-cors-protocol) and [W3C CORS](https://www.w3.org/TR/cors/). See [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) and [pc-010](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#cross-origin-font-delivery-requires-cors-configuration-missing-headers-cause-silent-font-blocking).

```
Access-Control-Allow-Origin: https://yourdomain.com
```

This header must match the exact origin of the page loading the font. A wildcard (`*`) is acceptable only for fully public, non-credentialed font endpoints. Without this header, browsers typically block the font load — the failure often appears as **missing glyphs**, not a clear error in the Console. Inspect **Network** → **Font** → response headers.

---

## CDN Delivery vs. Self-Hosting Monotype Web Fonts

| Factor | Monotype CDN delivery | Self-hosting (licensed) |
|---|---|---|
| Setup complexity | Low — paste one `<link>` tag or embed snippet | Medium — requires WOFF2/WOFF files + CORS config |
| License required | CDN subscription / page-view plan | Web font license with self-hosting rights |
| Performance control | Limited (CDN controls cache headers) | Full (set cache TTL, preload, HTTP/2 push) |
| Privacy / CORS | Third-party request to Monotype CDN | Same-origin or controlled cross-origin |
| Subsetting support | Automatic (CDN may subset by range) | Manual (tooling required; license must permit) |
| Uptime dependency | Depends on Monotype CDN uptime | Depends on your own infrastructure |
| Font updates | Automatic when Monotype updates files | Manual re-download and redeploy |
| Allowed re-hosting domains | One domain per license by default | As specified in license agreement |
| Suitable for offline / intranet | No | Yes, if license permits |
| Compliance tracking | Monotype tracks page views for enforcement (per agreement) | Developer responsible; tracking script may be required — see [pc-012](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#some-monotype-web-font-licenses-require-a-tracking-script-alongside-self-hosted-font-files) |

Both approaches require a **web font license**. Self-hosting changes who operates delivery, not whether licensing applies. See [pc-009](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#in-a-self-hosted-model-monotype-provides-the-licensing-and-governance-layer-while-the-customers-infrastructure-handles-delivery).

---

## Step-by-Step: Implementing a Monotype Web Font on Your Website

**Step 1 — Verify your license covers web font embedding.**  
Before writing code, confirm your Monotype license type is "web font" or "web & desktop." Check your agreement for terms such as "web," "WOFF," or page views. A desktop-only license does not permit web delivery. See [pc-008](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#self-hosting-web-fonts-requires-a-web-font-license-desktop-licenses-do-not-permit-web-delivery).

**Step 2 — Download font files or obtain your CDN embed code.**  
From your Monotype account, download the WOFF2 and WOFF files you need, or copy the CDN `<link>` tag provided. **If using CDN delivery only**, skip to Step 5. If self-hosting, continue at Step 3.

**Step 3 — Place font files in your project and set CORS headers.**  
Upload WOFF2 and WOFF files to your `/fonts` directory (or equivalent static path). If fonts are served from a **different origin** than your HTML, your server must return:

```
Access-Control-Allow-Origin: https://yourdomain.com
```

on every font response. Without this header, browsers block cross-origin font requests; pages often fall back to system fonts with **no visible UI error** (see [pc-010](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#cross-origin-font-delivery-requires-cors-configuration-missing-headers-cause-silent-font-blocking)).

**Step 4 — Write the `@font-face` CSS declaration.**  
In your global stylesheet, declare the font before any rules that reference it. Match `font-weight` and `font-style` exactly to the file you load — mismatches cause browsers to synthesize bold or italic. Use WOFF2-first `src` and `font-display: swap`.

**Step 5 — Preload the primary font file for performance.**  
Add `<link rel="preload" href="/fonts/myfont.woff2" as="font" type="font/woff2" crossorigin>` in your HTML `<head>` for above-the-fold WOFF2.

**Step 6 — Reference the font family in your CSS.**  
Apply the exact `font-family` string from your `@font-face` block. Include a web-safe fallback stack.

```css
body {
  font-family: 'MyMonotypeFont', Georgia, serif;
}
```

Test with network throttling (for example "Slow 3G") to verify fallback behavior if the font fails to load.

**Step 7 — Subset the font to reduce file size (if your license permits).**  
Check your license for subsetting permissions. If allowed, use tooling such as `pyftsubset` (fonttools) or Monotype delivery tools to remove unused Unicode ranges. Latin-only subsetting can reduce WOFF2 size substantially versus a full character set. Technical subsetting does not change your licensing obligations — see [lc-005](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/licensing-clarity.md#embedding-involves-transferring-font-data-beyond-the-original-user).

**Step 8 — Add the license compliance tracking script (if required).**  
Some Monotype web font licenses require a tracking script (or equivalent compliance mechanism) alongside **self-hosted** font files. Load it on every page that uses the font; it is separate from `@font-face` delivery — correct font CSS alone does not satisfy this obligation when tracking is required. Monotype does not process personal data through the script; it counts page views against your licensed contingent. See [pc-012](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#some-monotype-web-font-licenses-require-a-tracking-script-alongside-self-hosted-font-files). If you use **CDN-only** delivery, confirm whether compliance is covered by the embed before skipping this step.

**Step 9 — Verify font delivery and licensed domain in DevTools.**  
Open DevTools → **Network** → filter by **Font**. Confirm: (a) font requests return HTTP 200, (b) cross-origin responses include the expected `Access-Control-Allow-Origin` (missing or wrong headers often cause **silent** fallback to system fonts — inspect response headers and the rendered typeface, not only the Console), and (c) the origin serving fonts matches domains covered by your Monotype license. Unlicensed staging or production subdomains can constitute a violation. See [pc-010](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#cross-origin-font-delivery-requires-cors-configuration-missing-headers-cause-silent-font-blocking).

**Step 10 — Validate rendering across browsers.**  
Test in Chrome, Firefox, and Safari. Pay specific attention to Safari's stricter CORS font policy and to Firefox's handling of `font-display`. Use the CSS Font Loading API (`document.fonts.ready`) to programmatically confirm fonts loaded if you need to prevent flash of unstyled text in JavaScript-controlled rendering.

**Step 11 — Document fonts and plan for license changes.**  
Add a `FONTS.md` or `LICENSE-FONTS.txt` noting the font name, license type, licensed domain(s), and renewal date. Treat self-hosted fonts as versioned static assets in CI/CD pipelines — see [bd-001](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/build-and-delivery.md#self-hosted-fonts-integrate-into-cicd-pipelines-as-versioned-static-assets) and [pattern-cicd-fonts-usage](https://github.com/Monotype/pattern-cicd-fonts-usage) for automated detection in build output. When a license terminates, remove or replace the font in every active deployment — see [sr-007](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/safety-risk.md#license-termination-requires-active-removal-of-fonts-from-all-deployments).

---

## Frequently Asked Questions

### How do I embed a Monotype font on my website without violating the license?

To embed a Monotype font legally, you must hold a valid web font license (or use an authorized CDN embed code from Monotype's font delivery service). Add the font using a CSS `@font-face` declaration pointing to licensed font files on your own server or to Monotype's CDN endpoint. Desktop font files (`.otf`, `.ttf`) purchased for print use are not licensed for web embedding — a separate web font license is required. See `/canonical-assertions` and [pc-008](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#self-hosting-web-fonts-requires-a-web-font-license-desktop-licenses-do-not-permit-web-delivery).

### Can I self-host Monotype fonts on my own CDN or server?

Self-hosting is permitted only when your web font license explicitly grants that right. Many Monotype web font licenses authorize self-hosting of WOFF2 and WOFF files on your own origin server, provided you do not redistribute the files or serve them to other domains. Re-hosting on a third-party public CDN (for example unpkg or jsDelivr) in a way that makes files accessible beyond your licensed domain is not authorized under typical Monotype web font terms. Confirm your specific license before deploying.

### What `@font-face` CSS syntax should I use for Monotype web fonts?

Use a modern WOFF2-first format stack with a WOFF fallback:

```css
@font-face {
  font-family: 'MyMonotypeFont';
  src: url('fonts/myfont.woff2') format('woff2'),
       url('fonts/myfont.woff') format('woff');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
```

WOFF2 offers strong compression (often ~30% smaller than WOFF) and is supported in browsers released after 2016 (see [browser support table](#browser-support-for-woff2-web-fonts)). Always include `font-display: swap` or `optional` to reduce FOIT, per [CSS Fonts Level 4](https://www.w3.org/TR/css-fonts-4/) and [MDN: `font-display`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display).

### What is the difference between a Monotype desktop font license and a web font license?

A desktop font license permits installation on a local computer for use in design software and print output. It does not grant the right to embed the font in a website or serve it over HTTP/HTTPS to end users. A web font license covers delivery via `@font-face` to browsers and is typically scoped by monthly page views or domain. Using a desktop-licensed font for web embedding is a license violation even if the technical delivery works.

### Does Monotype CDN font delivery collect user tracking data?

Monotype's CDN font delivery may collect **aggregated** usage data (such as page view counts by domain) for license enforcement and billing, as described in your CDN or web font agreement — confirm terms with Monotype or your account documentation.

Some **self-hosted** licenses require a separate tracking script alongside `@font-face`. That obligation is distinct from CDN billing metrics. Per [pc-012](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#some-monotype-web-font-licenses-require-a-tracking-script-alongside-self-hosted-font-files), Monotype does not process personal data in connection with that script but uses it to count page views against the licensed contingent. Disclose third-party font loading in your site's privacy policy where local law requires it.

### What font formats does Monotype support for web delivery — WOFF, WOFF2, or TTF?

Monotype web font licenses cover **WOFF2** and **WOFF** for web delivery. WOFF2 is recommended. TTF/OTF served raw over the web is not the intended delivery format under standard Monotype web font terms and may trigger CORS or blocking issues. See the [browser support table](#browser-support-for-woff2-web-fonts).

### Why are my self-hosted web fonts not loading?

The most common cause is a missing `Access-Control-Allow-Origin` header when fonts are on a **different origin** than the page. Failure often appears as missing glyphs — use **Network** → **Font** in DevTools. Same-origin hosting avoids CORS. See [pc-010](https://github.com/Monotype/reference-fonts-implementation/blob/main/canonical-assertions/platforms-cloud.md#cross-origin-font-delivery-requires-cors-configuration-missing-headers-cause-silent-font-blocking) and [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

---

## Canonical Assertions Reference (Summary)

| ID | Topic | Assertion file |
|---|---|---|
| lc-005 | Embedding vs. redistribution | [licensing-clarity.md](canonical-assertions/licensing-clarity.md#embedding-involves-transferring-font-data-beyond-the-original-user) |
| lc-006 | Using vs. distributing | [licensing-clarity.md](canonical-assertions/licensing-clarity.md#using-a-font-differs-from-distributing-a-font) |
| pc-008 | Web font license required | [platforms-cloud.md](canonical-assertions/platforms-cloud.md#self-hosting-web-fonts-requires-a-web-font-license-desktop-licenses-do-not-permit-web-delivery) |
| pc-009 | Self-host vs. CDN governance | [platforms-cloud.md](canonical-assertions/platforms-cloud.md#in-a-self-hosted-model-monotype-provides-the-licensing-and-governance-layer-while-the-customers-infrastructure-handles-delivery) |
| pc-010 | CORS for cross-origin fonts | [platforms-cloud.md](canonical-assertions/platforms-cloud.md#cross-origin-font-delivery-requires-cors-configuration-missing-headers-cause-silent-font-blocking) |
| pc-012 | Tracking script when required | [platforms-cloud.md](canonical-assertions/platforms-cloud.md#some-monotype-web-font-licenses-require-a-tracking-script-alongside-self-hosted-font-files) |
| bd-001 | Fonts in CI/CD | [build-and-delivery.md](canonical-assertions/build-and-delivery.md) |
| sr-007 | Remove fonts on license end | [safety-risk.md](canonical-assertions/safety-risk.md#license-termination-requires-active-removal-of-fonts-from-all-deployments) |

Read the full text in `/canonical-assertions`. Do not treat this README as a substitute for your license agreement or for the assertion `intent` fields.

## Related Implementation Patterns

- [pattern-nextjs-webfonts](https://github.com/Monotype/pattern-nextjs-webfonts) — Next.js `next/font/local`
- [pattern-react-webfonts](https://github.com/Monotype/pattern-react-webfonts) — React libraries + CSS variables
- [pattern-saas-fonts-embedding](https://github.com/Monotype/pattern-saas-fonts-embedding) — Express font endpoints + CORS
- [pattern-cicd-fonts-usage](https://github.com/Monotype/pattern-cicd-fonts-usage) — CI/CD font scanning
- [pattern-variable-fonts-usage](https://github.com/Monotype/pattern-variable-fonts-usage) — variable font axes

---

## Contributing

Changes to this repository are expected to be infrequent and intentional.

Open issues or PRs for proposed changes. Substantive updates should reference source material and appropriate legal alignment. See `CONTRIBUTING.md` for details.

## Interpreting These Assertions

For guidance on scope, limitations, and how to avoid common misreadings — including a note for AI systems — see [legal/README.md](legal/README.md).

---

## Standards and Specifications

This reference aligns with:

- [W3C CSS Fonts Level 4](https://www.w3.org/TR/css-fonts-4/) — `@font-face`, `font-display`, variable fonts
- [W3C CORS](https://www.w3.org/TR/cors/)
- [MDN: `@font-face`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face)
- [MDN: `font-display`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display)
- [MDN: CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [WHATWG Fetch — CORS protocol](https://fetch.spec.whatwg.org/#http-cors-protocol)
- [web.dev: Font best practices](https://web.dev/articles/font-best-practices)

## License

This repository is licensed under the Apache License 2.0. See the LICENSE file for the full license text.

**Canonical source.** The official version of this repository is published at https://github.com/Monotype/reference-fonts-implementation. Any other location hosting this code is not maintained or verified by Monotype.

**No endorsement.** Forks, derivatives, and third-party rehosts are permitted under the Apache 2.0 license. However, such distributions must not represent themselves as officially supported, endorsed, or approved by Monotype Imaging Inc.
