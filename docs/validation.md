# Release validation


<!-- source-publication-version -->
> Companion source is published at [df93c3c](https://github.com/Loz888/durable-coding-agent-runtime-source/commit/df93c3c13b130b3d08a635a325cb592f4908fd52). [Evidence and limits](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/df93c3c13b130b3d08a635a325cb592f4908fd52/reports/runtime-followup-20261022/README.md) · [GitHub CI](https://github.com/Loz888/durable-coding-agent-runtime-source/actions). “Uncommitted/not pushed” wording below records acceptance-time snapshots rather than current publication status.


<!-- runtime-followup:start -->
## 2026-10-22 follow-up

Follow-up full/race each 407/0/0; offline entry 206 Go events plus 7 Python tests. Twelve new synthetic trials completed 12,003 tasks. The real upstream seed is an expected-failure/known-fix control, with zero model calls. Clock backsteps remain unresolved. See companion source local `reports/runtime-followup-20261022`; not pushed, no software CI in this showcase.
<!-- runtime-followup:end -->

前批本地完整/race各338 PASS、关键恢复十轮220 PASS，均0 FAIL/0 SKIP；数据库无关153 PASS。真实Docker4场景与双容器接管通过。Live为12合成任务/11请求、3382/4201 Token、估价$0.0060558，非账单；Live绑定其实际较早二进制。物理双主机、断电、真实开源缺陷泛化、长期容量和历史时钟根因仍未验证。没有提交或推送。


## Local upgrade · 2026-10-22 (unpublished)

This section describes the current working tree; older dated “pending/BLOCKED” statements are historical. No commit, push or remote-CI result is implied.

Implemented bounded multi-file patches (up to 8 allowlisted existing Go files per repair round), persisted logical model responses with conservative unknown-outcome handling, real Docker confinement and an independent scoped orphan reaper, OTLP/metrics/local alert acceptance, paced/real-Go-test benchmark workloads, and content-addressed portable Worktree recovery. Two isolated host containers plus a separate authenticated artifact service passed actual A-container SIGKILL, natural lease expiration and a real Epoch2 ClaimBatch takeover. This remains one physical computer, with no physical-host/power-loss claim.

The frozen 12-task synthetic live evaluation made **11 actual requests**, **3382 input / 4201 output tokens**: 9 fresh repairs and one response-recovery control passed holdout assertions; an immutable failure control failed as expected; an already-passing baseline made zero requests. Mode: High, requested/returned alias `deepseek-flash`. The peak cache-miss estimate is **$0.0060558**, not an invoice, against this run's separately authorized $2 cap. These simple manually constructed tasks do not measure general repository repair accuracy.

Explicit sealed input policies capture offline vendor inputs or root-contained workspace modules, with shared fixed-command planning for trusted-local and sandbox execution. Terminal Worktrees can be inspected/dry-run and retired with identity/state checks and a persisted deletion intent. A separate fresh local PostgreSQL cluster restored synthetic data and read-only role permissions.

Remaining unverified: a second physical host, network partition/power loss, arbitrary hostile repositories, representative real-world defects and human review, production soak/capacity, and the cause of two historical negative DB wall-clock durations. Monotonic benchmark timing does not remove the Lease clock assumption.

本仓库仅文档；完整可执行证据保存在配套源码本地 reports，尚未推送。


> This describes the companion source implementation. Run commands in the source checkout; this showcase contains documentation and runs no software CI. This alignment draft is local and has not been pushed.

[English](validation.md) | [简体中文](zh-CN/validation.md) | [Home](../README.md)

## Historical review snapshot · 2026-10-20

Full and race suites each pass 260 test/subtest events; database-independent suites pass 138, with no failed or skipped tests. CI at 645c88e passes the existing PostgreSQL, crash and Capture entrypoints. See [results and scope](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/docs/review-20261020/RESULTS.md), [local evidence](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/reports/review-20261020/verification.json) and [exact remote runs](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/reports/review-20261020/remote-ci.json). No new coverage percentages or paid-model measurements are inferred from these checks.

## Historical release snapshot · 2026-10-17

Local acceptance was executed on 2026-10-17 in the independent release checkout
using Go 1.27.1, PostgreSQL 16 and Linux/WSL. No live model API key was provided.
Only the two fixed dedicated test databases were used.

| Validation | Passed test entries | FAIL | SKIP |
| --- | ---: | ---: | ---: |
| Full project | 226 | 0 | 0 |
| Full project race detector | 226 | 0 | 0 |
| Full project coverage run | 226 | 0 | 0 |
| repair20 / durable / repair19 repeated ten times | 690 | 0 | 0 |
| Database-independent suites | 133 | 0 | 0 |
| Mock demo | 1 | 0 | 0 |
| Four-state + Day20 crash demo | 6 | 0 | 0 |

Counts include parent tests, subtests and crash-child helper entrypoints.
Format, build and go vet also passed. Every counted suite rejected actual
test SKIP; packages with no tests were not misreported as skipped tests.

## Day 21 and reproduction

The acceptance runner captured 12 fresh deterministic trials and strict scoring
passed all scenarios. It also recomputed the committed summary and Markdown
report byte-for-byte. Original cases, observations and recorded 400/1403 ms
P50/P95 were preserved. The fresh capture's latency is a separate observation,
not a replacement benchmark.

The new Day 21 runner owns a fresh external output directory and does not delete
committed JSONL. [Evaluation](evaluation.md) gives the recorded data, real capture
path, denominators, timing boundary and sample limitations.

## Measured statement coverage

| Package | Coverage |
| --- | ---: |
| cmd/day21-eval | 55.6% |
| cmd/day8-server | 87.3% |
| distributed | 66.3% |
| durable | 81.2% |
| eval16 | 89.4% |
| eval21 | 94.9% |
| executor | 93.1% |
| httpapi | 58.2% |
| internal/testdb | 35.3% |
| observability | 93.9% |
| persistence | 44.9% |
| reliability15 | 85.7% |
| repair17 | 83.3% |
| repair18 | 82.5% |
| repair19 | 66.9% |
| repair20 | 39.2% |
| repodiag | 84.7% |
| runtimecore | 94.3% |

internal/integration20 is test-only and has no production statements.
Command packages without unit tests report zero/no test files. Coverage values
describe this recorded run and are not a performance or production-reliability measure.

## Remote CI and scope

[Actions](https://github.com/Loz888/durable-coding-agent-runtime-source/actions) is the
authoritative current remote status. CI prepares PostgreSQL 16 over TCP, creates
both dedicated databases, applies migrations 001–009, then runs full ordinary
and race suites, the real Mock/crash demos and Day 21 Capture/report reproduction.
Local WSL validation used the corresponding Unix-socket connection.
No local result alone is asserted as remote CI success.

The source repository is public; no software license has been selected.
The recorded local results above were obtained in the independent release
checkout. The same implementation and test files were copied into this public
repository without importing its private Git history. Original development
files were read-only. Local reports, raw logs, credentials and backups stay
outside Git. The public Actions link above reports this repository's own CI.

Day 19 Rename, Day 20 final-checkpoint SIGKILL and Day 21 seeded takeover are
distinct experiments. No sandbox, arbitrary-effect exactly-once or multi-host
Worktree recovery is claimed.

## Local engineering acceptance · 2026-10-21 (unpublished)

The companion source implements bounded complete candidate snapshots, a versioned `verification_input_digest` and evidence-only completion after the last execution Claim, limited to standalone stdlib-only Go modules. Historical verdicts without a digest cannot be reused directly. Completion checks identity, complete inputs, state and Checkpoint CAS without another execution Claim/Epoch, repair round or model call; cancellation/Sweep competition and rollback have dedicated regression cases.

The final source full and race suites each passed **301 test/subtest events, zero failures/skips** after the Worker startup adjustment. The core five-package ten-repeat check passed **1,360 events, zero failures/skips** (`-p 1 -count=10 -timeout 1800s`; repair20, durable, repair19, internal/integration20 and reliability15). Exact commands and final source fingerprints are retained in the companion local `docs/review-20261020/RESULTS.md` and `reports/review-20261020/engineering-20261021.json`. Stop old Workers, apply migrations **001–011**, then start the new Worker; mixed-version rolling upgrades are not claimed.

These are local source results, not software CI in this showcase or remote CI. Container adapter code exists, but actual engine isolation remains unverified. Historical live, Day21 Mock and older regression records retain their own dates, versions and denominators. New companion files remain unpublished, so this page does not link to nonexistent GitHub paths.

The earlier ten-repeat attempt with a 600-second timeout exited 1 after 600.715 seconds, with 1,007 passing events and a package failure in durable. It is not a passing acceptance run. The later `-p 1 -timeout 1800s` repeat kept all five packages and ten repetitions and completed in 804.374 seconds with 1,360 passing events. The two private raw records remain separate; no test scenario or assertion was removed.

## Local extension batch 1 · 2026-10-21 (unpublished)

The companion source now includes optional OpenTelemetry SDK/local NDJSON tracing, a real-PostgreSQL synthetic throughput benchmark and a frozen 20-case scripted repair evaluation with an independent oracle. Latest full/race runs pass 309 events each; ten targeted race runs pass 90, and the database-independent entrypoint passes 147. All have zero FAIL/SKIP; events include parent/subtests and helper entries. This documentation repository still has no software CI.

The benchmark completes 2400 tasks over 12 trials with no failures/extra claims, using fixed 20ms callbacks, not model work or production capacity. All 20 expected behaviors match, including failure controls: 17 visible positives, 16 oracle positives, and one intentional overfit rejected by the oracle. There are no new live-model calls. Real containers remain BLOCKED; multi-file and multi-host recovery are incomplete. Local evidence is under the companion source's `reports/runtime-extension-20261021/verification.json`; these files are unpublished, so no premature remote file links are provided. Apply migrations 001–012 before tracing. Earlier 301/260 snapshots below remain historical.

Primary benchmark timing uses Go monotonic intervals in one process. Two negative raw database wall-clock measurements are retained; their cause is unconfirmed. Desktop trials vary substantially, so no best-run capacity or SLA claim is made. Only two dedicated databases in an owned private PG cluster were used; fixture rows are cleaned, and the original database was not migrated. The filesystem regression proves candidate readability for the non-root UID, not real-engine isolation.

## Remaining work status · 2026-10-21

As of 2026-10-21, these statuses describe the **local unpublished version**. A completed baseline does not mean the entire workstream is complete.

| Workstream | Status | Remaining work |
|---|---|---|
| Multi-file patches | **Not implemented** | Change manifests, per-file digests, partial-write recovery, unified verification and crash matrix |
| Sandbox isolation | **Adapter exists; real acceptance blocked** | Docker isolation/resource/deadline acceptance; Worker-SIGKILL orphan-container reclamation remains incomplete |
| Broader repair evaluation | **Offline baseline implemented; expansion pending** | Current 20 frozen synthetic cases use scripted proposals; real repositories/models, independent manual adjudication and cost accounting remain pending |
| OpenTelemetry | **Local tracing implemented; operations pending** | OTLP collector/backend, metrics, alerts and exporter-failure validation; SIGKILL can lose buffered spans |
| Measured throughput | **Synthetic benchmark complete; capacity acceptance pending** | Two negative database wall-clock durations remain unexplained; real workloads, sustained/burst load, lock contention and recovery timing remain pending |
| Multi-host Worktree recovery | **Neither implemented nor tested** | Portable artifacts, destination reconstruction/identity checks and two-host takeover; same-machine container simulation is not two-host evidence |

These are outstanding tasks, not new acceptance results. Source details are maintained locally in `docs/pilot/IMPROVEMENT_OUTLINE.zh-CN.md`; no unpublished remote path is linked.
