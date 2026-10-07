# MoE-Perf

Repository for maintaining ideas about MoE's expert prefetch acceleration.

## Main Ideas We Want to Track

### Idea A: Predictive, Fine-Grained Expert Prefetch

Modern MoE models contain hundreds of experts, but each token activates only a small Top-K subset. Idea A investigates whether these future expert choices become **highly selective and predictable before the router officially resolves them**, creating an opportunity for speculative hardware prefetch.

Unlike prior expert-prediction work focused primarily on moving whole experts from CPU/SSD into GPU HBM, we first characterize how early and how narrowly the future expert set can be identified. We will measure:

- Top-Q recall
- Prediction entropy
- Top-K margins
- Speculation and overfetch amplification
- Prediction-to-use lead time

If a small candidate set (for example, K–2K experts) captures most future Top-K selections sufficiently early, the architecture can use this information to prepare expert execution before routing completes—for example, by preconstructing descriptors or priming selected HBM weight streams/tiles rather than fetching entire experts into a large on-chip buffer.

The central question is therefore not simply **“Can we predict experts?”**, which prior work has established, but **“Are expert selections sufficiently picky, early, and low-overfetch that fine-grained architectural speculation can hide the remaining route-to-execution latency?”**

#### Idea A2: Dense Static Caches

We also want to introduce additional memory types, such as NVMe, as much denser static caches and investigate when their presence becomes useful to this work. The goal is to understand how these memory tiers can complement GPU memory and speculative prefetching, and under what access, capacity, and latency conditions they provide meaningful architectural value.

### Idea B: Context Fabric for Semantic Dataflow Execution

Modern GPUs execute MoE through a control-flow-oriented sequence of routing, token dispatch/permutation, communication, grouped-GEMM construction, expert computation, activation, and combine operations. MoE, however, naturally represents a dynamic **dataflow graph** in which routed tokens become work for particular experts.

Idea B explores a custom **Context Fabric** architecture where the fundamental hardware object is a semantic context—containing information such as token/request ID, layer, expert ID, routing weight, dependencies, and destination—rather than a sequence of software-launched kernels and address-based transfers.

Contexts flow through the machine and directly trigger expert computation when their operands and resources become available, allowing batching, scheduling, communication, and execution to emerge from data availability rather than explicit control-flow orchestration.

The goal is not simply to accelerate All-to-All or routing—areas already addressed by systems such as MoE-Hub and MoEA—but to determine which stages of today's MoE pipeline exist primarily because GPUs are control-flow machines, and whether a persistent semantic dataflow substrate can **eliminate those stages altogether**. This is similar in philosophy to architectures such as PADE, which obtain their advantage by restructuring the execution pipeline rather than merely accelerating its existing components.
