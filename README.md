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
| [`log/`](log/) | Weekly Monday plan and weekend report, grouped by month |
| [`notes/`](notes/) | Concept notes written in my own words, one file per topic |
| [`templates/`](templates/) | Templates for a project, a note, and the weekly report |

**Weekly loop** (detail in the [roadmap](ROADMAP.md)):

1. **Monday:** write the Monday plan in `log/` from [`templates/weekly-report.md`](templates/weekly-report.md).
2. **Tuesday to Thursday:** about 1 hour a day on the week's sources and prototype.
3. **Weekend:** about 4 to 5 hours to finish the prototype, the home assignment, the work assignment (on the OMS; lesson only in this repo), and the knowledge check.
4. **Weekend report:** same log entry, same weekend. Hours, what was built, percentile numbers, knowledge-check score, work-assignment lesson, how the week went, carry-over. The week is not done until this exists.
5. Tick track checkboxes only when the week's "done means" in the roadmap are true.

---

## Rules

1. **Measure before claiming.** Every performance claim comes with hardware, kernel, compiler and the exact command, plus a full latency distribution (p50/p99/p99.9/max), not an average.
2. **Correct before fast.** Check against a reference implementation or model before any optimization.
3. **Break it on purpose.** Every system gets a fault-injection or crash test, not just a happy-path test.
4. **Write it down.** No project is done without a `RESULTS.md`: what was slow or fragile, why, what changed, the numbers. Weekly work is written in the weekend report.
5. **Employer material stays out.** FlexTrade code, specs, configs, traces and numbers never go in this public repository. Work assignments are for applying the week's idea on a trading OMS core (order state machines, gateways, sequence recovery, market data, hot-path latency). Only the lesson goes in `log/`.

Using AI without outsourcing understanding: predict first, quiz rather than lecture, audit generated scaffolds, explain tool output against a manual, critique notes after they are written from memory, never trust AI numbers without measuring. Full text: [`tracks/00-foundations.md`](tracks/00-foundations.md).

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

## Status

Breadth phase (one subject per week). Depth projects are chosen at week 13.

| Week | Monday | Subject | State |
|:--:|:--|:--|:--|
| 1 | 2026-11-23 | Timers and latency histograms (SI-01 start) | next |
| 2 | 2026-11-30 | CPU pipeline and branch prediction | |
| 3 | 2026-12-07 | Caches and prefetching | |
| 4 | 2026-12-14 | TLB and huge pages | |
| 5 | 2026-12-21 | Coherence and false sharing | |
| 6 | 2026-12-28 | Syscalls, scheduler and context switches | |
| 7 | 2027-01-04 | perf, profiling and flame graphs | |
| 8 | 2027-01-11 | GPU and CUDA intro | |
| 9 | 2027-01-18 | LLM inference: vLLM or llama.cpp | |
| 10 | 2027-01-25 | Linux receive path and kernel bypass intro | |
| 11 | 2027-02-01 | TLA+ on an order state machine | |
| 12 | 2027-02-08 | Storage and io_uring | |
| 13 | 2027-02-15 | Review and specialization decision | |

After week 13, replace this table with the chosen specialization and the SI projects in flight. Full project list: [projects/README.md](projects/README.md).

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
