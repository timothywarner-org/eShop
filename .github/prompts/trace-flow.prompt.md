---
agent: agent
description: Trace a request across service boundaries and report every
  place correctness depends on execution order.
---

Trace the flow I name below from entry point to final side effect.

Produce exactly these sections:

## Path
Each hop in order: service, file, method. One line per hop.

## Boundaries
Every service-to-service or process-to-process transition, and what
carries the data across it.

## Order dependencies
Every point where correctness depends on the sequence of side effects,
on a database-generated value being populated, or on a transaction
boundary. For each one, state what breaks if the order changes.

## Existing warnings
Quote verbatim every `REVIEW`, `TODO`, `HACK`, or `FIXME` comment in
any file on this path. If there are none, say so explicitly.

## What I could not determine
Name what you looked for and did not find. Do not guess.

Flow to trace:
