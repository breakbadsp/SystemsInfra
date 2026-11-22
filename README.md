# SystemsInfra

Learning log and project workspace for one goal: **become an expert in systems infrastructure**, with the emphasis on **performance, reliability and robustness**, across three domains:

- **Trading and HFT infra:** market data, order paths, kernel bypass, tick-to-trade latency
- **AI infra:** GPU kernels, inference engines, distributed serving
- **The shared core:** CPUs, memory, OS, networking, storage, consensus, measurement

These domains are more alike than different. Each comes down to moving bytes through a hierarchy with bounded tail latency, without losing data when things fail. The roadmap covers the shared map first, then goes deep in one or two areas.

Owner: Sachin. Background: senior C++ on a sell-side OMS core. Time: about 8 to 10 hours a week.

---

## How this repo works

| Folder | What goes there |
|:--|:--|
| [`ROADMAP.md`](ROADMAP.md) | The live plan: one subject per week, then a specialization choice at week 13 |
| [`tracks/`](tracks/) | Topic catalog: checklists, resources, exit criteria. Reference while doing the weekly plan |
| [`projects/`](projects/) | Project catalog (SI-01 to SI-14) plus one folder per project when it starts |
| [`log/`](log/) | Weekly entries grouped by month |
| [`notes/`](notes/) | Concept notes written in my own words, one file per topic |
| [`templates/`](templates/) | Templates for projects, notes, and weekly entries |

Follow the [weekly workflow](ROADMAP.md#weekly-workflow) and keep one [entry](templates/weekly-report.md) per week.

---

## Rules

1. **Make measurements reproducible.** Record the workload, environment, exact command, and metrics that fit the claim. For latency distributions, record sample count and enough samples to support the reported percentiles.
2. **Check correctness first.** Use a reference, a model, or an observable invariant.
3. **Test failure behavior.** When the system has a failure mode, inject a fault and check the expected result.
4. **Record the evidence.** Use the weekly entry for labs and `RESULTS.md` for completed projects.
5. **Keep employer material private.** Use public or synthetic inputs for repository work. If you apply a lesson at work, record only the general lesson here. Keep employer code, specifications, configurations, traces, and measurements private.

---

## Tracks

| # | Track | Domain |
|:--:|:--|:--|
| 0 | [Foundations](tracks/00-foundations.md) | CPU, memory, OS, networking, C++/Rust |
| 1 | [Performance engineering](tracks/01-performance.md) | Measurement, profiling, benchmarking, tuning |
| 2 | [Reliability and correctness](tracks/02-reliability.md) | Consensus, deterministic simulation, fault injection, verification |
| 3 | [Low-latency and HFT infra](tracks/03-low-latency-hft.md) | Market data, kernel bypass, time sync, tick-to-trade |
| 4 | [AI infra](tracks/04-ai-infra.md) | GPUs, CUDA, inference engines, distributed serving |
| 5 | [Storage and distributed systems](tracks/05-storage-distributed.md) | Storage engines, io_uring, durability, replication |

Tracks are the catalog. The sequence is the [roadmap](ROADMAP.md).

---

## Prior work this builds on

| Repo | Relevant to |
|:--|:--|
| [modern_exchng_cpp](https://github.com/breakbadsp/modern_exchng_cpp) | Order path, epoll/io_uring gateway, SPSC rings, pinned engines |
| [libvsr](https://github.com/breakbadsp/libvsr) | Viewstamped Replication, deterministic simulation |
| [triangulator](https://github.com/breakbadsp/triangulator) | Thread-level observability for any Linux process |
| [spmalloc](https://github.com/breakbadsp/spmalloc) · [sp](https://github.com/breakbadsp/sp) | Allocators, lock-free primitives |
| [1brc](https://github.com/breakbadsp/1brc) · [MktDataAggregator](https://github.com/breakbadsp/MktDataAggregator) | mmap, parallel parsing, large-file throughput |
| [fixsim](https://github.com/breakbadsp/fixsim) · [FIXTester](https://github.com/breakbadsp/FIXTester) | FIX order entry |
