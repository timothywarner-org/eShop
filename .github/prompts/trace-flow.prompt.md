---
agent: agent
description: Trace a request across service boundaries and report every
  place correctness depends on execution order.
---

Trace the flow I name below from entry point to final side effect.
Use file inspection by default. Do not edit files or execute the application
unless explicitly requested. If `docs/repo-map.md` exists, read its relevant
entries for orientation, then verify the path against source. Correct stale
map claims in your report; do not treat the map as proof.

Produce exactly these sections:

## Path
Each hop in order: service, file, method. One line per hop.

## Boundaries
Every service-to-service or process-to-process transition, and what
carries the data across it.

## Order dependencies
Every point where correctness depends on the sequence of side effects,
on a database-generated value being populated, or on a transaction
boundary. Explain the failure mechanism if the order changes, and label an
untested consequence as predicted rather than reproduced.

## Existing warnings
Quote verbatim every `REVIEW`, `TODO`, `HACK`, or `FIXME` comment in
any file on this path. If there are none, say so explicitly.

## What I could not determine
Name what you looked for and did not find. Do not guess.

## What to preserve
Propose at most two useful findings. For each, give:
- Verified fact and supporting file/method.
- Reasoning rule: what to check in similar work, and why it matters.
- Scope and destination: where the rule applies and which existing map,
  instruction, or prompt should hold it.
Keep hypotheses out of permanent guidance. Propose these entries for review;
do not save them unless requested.

Flow to trace:
