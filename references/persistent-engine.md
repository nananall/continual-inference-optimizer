# Persistent-engine and two-tier experiments

Together Compiler describes a persistent engine with forward-pass hot reload and tiered fidelity. This document gives a **capability-gated adaptation** for SGLang/vLLM; it does not imply either engine ships a compatible hot-swap RPC.

## Capability discovery

Before promising fast hot-swap, verify all of the following from the checked-out code and a safe smoke test:
- Runtime permits replacing the relevant call path while weights, KV metadata and memory remain coherent.
- Each rank / PP stage has an explicit version or epoch handshake and validates code shape and dtype compatibility.
- Old CUDA/HIP graphs are invalidated and recaptured where necessary, and the new graph is actually executed.
- In-flight requests either drain safely or keep the old implementation consistently; no mixed-version execution.
- Failure rolls back atomically or restarts the isolated worker safely. Device-side illegal access mandates fresh process/context.
- Measurement can prove which candidate ran via trace label, version ID or an equivalent direct check.

If any condition cannot be verified, **do not hot-reload**. Fall back to an isolated full restart or test worker. Reusing compilation caches and model artifacts without unsafe graph reuse is still valuable.

## Suggested tier structure

**Fast worker:** a small number of representative layers or operator wrappers, real dtype/shapes and data distribution, minimal profiling overhead. Cheaply reject poor candidates, but label result as screening only.

**Representative worker:** more layers or actual inference calls, realistic expert skew, sparse indices, cache state and communication. Prefer an already loaded test engine only if the environment supports safe candidate switching.

**Full worker:** true end-to-end prefill + decode including routing, KV transport, scheduler and request mix. No partial model/engine timing can substitute for this acceptance gate.

## Concurrency and experiment scheduling

Agents can propose candidates in parallel, but serialize execution on shared GPU resources unless isolation and reliable accounting exist. Maintain an experiment ledger with owner, target commit, hardware reservation, input hash, start/end, candidate ID, status and result references. Keep the GPU busy with **verified** experiments rather than launching overlapping noisy tests.

Avoid confusing candidate build cache hits with model KV/prefix-cache hits; they affect different parts of the workflow.

## Protocol (if you implement your own runner)

1. Freeze a clean known-good baseline and input fixtures.
2. Build candidate in an isolated namespace; static check compile/import and signatures.
3. Drain or coordinate in-flight work, install candidate to all relevant ranks and update version ID.
4. Recapture affected graphs; assert graph and rank fingerprints.
5. Warm up before timing; compare identical representative workloads.
6. On error, roll back or restart; never reuse a context after device-side illegal access.
7. Use full-service replay to accept/reject and preserve logs/trace identifiers.

This is a design checklist, **not an API contract or a promise of a production-ready implementation**.
