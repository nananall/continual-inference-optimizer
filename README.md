# Continual Inference Optimizer

**A trace-driven Agent Skill for continual SGLang / vLLM inference optimization.**

Turn profiling evidence into validated code improvements through a repeatable loop:

**Measure → Locate bottleneck → Hypothesize → Implement → Screen → Validate → Keep or revert → Repeat**

This is an **independent, open-source skill inspired by** [Together Compiler: Continual Compilation for AI Inference](https://www.together.ai/blog/together-compiler-continual-compilation-for-ai-inference) and its [research paper](https://arxiv.org/abs/2611.14681). **It is not Together Compiler, not affiliated with Together AI, and does not include its proprietary persistent engine, hot-reload implementation, or cost model.** The skill describes how an agent should use tools available in your environment; it does not itself start a service or benchmark GPUs.

## What it does

- Discovers the **actual running code, model, layout, commands and traffic shape** before proposing optimizations.
- Identifies candidates from **trace evidence and source code**, not unsupported hardware guesses.
- Uses a **fast / representative / full end-to-end** measurement hierarchy.
- Reuses initialized engines or hot-reload **only when the local implementation demonstrably supports it**; otherwise restarts safely.
- Rejects changes that violate user-defined correctness, TTFT, TPOT, throughput, latency-tail, memory or layout constraints.
- Preserves reproducible evidence, cherry-pickable winners and rollback instructions.
- Treats traffic drift and changes in context length, cache-hit rate and MoE skew as reasons to re-evaluate prior winners.

## Layout

~~~text
.
├── SKILL.md
├── README.md
├── LICENSE
├── references/
│   ├── experiment-fidelity.md
│   ├── trace-and-bottleneck.md
│   ├── persistent-engine.md
│   ├── search-and-cost-model.md
│   └── correctness-and-rollback.md
├── templates/
│   └── experiment-record.md
└── examples/
    ├── sglang-pd-fixed-layout.md
    └── vllm-fixed-layout.md
~~~

The only mandatory entry point for an agent is [SKILL.md](SKILL.md); references are loaded as needed.

## Install

For Codex and other agents using the common Agent Skills directory:

~~~bash
git clone https://github.com/nananall/continual-inference-optimizer.git \
  ~/.agents/skills/continual-inference-optimizer
~~~

For Claude Code, clone under the current project's skills directory:

~~~bash
mkdir -p .claude/skills
git clone https://github.com/nananall/continual-inference-optimizer.git \
  .claude/skills/continual-inference-optimizer
~~~

If your agent uses another skills location, copy the repository directory into its supported location. Restart the agent/session if it caches skill discovery. The syntax for invoking a named skill varies by agent.

## Example prompt

~~~text
Use the continual-inference-optimizer skill to optimize my SGLang PD service.
Freeze the current P/D GPU counts and TP/EP/PP layout. Target both TTFT and
TPOT without throughput or correctness regression. Start by recovering the
exact running commit, launch commands, evaluation dataset and trace.
Use the existing benchmark runner and real request distributions. Explore
mechanism-level and kernel changes; screen cheaply, verify end-to-end, and
keep only improvements that satisfy my constraints. Return reproducible
commands, measured comparisons, trace links and a rollback plan.
~~~

See [SGLang PD example](examples/sglang-pd-fixed-layout.md) and [vLLM example](examples/vllm-fixed-layout.md).

## What this is / is not

| Supported workflow | Not supplied by this repository |
| --- | --- |
| Agent instructions and experiment discipline | A new SGLang/vLLM serving engine |
| Measurement, profiling and validation playbooks | Automatic access to your GPUs, logs or proprietary tools |
| Safe optional integration with engine reuse / hot swap | Together AI's hot-reload RPC or internal cost model |
| Configuration-search reasoning under explicit constraints | Guaranteed speedups or automatic production deployment |

Never invent a benchmark result. A kernel-only win is **not** an end-to-end service win. Fixed layout means fixed layout.

## References and attribution

- [Together Compiler — November 4, 2026](https://www.together.ai/blog/together-compiler-continual-compilation-for-ai-inference)
- [Together Compiler paper](https://arxiv.org/abs/2611.14681)
- [SGLang](https://github.com/sgl-project/sglang)
- [vLLM](https://github.com/vllm-project/vllm)

No source code or benchmark results from Together Compiler are redistributed here. MIT license; see [LICENSE](LICENSE).
