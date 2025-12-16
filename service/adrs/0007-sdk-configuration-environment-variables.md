# 7. SDK Configuration Pattern using Environment Variables

Date: 2025-10-17

## Status

Accepted

## Context

OpenFeature Protocol (OFREP) SDKs currently require configuration through code-based initialization, which can be inflexible in containerized and cloud-native environments. This approach requires rebuilding applications or modifying code to change configuration values such as endpoints, authentication headers, and timeout values.

OpenTelemetry has established a successful pattern for SDK configuration using well-defined environment variables (<https://opentelemetry.io/docs/concepts/sdk-configuration/>), which provides several benefits:

- Configuration externalization without code changes
- Better support for containerized deployments
- Consistency across different language SDKs
- Easier configuration management in CI/CD pipelines
- Separation of concerns between application logic and infrastructure configuration

Currently, OFREP consumers handle the configuration programmatically, leading to:

- Inconsistent configuration patterns across different SDK implementations
- Difficulty in managing configuration across multiple environments
- Need for custom configuration management solutions
- Tight coupling between application code and deployment configuration

The proposed standardization would align with industry best practices and improve the developer experience for OFREP adoption.

## Decision

Adopt a standardized environment variable configuration pattern for OFREP SDKs, following OpenTelemetry's approach. This will establish well-defined environment variable names that all OFREP SDK implementations should support.

Define the following standard environment variables:

- **OFREP_ENDPOINT**: The base URL for the OFREP service (e.g., `http://localhost:2321`)
- **OFREP_HEADERS**: Additional HTTP headers in comma-separated key=value format (e.g., "Authorization=Bearer 123,Content-Type=application/json")  
- **OFREP_TIMEOUT_MS**: Request timeout in milliseconds (e.g., "5000")

SDKs should:

1. Check for these environment variables during initialization
2. Use environment variable values as defaults when no programmatic configuration is provided
3. Allow programmatic configuration to override environment variables
4. Provide clear documentation on supported environment variables
5. Validate environment variable values and provide meaningful error messages

## Consequences

### Positive

- **Standardized Configuration**: Consistent configuration approach across all OFREP SDK implementations reduces learning curve for developers
- **Cloud-Native Compatibility**: Better support for containerized deployments and other cloud-native environments where environment variables are the preferred configuration method
- **Deployment Flexibility**: Applications can be configured without code changes, enabling the same artifact to be deployed across different environments
- **Industry Alignment**: Follows established patterns from OpenTelemetry, making adoption easier for teams already familiar with these patterns
- **Configuration Externalization**: Separates application logic from infrastructure configuration, improving maintainability

### Negative

- **Breaking Change Risk**: Existing SDK implementations may need updates to support environment variable configuration, potentially requiring migration effort
- **Configuration Complexity**: Multiple configuration sources (environment variables, programmatic configuration) may lead to confusion about precedence and debugging challenges  
- **Security Considerations**: Environment variables containing sensitive information (headers with authentication tokens) may be visible in process lists or container configurations
- **Platform Limitations**: Some deployment platforms may have restrictions on environment variable names or values that could impact adoption
- **Multi-Provider Limitation**: In scenarios where multiple OFREP providers need to be configured simultaneously (e.g., different providers for different feature flag domains), environment variables cannot easily specify provider-specific configuration without introducing complex naming conventions or requiring programmatic configuration

## Alternatives Considered

### Configuration Files (JSON/YAML)

- **Description**: Use external configuration files (e.g., ofrep.config.json) that SDKs would read during initialization
- **Rejection Reason**: Configuration files add complexity in containerized environments, require file mounting, and are less standard than environment variables in cloud-native deployments

### Configuration Management Integration

- **Description**: Direct integration with configuration management systems like HashiCorp Vault, AWS Parameter Store, or Azure Key Vault
- **Rejection Reason**: Would require SDK implementations to include vendor-specific dependencies and authentication logic, increasing complexity and reducing portability

## Implementation Notes

- **Precedence Order**: Programmatic configuration should take precedence over environment variables, allowing developers to override environment-based defaults when needed
- **Validation Strategy**: SDKs should validate environment variable formats during initialization and provide clear error messages for invalid values (e.g., invalid URLs, malformed headers)  
- **Documentation Requirements**: All SDK implementations should document supported environment variables, their expected formats, and provide examples for common deployment scenarios
- **Migration Support**: Existing applications should continue to work without changes, with environment variable support being additive functionality
