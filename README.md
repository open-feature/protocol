# OpenFeature Remote Evaluation Protocol (OFREP)

---

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/open-feature/community/0e23508c163a6a1ac8c0ced3e4bd78faafe627c7/assets/logo/horizontal/white/openfeature-horizontal-white.svg" />
    <img align="center" alt="OpenFeature Logo" src="https://raw.githubusercontent.com/open-feature/community/0e23508c163a6a1ac8c0ced3e4bd78faafe627c7/assets/logo/horizontal/black/openfeature-horizontal-black.svg" />
  </picture>
</p>

![WIP](https://img.shields.io/badge/status-WIP-red)  
:warning: __OpenFeature Remote Evaluation Protocol is a WIP initiative, expect breaking changes__ :warning:


## What is OFREP?
OpenFeature Remote Flag Evaluation Protocol, is an OpenAPI specification for flag management systems that allows the use of generic providers to connect to any feature flag service that supports the protocol.


## Goals
- develop a client/server feature flag evaluation protocol:
  - define payloads sent between client and server
  - define the resources and functions available on the server
  - define the supported transports and encoding methods
  - define mechanisms for metadata delivery and transport configuration (headers, etc)
- develop production-ready reference implementation
  - compliant server
  - compliant client (provider)
    - highly configurable and modular
    - compatible with any server implementation

## Non-Goals

- defining a schema for the definition or storage of feature flags
- defining an expression language for feature flag targeting rules

## Design Principles

We value the following:
  - adherence to OpenFeature semantics
  - maximum compatibility and ease-of-adoption for existing solutions
  - minimum traffic and payload size
  - low latency of flag evaluations
  - portability and interoperability between runtimes and languages (should be usable in web, mobile devices, and server)
  - minimum reliance on dependencies (leverage ubiquitous, native transports and encodings where possible)
 
## Contribution
If you are interested about the OpenFeature Remote Evaluation Protocol you can join the [`#openfeature-remote-evaluation-protocol`](https://cloud-native.slack.com/archives/C066A48LK35) slack channel on the [CNCF Slack](https://communityinviter.com/apps/cloud-native/cncf).