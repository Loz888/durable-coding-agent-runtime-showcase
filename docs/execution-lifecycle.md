# Execution lifecycle

[English](execution-lifecycle.md) | [简体中文](zh-CN/execution-lifecycle.md) | [Home](../README.md)

<!-- source-publication-version -->
> Companion source is published at [df93c3c](https://github.com/Loz888/durable-coding-agent-runtime-source/commit/df93c3c13b130b3d08a635a325cb592f4908fd52). [Evidence and limits](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/df93c3c13b130b3d08a635a325cb592f4908fd52/reports/runtime-followup-20261022/README.md) · [GitHub CI](https://github.com/Loz888/durable-coding-agent-runtime-source/actions). “Uncommitted/not pushed” wording below records acceptance-time snapshots rather than current publication status.


This page describes the implementation. Run reproduction commands in the [public source repository](https://github.com/Loz888/durable-coding-agent-runtime-source); this documentation repository contains no source and does not execute software CI.

1. Submit atomically creates Run metadata and revision-zero history.
2. ClaimBatch selects priority/available candidates with FOR UPDATE SKIP LOCKED,
   checks lease_epoch < max_attempts, increments Epoch and assigns owner/expiry
   in one SQL statement. Manual non-queue Claim also enforces budget.
3. Heartbeat starts before slow model/workspace initialization. Renewal failure
   cancels the Run. Claim, Renew and guarded Patch writes share the Run row.
4. Diagnosis resumes persisted pending tools before another model call; model
   decisions are saved before tools and outputs in a later checkpoint.
   Coding reconciles persisted Patch state before proposing another change.
5. FencedStore.Save checks owner, Epoch, TTL and expected Revision, then commits
   a newer snapshot. CodingHandler writes the final Agent checkpoint.
6. Scheduler releases completed work. Terminal Runs are not reclaimable.
7. RetryLater / Day15 transient policy release and reschedule; permanent
   failures terminate. A stale owner/CAS error cannot finalize a new owner.
8. The same Scheduler periodically performs bounded maintenance. In the unpublished
   2026-10-21 local source it first considers completion_pending Runs for evidence-only
   finalization: matching verified_pass evidence, identity, complete inputs and CAS
   are checked in the protected Run-row transaction, without another execution grant.
   SweepExhausted handles the remaining exhausted paths; pending completion is excluded
   from the generic failure sweep. Live leases, cancellation and terminal state remain
   protected. See [current acceptance scope](validation.md#local-engineering-acceptance--2026-10-21-unpublished).

| Counter | Meaning | Consumed by |
|---|---|---|
| lease_epoch vs max_attempts | Successful execution grants | Successful Claim |
| retry_count | Explicit reschedules | RetryLater / retry policy |
| rounds_used vs max_rounds | New logical Patch intents | First Prepare |

MaxAttempts ≠ retry_count ≠ rounds_used. SIGKILL may consume a Claim without
an explicit retry. Takeover never resets repair budget; recovery does not
allocate a new Patch round. Maintenance defaults to a one-second cadence;
errors are logged and the existing Scheduler continues.
