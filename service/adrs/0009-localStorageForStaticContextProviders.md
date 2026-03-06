# 9. Persist static-context evaluations in local storage by default for web and mobile providers

Date: 2026-03-06

## Status

Proposed

## Context

OFREP static-context providers evaluate all flags in one request and then serve evaluations from a local cache.
This model works well when the provider can reach the OFREP service during initialization and while polling for updates.

Web and mobile applications often operate with intermittent connectivity.
They are also frequently restarted, which means an in-memory cache is lost between sessions.
Today, a client may have a previously successful bulk evaluation, but if it starts while offline it cannot reuse that evaluation unless the provider implementation persists it locally.

This creates a poor experience for static-context providers in the environments they are primarily meant to support.
An offline user may see feature state regress to errors or code defaults even though the application already had a usable last-known evaluation.

OFREP already supports local cached evaluation, bulk evaluation, polling, and ETag-based revalidation.
Persisting the last successful static-context evaluation extends the existing cache model to survive application restarts and temporary loss of connectivity.

## Decision

Web and mobile OFREP providers that implement the static-context paradigm should persist their last successful bulk evaluation in local persistent storage by default.

The persisted entry should include:

- the bulk evaluation payload
- the associated `ETag`, if one was returned
- enough metadata to determine whether the entry applies to the current provider instance, such as the OFREP endpoint and the static evaluation context or a stable derived key for it
- the time the entry was written

The provider should continue to use its in-memory cache for normal flag evaluation.
Persistent local storage acts as the source used to bootstrap or recover that in-memory cache.

During initialization, a provider should:

1. Attempt to load a matching persisted bulk evaluation from local storage.
2. Attempt the normal `/ofrep/v1/evaluate/flags` request.
3. If the request succeeds, populate the in-memory cache and update the persisted entry.
4. If the request cannot complete because the client is offline or the network is temporarily unavailable, and a matching persisted entry exists, populate the in-memory cache from that persisted entry and continue operating from it.
5. If no matching persisted entry exists, preserve the existing initialization failure behavior.

Providers should only reuse a persisted evaluation when it matches the current static-context inputs.
At minimum, this includes the target OFREP service and the evaluation context.
Implementations may include additional inputs in the cache key when they affect the returned evaluation.

Fallback to persisted data is intended for offline or transient network failures.
Providers should not silently fall back to persisted data for authorization failures, invalid requests, or other server responses that indicate a configuration or protocol problem.

When connectivity returns, the provider should resume its normal refresh behavior.
If an `ETag` was stored with the persisted entry, the provider should use it with `If-None-Match` when revalidating the bulk evaluation.

Providers should allow applications to disable the default persistence behavior or replace the storage backend when platform requirements or policy constraints require it.

## Consequences

### Positive

- Static-context providers become resilient to offline application startup when a last-known evaluation exists
- Web and mobile applications preserve feature state across restarts instead of losing it with the in-memory cache
- The decision aligns with the existing OFREP model where static-context providers evaluate remotely once and then read locally
- Reusing the stored `ETag` allows efficient revalidation when connectivity returns
- Provider implementations get a consistent default expectation for offline behavior across ecosystems

### Negative

- Providers become more complex because they must manage persistence, cache-key matching, and recovery flows
- Persisted evaluations may become stale, so applications can continue using outdated flag values while offline
- Local persistent storage can be unavailable, limited in size, or restricted by platform policy
- Persisting evaluation data introduces security and privacy considerations, especially if flag metadata or context-derived identifiers are sensitive
- Mobile platforms do not share a single storage API, so providers may need platform-specific defaults behind a common abstraction

## Alternatives Considered

### Keep static-context caches in memory only

This keeps provider implementations simpler, but it means offline startup cannot use a previously successful evaluation.
That undermines a core advantage of static-context evaluation for web and mobile environments.

### Make persistence opt-in instead of the default

This reduces default behavior changes, but it produces inconsistent offline behavior across provider implementations and requires every application to rediscover and enable the same capability.
For web and mobile static-context providers, persistence is expected behavior rather than an exceptional optimization.

### Add protocol-level support for offline snapshots

This could standardize snapshot delivery more explicitly, but it would require protocol changes.
Persisting the existing bulk evaluation response is sufficient for the current need and can be implemented entirely within providers.

## Implementation Notes

- "Local storage" means a local persistent key-value store appropriate for the runtime, such as browser `localStorage` on the web or an equivalent mobile storage mechanism
- Providers should version their persisted format so future schema changes can be handled safely
- Providers should clear or replace persisted entries when the static context changes, such as on logout or user switch
- SDK documentation should describe that offline fallback uses the last successful bulk evaluation and may therefore serve stale values until connectivity returns
