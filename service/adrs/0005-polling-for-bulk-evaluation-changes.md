# 5. Polling for bulk evaluation changes

Date: 2024-03-01

## Status

Accepted

## Context

In most systems, feature flags can be changed at any time.
This is a challenge for bulk evaluation where the typical use case is caching the evaluated flags and evaluating from the cache.

Many feature flagging systems provide ways to inform clients about changes in feature flag values.
This can reach from polling clients over to server-sent events and websockets.

Some providers also define polling endpoints that list changed feature flags instead of polling the evaluation endpoint.

The [vendor survey](https://docs.google.com/forms/d/1NzqKx57XvRK_2lRQOFCRmF5exet6f15-sCjdEy0HCS8#responses) showed that many vendors do simple polling from clients or SSE.

## Decision

OFREP intends provider implementations to do polling. For now no streaming like SSEs or websockets is specified.
The reason is simplicity and openness to define additional mechanisms later.

Optional [ETag](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag) can be used with the bulk evaluation endpoint.
The motivation here is to save wire bandwidth and evaluation burden on the flag management system.
If present in response, the implementation will attach ETag when polling and respect 304 responses from the flag management system.

Individual flag evaluations may be cached to increase performance.
But this is out of the scope of the OpenAPI specification and is an implementation detail of the provider.

## Consequences

- OFREP stays simple
- There is no way of implementing real-time flag updates for now
  - Frequent evaluations from polling clients may introduce additional processing for feature flagging services
- Additional change detection mechanisms will need to be added later
