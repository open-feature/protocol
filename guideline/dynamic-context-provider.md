# Creating an OFREP server provider

OpenFeature Remote Evaluation Protocol (OFREP) is an API specification for feature flagging that allows the use of generic providers to connect to any feature flag management systems that supports the protocol.

In this document, we will specify how to write an OFREP provider using the [dynamic-context-paradigm](https://openfeature.dev/specification/glossary/#dynamic-context-paradigm) that is used on server side applications typically operate in the context of multiple users. 

**Pre-requisite:**
- Understanding of [general provider concepts](https://openfeature.dev/docs/reference/concepts/provider/)
- Understanding of the [OFREP](../../README.md)
- Understanding of the [OFREP OpenAPI specification](../../service/openapi.yaml)

## Configuration
An OFREP server provider implementation must, at the time of creation, accept at least these options
- `baseURL`: The base URL of the [flag management system](https://openfeature.dev/specification/glossary#flag-management-system).  
  This must be the base of the URL pointing before the `/ofrep` namespace of the API.
  - In the constructor, the provider should check if the `baseURL` is a valid URL and return an error if the URL is invalid.
- `headers`: The headers to use when calling the OFREP endpoints *(e.g.:`Authorization`, Custom headers, etc ...)*.
- `timeout`: This specifies the duration to wait for a response before canceling the HTTP request. If no value is provided, a default timeout of `10 seconds` must be applied.

## Evaluation
When an evaluation function is called the server provider will make a `POST` request to the `/ofrep/v1/evaluate/flags/{key}` endpoint *(where `{key}` is the flag name), with the evaluation context in the body.

**Request body example**:
```json
{
    "context": {
    "targetingKey": "f021d0f9-33b7-4b22-b0bd-9fec66ba1d7d",
    "firstname": "foo",
    "lastname": "bar",
    "email": "foo.bar@ofrep.dev"
    }
}
```

When calling the API the provider can receive those response codes:
- `400`: Bad evaluation request, this means that the flag management system has returned an error during the evaluation. In that situation the provider should map the error returned to an OpenFeature Error and return it.
- `401`, `403`: The provider is not authorized to call the OFREP API. In that situation the provider should return an error.
- `404`: Flag not found, this means that the flag management system does not have a flag for this flag key. In that situation the provider should return a `FLAG_NOT_FOUND` error.
- `429`: The provider has reached the rate limit of the flag management system. In that situation the provider should read the `Retry-After` header from the response and ensure that no call to the endpoint happened before this date. Every evaluation happening before the retry after date should error and return the default value.
- `200`: The flag management system has returned a valid evaluation details for this flag. In that situation the provider must map the API response into an evaluation details response, and return it.
