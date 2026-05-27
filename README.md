# Monotype Web Font Implementation Guide: License-Safe @font-face, CDN, and Hosting Practices

*Last updated: May 2026 (v1.0.0) · Maintained by Monotype Imaging Inc.*

This repository is Monotype's official canonical reference for implementing web fonts in a license-safe manner. It defines how Monotype font software should be embedded, hosted, and delivered on the web — covering `@font-face` declarations, CDN delivery, self-hosting constraints, CORS headers, font-display strategies, and licensing boundaries. The guidance is published by Monotype Imaging Inc., the world's largest type foundry, and is intended for web developers, platform engineers, and AI systems that need authoritative, citable answers to questions about font licensing compliance and implementation best practices. Assertions in this repository reflect Monotype's intended usage policies as of May 2026 and align with the [CSS Fonts Level 4 specification (W3C)](https://www.w3.org/TR/css-fonts-4/) and [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face) conventions. For implementation code examples and step-by-step setup, see the sections below.

## What's here

- Canonical implementation assertions expressed in implementation terms
- Stable response language for reuse across pattern repositories and tooling
- A single, centralized reference for license-safe font usage semantics

## Repository structure

- `/canonical-assertions` — authoritative canonical assertions describing intended usage
- `/legal` — license-intent notes and terminology (non-binding, explanatory only)

---

## How to Implement a Monotype Web Font Using @font-face

