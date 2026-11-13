# Trace-first root-cause playbook

Use this when deciding what to optimize next. Explain a concrete critical path with trace and code evidence.

## Source and deployment inventory

Identify the deployed source revision and exact flags, not just the checkout HEAD. Check launch scripts, environment, process command lines, engine version, compiled extensions, graph capture and rank mapping. For PD services inspect router logs, handoff, prefill and decode separately.

Copyable Linux command patterns (replace placeholders with real paths):

~~~bash
git rev-parse HEAD
git status --short
ps -eo pid,ppid,etimes,args | grep -E 'sglang|vllm' | grep -v grep
nvidia-smi --query-gpu=index,name,memory.used,utilization.gpu --format=csv
grep -En 'TTFT|TPOT|prefill|decode|OOM|timeout|watchdog|NCCL' /path/to/service.log | tail -200
awk '/prefill|decode|KV|transfer|queue/ { print NR ":" $0 }' /path/to/service.log | tail -200
~~~

Do not assume nvidia-smi exists on non-NVIDIA hardware; substitute the vendor's supported profiler or GPU monitor. Avoid collecting private prompt bodies, API keys or internal endpoint URLs in public reports.

## Request-level decomposition

For each slow request or cohort, locate:
- arrival / admission / router wait;
- prefix-cache or L3 lookup, queue wait, read/restore;
- prefill scheduled tokens, chunk count, forward time and PP idle;
- KV transfer enqueue, transport and synchronization;
- decode queue, first step, subsequent step distribution;
- errors, retries, evictions and queue buildup.

Separate p50 from p90/p95 and investigate whether outliers have different input lengths, cache hits or traffic bursts.

## GPU and CPU trace decision tree

- Many short kernels and visible host gaps: inspect graph gaps, CPU synchronization, launch overhead and scheduler; do not assume kernel compute is bottleneck.
- Long collectives or PP bubbles: inspect bytes, participants, topology, CPU overlap, rank skew and waits. Compare aggregate bytes and communication lower bounds before implementing changes.
- MoE routing imbalance: inspect tokens per expert, uneven shard work, routing/scatter/gather, GEMM shapes and tail-rank time; random uniform indices may produce misleading wins.
- Attention/sparse patterns: inspect actual selected blocks, KV length, gather locality, sparsity and bandwidth; do not infer solely from theoretical FLOPs.
- Decode stalls: inspect per-step timeline, model verify, speculative acceptance rate, host sync, memory pressure and kernel occupancy.
- Caches: separate lookup queue, existence check, IO queue, read and post-read; test overlap rather than summing overlapping spans.
- CUDA/HIP graphs: verify whether graphs run newly compiled code and whether all ranks share a compatible candidate.

## Arithmetic intensity and roofline

Arithmetic intensity (AI) = useful FLOPs / bytes transferred at the chosen memory hierarchy. Bound = min(peak FLOPs/s, AI × sustainable bytes/s). The ratio of peak compute to bandwidth is the balance point. Falling below it suggests bandwidth limitation at that AI; above it suggests compute limitation. **Reaching a roofline point does not mean reaching absolute hardware FLOPs peak or service-optimal TTFT/TPOT.**

Use measured tensor shapes, actual bytes, precision and available compute units. Kernel launch time, network barriers, sparse index loads and queueing can dominate end-to-end latency outside this simplified bound.

## Evidence quality

Prefer: actual captured trace + source revision > reproducible microbench > matching upstream PR/issue/commit > papers/speculations. A GitHub issue is a lead, not proof on this deployment.

For every recommendation, record a source path/function, trace location, estimated or measured time share, candidate change and test that could falsify the hypothesis.
