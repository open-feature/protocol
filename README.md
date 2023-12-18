# OpenFeature Remote Evaluation Protocol (OFREP)

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
