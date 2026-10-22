# Day 21 evaluation

> This describes the companion source implementation. Run commands in the source checkout; this showcase contains documentation and runs no software CI. This alignment draft is local and has not been pushed.

[English](evaluation.md) | [简体中文](zh-CN/evaluation.md) | [Home](../README.md)

<!-- source-publication-version -->
> Companion source is published at [df93c3c](https://github.com/Loz888/durable-coding-agent-runtime-source/commit/df93c3c13b130b3d08a635a325cb592f4908fd52). [Evidence and limits](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/df93c3c13b130b3d08a635a325cb592f4908fd52/reports/runtime-followup-20261022/README.md) · [GitHub CI](https://github.com/Loz888/durable-coding-agent-runtime-source/actions). “Uncommitted/not pushed” wording below records acceptance-time snapshots rather than current publication status.


## Evidence layers after the review

The [current results page](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/docs/review-20261020/RESULTS.md) separates review regression, this historical Mock capture and the archived live single-file repair. The structured diagnosis-v1 gate checks observed execution and references, not root-cause correctness. Neither a 260-event regression count nor a passing diagnostic verdict is a live-model accuracy denominator. The recorded Day21 data below are unchanged.

## What was measured

The committed dataset is a recorded, deterministic Mock experiment:
four controlled scenarios, three trials each, using trusted single-file Go fixtures.
It is not a live DeepSeek evaluation or an open-world bug benchmark.

| Scenario / case_id | Expected behavior | Recorded checks |
| --- | --- | ---: |
| divide-repair | A failing baseline is repaired; independent Go tests pass; checkpoint succeeds | 3/3 |
| baseline-pass | An already-passing baseline consumes no repair round or model call | 3/3 |
| two-round-exhaust | Two unsuccessful rounds exhaust the budget without claiming success | 3/3 |
| verified-pass-takeover | A new real claim reuses a verified result with no new model call | 3/3 |

The Capture harness is TestDay21Capture in durable/day21_capture_test.go.
It uses real PostgreSQL, Queue.ClaimBatch, Scheduler.execute, CodingHandler and the
repair19 Patch Store. Proposals come from day21Proposer, not an external API.

## Recorded results

| Metric | Value | Denominator / interpretation |
| --- | ---: | --- |
| Scenario correctness | 12/12 | All four kinds must satisfy their own expected behavior |
| Fresh repair | 3/3 | Only divide-repair trials |
| Verified-result recovery | 3/3 | Only verified-pass-takeover trials |
| Unsupported success claims | 0/9 | Only statuses repaired or already_passed; excludes three expected exhausted runs |
| P50 | 400 ms | Nearest-rank percentile over the 12 recorded durations |
| P95 | 1403 ms | With 12 samples, the nearest-rank P95 is the largest sample |
| Durable repair rounds | 12 | Sum of rounds_used |
| Mock proposer calls | 9 | Sum of model_calls; not a live-model usage measurement |
| Token usage known | 0/12 | The zero token accumulators do not imply measured zero cost |

These numbers belong to the committed recorded dataset. A fresh capture reports
its own timing; it never overwrites these figures or claims they are portable benchmarks.

## Auditable files and scoring

- [cases.json](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/evaldata/day21/cases.json): four kinds and exact trial counts.
- [runs.jsonl](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/evaldata/day21/runs.jsonl): 12 captured Observation records.
- [summary.json](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/evaldata/day21/summary.json): machine-readable aggregate.
- [report.md](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/evaldata/day21/report.md): generated narrative report.
- [Scorer](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/eval21/score.go) and [CLI](https://github.com/Loz888/durable-coding-agent-runtime-source/blob/main/cmd/day21-eval/main.go).

Observation fields are case_id, trial, run_id, status, baseline_failed, tests_passed,
original_clean, checkpoint_succeeded, recovered, rounds_used, attempts, model_calls,
usage_known, prompt_tokens, completion_tokens and duration_ms.
Records contain synthetic test Run IDs; no repository paths or contact details
were present, so the selected observations require no redaction. Capture logs,
local Worktrees and credentials are excluded from Git.

Score rejects unknown cases, missing/duplicate trials, reused Run IDs and invalid
numeric fields. The CLI rejects unknown observation fields and trailing JSON.
-require-all returns a nonzero exit code unless every scenario passes.

A positive claim requires independent tests_passed, original_clean and
checkpoint_succeeded evidence; repaired claims also require baseline_failed.
Scenario correctness applies additional kind-specific rules: recovery needs
recovered=true, model_calls=0 and attempts>=2; baseline-pass needs zero rounds;
budget exhaustion needs failed tests, no successful checkpoint and the exact
configured maximum rounds. These checks are bounded evidence for these fixtures,
not proof of arbitrary code correctness.

## Capture path and timing boundary

Fresh repair starts timing immediately before the actual claim and stops when
Scheduler.execute returns after final checkpoint handling. Independent proof
collection runs afterward and is excluded. For recovery, Worker A's preparation
of verified_pass is also excluded; the measured interval begins before Worker B's
new claim. Tests expire only their own Run; the new Epoch comes from ClaimBatch.

Day 21 recovery seeds a verified result and performs takeover. It does not kill a
Worker. The real Day 20 SIGKILL experiment is separate; the Day 19 Rename gap is
another boundary. See [crash recovery](crash-recovery.md).

## Reproduce the recorded report

This does not run the agent or need a database. Use Go and the checked-in files:

```bash
output="$(mktemp -d)"
env -u DEEPSEEK_API_KEY go run ./cmd/day21-eval   -cases evaldata/day21/cases.json -runs evaldata/day21/runs.jsonl   -report "$output/report.md" -summary "$output/summary.json" -require-all
cmp evaldata/day21/summary.json "$output/summary.json"
cmp evaldata/day21/report.md "$output/report.md"
```

## Capture a fresh experiment

After the dedicated database setup in [README](../README.md):

```bash
./scripts/day21_accept.sh
```

The runner checks both actual database names, unsets the real model key,
runs scorer/CLI unit tests, captures all 12 required subtests and checks the
committed report before scoring the new data. A fresh external directory owns
all output. DAY21_OUTPUT_DIR may name a new directory outside the repository;
an existing directory or repository-local path is rejected. Full ordinary/race
regression remains in test.sh and the CI jobs.

## Limits

The sample is small and repeated on controlled fixtures. Execution order,
compiler caches and host load affect latency. The recorded capture does not
contain a complete CPU/RAM/environment trace. Release validation uses Linux/WSL,
Go 1.27.1 and PostgreSQL 16, but that does not make 400/1403 ms a general claim.

No live-model success rate, API spending, throughput, sandbox security,
multi-host reconstruction or arbitrary-effect exactly-once conclusion follows.

## Local engineering acceptance · 2026-10-21 (unpublished)

The companion source implements bounded complete candidate snapshots, a versioned `verification_input_digest` and evidence-only completion after the last execution Claim, limited to standalone stdlib-only Go modules. Historical verdicts without a digest cannot be reused directly. Completion checks identity, complete inputs, state and Checkpoint CAS without another execution Claim/Epoch, repair round or model call; cancellation/Sweep competition and rollback have dedicated regression cases.

The final source full and race suites each passed **301 test/subtest events, zero failures/skips** after the Worker startup adjustment. The core five-package ten-repeat check passed **1,360 events, zero failures/skips** (`-p 1 -count=10 -timeout 1800s`; repair20, durable, repair19, internal/integration20 and reliability15). Exact commands and final source fingerprints are retained in the companion local `docs/review-20261020/RESULTS.md` and `reports/review-20261020/engineering-20261021.json`. Stop old Workers, apply migrations **001–011**, then start the new Worker; mixed-version rolling upgrades are not claimed.

These are local source results, not software CI in this showcase or remote CI. Container adapter code exists, but actual engine isolation remains unverified. Historical live, Day21 Mock and older regression records retain their own dates, versions and denominators. New companion files remain unpublished, so this page does not link to nonexistent GitHub paths.

## Local scripted extension / 本地冻结评测 · 2026-10-21（未发布）

Companion source `evaldata/repair-extension-20261021/cases.json` is frozen at SHA256 `48c8a693c519aae44b63daf9c3ab26cbb422a2804d5c46f328ecab4e53a7c014`. It contains 14 correct scripted patches, 2 already-passing cases, 2 exhausted budgets, 1 invalid fragment and 1 intentional overfit. Real Queue/Claim/Patch/Checkpoint paths run, followed by an independently specified holdout test on a copy.

All 20 expected behaviors match: 17 visible-test positives, 16 oracle positives and one intentional visible-positive/oracle-negative control. All original repositories stay clean, with real claim Epoch 1. Scripted proposals are not model outputs: this is neither a 20/20 repair accuracy claim nor a new live-model result. Historical Day21 and live evidence remain separate. Scope remains controlled single-file, trusted host execution; broader real-repository/model evaluation is incomplete.
