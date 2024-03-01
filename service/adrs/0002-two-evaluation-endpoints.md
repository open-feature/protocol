# 2. Two evaluation endpoints

Date: 2024-03-01

## Status

Accepted

## Context

Evaluating flags in the static context paradigm, is often done by bulk evaluating flags and updating them if
needed.
E.g. a [frontend would most times load all flags](https://openfeature.dev/blog/catering-to-the-client-side) and cache
them to prevent loading times and many requests to the feature
flagging service.

In the dynamic context paradigm, flags are most times evaluated on demand.
E.g. backend service often evaluates flags based on information from the specific request, so bulk evaluation is not
possible.

## Decision

OFREP contains two endpoints, one for individual flag evaluations and the other for bulk evaluations.
Individual flag evaluation is preferred for dynamic context paradigm whereas bulk evaluation is preferred for static
context paradigm.

## Consequences

- OFREP supports single and bulk evaluations
- Each endpoint might evolve to better serve the specific OpenFeature paradigm
- It might become challenging to prevent the endpoint from diverging
