# liblab Setup and Configuration

This document explains how liblab is configured to generate the MemNexus SDK, CLI, and MCP Server packages from the OpenAPI specification.

## Overview

**liblab** is an SDK generation platform that creates production-ready client libraries from OpenAPI specifications. For MemNexus, we use liblab to generate all three client-facing packages (SDK, CLI, MCP Server) from a single OpenAPI spec, ensuring consistency and unified versioning.

## liblab Account Setup

### 1. Create Account

1. Visit [liblab.com](https://liblab.com)
2. Sign up for an account
3. Navigate to API settings to generate an API token

### 2. Generate API Token

1. Go to [https://app.liblab.com/settings/tokens](https://app.liblab.com/settings/tokens)
2. Create a new API token
3. Copy the token (format: `liblab_xxxxx...`)

### 3. Add Token to GitHub Secrets

The API token is stored as a repository secret:

```bash
# Via GitHub UI:
# Settings → Secrets and variables → Actions → New repository secret
# Name: LIBLAB_API_KEY
# Value: liblab_xxxxx...
```

Or via GitHub CLI:

```bash
gh secret set LIBLAB_API_KEY --body "liblab_xxxxx..." --repo memnexus-ai/mx-releases
```

## Configuration File: liblab.config.json

The root `liblab.config.json` file configures SDK generation for all packages.

### Current Configuration

```json
{
  "sdkName": "memnexus",
  "apiVersion": "1.0.0",
  "apiName": "MemNexus",
  "specFilePath": "specs/openapi.yaml",
  "languages": ["typescript"],
  "languageOptions": {
    "typescript": {
      "bundle": true,
      "githubRepoName": "memnexus-ai/mx-releases",
      "homepage": "https://github.com/memnexus-ai/mx-releases",
      "liblabVersion": "2",
      "sdkVersion": "1.0.0",
      "httpLibrary": "axios"
    }
  },
  "publishing": {
    "npm": {
      "packageName": "@memnexus-ai/sdk",
      "githubRepoName": "memnexus-ai/mx-releases"
    }
  },
  "customizations": {
    "packageName": "@memnexus-ai/sdk",
    "devContainer": false,
    "generateEnv": true,
    "inferServiceNames": false,
    "license": {
      "type": "MIT",
      "url": "https://opensource.org/licenses/MIT"
    },
    "retry": {
      "enabled": true,
      "maxAttempts": 3,
      "retryDelay": 150,
      "maxDelay": 5000,
      "retryDelayJitter": 50,
      "backOffFactor": 2,
      "httpCodesToRetry": [408, 429, 500, 502, 503, 504],
      "httpMethodsToRetry": ["GET", "POST", "PUT", "DELETE", "PATCH", "HEAD", "OPTIONS"]
    }
  },
  "deliverables": [
    {
      "type": "compiled-sdk",
      "outputPath": "packages/sdk"
    }
  ],
  "docs": ["api"],
  "auth": []
}
```

### Key Configuration Options

#### Basic Settings

- **sdkName**: Base name for the SDK (used in client class names)
- **apiVersion**: Version of the API (sync with VERSION.yaml)
- **apiName**: Human-readable API name
- **specFilePath**: Path to OpenAPI specification file
- **languages**: Array of target languages (currently just TypeScript)

#### Language Options

- **bundle**: Enable bundling for browser compatibility
- **githubRepoName**: Repository for GitHub integration
- **homepage**: Package homepage URL
- **liblabVersion**: liblab SDK generation version
- **sdkVersion**: SDK version (sync with VERSION.yaml)
- **httpLibrary**: HTTP client library ("axios" for TypeScript)

#### Publishing

- **npm.packageName**: npm package name (`@memnexus-ai/sdk`)
- **npm.githubRepoName**: Repository for package publishing

#### Customizations

- **packageName**: Override generated package name
- **generateEnv**: Generate .env.example file
- **license**: MIT license configuration
- **retry**: Automatic retry configuration for failed requests
  - Exponential backoff
  - Configurable HTTP codes and methods to retry
  - Jitter to prevent thundering herd

#### Deliverables

- **type**: `compiled-sdk` for pre-built SDK
- **outputPath**: Where to place generated code

#### Documentation

- **docs**: Array of documentation types to generate
  - `"api"`: API reference documentation
  - `"snippets"`: Code snippets and examples
  - `"enhancedApiSpec"`: Enhanced OpenAPI specification

## Local SDK Generation

### Prerequisites

```bash
# Install liblab CLI
npm install -g liblab

# Verify installation
liblab --version
```

### Generate SDK Locally

```bash
# Set API token
export LIBLAB_TOKEN="liblab_xxxxx..."

# Generate SDK
cd /path/to/mx-releases
liblab build --yes

# Output will be in ./output/typescript/
```

### Verify Generated SDK

```bash
# Check generated structure
tree output/typescript -L 2

# Verify package.json
cat output/typescript/package.json

# Build the SDK
cd output/typescript
npm install
npm run build

# Run type checks
npm test
```

## Generated SDK Structure

```
output/typescript/
├── LICENSE                    # MIT license
├── README.md                  # Usage documentation
├── package.json               # Package configuration
├── tsconfig.json              # TypeScript configuration
├── src/
│   ├── index.ts               # Main entry point
│   ├── http/                  # HTTP client layer
│   │   ├── types.ts
│   │   ├── handlers.ts
│   │   └── environment.ts
│   └── services/              # Generated service classes
│       ├── MemoriesService.ts
│       ├── ConversationsService.ts
│       ├── FactsService.ts
│       └── ...
├── documentation/             # API documentation
│   ├── models/
│   └── services/
├── examples/                  # Usage examples
│   ├── README.md
│   ├── src/
│   └── package.json
└── scripts/
    └── prepublish.sh          # Pre-publish script
```

## Integration with Workflows

### GitHub Actions Workflow

```yaml
- name: Install liblab CLI
  run: npm install -g liblab

- name: Generate SDK
  env:
    LIBLAB_TOKEN: ${{ secrets.LIBLAB_API_KEY }}
  run: |
    liblab build --yes
    # Move output to packages/sdk
    rm -rf packages/sdk/*
    cp -r output/typescript/* packages/sdk/

- name: Update package.json version
  run: |
    cd packages/sdk
    npm version ${{ env.SDK_VERSION }} --no-git-tag-version

- name: Build SDK
  run: |
    cd packages/sdk
    npm install
    npm run build
```

## Multi-Package Generation (Future)

Currently, liblab generates only the TypeScript SDK. To generate CLI and MCP Server:

### Option 1: Post-Processing

Generate SDK, then create CLI and MCP Server wrappers:

```
SDK (liblab) → CLI wrapper → MCP wrapper
```

### Option 2: Multiple Configurations

Create separate liblab configs for each package:

```
liblab.config.sdk.json  → @memnexus-ai/sdk
liblab.config.cli.json  → @memnexus-ai/cli
liblab.config.mcp.json  → @memnexus-ai/mcp-server
```

### Option 3: Custom Templates

Use liblab hooks and custom templates to generate all three in one operation.

**Current Approach:** We use liblab for SDK generation, then create CLI and MCP Server as wrapper packages that depend on the SDK.

## Version Synchronization

All packages share the same version from VERSION.yaml:

```yaml
generated:
  version: 1.0.0
```

Update workflow:
1. Update `VERSION.yaml`
2. Workflow reads version
3. liblab generates SDK with version
4. CLI and MCP package.json updated to match
5. All publish together

## Customization Options

### HTTP Library

Choose HTTP client (currently axios):

```json
{
  "languageOptions": {
    "typescript": {
      "httpLibrary": "axios"  // or "fetch", "node-fetch"
    }
  }
}
```

### Retry Configuration

Customize automatic retry behavior:

```json
{
  "customizations": {
    "retry": {
      "enabled": true,
      "maxAttempts": 3,
      "retryDelay": 150,
      "maxDelay": 5000,
      "httpCodesToRetry": [408, 429, 500, 502, 503, 504]
    }
  }
}
```

### Authentication

Configure authentication handling:

```json
{
  "auth": []  // Auto-detected from OpenAPI securitySchemes
}
```

liblab automatically detects and implements authentication from the OpenAPI spec's `securitySchemes`.

## Troubleshooting

### Issue: "ZodError" validation errors

**Cause:** Invalid liblab.config.json structure

**Solution:** Validate config against liblab schema:
```bash
liblab config validate
```

Common issues:
- `docs` must be array of strings, not objects
- `authentication.access` must be object, not string
- Unknown fields in config

### Issue: Generation fails with API errors

**Cause:** Invalid OpenAPI specification

**Solution:** Validate OpenAPI spec:
```bash
npx @openapitools/openapi-generator-cli validate -i specs/openapi.yaml
```

### Issue: Generated SDK has wrong package name

**Cause:** `customizations.packageName` overrides `publishing.npm.packageName`

**Solution:** Ensure both are set correctly:
```json
{
  "publishing": {
    "npm": {
      "packageName": "@memnexus-ai/sdk"
    }
  },
  "customizations": {
    "packageName": "@memnexus-ai/sdk"
  }
}
```

### Issue: Build fails after generation

**Cause:** Missing dependencies or TypeScript errors

**Solution:**
```bash
cd output/typescript
npm install
npm run build
```

Check for TypeScript errors in generated code and report to liblab if needed.

## Best Practices

1. **Version Control liblab.config.json** - Commit configuration to git
2. **Validate OpenAPI Spec First** - Ensure spec is valid before generation
3. **Review Generated Code** - Check generated SDK for quality
4. **Test Generated SDK** - Run unit tests against generated code
5. **Lock liblab Version** - Use specific liblab CLI version in CI/CD
6. **Monitor Generation** - Check liblab portal for build status
7. **Cache node_modules** - Speed up workflow with dependency caching

## Resources

- [liblab Documentation](https://liblab.com/docs)
- [liblab Portal](https://app.liblab.com)
- [liblab CLI Reference](https://liblab.com/docs/cli)
- [TypeScript SDK Guide](https://liblab.com/docs/languages/typescript)
- [OpenAPI Specification](https://spec.openapis.org/oas/v3.0.3)

---

**Last Updated:** 2025-12-10
**liblab CLI Version:** 0.49.48
**Status:** Active
