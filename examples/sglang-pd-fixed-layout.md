# Example: SGLang PD inference, fixed GPU layout

Use as a starting prompt, adjusting model and flags to the **real** deployment:

~~~text
Use the continual-inference-optimizer skill to optimize my SGLang service.

The Prefill/Decode layout is fixed at P4/D4. Read the current launch
commands to recover the exact TP/PP/EP/DP values; never silently modify
these or the model/precision/API contract.

I want lower TTFT and TPOT with no throughput or accuracy regression.
Start by identifying the baseline revision, current best result, fixed
request dataset and actual P/D traces. Separate admission, prefill,
caching, transfer, decode queue, verify and token-step latency.

Optimize mechanisms and kernels based on evidence, including cache,
communication, scheduling, attention, MoE, MTP and CUDA Graph behavior.
Use safe fast screening, realistic representative tests and mandatory
end-to-end replay. Rebenchmark combinations, not just individual wins.

Provide exact source diffs/commits, startup/replay commands, trace
identifiers, before/after tables, rejected candidates and rollback steps.
Do not invent measurements, hot-swap support or performance gains.
~~~

If the target has a fixed SLO (e.g. p90 TTFT <= 20 seconds, mean TPOT <= 30ms), add the **actual** targets and window to the experiment contract, along with offered QPS, model and hardware. Never use the example threshold as an assumed requirement.

A lower prefill TTFT does not guarantee an increase in total tokens/s if decode remains the throughput bottleneck. Measure both.
