# Testing strategy

This document describes tests in the private implementation.
The public documentation repository contains no test suite, runnable demo,
application source or database migrations. The following sections explain
the engineering acceptance criteria rather than provide shell instructions.

## Environment and isolation

Recorded implementation runs used Linux/WSL, Go 1.27.1 and PostgreSQL 16.
Database scenarios used two dedicated test databases. Fixtures owned their
rows and random Patch schemas; production task databases were not used.

Independent fixtures sharing the queue serialized setup through a PostgreSQL
advisory lock. Within-fixture owner renewal and Patch row-lock contention
remained concurrent.

## Required scenarios

| Area | Acceptance condition |
| --- | --- |
| Real claims | Takeover acquires work through the actual Queue claim operation |
| Four Patch states | SIGKILL recovery covers prepared, applied, verified_pass and verified_fail |
| Final checkpoint gap | A verified Patch survives a crash before the Agent checkpoint |
| Worker fencing | An old Epoch cannot commit a new owner's result |
| Worktree identity | Missing, replaced, rebased or incorrectly bound workspaces fail closed |
| Repair budget | Takeover preserves the logical round budget |
| Heartbeat contention | Renewal and Patch row-lock competition exercise real concurrency |
| Source isolation | Repairs modify only the managed Worktree |
| Cached verification | Persisted verification can be reused without another model call |
| Terminal state | Completed Runs cannot be claimed again |

Only child processes created by the test harness were killed. The harness
advanced only its own Run's Lease expiry to bound waiting time; the next Epoch
still came from an actual Queue claim.

## Acceptance rules

Implementation acceptance rejected failed tests, skipped tests, missing
required scenarios and empty results. Database connection configuration was
provided explicitly. Package entries with no test files were distinguished
from skipped test cases.

The full project ran normally and with the Go race detector. A separate
database-independent suite, repeated repair tests, static analysis and builds
were also checked. See [recorded results](validation.md).

## Models and fixtures

Mock repair used a deterministic proposal for a controlled Divide defect.
HTTP model-adapter tests used local HTTP test servers. These observations
are not a live-model quality evaluation; no real model API key was injected.

Four intentionally failing nested Go modules were diagnosis inputs; a fifth
was a passing control. Their intentional failures were not deleted to create
a misleading successful result.

## Compiler-cache handling

The private race CI compiled normal test dependencies before the full race
suite. Nested ordinary Go test processes use a different standard-library
cache from race-instrumented processes. Cache preparation kept cold compiler
work from consuming strict fixture execution deadlines.

This preparation replaced no acceptance scenario. Runtime deadlines,
Lease guards and all assertions remained unchanged.

## Public verification boundary

Readers can inspect the design documents and their consistency.
The implementation source, private CI logs and executable fixtures are not
available from this repository, so the reported test outcomes are author-reported
engineering evidence rather than independently reproducible public results.
