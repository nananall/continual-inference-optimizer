# Benchmark contracts and measurement fidelity

Load this reference before running or comparing benchmarks.

## Frozen benchmark contract

Record: engine git SHA/dirty diff; model revision; GPU count/type/topology; P/D layout; TP/PP/EP/DP/DCP; dtype/quantization; driver/runtime/compiler; seed/sampling configuration; routing and KV transport; cache policy; prefix/cache hit; graph shape/capture; concurrent requests; offered QPS; prompt/output length distributions; dataset hash; start/end clock and warmup; whether the profiler was enabled.

A baseline/candidate comparison is **invalid** when a relevant condition differs without explicit stratification. Do not compare 500 fixed requests with a synthetic workload and call that an optimization.

## Measurement hierarchy

1. **Static:** compile/import and test basic correctness with no performance claim.
2. **Micro:** use accurate shapes, strides, dtype, top-k distribution and real device. Time using synchronized device events or a trusted kernel benchmark harness, not unsynchronized Python clocks.
3. **Representative:** partial layer/model or full engine phase with realistic scheduling, graph capture, cache state and communication.
4. **End-to-end:** fixed service replay on all ranks, representative steady-state offered load, identical correctness contract.

Run multiple samples when variance matters. Use paired alternation (A/B/A/B) where practical to reduce effects from thermal/clock drift, noisy neighbors and cache warming. Report medians/quantiles and sample size; avoid fake precision. Separate profiler captures from authoritative unprofiled timing when overhead is material.

## Metrics

- **TTFT:** client request start to first generated token received. Decompose scheduling, prefill computation, handoff and decode queue where tools allow.
- **TPOT / ITL:** define exactly whether this is mean interval over subsequent tokens, per-request average then aggregate, or all-token weighted aggregate. Do not compare incompatible definitions.
- **Output TPS:** sum of completed output tokens divided by shared wall-clock measurement duration. Specify whether first tokens or draft tokens are counted.
- **TPM:** output TPS × 60 for the same boundary and definition. Include prompt tokens only if the metric explicitly says total tokens.
- **QPS:** completed requests / elapsed wall clock; distinguish offered QPS from achieved QPS.
- **Tail latency:** percentiles computed over same request population; do not use p90 TTFT on one dataset and mean on another.
- **Memory:** static reservation, peak actual allocated, KV occupied, cache host/device, transfer buffers, graph pools; include OOM and non-zero error count.
- **SLO:** specify thresholds and window (e.g. p90 TTFT, mean TPOT, 5/30-minute windows). Reject violating constraints even with higher aggregate throughput.

## Correct comparisons

- Report cold- and warm-prefix-cache separately when both matter.
- In PD deployments, collect P queue, P forward, P-to-D KV send/receive, D queue, first decode and token-step data separately where available.
- If the concurrency is low and the objective is TTFT, ensure offered load and achieved throughput are still comparable.
- At stable high load, a faster prefill does not imply a throughput increase if decode is saturated.
- If the user says throughput must not decrease, require an explicit comparison under fixed load; lack of measurement means **unknown**, not **passed**.
- Do not include compiler setup, load time or profiler collection in token-generation throughput unless the workload explicitly evaluates startup.

## Outcome labels

**ACCEPTED:** end-to-end objective improves beyond noise, correctness passes, all constraints pass.

**REJECTED:** correctness fails, any hard constraint fails, or reliable comparison loses.

**INCONCLUSIVE:** samples insufficient, cache/workload changed, regression within noise, mismatched code/graph, missing end-to-end check.

**NOT RUN:** commands are provided but the agent had no authority/access to run them.
