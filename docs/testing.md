# Testing and reproduction

> This describes the companion source implementation. Run commands in the source checkout; this showcase contains documentation and runs no software CI. This alignment draft is local and has not been pushed.

[English](testing.md) | [简体中文](zh-CN/testing.md) | [Home](../README.md)

<!-- source-publication-version -->
> Companion source is published at [df93c3c](https://github.com/Loz888/durable-coding-agent-runtime-source/commit/df93c3c13b130b3d08a635a325cb592f4908fd52). [Evidence and limits](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/df93c3c13b130b3d08a635a325cb592f4908fd52/reports/runtime-followup-20261022/README.md) · [GitHub CI](https://github.com/Loz888/durable-coding-agent-runtime-source/actions). “Uncommitted/not pushed” wording below records acceptance-time snapshots rather than current publication status.


Use Linux/WSL2, the Go version in go.mod, Git, Bash, Python3 and PostgreSQL16.
README Quick Start creates agent_runtime_day14_test and agent_runtime_test.
The role must own test databases or be able to create/drop schemas.

Export DAY12_TEST_DATABASE_URL and TEST_DATABASE_URL. Never use production
DATABASE_URL for tests. The current local setup_test_db.sh checks exact names and applies 001–011
to the queue test database, and leaves task schema creation to isolated
persistence tests. TCP DSNs can use TEST_USER:TEST_PASSWORD at localhost with
the same database names. CI's ci-only-password is a disposable service-container
credential, not a deployment or user secret.

## Suites
- test_unit.sh runs complete PostgreSQL-independent package suites: cmd/day21-eval, eval21, distributed,
  eval16, executor, internal/testdb, internal/providererr, internal/boundedlog, observability, repair17/18/20, repodiag and
  runtimecore. Mixed packages run fully in the integration job.
- test.sh ./... runs every root-module test with real PostgreSQL.
- test.sh -race ./... runs the same suite under the race detector.
- go test -p 1 -count=10 -timeout 1800s ./repair20 ./durable ./repair19 ./internal/integration20 ./reliability15
  repeats the five core/integration packages; the final recorded run uses this scope.
- go test -count=1 -timeout 300s ./durable ./repair19 ./internal/integration20
  exercises combined authorization, persistence and recovery.

Fixtures own rows and random Patch schemas. Independent fixtures sharing the
fixed day12 queue serialize via an advisory lock; within-fixture owner/renewal
and Patch-lock competition stays real. Child helper tests return normally
unless started by the fault parent; parents execute them in actual subprocesses.
This is not SKIP.

## No-SKIP acceptance
test.sh records Go JSON events and check_test_results.py rejects failed/skipped
tests, package failure, zero test results or absent required demo scenarios.
Some original optional DB helpers still SKIP without URLs; that is not release
acceptance. CI and release tests provide both connections and reject every SKIP.

## Models and intentional failures
Mock uses ScriptedFix for the controlled Divide defect; HTTP model tests use
local httptest. These are not a real DeepSeek evaluation. Real-model runs are
opt-in and never CI or these demos.

Four nested evaldata/fixtures modules intentionally fail tests as oracle inputs;
do not delete them or fix their defects to make an evaluation appear green.
They are excluded from root-module traversal by their own go.mod files.

## Cold compiler caches in race CI

The race job first compiles normal test dependencies with go test -run '^$'.
Go's race-instrumented standard library cache differs from the cache used by
the nested ordinary go test commands in trusted fixtures. Warming the latter
prevents a fresh runner's compiler work from consuming a fixture's strict
execution timeout. This preparation executes no acceptance cases; the full
race suite still executes every test, checks JSON results and rejects SKIP.
Runtime deadlines, Lease guards and all assertions remain unchanged.

## Day 21 and output isolation

day21_accept.sh captures exactly the 12 required TestDay21Capture subtests,
recomputes committed summary/report files and scores a fresh dataset.
The full suite, Mock/crash demos and unit runner unset DAY21_RUNS_OUT to avoid
appending duplicate observations to another capture. The accept script passes
that variable only to the one capture command.

Ordinary/race checks use a 360-second timeout. The five-package ten-repeat
check below used 1,800 seconds and -p 1; earlier three-package runs used 900 seconds.
Timeouts do not remove any test or assertion; no fresh coverage result is claimed.
See [evaluation](evaluation.md) for report reproduction and fresh capture.

## Pilot integration database roles

The new pilot API fixtures provision a separate restricted login with a random fixture-only password, including TCP/SCRAM CI. Their setup connection requires authority to create/manage test roles and grant access inside the two dedicated test databases. Run them in an isolated test PostgreSQL cluster or the provided CI service. Do not promote the production API role to run tests. The API itself refuses superuser/owner credentials.

## Local engineering acceptance · 2026-10-21 (unpublished)

The companion source implements bounded complete candidate snapshots, a versioned `verification_input_digest` and evidence-only completion after the last execution Claim, limited to standalone stdlib-only Go modules. Historical verdicts without a digest cannot be reused directly. Completion checks identity, complete inputs, state and Checkpoint CAS without another execution Claim/Epoch, repair round or model call; cancellation/Sweep competition and rollback have dedicated regression cases.

The final source full and race suites each passed **301 test/subtest events, zero failures/skips** after the Worker startup adjustment. The core five-package ten-repeat check passed **1,360 events, zero failures/skips** (`-p 1 -count=10 -timeout 1800s`; repair20, durable, repair19, internal/integration20 and reliability15). Exact commands and final source fingerprints are retained in the companion local `docs/review-20261020/RESULTS.md` and `reports/review-20261020/engineering-20261021.json`. Stop old Workers, apply migrations **001–011**, then start the new Worker; mixed-version rolling upgrades are not claimed.

These are local source results, not software CI in this showcase or remote CI. Container adapter code exists, but actual engine isolation remains unverified. Historical live, Day21 Mock and older regression records retain their own dates, versions and denominators. New companion files remain unpublished, so this page does not link to nonexistent GitHub paths.
