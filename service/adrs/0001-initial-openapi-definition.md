# 1. initial openapi definition

Date: 2024-02-29

## Status

Accepted

## Context

Technical decisions for the initial version of the OpenAPI definition of OFREP

## Decision

- JSON over HTTP for the protocol

Technology selection is based on the survey feedback from vendors. Most of the vendors were using JSON & HTTP in their current implementations.
Hence, the adoption could be easy for them.

- Two evaluation endpoints

Initial specification contains two endpoints, one for individual flag evaluations and the other for bulk evaluations.
Individual flag evaluation is preferred for dynamic context paradigm whereas bulk evaluation is preferred for static context paradigm.

- No type details in request or response

There were many discussions about this point. 
However, in the community meeting on 2024-02-29, it was concluded **not** to include type details in any payload.
The reason for this is to keep the API simple.

- Not include flags field in bulk evaluation request

Initially, it was considered to add `flags` field containing a list of flags for bulk evaluation requests, later it was decided to remove this.
For systems that rely on a list of flags, there is an alternative to include them through the `context` object.

- Polling for bulk evaluation changes

Decided to use polling for bulk evaluation changes. This is to simplify the initial version of the protocol.
There were suggestions on introducing a dedicated endpoint for change detection, but this was not well received. 

- ETag for bulk evaluation changes

Optional ETag can be used with the bulk evaluation endpoint. 
The motivation here is to save wire bandwidth and evaluation burden on the flag management system. 
If present in response, implementation will attach ETag when polling and respect 304 responses from the flag management system.


## Consequences

- Type inference at API client

Given that type is not included in either request or response, client implementation (the provider) will need to perform a type cast to match with OpenFeature evaluation types.

- Caching for single flag evaluations

Individual flag evaluations may be cached to increase performance. 
But this is out of scope of the OpenAPI specification and is an implementation detail of the provider.
