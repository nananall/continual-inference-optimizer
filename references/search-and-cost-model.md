# Candidate search and parallelism cost modeling

Read this when deciding the next experiment or considering topology/configuration. User-fixed layouts always override search.

## Candidate prioritization

Choose candidates based on measured critical-path contribution, attainable improvement, correctness risk, reversibility and experiment cost. Do not hard-code an optimization order. A kernel speedup is only useful if it reduces a limiting stage and does not add offsetting queue, communication or host overhead.

A compact hypothesis:

~~~text
Observed: decode verify phase spends X ms with a host/device synchronization.
Hypothesis: eliminating redundant synchronization reduces token-step critical path.
Patch: <diff / commit>; scope: <rank/shape/dtype>.
Fast test: <reproducer>; representative test: <real workload>.
Full gate: <TTFT/TPOT/output TPS/accuracy/peak memory>.
Rollback: <commit/command>.
~~~

Use X only when measured. Avoid inventing numbers in a filled experiment record.

## Search stages

1. Inspect measured top bottlenecks and source. Form multiple falsifiable hypotheses.
2. Estimate theoretical bounds, but flag assumptions. Use microbenches to prune obvious losers.
3. Probe representative forward passes with realistic data, including experts, cache and attention sparsity.
4. Verify selected candidate(s) under full engine timing and user SLO.
5. Reprofile; change the next hypothesis if the bottleneck moves.
6. Combine compatible winning patches and rerun the **entire** acceptance gate.

## Layout selection: only if permitted

A candidate layout (TP/PP/EP/DP/DCP, P/D ratio) trades communication, memory capacity, batch size, per-device compute, KV replication and workload imbalance. Raw chip peak and link bandwidth alone do not select the best configuration.

When layout search is authorized:
- enumerate memory-feasible alternatives;
- reject hard SLO and capacity failures analytically;
- calibrate communication and representative forward costs with real measurements;
- evaluate top options with the real traffic distribution and full service;
- record why topology or concurrency changes the result.

When the user says **P TP2 / D TP2 fixed** or **P4D4 fixed**, do not execute layout alternatives. Use this analysis only to understand bottlenecks; restrict actual patches to allowed degrees of freedom.

## Traffic drift

Watch for changes in offered QPS, burstiness, prefill length histogram, output length, cache hit ratio, MoE token-per-expert distribution, sparse selection and failures. Re-run relevant representative and end-to-end benchmarks when drift makes the original comparison invalid.

Do not promise unattended monitoring without an actual scheduler/automation. A new SGLang or vLLM version can invalidate assumptions about kernels, graph semantics or dtype; rerun compatibility tests.
