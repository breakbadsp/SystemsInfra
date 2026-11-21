# SystemsInfra

Learning log and project workspace for one goal: **become an expert in systems infrastructure**, with the emphasis on **performance, reliability and robustness**, across three domains:

- **Trading and HFT infra:** market data, order paths, kernel bypass, tick-to-trade latency
- **AI infra:** GPU kernels, inference engines, distributed serving
- **The shared core:** CPUs, memory, OS, networking, storage, consensus, measurement

These domains are more alike than different. Each comes down to moving bytes through a hierarchy with bounded tail latency, without losing data when things fail. The roadmap builds the shared core once and applies it to each domain.

---

## How this repo works

| Folder | What goes there |
|:--|:--|
| [`ROADMAP.md`](ROADMAP.md) | The 18-month plan: which track and project each quarter |
| [`tracks/`](tracks/) | One file per learning track: topics as checklists, resources, exit criteria |
| [`projects/`](projects/) | Project catalog plus one folder per project (code, benchmarks, write-up) |
| [`log/`](log/) | Monthly log, one short entry per week: hours, learned, built, numbers |
| [`notes/`](notes/) | Concept notes written in my own words, one file per topic |
| [`templates/`](templates/) | Templates for a new project, a note and a log entry |

**Weekly loop:** pick the next unchecked item in the current track → study → apply it in the current project → measure → write a log entry → tick the checkbox.

## Rules

1. **Measure before claiming.** Every performance claim comes with hardware, kernel, compiler and the exact command, plus a full latency distribution (p50/p99/p99.9/max), not an average.
2. **Correct before fast.** Check against a reference implementation or model before any optimization.
3. **Break it on purpose.** Every system gets a fault-injection or crash test, not just a happy-path test.
4. **Write it down.** No project is done without a `RESULTS.md`: what was slow or fragile, why, what changed, the numbers.

## Tracks

| # | Track | Domain |
|:--:|:--|:--|
| 0 | [Foundations](tracks/00-foundations.md) | CPU, memory, OS, networking, C++/Rust |
| 1 | [Performance engineering](tracks/01-performance.md) | Measurement, profiling, benchmarking, tuning |
| 2 | [Reliability and correctness](tracks/02-reliability.md) | Consensus, deterministic simulation, fault injection, verification |
| 3 | [Low-latency and HFT infra](tracks/03-low-latency-hft.md) | Market data, kernel bypass, time sync, tick-to-trade |
| 4 | [AI infra](tracks/04-ai-infra.md) | GPUs, CUDA, inference engines, distributed serving |
| 5 | [Storage and distributed systems](tracks/05-storage-distributed.md) | Storage engines, io_uring, durability, replication |

## Status

| Quarter | Focus | Projects | State |
|:--|:--|:--|:--|
| Q1 · Dec 2026 – Feb 2027 | Measurement core | SI-01, SI-02, SI-03 | ▶ next |
| Q2 · Mar – May 2027 | GPUs and CUDA | SI-04, SI-05 | |
| Q3 · Jun – Aug 2027 | Low-latency trading | SI-06, SI-07, SI-08 | |
| Q4 · Sep – Nov 2027 | Inference engine | SI-09 | |
| Q5 · Dec 2027 – Feb 2028 | Reliability and storage | SI-10, SI-11, SI-12 | |
| Q6 · Mar – May 2028 | Distributed serving | SI-13, SI-14 | |

Full project list: [projects/README.md](projects/README.md).

## Prior work this builds on

| Repo | Relevant to |
|:--|:--|
| [modern_exchng_cpp](https://github.com/breakbadsp/modern_exchng_cpp) | Order path, epoll/io_uring gateway, SPSC rings, pinned engines |
| [libvsr](https://github.com/breakbadsp/libvsr) | Viewstamped Replication, deterministic simulation |
| [triangulator](https://github.com/breakbadsp/triangulator) | Thread-level observability for any Linux process |
| [spmalloc](https://github.com/breakbadsp/spmalloc) · [sp](https://github.com/breakbadsp/sp) | Allocators, lock-free primitives |
| [1brc](https://github.com/breakbadsp/1brc) · [MktDataAggregator](https://github.com/breakbadsp/MktDataAggregator) | mmap, parallel parsing, large-file throughput |
| [fixsim](https://github.com/breakbadsp/fixsim) · [FIXTester](https://github.com/breakbadsp/FIXTester) | FIX order entry |
