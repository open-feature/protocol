# Creating an OFREP client provider

OpenFeature Remote Evaluation Protocol (OFREP) is an API specification for feature flagging that allows the use of generic providers to connect to any feature flag management systems that supports the protocol.

In this specification we will specify how to write an OFREP provider using the [static-context-paradigm](https://openfeature.dev/specification/glossary/#static-context-paradigm) that is used on client side applications typically operate in the context of a single user. 
We will keep the specification language agnostic.

**Pre-requisite:**
- Understanding of [general provider concepts](https://openfeature.dev/docs/reference/concepts/provider/)
- Understanding of the [OFREP](../../README.md)
- Understanding of the [OFREP OpenAPI specification](../../service/openapi.yaml)


## Constructor
An implementation of an OFREP client provider should allow in the creation of the provider to take as options:
- `baseURL`: The base URL of the [flag management system](https://openfeature.dev/specification/glossary#flag-management-system). This should be the base of the URL pointing before the `/ofrep` namespace of the API.
- `headers`: The headers to use when calling the OFREP endpoints *(ex:`Authorization`, Custom headers, etc ...)*.
- `pollInteral`: The polling interval defining when to call again the flag management system.

In the constructor, the provider should check if the `baseURL` is a valid URL and return an error if the URL is invalid.

## Initialize the provider
An implementation of an OFREP client provider should start with an initialization of the provider.

The `initialize()` function should follow those steps:
1. Make a GET request to the `/ofrep/v1/configuration` endpoint to retrieve the configurations return by the flag management system and store them in memory to be available for all the function of the provider. *(See [Annexe 1](#annexe-1) for the description of the endpoint response)*
2. Make a POST request to the `/ofrep/v1/evaluate/flags` endpoint with the evaluation context in the body.  
   - If the endpoint returns an error, the `initialize()` function must error and exit.  
   - If the request is successful, we should store in a local cache all of the flags evaluation results returned by the API in a local cache. We should also store the `ETag` header in the provider to be able to send it back later.
3. If polling is enabled, the function should start the polling loop *(See [polling section](#polling))*.

## Evaluation
The evaluation should not perform any remote API calls.

When calling an evaluation function the provider should check if the associated type is supported, by checking the key `capabilities.flagEvaluation.unsupportedTypes` from the configuration endpoint.
- If the type is unsupported, we should exit early and directly return an error.
- If the type is supported, we should retrieve the flag from the local cache of the flags evaluation.
  - If you are not able to find the flag, you should return an FlagNotFound error.
  - If the remote evaluation for this flag has return an error, you should map the error in provider and return the associated error.
  - If the value retrieve from the cache has a different type than the one expected you should return a TypeMismatch error.
  - If the cached evaluation is in success you should return the evaluation response.

## Polling
The polling system will make a POST request periodically tp the `/ofrep/v1/evaluate/flags` endpoint to check if there is a change in the flags evaluation to be able to store it.

If we have a `ETag` available we should always add the header `If-None-Match` with the `ETag` value.
- If the cache is still up-to-date we will receive a `304` telling us that the nothing has changed on the flag management system side.
- If the cache is outdated we will receive a `200` with the new values of all the flags. In that situation we should:
  1. Replace the actual local cache of flags evaluations.
  2. Store the new `ETag` value for the future call.


## Annexe 1
**`/ofrep/v1/configuration` description:**
The endpoint will return a list of configurations for the provider:
- `name`: Name of the flag management system, it should be used in the `metadata.name` name (ex: `OFREP Web Provider ${metadata.name}`).
- `capabilities`: List of capabilities of the flag management system and their associated configuration. *Check the [capabilities section](#capabilities) for the details of each capability.*

### Capabilities
In this section we will describe each capabilities and describe their default value and usage.

#### Cache Invalidation
The capability `cacheInvalidation` describe how the mechanism of cache invalidation is working.

##### Polling
`polling`: define how the provider should do the polling

- `enabled`: if `true` the provider should poll the `/ofrep/v1/evaluate/flags` regularly to check if any flag evaluation has changed in the flag management system.
- `minPollingInterval`: define the minimum polling interval acceptable by the flag management system. If for any reason the `pollInteral` provided in the constructor is lower than this `minPollingInterval` we should default on this value.

If the key `polling` is not available, the provider should use those default values:
```json
{
    "enabled": true,
    "minPollingInterval": 0 
}
```

#### Flag Evaluation
`flagEvaluation`: define the how to managed flag evaluation in the provider.
- `unsupportedTypes`: Some flag management system are not supporting all types. This array should contains all the types not supported by the flag management system.Acceptable values are `int`, `float`,`string`, `boolean`,`object`.  
  Default value: `[]`