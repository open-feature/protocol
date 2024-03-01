# 2. Two evaluation endpoints

Date: 2024-03-01

## Status

Accepted

## Context

Evaluating flags in the static context paradigm, is often done by bulk evaluating flags and updating them if needed.
For example, [frontend would most times load all flags](https://openfeature.dev/blog/catering-to-the-client-side). 
And the payload is cached to prevent loading times and to avoid overloading the feature flagging service.

In the dynamic context paradigm, flags are usually evaluated on demand.
For example, backend service often evaluates flags based on the information present in the specific request.
This makes bulk evaluation less favourable for dynamic context paradigm. 

## Decision

OFREP contains two endpoints, one for individual flag evaluations and the other for bulk evaluations.
Individual flag evaluation is preferred for the dynamic context paradigm whereas the bulk evaluation is preferred for static context paradigm.

## Consequences

- OFREP supports single and bulk evaluations
- Each endpoint might evolve to better serve the specific OpenFeature paradigm
- It might become challenging to prevent the endpoint from diverging
