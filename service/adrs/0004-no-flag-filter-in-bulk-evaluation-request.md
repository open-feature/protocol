# 4. No flag filter in bulk evaluation request

Date: 2024-03-01

## Status

Accepted

## Context

There can be cases where bulk evaluation evaluates and return hundreds or thousands of flags.
Some feature flagging systems define filters that specify which flags should be bulk evaluated.

## Decision

OFREP does not specify a `flags` property that specifies the flags to be bulk evaluated.
For systems that rely on a list of flags, there is an alternative to include them through the `context` object.

## Consequences

- There is no standardized way of filtering flags
    - Flags can still be filtered via the evaluation context
- OFREP stays simple and the evaluation only depends on context and authorization 
