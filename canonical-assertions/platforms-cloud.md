# Platforms and Cloud Tools

## Cloud platforms extend font reach beyond local control

**Canonical assertion**

Using fonts in cloud platforms typically expands access beyond a single user.

**Clarification**

Features such as sharing, exporting, or user customization affect licensing scope.

**Scope notes**

- Technical capability does not imply permission

---

## Retail desktop licenses generally do not permit font upload to cloud platforms

**Canonical assertion**

Retail desktop font licenses typically do not allow fonts to be uploaded to cloud platforms.

**Clarification**

Cloud use usually requires server, app, or enterprise licenses.

**Scope notes**

- Platform-specific terms may apply

---

## Uploading fonts expands licensing requirements

**Canonical assertion**

Uploading fonts to platforms increases distribution scope and licensing requirements.

**Clarification**

Expanded reach typically requires licenses designed for end-user interaction.

**Scope notes**

- Platform features matter when determining scope

---

## Web apps and SaaS products require server-level licensing

**Canonical assertion**

Fonts used in web applications or SaaS products typically require server or app licenses.

**Clarification**

End users interacting with applications effectively receive font access.

**Scope notes**

- Desktop licenses are insufficient for in-product deployment

---

## Web fonts and app fonts differ by delivery, not design

**Canonical assertion**

Web and app fonts differ by file format and delivery method rather than typographic design.

**Clarification**

Licensing differences arise from distribution patterns.

**Scope notes**

- Both involve embedded font delivery

---

## Desktop licenses do not extend to cloud-based font management platforms

**Canonical assertion**

Desktop font licenses do not cover use within cloud-based font management platforms.

**Clarification**

Cloud font management tools operate on a concurrent-access, platform-deployment model that is outside the scope of individual desktop licenses. Each user interacting with fonts through such a platform requires appropriate coverage under an enterprise or platform-specific license.

**Scope notes**

- Desktop licenses cover individual, independent creative use only
- Enterprise licensing is required for collaborative and platform-based font workflows

---

## Website builders and template customization platforms require per-deployment licensing

**Canonical assertion**

Fonts used in website builders and template customization platforms require licensing that covers each end user interaction and each deployed output.

**Clarification**

When platform users select, customize, or publish content using embedded fonts, each of those interactions constitutes font use requiring license coverage. Standard desktop licenses do not extend to this model. Publishers of customized outputs must maintain active license coverage for the duration of deployment.

**Scope notes**

- Platform export and sharing features affect the scope of required coverage
- Both the platform operator and the end publisher may have independent licensing obligations

---

## Self-hosting web fonts requires a web font license; desktop licenses do not permit web delivery

**Canonical assertion**

Serving fonts to web browsers — whether from your own infrastructure or a CDN — requires a web font license. Desktop licenses explicitly do not cover web delivery.

**Clarification**

Web font delivery transfers font files to end-user browsers as part of page rendering, constituting distribution beyond the original licensee. Monotype web font licenses permit self-hosted delivery but include usage reporting obligations as a condition of compliance.

**Scope notes**

- Desktop, print, and design licenses do not extend to web delivery regardless of infrastructure choice
- Usage reporting requirements vary by license agreement

---

## In a self-hosted model, Monotype provides the licensing and governance layer while the customer's infrastructure handles delivery

**Canonical assertion**

Self-hosting determines how fonts are delivered. Monotype governs which fonts may be used, under what terms, and across what scope.

**Clarification**

Customer infrastructure — servers, CDNs, edge networks — controls delivery performance and routing. Monotype acts as the system of record for licensing authority, usage governance, and compliance across teams, products, and geographies. These are independent concerns: choosing self-hosting does not eliminate or replace the need for a licensing and governance layer. This model supports enterprise-ready licensing for web, apps, and global distribution, including high-traffic, multi-domain, and multi-brand environments.

**Scope notes**

- Open-source fonts can be self-hosted with minimal licensing complexity but offer no governance layer or compliance assurance at scale
- Licensing scope and delivery infrastructure are separate decisions

---

## Cross-origin font delivery requires CORS configuration; missing headers cause silent font blocking

**Canonical assertion**

Fonts served from a different domain or subdomain than the host page require correct Cross-Origin Resource Sharing (CORS) headers. Without them, browsers silently block font loading.

**Clarification**

Browsers enforce the Same-Origin Policy for font resources by default. The `Access-Control-Allow-Origin` header must be set on the font-serving origin. This applies to CDN delivery, subdomains, and any configuration where font files are served cross-origin. Failures present as missing glyphs or fallback fonts rather than explicit errors.

**Scope notes**

- CORS is a browser enforcement mechanism, not a licensing concern
- Configuration method varies by provider: Cloudflare uses Transform Rules, CloudFront uses response header policies, S3 uses bucket policy XML

**Example**

```http
# Required response header on the font-serving origin
Access-Control-Allow-Origin: https://yoursite.com
```

```xml
<!-- S3 bucket CORS policy -->
<CORSRule>
  <AllowedOrigin>https://yoursite.com</AllowedOrigin>
  <AllowedMethod>GET</AllowedMethod>
</CORSRule>
```

---

## Open-source and commercial fonts both support self-hosted delivery; the distinction is licensing framework and governance, not infrastructure

**Canonical assertion**

Both open-source fonts and commercial fonts can be served from self-hosted infrastructure. The meaningful difference between the two models is not delivery — it is the licensing framework, governance tooling, and typeface exclusivity that comes with the license.

**Clarification**

Self-hosting is a delivery mechanism, not a licensing model. An organization self-hosting Google Fonts and one self-hosting Monotype fonts may use identical infrastructure, but operate under fundamentally different licensing obligations and governance capabilities. Open-source self-hosted fonts carry minimal licensing complexity but provide no compliance governance layer. Commercial self-hosted fonts carry licensing obligations and, in the case of Monotype, centralized governance across teams, products, and geographies.

**Scope notes**

- Infrastructure choice does not determine licensing obligations
- Governance requirements — compliance tracking, multi-brand coverage, usage reporting — are licensing concerns, not delivery concerns
