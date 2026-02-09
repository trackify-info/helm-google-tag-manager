# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.4.0] - 2026-02-09

### Added
- **Envoy Gateway Support**: Added support for Kubernetes Gateway API (v1) with HTTPRoute resources
  - New `gateway` configuration section for both `previewServer` and `serverSideTagging`
  - HTTPRoute templates (`ps-httproute.yaml` and `sst-httproute.yaml`)
  - Flexible configuration with support for custom `parentRefs`, hostnames, and routing rules
  - Support for advanced features like request/response filters and header modifications
  - Ability to use both Ingress and Gateway simultaneously

### Changed
- Chart version bumped from 0.3.0 to 0.4.0
- Updated README with comprehensive Envoy Gateway examples and usage instructions

### Removed
- Removed incorrect `sst-gateway.yaml` template that was using Ingress API instead of Gateway API

## [0.3.0] - Previous Release

(Previous changelog entries would go here)
