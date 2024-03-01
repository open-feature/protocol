# 3. No flag type in request or response

Date: 2024-03-01

## Status

Accepted

## Context

Feature flags in OpenFeature can have several different types.
Often you see `boolean | number | string | structure` and often specific number types like `integer | float`.
Details can be seen in the [OF specification](https://openfeature.dev/specification/types).

When evaluating a flag via OFREP, the provider implementation will have to parse the flag value from the JSON response.
There are different approaches to this. Some feature flagging services define an expected flag type in the request.
Some include the flag type or even a schema in the response and some do not specify any type either in request or response.

## Decision

In the community meeting on 2024-02-29, it was concluded **not** to include type details in any payload.
The reason for this is to keep the API simple till there is a need for improvements.

Provider implementations are expected to infer the types from the JSON response.
Given that OpenFeature evaluation supports primitive types, value conversions may focus on evaluation type and deserialized JSON value type.
To differentiate integers and floats, the provider implementations will need to enforce JSON number parsing rules in the respective language.

## Consequences

- OFREP stays simple
- Parsing flag types can be done by using the JSON value type
- Parsing language specific types like floats will be up to the provider implementations
- Not having the request type in the request results in the feature flagging service, not knowing when there is a flag type mismatch
