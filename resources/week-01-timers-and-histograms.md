# Week 1 · Timers and latency histograms

## Why it matters and where it is used

Every performance claim is a difference between two clock reads. If the clock read is slow, jittery, or drifting, every number after it is wrong. A 20 ns timer overhead is invisible on a 10 ms request, but it is half the measurement on a 40 ns cache miss.

This shows up in several places:

- **Trading.** Tick-to-trade is a chain of timestamps from NIC to order out. Use TSC for in-process stamps, and hardware or PTP timestamps across machines. MiFID II requires HFT clocks synced to UTC within 100 microseconds.
- **OMS.** Latency dashboards and SLAs depend on how timestamps were taken and turned into percentiles.
- **AI infra.** Serving measures time to first token and time per output token at p50 and p99. GPU kernels are timed with CUDA events.
- **Profilers.** Tools such as perf, eBPF, and flame graphs rest on timestamps and counters.
- **Distributed systems.** Clock sync (NTP, PTP) or logical clocks are needed to order events.

### Why histograms and percentiles

Latency is a distribution, not one number. Each order lives through its own latency, so the average describes no real order. Slow cases have real causes, like page faults, interrupts, a lock, a log flush, or a preemption. They bunch up under load, which is exactly when speed matters most. A percentile tells you what fraction of requests did worse than a given value. Max tells you the worst thing that actually happened. There is also a trap: if the test driver waits for each reply before it sends the next request, a stall also hides the requests that should have arrived during it. That is coordinated omission, and it makes p99 look better than what users saw.

This shows up in several places:

- **Trading.** Tick-to-trade budgets are set at p99, p99.9, and max, because the trades you lose sit in the tail during market bursts.
- **OMS.** A client sees one slow ack, not the mean. With several hops (gateway, validation, risk, router), each hop adds another chance to hit a tail.
- **AI infra.** Serving targets are set on p50 and p99 of time to first token and time per output token. In synchronous data-parallel training, every step waits for the slowest GPU, so one worker's tail sets the step time for all of them.
- **At scale.** Dean and Barroso at Google showed that if a server responds in 10 ms but takes 1 second at p99, then a request that fans out to 100 such servers takes more than 1 second 63% of the time. At scale, tails decide what users see, not averages.

**Before reading:** write down a guess for how many cycles back-to-back `rdtsc` takes, then run `timers.cpp` and compare.

## Sources (reading order)

Start with the Intel paper and Gregg's post.

1. [How to Benchmark Code Execution Times on Intel IA-32 and IA-64 Instruction Set Architectures](https://www.intel.com/content/dam/www/public/us/en/documents/white-papers/ia-32-ia-64-benchmark-code-execution-paper.pdf) (Intel). Primary source on `rdtsc`/`rdtscp`, serialization, and measuring the measurement overhead. About 30 minutes.

2. [The Speed of Time](https://www.brendangregg.com/blog/2021-09-26/the-speed-of-time.html) (Brendan Gregg). Production case where the clocksource made `clock_gettime` slow. About 15 minutes.

3. [`vdso(7)`](https://man7.org/linux/man-pages/man7/vdso.7.html) and [`clock_gettime(2)`](https://man7.org/linux/man-pages/man2/clock_gettime.2.html). Why reading time skips the syscall, and how `CLOCK_MONOTONIC` differs from `CLOCK_MONOTONIC_RAW`. Short man pages.

4. [How NOT to Measure Latency](https://www.youtube.com/watch?v=lJ8ydIuPFeU) (Gil Tene). Averages lie, and coordinated omission hides stall time. About 1 hour.

5. [The Tail at Scale](https://cacm.acm.org/research/the-tail-at-scale/) (Jeffrey Dean and Luiz Andre Barroso, Communications of the ACM 56, 2013; also at [research.google](https://research.google/pubs/the-tail-at-scale/)). Why tail latency, not the average, sets user-visible latency once requests fan out. A 10 ms typical response and 1 s at p99, fanned out to 100 servers, gives 63% of requests over 1 s. About 20 minutes for the first two sections.

6. [HdrHistogram_c](https://github.com/HdrHistogram/HdrHistogram_c). Histogram library for [SI-01](../projects/README.md#si-01-benchkit).

7. [Performance Analysis and Tuning on Modern CPUs](https://github.com/dendibakh/perf-book) (Denis Bakhvalov, perf-book). Measurement chapter: warm-up, frequency scaling, and noise.
