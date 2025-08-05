# 6. Optional value field for code default deferral

Date: 2025-07-29

## Status

Accepted

## Context

Feature flag management systems often follow a standard release lifecycle where flags transition through different states. During early stages of a feature rollout, it's common practice to defer to the default value defined in the application code rather than returning an explicit value from the flag management system.

This pattern provides several benefits:
- Developers define default values once in code, avoiding duplication in the flag management system
- Flag management systems can gradually roll out features without needing to know the code default value
- The transition from code defaults to managed values becomes part of the standard feature flag lifecycle

Currently, OFREP requires the `value` field to be present in all successful evaluation responses. This forces flag management systems to either:
1. Duplicate the default value from code into their system
2. Return an error response when they want to defer to code defaults (misleading telemetry and error handling)

Neither approach is ideal. Returning errors for what is actually a successful evaluation leads to incorrect metrics and may trigger error handling logic in applications.

## Decision

Introduce a new flag type schema `codeDefaultFlag` that explicitly indicates the provider should use the default value defined in the application code. This new type would be used alongside existing flag types (booleanFlag, stringFlag, etc.) in evaluation responses.

Key implementation details:
- Add a new `codeDefaultFlag` schema type with no `value` field
- Existing flag type schemas remain unchanged (backward compatible)
- When a response uses `codeDefaultFlag` type, providers MUST use the code default value
- All other fields in the evaluation response (key, reason, variant, metadata) remain unchanged
- This behavior applies to both single flag evaluations and bulk evaluations
- The `oneOf` discriminator in evaluation responses would include this new type

Example OpenAPI schema addition:
```yaml
codeDefaultFlag:
  description: A flag evaluation that defers to the code default value
  type: object
  # Note: No value property - provider must use code default
```

The evaluation success response would be updated to include this type:
```yaml
- oneOf:
    - $ref: "#/components/schemas/booleanFlag"
    - $ref: "#/components/schemas/stringFlag"
    - $ref: "#/components/schemas/integerFlag"
    - $ref: "#/components/schemas/floatFlag"
    - $ref: "#/components/schemas/objectFlag"
    - $ref: "#/components/schemas/codeDefaultFlag"  # New type
```

Example response deferring to code default:
```json
{
  "key": "new-feature",
  "reason": "DEFAULT",
  "variant": "control"
  // No value field - indicates codeDefaultFlag type
}
```

Example response with explicit value (existing behavior):
```json
{
  "key": "new-feature",
  "value": true,
  "reason": "TARGETING_MATCH",
  "variant": "treatment"
}
```

## Consequences

### Positive

- **Eliminates default value duplication**: Developers maintain default values in one place (code) rather than synchronizing between code and flag management system
- **Supports standard feature flag lifecycle**: Natural progression from code defaults to managed values
- **Accurate telemetry**: Deferred evaluations are correctly recorded as successful rather than errors
- **Flexibility for flag management systems**: Systems can choose when to provide explicit values vs. deferring to code
- **Backward compatible**: Existing implementations continue to work without modification

### Negative

- **Breaking change for providers**: While the API remains backward compatible, all provider implementations must be updated to handle the new `codeDefaultFlag` type. This change should be implemented before the 1.0 release.
- **Delayed type mismatch detection**: When deferring to code defaults, type mismatches between the flag configuration and code won't be detected until the flag management system returns an explicit value. This could lead to runtime errors that would have been caught earlier with mandatory values.
- **Provider complexity**: Providers must carefully handle the new flag type case and ensure code defaults are always available when needed.

### Implementation Requirements

1. **Provider updates**: All OFREP providers must be updated to:
   - Recognize the `codeDefaultFlag` type in responses
   - Use code default values when this type is received
   - Maintain existing behavior for other flag types

2. **Type validation**: Providers should validate that code default types match expected flag types when possible, though this validation may be limited when values are omitted.

3. **Error handling**: Providers must ensure that appropriate errors are returned if a code default is not available when the `codeDefaultFlag` type is used.

4. **Bulk evaluations**: The same logic applies - individual flags within a bulk response may use the `codeDefaultFlag` type independently.
