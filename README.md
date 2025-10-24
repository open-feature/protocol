<!-- markdownlint-disable MD033 -->
<!-- x-hide-in-docs-start -->
<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/open-feature/community/0e23508c163a6a1ac8c0ced3e4bd78faafe627c7/assets/logo/horizontal/white/openfeature-horizontal-white.svg" />
    <img align="center" alt="OpenFeature Logo" src="https://raw.githubusercontent.com/open-feature/community/0e23508c163a6a1ac8c0ced3e4bd78faafe627c7/assets/logo/horizontal/black/openfeature-horizontal-black.svg" />
  </picture>
</p>

<h2 align="center">OpenFeature Remote Evaluation Protocol (OFREP)</h2>

![Protocol version](https://img.shields.io/github/v/release/openfeature/protocol)

## What is OFREP?
<!-- x-hide-in-docs-end -->

The **OpenFeature Remote Evaluation Protocol (OFREP)** is an API specification for feature flagging that enables vendor-agnostic communication between applications and flag management systems. It defines a standard API layer between the provider and the flag management system, allowing any OpenSource or commercial system to implement the protocol and be compatible with community-maintained providers.

### Key Benefits

- **Vendor Agnostic**: Connect to any OFREP-compliant flag management system without vendor-specific implementations
- **Standardized**: Built on a common OpenAPI specification for consistent integration
- **Flexible**: Works on both client-side and server-side applications
- **Community Maintained**: Generic OFREP providers maintained by the OpenFeature community
- **Simple Migration**: Switch between flag management systems without changing application code

## How It Works

OFREP is **a protocol, not a provider**. It defines how to communicate with feature flag management systems and works on top of OpenFeature SDKs by providing standardized providers.

```mermaid
graph LR
    A[Your Application] --> B[OpenFeature SDK]
    B --> C[OFREP Provider]
    C --> D[OFREP API]
    D --> E[Flag Management System]
```

1. Your application uses the OpenFeature SDK
2. The OpenFeature SDK uses an OFREP provider
3. The OFREP provider communicates with your flag management system via the standardized OFREP API
4. Your flag management system implements the OFREP specification

### Server vs Client

OFREP supports both paradigms defined by OpenFeature:

#### Server-Side (Dynamic Context)

- Evaluation happens by calling the OFREP API with context
- No in-process evaluation inside OFREP providers, API request made on every evaluation.
- Context changes frequently (per request, per user)

#### Client-Side (Static Context)

- Evaluation happens locally with pre-fetched flag data
- Provider reconciles state using the OFREP API when context changes
- Context represents a single user or session

## Using OFREP

### Available Providers

The OpenFeature community maintains OFREP providers for multiple languages. View the complete list of [OFREP providers in the ecosystem](https://openfeature.dev/ecosystem/?instant_search%5BrefinementList%5D%5Bvendor%5D%5B0%5D=OFREP).

### Available Flag Management Systems

Many flag management systems support OFREP. View the complete list of [OFREP-compliant systems in the ecosystem](https://openfeature.dev/ecosystem/?instant_search%5BrefinementList%5D%5Btype%5D%5B0%5D=OFREP%20API).

## Implementing OFREP

### For Provider Developers

To create an OFREP provider for a new language:

1. Review the [server provider guideline](./guideline/dynamic-context-provider.md) or [client provider guideline](./guideline/static-context-provider.md)
2. Implement the provider interface for your SDK
3. Add HTTP client logic to call OFREP endpoints
4. Handle error cases and response codes according to the specification
5. Register your provider in the [OpenFeature ecosystem](https://github.com/open-feature/openfeature.dev/issues)

### For Flag Management System Developers

To make your flag management system OFREP-compliant:

1. Implement the [OFREP OpenAPI specification](./service/openapi.yaml)
2. Expose OFREP endpoints at `/ofrep/v1/evaluate/...`
3. Support both bulk evaluation (for client-side) and single flag evaluation (for server-side)
4. Test your implementation with existing OFREP providers
5. Register your system in the [OpenFeature ecosystem](https://github.com/open-feature/openfeature.dev/issues)

## Resources

- **OpenAPI Specification**: [OFREP OpenAPI Spec](./service/openapi.yaml)
- **Provider Guidelines**:
  - [Server Provider Guide](./guideline/dynamic-context-provider.md)
  - [Client Provider Guide](./guideline/static-context-provider.md)
- **Ecosystem**:
  - [OFREP Providers](https://openfeature.dev/ecosystem/?instant_search%5BrefinementList%5D%5Bvendor%5D%5B0%5D=OFREP)
  - [OFREP-Compliant Systems](https://openfeature.dev/ecosystem/?instant_search%5BrefinementList%5D%5Btype%5D%5B0%5D=OFREP%20API)
- **CNCF Slack**: Join [#openfeature-remote-evaluation-protocol](https://cloud-native.slack.com/archives/C066A48LK35)

## Get Involved

OFREP is an open standard maintained by the OpenFeature community. We welcome contributions:

- **Implement OFREP** in your flag management system
- **Create providers** for additional languages
- **Provide feedback** on the specification
- **Share your experience** using OFREP

Join the [`#openfeature-remote-evaluation-protocol`](https://cloud-native.slack.com/archives/C066A48LK35) channel on the [CNCF Slack](https://communityinviter.com/apps/cloud-native/cncf) to get involved.