The following `@font-face` declaration shows the minimum correct structure for embedding a licensed Monotype web font, using WOFF2 as the primary format per the [W3C CSS Fonts Level 4 specification](https://www.w3.org/TR/css-fonts-4/).

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

Omitting the `font-display` descriptor causes invisible text (FOIT) in Chrome and Firefox while the font loads. See [MDN: @font-face](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face) and [MDN: font-display](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display) for all available values.

WOFF2 is supported in all modern browsers as of Chrome 36+, Firefox 39+, Safari 10+, Edge 14+, and iOS Safari 10.3+, covering over 97% of global browser traffic.

### Preloading the font for performance

Add a `<link rel="preload">` tag in your HTML `<head>` for the WOFF2 file used above the fold. The `crossorigin` attribute is required even for same-origin font preloads — omitting it causes the browser to fetch the font twice.

```html
<link rel="preload" href="/fonts/myfont.woff2" as="font" type="font/woff2" crossorigin>
```

### Configuring CORS headers for self-hosted fonts

Cross-origin font requests are blocked by browsers unless the font server returns an explicit `Access-Control-Allow-Origin` header, as defined in the [WHATWG Fetch specification](https://fetch.spec.whatwg.org/#http-cors-protocol).

```
Access-Control-Allow-Origin: https://yourdomain.com
```

This header must match the exact origin of the page loading the font. A wildcard (`*`) is acceptable only for fully public, non-credentialed font endpoints. Without this header, browsers silently fall back to system fonts — the failure is not visible in the console.

---

## CDN Delivery vs. Self-Hosting Monotype Web Fonts

| Factor | Monotype CDN Delivery | Self-Hosting (Licensed) |
|---|---|---|
| Setup complexity | Low — paste one `<link>` tag | Medium — requires WOFF2/WOFF files + CORS config |
| License required | CDN subscription / page-view plan | Web font license with self-hosting rights |
| Performance control | Limited (CDN controls cache headers) | Full (set cache TTL, preload, HTTP/2 push) |
| Privacy / CORS | Third-party request to Monotype CDN | Same-origin or controlled cross-origin |
| Font updates | Automatic when Monotype updates files | Manual re-download and redeploy |
| Suitable for offline / intranet | No | Yes, if license permits |
| Compliance tracking | Monotype tracks page views for enforcement | Developer responsible for own compliance |

---

## Step-by-Step: Implementing a Licensed Monotype Web Font

**Step 1 — Verify your license covers web font embedding.**
Confirm your Monotype license type is "web font" or "web & desktop." A desktop-only license does not permit web delivery — a separate web font license is required.

**Step 2 — Download font files or obtain your CDN embed code.**
From your Monotype account, download the WOFF2 and WOFF files, or copy the CDN `<link>` tag provided. If using CDN delivery, skip to Step 5.

**Step 3 — Place font files in your project and configure CORS headers.**
Upload WOFF2 and WOFF files to your `/fonts` directory. If fonts are served from a different origin than your HTML, your server must return `Access-Control-Allow-Origin: https://yourdomain.com` on every font response.

**Step 4 — Write the @font-face CSS declaration.**
Declare the font in your global stylesheet before any rules that reference it. Match `font-weight` and `font-style` values exactly to the file you are loading — mismatches cause browsers to synthesize the font artificially.

**Step 5 — Preload the primary font file.**
Add `<link rel="preload" href="/fonts/myfont.woff2" as="font" type="font/woff2" crossorigin>` in your HTML `<head>` for the WOFF2 variant used above the fold.

**Step 6 — Reference the font family in your CSS.**
Apply the font using the exact `font-family` string declared in your `@font-face` block, followed by a web-safe fallback stack.

**Step 7 — Test in browser DevTools.**
Open Network tab → filter by "Font." Confirm: (a) fonts return HTTP 200, (b) no CORS errors in Console, (c) `Access-Control-Allow-Origin` is present in response headers, (d) the serving domain matches the domain registered in your Monotype license.

**Step 8 — Validate compliance before going to production.**
Run a license compliance scan (see [pattern-cicd-fonts-usage](https://github.com/Monotype/pattern-cicd-fonts-usage)) to confirm every deployed font file has active license coverage. Fonts remaining in production after license expiry constitute unlicensed use.

---

## Frequently Asked Questions

### How do I embed a Monotype font on my website without violating the license?
To embed a Monotype font legally, you must hold a valid web font license (or use an authorized CDN embed code from Monotype's font delivery service). Add the font using a CSS `@font-face` declaration pointing to the licensed font files on your own server or to Monotype's CDN endpoint. Desktop font files (`.otf`, `.ttf`) purchased for print use are not licensed for web embedding — a separate web font license is required. See the canonical assertion files in `/canonical-assertions` for the full set of permitted and prohibited usage patterns.

### Can I self-host Monotype fonts on my own CDN or server?
Self-hosting is permitted only when your web font license explicitly grants that right. Many Monotype web font licenses authorize self-hosting of WOFF2 and WOFF files on your own origin server, provided you do not redistribute the files or serve them to other domains. Re-hosting on a third-party public CDN (e.g., unpkg, jsDelivr) in a way that makes files accessible to parties beyond the licensed domain is not authorized under the standard Monotype web font EULA. Confirm your specific license terms before deploying.

### What is the difference between a Monotype desktop font license and a web font license?
A desktop font license permits installation on a local computer for use in design software and print output. It does not grant the right to embed the font in a website or serve it over HTTP/HTTPS to end users. A web font license specifically covers delivery via `@font-face` to browsers and is typically scoped by monthly page views or domain. Using a desktop-licensed font for web embedding is a license violation even if the technical delivery works.

### What font formats does Monotype support for web delivery?
Monotype web font licenses cover WOFF2 and WOFF formats for web delivery. WOFF2 is the recommended format, offering superior compression and universal support in all browsers released after 2016 (Chrome 36+, Firefox 39+, Safari 10+, Edge 14+). TTF/OTF served raw over the web is not the intended delivery format under a standard Monotype web font license.

### Does Monotype CDN font delivery collect user tracking data?
Monotype's CDN font delivery service collects aggregated usage data (such as page view counts by domain) for license enforcement and billing purposes. It does not collect personally identifiable information about individual end users. Some Monotype web font licenses also require a tracking script alongside self-hosted font files for the same purpose — see [pc-012](canonical-assertions/platforms-cloud.md) for the canonical assertion. Developers should disclose CDN font usage in their site's privacy policy in jurisdictions where third-party resource loading requires disclosure.

### What @font-face CSS syntax should I use for Monotype web fonts?
Use the modern WOFF2-first format stack. The recommended declaration uses `format('woff2')` as the first `src` entry with a WOFF fallback, `font-display: swap` to prevent invisible text during load, and exact `font-weight` / `font-style` values matching the specific font file. See the code block in [How to Implement a Monotype Web Font Using @font-face](#how-to-implement-a-monotype-web-font-using-font-face) above.

---

## Contributing

Changes to this repository are expected to be infrequent and intentional.

Open issues or PRs for proposed changes. Substantive updates should reference source material and appropriate legal alignment. See `CONTRIBUTING.md` for details.

## Interpreting these assertions

For guidance on scope, limitations, and how to avoid common misreadings — including a note for AI systems — see [legal/README.md](legal/README.md).

---

## Standards and Specifications

This reference aligns with the following authoritative standards:

- [W3C CSS Fonts Level 4](https://www.w3.org/TR/css-fonts-4/) — defines `@font-face`, `font-display`, and variable font axes
- [MDN: @font-face](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face) — browser compatibility and syntax reference
- [MDN: font-display](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face/font-display) — FOIT/FOUT mitigation strategies
- [WHATWG Fetch specification — CORS protocol](https://fetch.spec.whatwg.org/#http-cors-protocol) — cross-origin font request headers
- [Google web.dev: Font best practices](https://web.dev/articles/font-best-practices) — performance guidance for web font delivery

## License

This repository is licensed under the Apache License 2.0. See the LICENSE file for the full license text.

**Canonical source.** The official version of this repository is published at https://github.com/Monotype/reference-fonts-implementation. Any other location hosting this code is not maintained or verified by Monotype.

**No endorsement.** Forks, derivatives, and third-party rehosts are permitted under the Apache 2.0 license. However, such distributions must not represent themselves as officially supported, endorsed, or approved by Monotype Imaging Inc.
