# Changelog

All notable changes to the MemNexus platform will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]
## [1.1.1] - 2025-12-31

### Changed
- Platform release 1.1.1
- Generated packages (SDK, CLI, MCP Server) updated to 1.1.1
- OpenAPI specification synced

## [1.1.1] - 2025-12-31

### Changed
- Platform release 1.1.1
- Generated packages (SDK, CLI, MCP Server) updated to 1.1.1
- OpenAPI specification synced

## [1.1.1] - 2025-12-30

### Changed
- Platform release 1.1.1
- Generated packages (SDK, CLI, MCP Server) updated to 1.1.1
- OpenAPI specification synced


### Infrastructure
- Established unified release coordination repository
- Created automated release pipeline (7 phases)
- Set up liblab-based package generation

---

## [1.0.0] - 2025-12-10

### Added
- Initial unified release system
- SDK, CLI, and MCP Server generation from single OpenAPI spec
- Centralized version coordination via VERSION.yaml
- Automated release pipeline from code commit to npm publish
- Platform-wide changelog and release notes

### Changed
- Migrated from fragmented release process to unified mx-releases repository
- Renamed `@memnexus-ai/contracts` to `@memnexus-ai/sdk`
- Moved from GitHub Packages to public npm registry

### Infrastructure
- Created mx-releases repository structure
- Set up VERSION.yaml as single source of truth
- Prepared for liblab integration for package generation
- Established 7-phase automated release pipeline

---

## Release Notes

### Platform Version 1.0.0

**Release Date:** 2025-12-10

**Components:**
- Core API: 1.0.0
- API Gateway: 1.0.0
- Generated Packages: 1.0.0
  - SDK: `@memnexus-ai/sdk`
  - CLI: `@memnexus-ai/cli`
  - MCP Server: `@memnexus-ai/mcp-server`

**Summary:**
This is the inaugural release of the unified MemNexus platform release system. All client-facing packages (SDK, CLI, MCP Server) are now generated from a single OpenAPI specification and share the same version number, ensuring perfect compatibility across all interfaces.

**Breaking Changes:**
- Package name change: `@memnexus-ai/contracts` → `@memnexus-ai/sdk`
- Publishing location: GitHub Packages → npmjs.com (public registry)

**Migration Guide:**
See [README.md#migration-from-old-packages](./README.md#migration-from-old-packages)

---

[Unreleased]: https://github.com/memnexus-ai/mx-releases/compare/platform-v1.0.0...HEAD
[1.0.0]: https://github.com/memnexus-ai/mx-releases/releases/tag/platform-v1.0.0
