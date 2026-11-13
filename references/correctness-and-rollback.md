# Correctness, attribution and rollback gates

Run correctness checks before and after benchmark candidate selection, not only at the end.

## Functional API invariants

Verify shape, dtype, padding, sequence boundaries, decoding length, streaming API schema, tokenizer behavior, tool-call formatting where applicable, and absence of unexpected 4xx/5xx responses.

Inspect all ranks/stages for exceptions, watchdog timeouts, invalid memory access, asynchronous error propagation, deadlocks and CPU/cgroup OOM.

## Numerical precision

- Use a deterministic reference path when possible. For quantization/fusion changes, define finite-value, tolerance and application-level acceptance criteria before measurement.
- Compare logits or intermediate tensor samples when appropriate, not just one output string.
- Treat nondeterministic kernels and sampling as distribution-level questions. Differences with the same seed alone do not prove incorrectness if execution order or RNG consumption changes.
- For speculative decoding/MTP, test the intended sampler's correctness assumptions. Acceptance length and rate are performance metrics; equivalent task score is not proof of exact target distribution.
- Greedy exact equality, per-token numerical tolerance and task benchmark accuracy are different gates; choose the required gate deliberately.

## Attribution

A result is not trustworthy if the tested commit differs from the running ranks, an old CUDA/HIP graph remains active, graph fallback silently changed the implementation, cache state drifted, input mixes differ or the profiler itself dominates runtime.

A kernel-only reduction in duration must be kept separate from end-to-end latency. Record A/B sample sizes and noise estimates where possible. Beware averaging ratios across runs with different denominators.

## Rollback

Keep the previous verified git SHA and launch flags. Before a risky patch, make recovery safe: capture commands, dirty diff and output artifacts. If test workers hang, avoid broad kill commands that might affect unrelated services.

On driver error, illegal device access, deadlock or OOM: stop the candidate on the isolated test service, collect minimal diagnostics, restart affected processes/context, reestablish the baseline and only then resume testing. Do not publish a speedup measured after an invalid context.

If a winner is accepted, tag or record its commit and exactly why it passed. If its combination with another winner regresses, bisect the interaction rather than silently reporting summed gains.
