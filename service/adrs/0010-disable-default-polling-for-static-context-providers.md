# 10. Disable default polling for static-context providers

Date: 2026-03-19

## Status

Accepted

## Context

OFREP static-context providers currently poll on a fixed timer by default. The JS web provider polls every 30 seconds, the Swift provider every 30 seconds, and the Kotlin provider every 5 minutes. This was originally specified in [ADR-0005](0005-polling-for-bulk-evaluation-changes.md), which established polling as the change detection mechanism for OFREP.

Timer-based polling is problematic for static-context providers for two reasons:

1. **Sticky bucketing** — while bucketing is generally deterministic for the same context, time-based targeting rules (commonly used in rollouts) can get triggered between polling intervals, causing evaluated values to change mid-session. Most platforms expect feature flag bucketing to remain sticky for a user's whole session. Changing flag values mid-session can cause UI flicker, inconsistent experiences, and breaks experiment integrity.

2. **Resource efficiency** — timer-based polling is wasteful for static-context providers where the evaluation context rarely changes during a session. On mobile, frequent polling impacts battery life and data usage unnecessarily.

This is out of step with how vendor SDKs handle client-side refresh. The majority of vendors do not use timer-based polling as the default client-side refresh mechanism:

| Vendor | Default Mechanism | Timer Polling? |
|--------|-------------------|----------------|
| **DevCycle** | Init + context change + foreground + SSE push | No |
| **LaunchDarkly** | SSE streaming foreground, 60min poll background only | No (streams) |
| **Statsig** | Init + user change only | No |
| **Eppo** | Init only, polling opt-in | No (off by default) |
| **ConfigCat** | Auto polling (60s) | Yes |
| **Unleash** | Polling (30s) | Yes |

DevCycle's mobile and web SDKs re-fetch configuration only on initialization, user change (`identifyUser`/`resetUser`), app foreground / page visibility, and SSE push events. There is no timer-based polling for config. LaunchDarkly uses SSE streaming when foregrounded and only polls in the background at 60-minute intervals. Statsig and Eppo do not poll at all by default.

[ADR-0008](0008-sse-for-bulk-evaluation-changes.md) introduces SSE as an optional real-time change notification mechanism. Combined with event-driven refresh triggers, this provides a complete alternative to timer-based polling for static-context providers.

## Decision

Static-context providers should not poll on a fixed timer by default. Instead, providers should re-fetch bulk evaluations based on event-driven triggers:

1. **Initialization** — as defined in [ADR-0009](0009-localStorageForStaticContextProviders.md), providers fetch bulk evaluations during initialization (or load from cache and refresh in the background).

2. **Context change** — when `onContextChange` is invoked (e.g., user switch, logout), the provider re-fetches bulk evaluations for the new context. This is already handled by existing provider implementations.

3. **App foreground / page visibility** — when the application returns from the background or the page becomes visible, the provider should re-fetch bulk evaluations. This ensures values are fresh when the user returns without requiring continuous polling.

4. **SSE push** — when an SSE `refetchEvaluation` event is received (as defined in [ADR-0008](0008-sse-for-bulk-evaluation-changes.md)), the provider re-fetches bulk evaluations. This provides real-time updates when the server supports it.

Timer-based polling should remain available as an opt-in configuration for applications that want it, but the default poll interval should be disabled (`0`).

### Foreground detection

Providers should detect app foreground / page visibility using platform-appropriate mechanisms and re-fetch bulk evaluations when the application becomes active.

On foreground, if the provider has an active SSE connection, no re-fetch is needed as the SSE connection handles change detection. If the SSE connection was closed during inactivity (as described in [ADR-0008](0008-sse-for-bulk-evaluation-changes.md)), the provider should reconnect and perform a full unconditional re-fetch.

If SSE is not configured or not supported by the server, the provider should re-fetch on foreground.

### Interaction with SSE (ADR-0008)

The `changeDetection` configuration option defined in [ADR-0008](0008-sse-for-bulk-evaluation-changes.md) already covers the relevant modes:

- `sse` (default): Use SSE if available, with foreground refresh as a complement for resuming from inactivity.
- `polling`: Opt-in timer-based polling for applications that want it. When this mode is selected, the provider should still re-fetch on foreground in addition to polling.
- `none`: No background refresh. Rely solely on initialization and `onContextChange` calls.

This ADR changes the behavior within the `sse` default mode: instead of falling back to timer-based polling when SSE is unavailable, the provider relies on foreground refresh and explicit context change triggers. Timer-based polling is only used when explicitly configured via the `polling` mode.

### ETag revalidation

All re-fetch triggers (foreground, SSE push, context change, opt-in polling) should use the stored `ETag` with `If-None-Match` for bandwidth efficiency, except when performing a full unconditional re-fetch after SSE inactivity (as specified in [ADR-0008](0008-sse-for-bulk-evaluation-changes.md)).

## Consequences

### Positive

- Flag values remain consistent within a session, preserving sticky bucketing expectations
- Reduced network traffic and battery usage on mobile devices
- Aligns with the established pattern used by the majority of vendor SDKs (DevCycle, LaunchDarkly, Statsig, Eppo)
- App foreground refresh ensures values are fresh when the user returns from background
- Combined with SSE (ADR-0008), provides real-time updates without the overhead of continuous polling

### Negative

- Flag changes made server-side won't reach the client until the next foreground event, context change, or SSE push (if configured). Applications that need mid-session updates without SSE must opt into polling.
- Providers must implement platform-specific foreground detection, adding complexity across iOS, Android, and web
- Breaking change for existing applications that rely on the current default polling behavior to receive mid-session flag updates

## Implementation Notes

- Foreground detection should be implemented behind a platform abstraction so it can be tested and swapped
- The existing `pollInterval` configuration should default to `0` (disabled) instead of the current positive values (JS: 30000ms, Swift: 30s, Kotlin: 5min). Applications can opt into polling by setting a positive value.
- This ADR supersedes the default polling behavior established in [ADR-0005](0005-polling-for-bulk-evaluation-changes.md) for static-context providers only. ADR-0005 remains valid for the polling protocol mechanism itself and for dynamic-context (server-side) providers.
- Providers should emit `ConfigurationChanged` when a foreground re-fetch returns updated flag values
- SDK documentation should note the change in default behavior and how to opt into timer-based polling if needed

## Open Questions

1. ~~Should the ADR specify a recommended cooldown duration for foreground re-fetches, or leave this to implementations?~~
   - **Answer:** Not needed. ETag/304 already short-circuits redundant requests, and rapid foreground/background cycling is not a realistic concern.
2. How should providers handle the transition for existing applications that depend on the current default polling behavior? Should there be a deprecation period or migration guide?
   - **Answer:** This can be treated as a breaking provider change. OFREP providers are all sub-v1, so there is no semver obligation to provide a deprecation period. Breaking provider changes are typically just changes in options or behaviors and are straightforward to handle.

## Related

- [ADR-0005: Polling for bulk evaluation changes](0005-polling-for-bulk-evaluation-changes.md)
- [ADR-0008: SSE for bulk evaluation changes](0008-sse-for-bulk-evaluation-changes.md)
- [ADR-0009: Local storage for static-context providers](0009-localStorageForStaticContextProviders.md)
- [Issue #68: OFREP Static-Context Provider Default Refresh Behavior](https://github.com/open-feature/protocol/issues/68)
