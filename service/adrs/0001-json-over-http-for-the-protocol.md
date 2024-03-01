# 1. JSON over HTTP for the protocol

Date: 2024-02-29

## Status

Accepted

## Context

OFREP needs a network protocol and serialization format for the feature flag evaluation information.
The protocol and serialization format must be supported in web, mobile and server environments.

## Decision

Technology selection was based on the [survey feedback from vendors](https://docs.google.com/forms/d/1NzqKx57XvRK_2lRQOFCRmF5exet6f15-sCjdEy0HCS8#responses).
The majority of the vendors were using JSON & HTTP in their current implementations.
Based on this, it was decided to use JSON over HTTP for OFREP.

## Consequences

- Web, mobile and server environments have great support for JSON over HTTP
- Many developers are familiar with this technology which may reduce the implementation burden
- Most vendors are already using this technology, hence it will be easier for them to adopt OFREP
