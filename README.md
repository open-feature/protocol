# OpenFeature Remote Evaluation Protocol (OFREP)

---

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/open-feature/community/0e23508c163a6a1ac8c0ced3e4bd78faafe627c7/assets/logo/horizontal/white/openfeature-horizontal-white.svg" />
    <img align="center" alt="OpenFeature Logo" src="https://raw.githubusercontent.com/open-feature/community/0e23508c163a6a1ac8c0ced3e4bd78faafe627c7/assets/logo/horizontal/black/openfeature-horizontal-black.svg" />
  </picture>
</p>

![Protocol version](https://img.shields.io/github/v/release/openfeature/protocol)

## What is OFREP?
**OpenFeature Remote Flag Evaluation Protocol**, is an API specification for feature flagging that allows the use of generic providers to connect to any feature flag management systems that supports the protocol.

## Goal
The primary goal of the OpenFeature Remote Evaluation Protocol (OFREP) is to establish a standardized, vendor-agnostic communication layer for feature flag evaluation. This protocol aims to decouple applications from specific feature flag vendors, fostering a more flexible and interoperable ecosystem.

At its heart, OFREP defines a standard API layer between the provider and the flag management system, allowing OpenSource and commercial feature flag management systems to implement the protocol and to be compatible with the community maintained providers. It enables out-of-the-box compatibility with any OFREP compliant flag management system, regardless if they have a specific OpenFeature provider implementation or not.

If you are building an application that uses feature flags and you don't want to be tied to a specific vendor, you can use the OpenFeature Remote Evaluation Protocol to connect to any flag management system that supports the protocol, without needing to implement a specific provider for it.

## Things to know
- OFREP is a protocol, not a provider. It defines how to communicate with feature flag management systems.
- OFREP works on top of OpenFeature SDKs, by providing standardized providers that can be used to connect to any OFREP compliant flag management system.
- OFREP works on the client side and the server side. It can be used in web, mobile, and server applications.
- On the servers implementation, OFREP is designed to do evaluation by calling the OFREP API. There is no inprocess evaluation inside the OFREP providers.
- OpenFeature community has built a set of providers that implement the OFREP protocol, allowing you to connect to any OFREP compliant flag management system.

## OFREP Support
The OpenFeature community is maintaining a set of providers implementing the OFREP protocol, you can find the list of available providers in the [ecosystem page](https://openfeature.dev/ecosystem/?instant_search%5BrefinementList%5D%5Bvendor%5D%5B0%5D=OFREP) of the OpenFeature website.

You can also find the list of flag management systems that support the OFREP protocol in the [ecosystem page](https://openfeature.dev/ecosystem/?instant_search%5BrefinementList%5D%5Btype%5D%5B0%5D=OFREP%20API).


## Create a provider for OFREP.
If your language is not yet supported and you want to create a provider for OFREP, you can follow the OFREP provider guidelines, they will help you to implement a provider that is compatible with the OpenFeature Remote Evaluation Protocol.

- [OFREP Server Provider Guideline](./guideline/dynamic-context-provider.md)
- [OFREP Client Provider Guideline](./guideline/static-context-provider.md)
- [OpenAPI Specification](./service/openapi.yaml)

> [!NOTE]
> After implementing the provider, you can register it in the OpenFeature ecosystem page by [creating an issue](https://github.com/open-feature/openfeature.dev/issues).

## Implement OFREP in your flag management system.

If you want to implement the OpenFeature Remote Evaluation Protocol in your flag management system, you must support the OpenAPI specification and implement the different endpoints defined.

- [OpenAPI Specification](./service/openapi.yaml)

> [!NOTE]
> After implementing the OpenAPI specification, you can register your flag management system in the OpenFeature ecosystem page by [creating an issue](https://github.com/open-feature/openfeature.dev/issues), so users can find it and use it with the OFREP providers.

## Contribution
If you are interested about the OpenFeature Remote Evaluation Protocol you can join the [`#openfeature-remote-evaluation-protocol`](https://cloud-native.slack.com/archives/C066A48LK35) slack channel on the [CNCF Slack](https://communityinviter.com/apps/cloud-native/cncf).
