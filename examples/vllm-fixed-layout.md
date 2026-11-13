# Example: vLLM on non-NVIDIA hardware, fixed TP layout

~~~text
Use the continual-inference-optimizer skill to improve TTFT in our
vLLM serving stack. Prefill TP=2 and Decode TP=2 are fixed.

Do not change deployment layout, model semantics, output correctness,
or reduce achieved throughput or worsen TPOT. Recover the exact running
commit, compatible device backend, startup arguments, baseline results
and profiler traces first.

Inspect router and prefill queueing, tokenization, chunking,
attention/MoE operator shapes, inter-rank collectives, KV transfer,
host synchronization and decode backpressure. Prioritize trace-backed
candidates. Do not assume NVIDIA-specific utilities or kernel APIs
are available on this GPU; inspect the actual backend.

Use realistic traffic, preserve cache conditions, validate both fast
and full-service tests, and report exact reproducible commands and
measured comparisons. Mark missing tests as NOT RUN.
~~~

When CUDA-specific advice is not applicable, use backend-supported profiling and kernels rather than silently translating CUDA behavior into hardware claims.
