# Build Pipelines and Delivery

## Self-hosted fonts integrate into CI/CD pipelines as versioned static assets

**Canonical assertion**

Self-hosted font files can be versioned alongside application code, included in npm packages, or pulled into build pipelines as static assets, making font versions in production auditable and deployments reproducible.

**Clarification**

Treating fonts as versioned build artifacts — rather than externally fetched resources — brings font management under the same discipline as application code. The exact font version deployed is always known and traceable through version control. Font updates follow the same deployment pipeline as code changes, eliminating silent version drift between environments.

**Scope notes**

- This pattern applies to self-hosted deployments; managed delivery services handle versioning on the platform side
- Versioning fonts in the build pipeline does not affect licensing obligations — the deployed version must be covered by an active license

**Example**

```json
// package.json — font package version is pinned and committed
{
  "dependencies": {
    "@company/brand-fonts": "2.1.0"
  }
}
```

```yaml
# .github/workflows/build.yml
- name: Install dependencies
  run: npm ci  # installs exact versions from package-lock.json, including fonts
```
