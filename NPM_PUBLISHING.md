# npm Trusted Publishing Setup

This document explains how npm package publishing is configured for the mx-releases repository using **npm Trusted Publishing** with GitHub Actions OIDC.

## Overview

All three packages (`@memnexus-ai/sdk`, `@memnexus-ai/cli`, `@memnexus-ai/mcp-server`) are published to the public npm registry ([npmjs.com](https://www.npmjs.com)) using **npm Trusted Publishing**, which eliminates the need for long-lived npm tokens stored as secrets.

## npm Trusted Publishing (OIDC)

### What is npm Trusted Publishing?

npm Trusted Publishing uses **OpenID Connect (OIDC)** to create a secure, short-lived connection between GitHub Actions and npm. Instead of storing an npm access token as a secret, GitHub Actions authenticates directly with npm using OIDC tokens.

**Benefits:**
- ✅ No secrets to manage or rotate
- ✅ Automatic authentication via OIDC
- ✅ Enhanced security (short-lived tokens)
- ✅ Provenance attestations for supply chain security

**Reference:** [npm Trusted Publishing Documentation](https://github.blog/changelog/2023-04-19-npm-provenance-public-beta/)

## Configuration

### 1. npm Organization Setup

The `@memnexus-ai` npm organization must be configured to allow Trusted Publishing from GitHub Actions.

**Requirements:**
- npm account with `@memnexus-ai` organization access
- Organization configured for Trusted Publishing
- Packages reserved:
  - `@memnexus-ai/sdk`
  - `@memnexus-ai/cli`
  - `@memnexus-ai/mcp-server`

### 2. GitHub Actions Workflow Permissions

Workflows that publish to npm require specific permissions:

```yaml
permissions:
  contents: write    # For git operations (tags, releases)
  id-token: write    # Required for npm Trusted Publishing via OIDC
```

The `id-token: write` permission allows GitHub Actions to generate OIDC tokens that npm can verify.

### 3. Node.js Setup

Configure Node.js with the npm registry:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '20.x'
    registry-url: 'https://registry.npmjs.org'
    scope: '@memnexus-ai'
```

### 4. Publishing with Provenance

Enable provenance attestations when publishing:

```yaml
- name: Publish to npm Registry
  env:
    NPM_CONFIG_PROVENANCE: true
  run: |
    npm publish --access public --provenance
```

**What is Provenance?**
- Cryptographically signed attestation linking the published package to its source code
- Provides supply chain transparency
- Shows exactly which commit, workflow, and repository produced the package
- Visible on npmjs.com package pages

### 5. Environment Configuration

Use GitHub environments for additional security:

```yaml
environment:
  name: npm-registry
  url: https://www.npmjs.com/package/@memnexus-ai/sdk
```

This allows you to configure:
- Required reviewers before publishing
- Wait timers
- Environment-specific secrets (if needed)

## Workflow Example

Here's a minimal example of publishing with npm Trusted Publishing:

```yaml
name: Publish Package

on:
  push:
    branches:
      - main

jobs:
  publish:
    name: Publish to npm
    runs-on: ubuntu-latest
    permissions:
      contents: write
      id-token: write
    environment:
      name: npm-registry
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20.x'
          registry-url: 'https://registry.npmjs.org'
          scope: '@memnexus-ai'

      - name: Ensure latest npm
        run: npm install -g npm@latest

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Publish
        env:
          NPM_CONFIG_PROVENANCE: true
        run: npm publish --access public --provenance
```

## Version Management

### Check if Version Exists

Before publishing, check if the version already exists on npm:

```bash
if npm view @memnexus-ai/sdk@1.0.0 version 2>/dev/null; then
  echo "Version already exists"
  exit 0
fi
```

### Bump Version

Use semantic versioning:

```bash
# Patch (1.0.0 → 1.0.1)
npm version patch --no-git-tag-version

# Minor (1.0.0 → 1.1.0)
npm version minor --no-git-tag-version

# Major (1.0.0 → 2.0.0)
npm version major --no-git-tag-version
```

## Package Configuration

Each package's `package.json` must include:

```json
{
  "name": "@memnexus-ai/sdk",
  "version": "1.0.0",
  "publishConfig": {
    "access": "public",
    "registry": "https://registry.npmjs.org"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/memnexus-ai/mx-releases.git",
    "directory": "packages/sdk"
  }
}
```

**Key fields:**
- `publishConfig.access: "public"` - Makes package publicly accessible
- `publishConfig.registry` - Ensures publishing to npmjs.com
- `repository` - Links package to source code for provenance

## Unified Version Management

All three packages (SDK, CLI, MCP) share the same version number because:
- They're generated from the same OpenAPI spec
- They're generated in a single liblab operation
- They represent the same API surface
- They're released atomically (all together or none)

**VERSION.yaml** is the single source of truth:

```yaml
generated:
  version: 1.0.0  # Applied to all three packages
  packages:
    sdk:
      package: "@memnexus-ai/sdk"
    cli:
      package: "@memnexus-ai/cli"
    mcp:
      package: "@memnexus-ai/mcp-server"
```

## Troubleshooting

### Issue: "npm ERR! 403 Forbidden"

**Solution:** Ensure the GitHub repository is configured in npm Trusted Publishing settings for the `@memnexus-ai` organization.

### Issue: "npm ERR! need auth"

**Solution:** Verify that:
- `id-token: write` permission is set in the workflow
- `registry-url` is configured in `actions/setup-node`
- npm version is up to date (`npm@latest`)

### Issue: "Version already exists"

**Solution:** This is expected behavior if the version was already published. Either:
- Skip publishing (version check in workflow)
- Bump the version number
- Delete the version from npm (not recommended)

### Issue: Provenance not showing on npmjs.com

**Solution:** Ensure:
- `NPM_CONFIG_PROVENANCE: true` is set
- `--provenance` flag is used in `npm publish`
- npm version is 9.5.0 or higher

## Security Best Practices

1. **Use Trusted Publishing** - Never store npm tokens as secrets
2. **Enable Provenance** - Provides supply chain transparency
3. **Use Environments** - Add required reviewers for production publishes
4. **Version Checking** - Prevent accidental republishing
5. **Monitor npm Audit** - Regularly check for vulnerabilities

## References

- [npm Trusted Publishing](https://github.blog/changelog/2023-04-19-npm-provenance-public-beta/)
- [GitHub OIDC](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
- [npm Provenance](https://docs.npmjs.com/generating-provenance-statements)
- [actions/setup-node](https://github.com/actions/setup-node)

---

**Last Updated:** 2025-12-10
**Status:** Active
