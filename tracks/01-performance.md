# Track 1: Performance engineering

How to measure, find the bottleneck and prove an improvement. This is the method used across every other track.

The live week-by-week plan is [`ROADMAP.md`](../ROADMAP.md). Week 1 starts SI-01 (timers and histograms). Week 7 is `perf` and flame graphs. Remaining items are for after week 13, in whichever specialization needs them. Keep ticking checkboxes here.

## Measurement

- [ ] Timers: `rdtsc`/`rdtscp`, invariant TSC, calibration, `clock_gettime` cost
- [ ] Latency histograms: HdrHistogram, percentiles, why averages lie
- [ ] Coordinated omission and how to correct for it in load generators
- [ ] Open-loop vs closed-loop load; Poisson arrivals
- [ ] Benchmark hygiene: warm-up, CPU frequency pinning, isolation, run-to-run variance, confidence intervals
- [ ] Microbenchmark pitfalls: dead-code elimination, `DoNotOptimize`, measuring the harness instead of the code

## Profiling

- [ ] `perf stat`, `perf record`, `perf c2c` (false sharing), `perf mem`
- [ ] Hardware counters via `perf_event_open` directly from code
- [ ] Flame graphs, off-CPU analysis, wakeup chains
- [ ] eBPF: `bpftrace`, `bcc` tools, writing a small custom probe
- [ ] Intel VTune / AMD uProf top-down analysis
- [ ] GPU: Nsight Systems and Nsight Compute (used in track 4)
- [ ] Thread-level view with triangulator alongside the above

## Analysis

- [ ] Roofline model for CPU and GPU
- [ ] Little's law, queueing theory basics (M/M/1, utilization vs latency curve)
- [ ] Amdahl and the Universal Scalability Law; fitting USL to measured scaling
- [ ] Tail latency causes: GC/allocation, page faults, IRQs, SMIs, frequency changes, lock convoys

## Tuning

- [ ] Data layout: AoS vs SoA, hot/cold splitting, padding
- [ ] Allocation: arenas, pools, slab allocators, comparison of jemalloc / mimalloc / tcmalloc
- [ ] Batching against latency, adaptive batching
- [ ] Profile-guided optimization, BOLT

## Resources

- ★ Brendan Gregg, *Systems Performance* (2nd ed.) and *BPF Performance Tools*
- ★ Gil Tene, "How NOT to Measure Latency" (talk)
- Denis Bakhvalov's easyperf blog
- Neil Gunther, *Guerrilla Capacity Planning* (USL)
- Martin Thompson's Mechanical Sympathy blog

## Exit criteria

- [ ] SI-01 used by at least three other projects
- [ ] One write-up that diagnoses a real tail-latency problem end to end (in any project or at work)
