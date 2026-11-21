# Roadmap

18 months at about 8–10 hours a week. One **primary track** per quarter, with project work in that track. Reliability (track 2) runs through every quarter: each project gets a fault or crash test, and Q5 goes deep on it.

Quarters alternate between trading and AI so neither goes stale, and the measurement core comes first because every later project reuses it.

---

## Q1 · Dec 2026 – Feb 2027: Measurement core

**Tracks:** [0 Foundations](tracks/00-foundations.md), [1 Performance](tracks/01-performance.md)

| Project | Weeks | Why now |
|:--|:--:|:--|
| [SI-01 benchkit](projects/README.md#si-01-benchkit) | 3 | Shared latency/throughput harness every later project uses |
| [SI-02 memlab](projects/README.md#si-02-memlab) | 4 | Measured cache, TLB, NUMA and false-sharing numbers for my own machines |
| [SI-03 jitterlab](projects/README.md#si-03-jitterlab) | 3 | OS jitter, and the measured effect of each Linux tuning knob |

**Exit:** I can say, with numbers from my own hardware, what an L1/L2/L3/DRAM access, a context switch, a syscall, a cross-core cache-line transfer and an IRQ cost.

## Q2 · Mar – May 2027: GPUs and CUDA

**Track:** [4 AI infra](tracks/04-ai-infra.md), phases A–B

| Project | Weeks | |
|:--|:--:|:--|
| Model internals + C++ CPU forward pass | 3 | Phase A of the track |
| [SI-04 SGEMM ladder](projects/README.md#si-04-sgemm-ladder) | 4 | ≥ 80% of cuBLAS |
| [SI-05 fused kernels](projects/README.md#si-05-fused-kernels) | 5 | Softmax, RMSNorm, FlashAttention-style forward pass |

**Exit:** I can read a Nsight Compute report and name the limiter, and SI-04 reaches 80% of cuBLAS.

## Q3 · Jun – Aug 2027: Low-latency trading

**Track:** [3 Low-latency and HFT](tracks/03-low-latency-hft.md)

| Project | Weeks | |
|:--|:--:|:--|
| [SI-06 ITCH feed handler](projects/README.md#si-06-itch-feed-handler) | 4 | Full-day replay, book per symbol, ns/msg |
| [SI-07 kernel-bypass receive](projects/README.md#si-07-kernel-bypass-receive) | 4 | Kernel socket → busy poll → AF_XDP → DPDK |
| [SI-08 tick-to-trade harness](projects/README.md#si-08-tick-to-trade-harness) | 5 | Timestamps at every hop, end to end |

**Exit:** a measured tick-to-trade distribution, with a breakdown of where each microsecond goes.

## Q4 · Sep – Nov 2027: Inference engine

**Track:** [4 AI infra](tracks/04-ai-infra.md), phase C

| Project | Weeks | |
|:--|:--:|:--|
| [SI-09 spinfer](projects/README.md#si-09-spinfer) | 12 | C++/CUDA engine: paged KV, continuous batching, quantization |

**Exit:** `spinfer` has a published comparison against llama.cpp and vLLM on the same GPU.

## Q5 · Dec 2027 – Feb 2028: Reliability and storage

**Tracks:** [2 Reliability](tracks/02-reliability.md), [5 Storage](tracks/05-storage-distributed.md)

| Project | Weeks | |
|:--|:--:|:--|
| [SI-10 Jepsen-style checker](projects/README.md#si-10-jepsen-style-checker) | 5 | Linearizability checker and fault nemesis, aimed at libvsr and the exchange |
| [SI-11 storage engine](projects/README.md#si-11-storage-engine) | 6 | WAL + LSM on io_uring, crash-consistency tests |
| [SI-12 upstream perf PR](projects/README.md#si-12-upstream-perf-pr) | ongoing | A merged PR in vLLM / SGLang / llama.cpp |

**Exit:** SI-10 has found a real bug, or shown none under a documented fault model. SI-11 survives the crash test suite.

## Q6 · Mar – May 2028: Distributed serving

**Track:** [4 AI infra](tracks/04-ai-infra.md), phase D

| Project | Weeks | |
|:--|:--:|:--|
| [SI-13 tensor parallelism](projects/README.md#si-13-tensor-parallelism) | 5 | `spinfer` across 2–4 GPUs |
| [SI-14 cache-aware router](projects/README.md#si-14-cache-aware-router) | 6 | Prefix-affinity routing, failover, fault tests |

**Exit:** measured scaling efficiency for SI-13; SI-14 beats round-robin and survives losing a replica.

---

## Adjusting

- **Reorder quarters freely.** Only SI-01 is a hard prerequisite; SI-09 needs SI-04/05, and SI-13/14 need SI-09.
- **Slipping is fine; skipping the write-up is not.** Better to finish fewer projects with `RESULTS.md` than many without.
- **Review each quarter:** update the status table in the [README](README.md), and move anything unfinished.
