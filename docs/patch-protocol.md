# Patch protocol

[English](patch-protocol.md) | [简体中文](zh-CN/patch-protocol.md) | [Home](../README.md)

<!-- source-publication-version -->
> Companion source is published at [df93c3c](https://github.com/Loz888/durable-coding-agent-runtime-source/commit/df93c3c13b130b3d08a635a325cb592f4908fd52). [Evidence and limits](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/df93c3c13b130b3d08a635a325cb592f4908fd52/reports/runtime-followup-20261022/README.md) · [GitHub CI](https://github.com/Loz888/durable-coding-agent-runtime-source/actions). “Uncommitted/not pushed” wording below records acceptance-time snapshots rather than current publication status.


This page describes the implementation. Run reproduction commands in the [public source repository](https://github.com/Loz888/durable-coding-agent-runtime-source); this documentation repository contains no source and does not execute software CI.

```text
prepared → applied → verified_pass / verified_fail

CurrentSHA == BeforeSHA → guarded Apply
CurrentSHA == AfterSHA  → RecoverApplied
otherwise              → Conflict; stop
```

Prepare persists path, hashes, exact replacement, preparing Epoch and causal
round_number, consuming one logical repair round in the same transaction.
Identical Patch ID/content is idempotent; different content under that ID
conflicts. New intents cannot overlap prepared, applied or verified_pass.
After verified_fail, a new intent needs budget and the preceding known AfterSHA.

The Store locks the Lease row and rechecks TTL after acquiring it.
ApplyGuarded checks authority immediately before Rename and before result
commit; a durably bound Worktree rejects older unguarded standalone Apply.
If expiry follows Rename, old result commit is rejected and the prepared
Intent remains recoverable.

VerifyGo runs outside the long transaction. Verdict persistence revalidates
Lease, Worktree binding and current hash. In the unpublished 2026-10-21 local
source, cached output/exit-code reuse additionally requires a matching versioned
verification_input_digest for the bounded candidate tree, test inputs and execution
configuration. A missing or changed digest cannot inherit a previous passing verdict;
a fresh verification needs valid execution authority. This currently supports only
standalone stdlib-only Go modules. DurationMS is not persisted in the legacy table,
so cached duration is not a fresh test measurement. See [current local scope](validation.md#local-engineering-acceptance--2026-10-21-unpublished).

Identity combines a random Git-metadata token with baseline, registration and
path checks. It protects accidental path reuse in the stated trusted-writer
model, not malicious processes with equal OS access.
