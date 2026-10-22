# Crash recovery

[English](crash-recovery.md) | [简体中文](zh-CN/crash-recovery.md) | [Home](../README.md)

<!-- source-publication-version -->
> Companion source is published at [df93c3c](https://github.com/Loz888/durable-coding-agent-runtime-source/commit/df93c3c13b130b3d08a635a325cb592f4908fd52). [Evidence and limits](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/df93c3c13b130b3d08a635a325cb592f4908fd52/reports/runtime-followup-20261022/README.md) · [GitHub CI](https://github.com/Loz888/durable-coding-agent-runtime-source/actions). “Uncommitted/not pushed” wording below records acceptance-time snapshots rather than current publication status.


This page describes the implementation. Run reproduction commands in the [public source repository](https://github.com/Loz888/durable-coding-agent-runtime-source); this documentation repository contains no source and does not execute software CI.

SIGKILL prevents cleanup, release and final checkpoint writes. PostgreSQL may
still show running while the file already contains a Patch.

A replacement Worker that continues execution must obtain a real new Claim after expiry. Epoch advances
only through Claim; old-owner writes are rejected even if that process resumes.
Revision CAS independently prevents overwriting one checkpoint version twice.

repair20.Agent first opens its bound Worktree and scans recoverable Patch
history ordered by causal round_number, not wall-clock created_at. Prepared
state is reconciled with file SHA; applied is verified; verified states reuse
persisted test evidence only when the verification-input digest matches in the
current local source. See [Patch protocol](patch-protocol.md). Only failed verification plus remaining shared budget
allows another proposal.

A verified_pass Patch may survive without the final Agent checkpoint.
CodingHandler recovers it without a new model call and finishes through
FencedStore. Patch completion and Run completion are separate boundaries.

scripts/demo_crash_recovery.sh runs the four-state child SIGKILL matrix and a
true SIGKILL between verified_pass and final checkpoint. It verifies real
ClaimBatch takeover, fencing, cached verdicts, shared budget and source isolation.
Only that test Run's expiry is advanced to avoid a long wait; Epoch is never
synthesized. Only an owned child is killed, never the shell or database.

The Mock harness drives existing Scheduler.execute; the checkpoint-gap recovery
calls the same public CodingHandler. These are bounded integration demos, not
a daemon load benchmark. Rename and Commit remain separate; unknown file bytes,
missing/replaced Worktrees, hostile writers, power loss and multi-host recovery
are not made safe by declaring the demo successful.

The unpublished 2026-10-21 local implementation also covers an exhausted last Claim:
the existing Scheduler maintenance path can finalize matching persisted successful
evidence, after expiry, in a protected Run-row transaction. It neither obtains another
execution Claim nor increments Epoch, runs tests, generates patches or calls a model.
Stale/missing verification inputs cannot be finalized as success. This restricted
completion path is distinct from a replacement Worker continuing execution.
A missing Worktree, wrong identity or mismatched digest is invalid evidence and this
restricted path records failure. A missing matching-profile new Worker or a temporary
database/check error may instead leave completion_pending awaiting intervention.
Restart using the same absolute Worktree base and verification configuration; do not
reset a budget or create a replacement empty directory to bypass the evidence checks.

## Distinct fault windows

| Experiment | Failure / takeover boundary | What it proves |
| --- | --- | --- |
| Day 19 legacy CLI | -inject-sigkill after Rename and before Patch result persistence | Intent/hash reconciliation; its legacy lab uses direct ownership changes rather than real Queue takeover |
| Day 19 Store unit test | TestDay19CrashBetweenRenameAndResult returns an injected hook error | The Rename/result gap is recoverable; this unit test is explicitly not a real SIGKILL |
| Four-state matrix | Owned child is killed after prepared, applied, verified_pass or verified_fail is durable | Recovery per committed state with actual ClaimBatch takeover |
| Day 20 checkpoint gap | TestDay20RealSIGKILLBeforeCheckpoint kills its own child after verified_pass, before final Agent checkpoint | Patch verdict survives and a real new owner finishes the checkpoint without another model call |
| Local 2026-10-21 last Claim | TestInterviewLastClaimSIGKILLFinalizesWithoutExecution kills its child after verified_pass, digest and completion_pending are durable | Scheduler maintenance finalizes with Epoch/max_attempts/repair rounds still 1 and no additional model calls; no second execution Claim |
| Day 21 evaluation | Seed verified_pass, expire only the test lease, claim as Worker B | Deterministic cached-result takeover measurements; no SIGKILL is injected here |

The Rename/result gap and verified-result/final-checkpoint gap are different.
The current local demo runs the real four-state matrix, Day 20 checkpoint test
and last-Claim evidence-only completion test;
retained Day 19 CLI/unit results are not substituted for those real-claim tests.
