# Projects

Fourteen projects. Each gets its own folder here (`projects/si-NN-name/`) when it starts, created from [the project template](../templates/project.md). Code, benchmarks, a `RESULTS.md` and a fault/crash test live in that folder.

Follow the [roadmap](../ROADMAP.md) for the breadth labs. In week 13, choose one specialization and its first depth project. Keep the other projects optional.

**Status:** not started · in progress · done

| ID | Project | Track | When it fits | Status |
|:--|:--|:--:|:--|:--|
| SI-01 | [benchkit](#si-01-benchkit) | 1 | Breadth week 1 (required). Finish remaining features when a depth project needs them. | not started |
| SI-02 | [memlab](#si-02-memlab) | 0 | Started in the breadth memory lab. Finish in any specialization that still needs machine numbers. | not started |
| SI-03 | [jitterlab](#si-03-jitterlab) | 0, 3 | Low-latency trading depth. Useful for AI host-thread tails. | not started |
| SI-04 | [SGEMM ladder](#si-04-sgemm-ladder) | 4 | AI inference depth. Week 8 is only a toy kernel. | not started |
| SI-05 | [Fused kernels](#si-05-fused-kernels) | 4 | AI inference depth, after SI-04. | not started |
| SI-06 | [ITCH feed handler](#si-06-itch-feed-handler) | 3 | Low-latency trading depth. | not started |
| SI-07 | [Kernel-bypass receive](#si-07-kernel-bypass-receive) | 3 | Low-latency trading depth, after SI-06. Week 10 is kernel path plus AF_XDP reading only. | not started |
| SI-08 | [Tick-to-trade harness](#si-08-tick-to-trade-harness) | 3 | Low-latency trading depth, after SI-06 and SI-07. | not started |
| SI-09 | [spinfer](#si-09-spinfer) | 4 | AI inference depth, after SI-04/05. Week 9 only runs vLLM or llama.cpp. | not started |
| SI-10 | [Jepsen-style checker](#si-10-jepsen-style-checker) | 2 | Reliability depth. Pairs with trading (exchange, libvsr) or AI (SI-14). | not started |
| SI-11 | [Storage engine](#si-11-storage-engine) | 5, 2 | Reliability depth. Optional for trading sequence store or AI checkpoint durability. The breadth lab covers restart recovery. | not started |
| SI-12 | [Upstream perf PR](#si-12-upstream-perf-pr) | 4 | AI inference depth, once you can profile a real engine (after week 9, stronger after SI-09). | not started |
| SI-13 | [Tensor parallelism](#si-13-tensor-parallelism) | 4 | AI serving depth, after SI-09. | not started |
| SI-14 | [Cache-aware router](#si-14-cache-aware-router) | 4, 2 | AI serving plus reliability, after SI-09. | not started |

Dependencies: SI-01 before serious measurement · SI-04/05 before SI-09 · SI-09 before SI-12, SI-13 · SI-13 before SI-14 if the router fronts spinfer · SI-06 before SI-07 before SI-08.

## Specializations

| Specialization | Primary projects | Optional |
|:--|:--|:--|
| Low-latency trading | SI-02 (finish), SI-03, SI-06, SI-07, SI-08 | The AI catalog. SI-10 if replication matters. SI-11 if durable sequence store matters. |
| AI inference and serving | SI-04, SI-05, SI-09, SI-12, then SI-13, SI-14 | The trading catalog. SI-03 if host tails matter. SI-11 if durability matters. |
| Reliability | SI-10, SI-11, further TLA+ | GPU kernels. Feed handler unless it is the system under test. |

---

## SI-01 benchkit

A small C++ library and CLI for latency and throughput measurement, used by every later project.

- `rdtscp` timer with calibration and invariant-TSC check; `clock_gettime` fallback
- HdrHistogram-style recorder; p50/p90/p99/p99.9/p99.99/max output; merge across threads
- Open-loop load generator with Poisson arrivals and coordinated-omission correction
- Hardware counters through `perf_event_open` (cycles, instructions, cache and branch misses) around a measured region
- Environment capture: CPU model, frequency governor, kernel, isolation flags, compiler, written into every result file

**Acceptance**
- [ ] Measurement overhead documented (cost of one timed region)
- [ ] Validated against a known-latency workload (e.g. a calibrated spin)
- [ ] Used to re-measure one existing repo (modern_exchng_cpp or libvsr)

## SI-02 memlab

Microbenchmarks that produce "latency numbers" for my own machines.

- Pointer-chasing latency across working-set sizes → L1/L2/L3/DRAM curve, with and without huge pages
- Bandwidth: sequential read, write, copy per core and all cores (STREAM-like)
- Cache-line ping-pong between core pairs → core-to-core latency matrix, same and cross socket
- False sharing demo with `perf c2c` confirmation
- NUMA local vs remote; prefetcher on/off where the BIOS allows

**Acceptance**
- [ ] Plots and tables for at least two machines
- [ ] Each number explained against vendor specs

## SI-03 jitterlab

Measure OS-induced jitter and the effect of each tuning step.

- A `sysjitter`/`hiccups`-style tool: tight loop on an isolated core recording gaps above a threshold
- Apply one change at a time: `isolcpus`, `nohz_full`, `rcu_nocbs`, IRQ affinity, C-states off, THP off, turbo fixed, SMT off
- Correlate gaps with interrupts (`/proc/interrupts`, `bpftrace`)

**Acceptance**
- [ ] Before/after gap histogram for every knob
- [ ] A tuning playbook ranked by measured effect, reused in SI-07/08

## SI-04 SGEMM ladder

Hand-written FP32 matmul, optimized step by step.

Steps: naive → coalesced → shared-memory tiling → 1D/2D register blocking → vectorized loads → warp tiling → `cp.async` double buffering → stretch: FP16/BF16 tensor cores (WMMA/MMA) vs cuBLAS HGEMM.

**Acceptance**
- [ ] Correct against cuBLAS for square and non-square shapes
- [ ] ≥ 80% of cuBLAS SGEMM at M=N=K=4096
- [ ] GFLOP/s and Nsight Compute limiter for every step
- [ ] Write-up of each step in roofline terms

## SI-05 Fused kernels

In CUDA and in Triton: online softmax, RMSNorm, fused RoPE, FlashAttention-style forward pass, flash-decoding for long KV caches.

**Acceptance**
- [ ] Matches PyTorch within tolerance in FP16/BF16
- [ ] Attention memory linear in sequence length
- [ ] Bandwidth-bound kernels at ≥ 70% of peak memory bandwidth
- [ ] CUDA vs Triton: code size, time to write, speed

## SI-06 ITCH feed handler

Decode a full day of NASDAQ TotalView-ITCH 5.0 and build L3 order books for every symbol.

- Zero-copy parser over an `mmap`ed file; message dispatch without virtual calls
- Order book with O(1) add/cancel/execute by order ID, cache-friendly price levels
- Per-message latency histogram (SI-01), throughput in messages/s
- Replay at recorded rate and at maximum rate over UDP multicast (feeds SI-07)
- Gap detection and recovery on the multicast replay

**Acceptance**
- [ ] Book state matches a reference (published end-of-day stats or a second implementation) at checkpoints
- [ ] Full day processed, with ns/msg p50/p99/max
- [ ] Fault test: inject drops and reordering in the replay; recovery verified

## SI-07 Kernel-bypass receive

The same multicast feed, received four ways, with the latency distribution of each.

1. Standard kernel UDP socket
2. Kernel socket with busy polling and tuned IRQs (SI-03 playbook)
3. AF_XDP (zero-copy where the driver allows)
4. DPDK (or ef_vi on Solarflare/AMD hardware)

Measured with hardware timestamps where the NIC supports them, from wire to application.

**Acceptance**
- [ ] Wire-to-app p50/p99/p99.9 for each mode, same hardware
- [ ] Cost of each mode documented: CPU usage, complexity, operational constraints

## SI-08 Tick-to-trade harness

End to end: feed (SI-06/07) → book → strategy stub → risk check → order gateway (OUCH or FIX via fixsim) → wire.

- Timestamp at every hop; correlate across hosts with PTP (linuxptp) if two machines are available
- Mock exchange that echoes orders and timestamps receipt

**Acceptance**
- [ ] Per-hop latency breakdown with p50/p99
- [ ] One optimization that measurably cuts p99, with before/after
- [ ] Fault test: gateway disconnect, feed gap, slow consumer; behaviour documented

## SI-09 spinfer

A C++/CUDA LLM inference engine for Llama-family models, built like modern_exchng_cpp: fixed pools, pinned threads, measured end to end.

```
HTTP (OpenAI-compatible) → SPSC request queue → pinned scheduler thread
    → batch builder → GPU worker (CUDA graphs) → detokenize → stream
                    ↘ block allocator (paged KV) ↗
```

Milestones: batch-1 generation → paged KV → continuous batching → chunked prefill → INT8/INT4 weight-only quantization → prefix caching → CUDA graphs → load generator (SI-01).

**Acceptance**
- [ ] Llama-3.2-1B (and one 7 to 8B model on a large enough GPU) with greedy output matching the reference
- [ ] TTFT, TPOT, tokens/s, p50/p99 against request rate
- [ ] Comparison table against llama.cpp and vLLM on the same GPU and workload
- [ ] KV memory waste below 5% under mixed-length load
- [ ] Monitored with triangulator under load

## SI-10 Jepsen-style checker

A test harness that finds consistency bugs in replicated systems, aimed first at libvsr and the replicated modern_exchng_cpp.

- History recorder for client operations (invoke/ok/fail/info)
- Linearizability checker (Wing & Gong with Porcupine-style optimizations) in C++
- Nemesis: partitions (network namespaces, iptables), process kill/pause, `tc netem` delay/loss, clock skew
- Deterministic replay of a failing schedule where possible

**Acceptance**
- [ ] Checker validated on known-good and known-bad histories
- [ ] Run against libvsr under every nemesis; any bug found is reported with a reproducer
- [ ] Documented fault model and what was not tested

## SI-11 Storage engine

A key-value storage engine: WAL + LSM tree on io_uring.

- WAL with group commit, checksums and recovery
- Memtable → SSTables, leveled compaction, bloom filters, block cache
- io_uring with registered buffers; comparison against `pread`/`pwrite` and `O_DIRECT`
- Crash-consistency suite: kill at random points, `dm-flakey` and simulated power loss; recover and check against a model

**Acceptance**
- [ ] Recovery correct across thousands of random crash points
- [ ] Throughput and p99 for fill, read-random and mixed workloads; fsync cost measured
- [ ] Comparison against RocksDB on the same workload (not to win, to understand the gap)

## SI-12 Upstream perf PR

Land a measured performance improvement in vLLM, SGLang or llama.cpp.

1. Build from source; pin a benchmark on fixed hardware
2. Profile GPU (Nsight Systems) and CPU side (`py-spy`, `perf`, triangulator)
3. Pick an open perf issue or a measured hot spot; reproduce, fix, measure
4. Submit, respond to review, write it up

**Acceptance**
- [ ] At least one merged PR with before/after numbers
- [ ] Public write-up with profiles

## SI-13 Tensor parallelism

Megatron-style tensor parallelism in spinfer: column-parallel QKV/up, row-parallel output/down, one all-reduce per block. NCCL for collectives, plus a hand-written ring all-reduce for understanding.

**Acceptance**
- [ ] Output identical to single GPU
- [ ] Scaling efficiency for 1 → 2 (→ 4) GPUs, with communication share per step
- [ ] `nccl-tests` bandwidth on the rented node vs what spinfer achieves

## SI-14 Cache-aware router

A router in front of several spinfer (or vLLM) replicas.

- Policies: round-robin, least-outstanding-tokens, prefix-cache affinity, hybrid
- Health checks, draining, failover
- Stretch: router metadata replicated with libvsr; deterministic simulation of replica and network faults

**Acceptance**
- [ ] Prefix-affinity beats round-robin on TTFT and throughput for a shared-system-prompt workload
- [ ] Losing a replica loses no requests beyond those in flight
- [ ] Fault-injection test suite (reuse SI-10 nemesis)
