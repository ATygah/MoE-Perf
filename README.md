# MoE-Perf

This repository tracks architecture research ideas for improving Mixture-of-Experts (MoE) system performance, with emphasis on reducing route-to-execution latency and rethinking MoE execution beyond control-flow GPU pipelines.

## Purpose

Modern MoE models activate only a small Top-K subset of experts per token out of a much larger expert pool. This repo is used to:
- Define and refine research hypotheses
- Track progress and evaluation milestones
- Capture architectural implications for future systems

## Idea A — Early, Selective Expert Prediction for Fine-Grained Speculation

Modern MoE models contain hundreds of experts, but each token activates only a small Top-K subset. Idea A investigates whether future expert choices become highly selective and predictable before the router officially resolves them, creating an opportunity for speculative hardware prefetch.

Unlike prior expert-prediction work focused mainly on moving whole experts from CPU/SSD into GPU HBM, this direction first characterizes how early and how narrowly the future expert set can be identified. Key measurements include:
- Top-Q recall
- Prediction entropy
- Top-K margins
- Speculation/overfetch amplification
- Prediction-to-use lead time

If a small candidate set (e.g., K–2K experts) captures most future Top-K selections sufficiently early, hardware could prepare expert execution before routing completes (for example, preconstructing descriptors or priming selected HBM weight streams/tiles rather than fetching entire experts into a large on-chip buffer).

The central question is not just “Can we predict experts?” but whether expert selections are sufficiently picky, early, and low-overfetch for fine-grained architectural speculation to hide the remaining route-to-execution latency.

## Idea A2 — Expert Prediction Under Alternate Memory Hierarchies

Idea A2 extends Idea A by changing the available memory hierarchy (for example, introducing NVMe-backed capacity tiers) and evaluating whether denser memory options are worthwhile given prediction quality, latency, bandwidth, and amplification effects.

## Idea B — Context Fabric for Semantic MoE Dataflow Execution

Modern GPUs execute MoE through a control-flow-oriented sequence: routing, token dispatch/permutation, communication, grouped-GEMM construction, expert computation, activation, and combine operations. Idea B explores a custom Context Fabric architecture where the fundamental hardware object is a semantic context (token/request ID, layer, expert ID, routing weight, dependencies, destination), not software-launched kernels and address-based transfers.

Contexts flow through the machine and directly trigger expert computation when operands/resources are available, so batching, scheduling, communication, and execution emerge from data availability rather than explicit control-flow orchestration.

The goal is not merely to accelerate routing or All-to-All (already addressed by systems such as MoE-Hub and MoEA), but to identify which current pipeline stages exist primarily because GPUs are control-flow machines and whether a persistent semantic dataflow substrate can eliminate those stages altogether (in the spirit of pipeline-restructuring architectures such as PADE).

## Progress Tracking

- [ ] Idea A: Define baseline workload(s), router checkpoints, and measurement protocol
- [ ] Idea A: Quantify early predictability (Top-Q recall, entropy, Top-K margins)
- [ ] Idea A: Measure overfetch amplification and prediction-to-use lead time
- [ ] Idea A: Evaluate speculative execution preparation strategies and latency hiding
- [ ] Idea A2: Model alternate memory tiers (including NVMe) and access paths
- [ ] Idea A2: Compare cost/performance trade-offs versus baseline hierarchy
- [ ] Idea B: Define semantic context format and lifecycle across MoE pipeline
- [ ] Idea B: Map current GPU MoE stages to context-triggered dataflow equivalents
- [ ] Idea B: Identify/eliminate control-flow-only stages and quantify impact
