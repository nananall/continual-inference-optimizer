# Experiment record — <id>

## Identity

- Status: NOT RUN / INCONCLUSIVE / ACCEPTED / REJECTED
- Timestamp and operator:
- Engine / model / revision:
- Baseline SHA and dirty diff:
- Candidate SHA and diff:
- GPU / driver / runtime:
- P/D split, TP/PP/EP/DP/DCP:
- Dtypes and model/weights revision:
- Launch commands and environment:
- Dataset and seed/hash:
- Traffic: QPS, concurrency, ISL/OSL distributions, cache-hit ratio, MoE/sparse distributions:
- Evidence links: logs, profiler traces, benchmark records:

## Hypothesis

- Observed critical path:
- Evidence and source path:
- Change / why it should help:
- Allowed and forbidden changes:
- Fast screen command + result:
- Representative test command + result:

## End-to-end validation

| Metric | Baseline | Candidate | Method/window | Pass? |
| --- | --- | --- | --- | --- |
| Accuracy / correctness | — | — | — | — |
| TTFT p50/p90/p95 (ms) | — | — | — | — |
| Mean TPOT / ITL (ms) | — | — | — | — |
| Output TPS / TPM | — | — | — | — |
| Achieved QPS | — | — | — | — |
| Peak GPU/CPU memory | — | — | — | — |
| Errors / timeouts / OOM | — | — | — | — |

- Number of runs, sample sizes and estimated variance:
- Cold/hot cache comparable?
- All ranks + graph use candidate?
- Full user SLO and constraints satisfied?
- Decision and rationale:
- Rollback commands:
- Highest-value next experiment:

> Leave measurements as unknown rather than inventing numbers. Never mark a candidate accepted from a microbenchmark alone.
