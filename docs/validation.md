# Recorded implementation validation

The privately maintained implementation was validated on Linux/WSL with Go 1.27.1 and PostgreSQL16, without a model API key.
These are functional test observations, not throughput/latency benchmarks.

| Command group | Passed test entries | FAIL | SKIP |
|---|---:|---:|---:|
| Full project | 207 | 0 | 0 |
| Full project race detector | 207 | 0 | 0 |
| repair20 repeated ten times | 50 | 0 | 0 |
| durable / repair19 / integration20 | 61 | 0 | 0 |

Counts include parent tests, subtests and crash-child helper entrypoints.
All complete suites used both dedicated PostgreSQL test databases.
Vet and build exited zero. Database-independent suites passed 127 test entries.
The executable Mock demo passed one required scenario; the crash demo passed
six entries covering the four-state matrix and final-Checkpoint crash gap.

## Measured statement coverage

| Package | Coverage |
|---|---:|
| cmd/day8-server | 87.3% |
| distributed | 66.3% |
| durable | 81.0% |
| eval16 | 89.4% |
| executor | 93.1% |
| httpapi | 58.2% |
| internal/integration20 | No statements (test-only package) |
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

Command packages without unit tests report zero/no test files. External demo
execution is not automatically counted as coverage instrumentation. All
coverage test packages passed; these values describe this run only.

## Evidence and public verification boundary

These figures summarize recorded runs of the private implementation.
The public repository contains documentation only; it does not distribute
source code, tests, migrations, raw logs, private workflow links or credentials.

The private implementation CI passed again after commit-identity cleanup.
No software CI badge or public execution result is asserted for this
documentation repository. Readers cannot reproduce the software tests or
coverage measurements from these documents alone.

The validation counts include parent tests, subtests and crash-child helpers;
they are not a count of independent production workloads. Coverage values
describe one recorded run and are not a throughput or reliability benchmark.

The documented crash experiments used actual Queue claims, concurrent fencing,
four persisted Patch states and owned-child SIGKILL. They do not prove power-loss
safety, arbitrary external-effect exactly-once or multi-host recovery.
