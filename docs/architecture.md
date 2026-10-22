# Architecture


<!-- source-publication-version -->
> Companion source is published at [df93c3c](https://github.com/Loz888/durable-coding-agent-runtime-source/commit/df93c3c13b130b3d08a635a325cb592f4908fd52). [Evidence and limits](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/df93c3c13b130b3d08a635a325cb592f4908fd52/reports/runtime-followup-20261022/README.md) · [GitHub CI](https://github.com/Loz888/durable-coding-agent-runtime-source/actions). “Uncommitted/not pushed” wording below records acceptance-time snapshots rather than current publication status.


<!-- runtime-followup:start -->
## 2026-10-22 follow-up

The shared metadata-only internal/goinput planner connects repair17/executor; Worker sandbox v4 binds input mode. Four real sealed-input Docker scenarios passed. Artifact inventory is read-only, not a deletion protocol. See companion source local `reports/runtime-followup-20261022`; not pushed, no software CI in this showcase.
<!-- runtime-followup:end -->

## Local upgrade · 2026-10-22 (unpublished)

This section describes the current working tree; older dated “pending/BLOCKED” statements are historical. No commit, push or remote-CI result is implied.

Implemented bounded multi-file patches (up to 8 allowlisted existing Go files per repair round), persisted logical model responses with conservative unknown-outcome handling, real Docker confinement and an independent scoped orphan reaper, OTLP/metrics/local alert acceptance, paced/real-Go-test benchmark workloads, and content-addressed portable Worktree recovery. Two isolated host containers plus a separate authenticated artifact service passed actual A-container SIGKILL, natural lease expiration and a real Epoch2 ClaimBatch takeover. This remains one physical computer, with no physical-host/power-loss claim.

The frozen 12-task synthetic live evaluation made **11 actual requests**, **3382 input / 4201 output tokens**: 9 fresh repairs and one response-recovery control passed holdout assertions; an immutable failure control failed as expected; an already-passing baseline made zero requests. Mode: High, requested/returned alias `deepseek-flash`. The peak cache-miss estimate is **$0.0060558**, not an invoice, against this run's separately authorized $2 cap. These simple manually constructed tasks do not measure general repository repair accuracy.

Explicit sealed input policies capture offline vendor inputs or root-contained workspace modules, with shared fixed-command planning for trusted-local and sandbox execution. Terminal Worktrees can be inspected/dry-run and retired with identity/state checks and a persisted deletion intent. A separate fresh local PostgreSQL cluster restored synthetic data and read-only role permissions.

Portable recovery requires a previously published artifact and durable patch history. Disk loss before the first publication and unobserved model responses are not automatically recovered.

Remaining unverified: a second physical host, network partition/power loss, arbitrary hostile repositories, representative real-world defects and human review, production soak/capacity, and the cause of two historical negative DB wall-clock durations. Monotonic benchmark timing does not remove the Lease clock assumption.

```mermaid
flowchart TD
    Submit[Submit / admission] --> Queue[(PostgreSQL Queue + Checkpoint)]
    Queue --> Worker[Existing Scheduler / Worker]
    Worker --> Lease[Lease + Epoch + Revision CAS]
    Worker --> Response[(Intent / observed model response)]
    Response --> Budget[(Persistent physical-call budget)]
    Response --> Patch[(One bounded multi-file Patch Intent)]
    Patch --> Worktree[Managed identity / per-file guarded Rename]
    Worktree --> Verify[Whole-input snapshot / tests]
    Verify --> Patch
    Verify --> Queue
    Worktree --> Artifact[(Private content-addressed artifact)]
    Artifact --> Rebuild[Fresh claim: local clone + intent replay + new identity + retest]
    Rebuild --> Worktree
    Verify --> Sandbox[Optional Docker + independent scoped reaper]
    Worker --> OTLP[Optional OTLP Collector]
    OTLP --> Backends[Jaeger / Prometheus / local controlled alert]
    Rebuild -. unverified physical fault domain .-> Physical[Second physical machine / power loss]
```

---


> This describes the companion source implementation. Run commands in the source checkout; this showcase contains documentation and runs no software CI. This alignment draft is local and has not been pushed.

[English](architecture.md) | [简体中文](zh-CN/architecture.md) | [Home](../README.md)

## Responsibilities and dependencies
runtimecore runs task.Task through executor.Registry with concurrent Workers
and result collection. persistence/httpapi provide a persistent task API.
This in-memory task runtime is distinct from durable Agent Run scheduling.

durable owns Queue, LeaseManager, FencedStore and Scheduler. Diagnosis providers
use durable.Runner plus repodiag. coding:* providers use CodingHandler and
repair20.Agent. Both reuse the same claims, heartbeat and Scheduler lifecycle;
the diagnosis model factory is not invoked for coding proposals.

Production dependencies: durable → repair20/repair19;
repair20 → repair19/repair18/repair17; repair19 → repair17.
internal/integration20 is test-only and references those packages one-way.
No production module imports it or needs a private-test export bridge.

## Schemas
Apply migrations in order: 001 creates day12.agent_runs/checkpoints; 002 adds
Lease owner/Epoch/expiry; 003 adds queue/priority/availability/retry_count;
004 adds budgets/failures/times/run_events; 005 creates day19 repair_runs and
patch_ops; 006 adds Worktree Identity and causal round_number; 007 adds scoped pilot identities and execution profiles; 008 adds shared model admission/accounting; 009 adds outcome_verdict and diagnostic_report. In the current local source,
010 adds patch verification_input_digest; 011 adds Run completion_pending.
Apply through 011 after stopping old Workers and before starting the new version.

HTTP persistence uses its own tasks schema through EnsureTaskSchema; tests
apply schema.sql in random runtime_test_* schemas.
Production events are unique by (run_id, revision, event_type); a legacy Day15
fixture has another shape, detected by Store. Equal-revision same-type
observations may merge; this is not one guaranteed event per attempt.

## Execution status and diagnostic evidence

Terminal checkpoint writes freeze finished_at in the protected transaction. Duration summaries separate completed, still-running and unknown historical timestamps. Diagnostic succeeded means execution completed; outcome_verdict separately records evidence_consistent, insufficient_evidence or inconsistent_evidence. Structured reports check current tool-call references, persisted-output hashes and actual test execution metadata. This does not prove narrative root causes. Coding uses tests_passed/tests_failed. Both provider adapters share internal/providererr; details and tests are in the [review implementation](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/docs/review-20261020/IMPLEMENTATION.md).

## Worktree lifecycle
repair17.Prepare creates a detached Git Worktree with Run/baseline metadata
and a random identity token. repair19.Register binds its absolute path, source
commit and identity to the logical Run. Recovery opens and validates that
same binding. Missing, symlinked, replaced or rebased workspaces fail closed.
Retain an unfinished Run's workspace for recovery; integration fixtures own
their temporary sources and cleanup.
The current local Worker normalizes its relative Worktree base at startup. Restart
with the same explicit absolute -worktree-base / WORKTREE_BASE, toolchain and allowlist
configuration; changing the launch directory does not discover the previous Worktrees.

## Persistence boundary
Short operations lock agent_runs, then repair_runs/patch_ops. Claim and Renew
contend on the same Run row. Guarded Rename is followed by authority/hash
validation before result commit. Model calls and Go tests run outside those
row-lock transactions. This is a trusted single-writer/shared-local-filesystem
model, not a sandbox or filesystem/database distributed transaction.

## Evaluation boundary

eval21.Score aggregates validated Observation records; cmd/day21-eval parses
manifests/JSONL and writes reports. Neither adds another Scheduler or Repair Loop.
TestDay21Capture drives the existing Scheduler with deterministic proposals and
collects independent baseline, Worktree-test, source-cleanliness and checkpoint evidence.
See [evaluation](evaluation.md).

## Local engineering acceptance · 2026-10-21 (unpublished)

The companion source implements bounded complete candidate snapshots, a versioned `verification_input_digest` and evidence-only completion after the last execution Claim, limited to standalone stdlib-only Go modules. Historical verdicts without a digest cannot be reused directly. Completion checks identity, complete inputs, state and Checkpoint CAS without another execution Claim/Epoch, repair round or model call; cancellation/Sweep competition and rollback have dedicated regression cases.

The final source full and race suites each passed **301 test/subtest events, zero failures/skips** after the Worker startup adjustment. The core five-package ten-repeat check passed **1,360 events, zero failures/skips** (`-p 1 -count=10 -timeout 1800s`; repair20, durable, repair19, internal/integration20 and reliability15). Exact commands and final source fingerprints are retained in the companion local `docs/review-20261020/RESULTS.md` and `reports/review-20261020/engineering-20261021.json`. Stop old Workers, apply migrations **001–011**, then start the new Worker; mixed-version rolling upgrades are not claimed.

These are local source results, not software CI in this showcase or remote CI. Container adapter code exists, but actual engine isolation remains unverified. Historical live, Day21 Mock and older regression records retain their own dates, versions and denominators. New companion files remain unpublished, so this page does not link to nonexistent GitHub paths.

## Optional tracing / 可选追踪 · 2026-10-21（本地未发布）

Submit/API persists W3C trace context in the Run transaction. Workers restore it to correlate Patch/Checkpoint operations, with the real SDK exporting per-process private NDJSON files. A separate worker-process regression checks TraceID and ParentSpanID; it does not establish multi-host Worktree recovery. Export is opt-in metadata only, excluding prompts, source, connection strings and raw errors. Apply migrations 001–012 first. OTLP, metrics and alerts are not configured; SIGKILL may lose buffered spans, and trace context grants no Lease/Epoch authority. New source full/race runs pass 309 events each; the earlier 301 count belongs to the previous completion snapshot.
