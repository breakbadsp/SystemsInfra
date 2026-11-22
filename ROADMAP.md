# Roadmap

Breadth first, then depth. About 8 to 10 hours a week. One subject per week.

Weeks 1 to 12 cover the whole map at light depth: enough to measure the effect on a Linux box, not enough to finish the project catalog. Week 13 is the decision point. After that, pick one or two specializations and go deep using the existing projects (SI-01 through SI-14). Unpicked projects are optional. Leaving them unbuilt is not a failure.

[`tracks/`](tracks/) and [`projects/README.md`](projects/README.md) stay as the topic and project catalog. Tick checkboxes there as weeks complete. This file is the sequence to follow.

SI-01 benchkit starts in week 1 because every later prototype reuses a timer and a latency histogram.

The plan starts the week of Monday 2026-11-23.

---

## Why this shape

The old plan was one deep track per quarter for 18 months. That delays GPUs, inference, kernel bypass and TLA+ until much later, and makes it hard to know which area is worth a year of nights. This plan spends about 12 weeks getting a measured picture of the whole map, then chooses.

---

## Weekly rhythm

One subject. Roughly 8 to 10 hours.

| When | What |
|:--|:--|
| **Monday (about 30 min)** | Open this week's section below. Write the Monday plan in `log/` from [`templates/weekly-report.md`](templates/weekly-report.md): hours available, the two resources, the prototype, a prediction, and a work assignment that can be done on a trading OMS core without copying anything out. |
| **Tuesday to Thursday (about 1 hour a day)** | Study the primary sources. Start the prototype. Measure something small so the weekend is not a cold start. |
| **Weekend (about 4 to 5 hours)** | Finish the prototype and the home assignment. Run the knowledge check with the notes closed. Do the work assignment on the OMS (or write why it could not be applied). |
| **Weekend report (first-class, about 15 min)** | Fill the weekend report in the same log entry: hours, what was built, numbers, knowledge-check score and weak spots, work-assignment lesson, how the week went, what to carry over. The week is not done until this is written. |

If a week slips, finish its weekend report anyway and carry the prototype into the next week. Do not skip the report.

---

## Rules

Same as the [README](README.md):

1. **Measure before claiming.** Hardware, kernel, compiler, exact command, and p50/p99/p99.9/max. Not an average as the headline.
2. **Correct before fast.** Check against a reference or a model before optimizing.
3. **Break it on purpose.** Change one thing that should make the effect worse or disappear, and confirm that it does.
4. **Write it down.** Prototype numbers go in the weekend report. Project-scale work still needs `RESULTS.md`.

Added for this public repo:

5. **Employer material stays out.** FlexTrade code, specs, configs, traces and numbers never go in this repository. The work assignment is for learning on the OMS (order state machines, gateways, sequence recovery, market data, hot-path latency). Only the lesson goes in `log/`.

---

## Using AI without outsourcing understanding

The skill is a personal model of the machine. AI is for sharpening that model, not replacing it. Full prompts and the per-topic loop live in [`tracks/00-foundations.md`](tracks/00-foundations.md).

**Do this:** predict in the Monday plan before asking; use AI to quiz, not to lecture; let it write benchmark scaffolding and then audit every line of the timed region; paste `perf` or compiler output and ask what it means, then confirm against the manual; after notes are written from memory, ask it to find errors.

**Do not:** trust a latency, bandwidth or cost it states until it is measured on this hardware; let it write `notes/`; skip the prediction; treat generated concurrency or memory-ordering code as correct without a test that tries to break it.

---

## How a week is done

Tick the matching track checkboxes only when all of these are true:

- The weekend report is in `log/`, including the environment and the percentile numbers (or an explicit note that the prototype could not run, and why).
- The prototype was built, measured and broken on purpose.
- The knowledge check was answered with notes closed, score recorded, weak spots named.
- The work assignment produced a lesson in my own words, with no employer detail in the repo.
- I can explain the week's subject out loud in two minutes, including one number from my own machine.

---

## Breadth map (weeks 1 to 13)

| Week | Monday | Subject | Catalog |
|:--:|:--|:--|:--|
| 1 | 2026-11-23 | Timers and latency histograms | SI-01 start; track 1 Measurement |
| 2 | 2026-11-30 | CPU pipeline and branch prediction | track 0 CPU |
| 3 | 2026-12-07 | Caches and prefetching | track 0 Memory; SI-02 start |
| 4 | 2026-12-14 | TLB and huge pages | track 0 Memory |
| 5 | 2026-12-21 | Coherence and false sharing | track 0 Memory |
| 6 | 2026-12-28 | Syscalls, scheduler and context switches | track 0 OS |
| 7 | 2027-01-04 | `perf`, profiling and flame graphs | track 1 Profiling |
| 8 | 2027-01-11 | GPU and CUDA intro (toy kernel) | track 4 Phase B slice |
| 9 | 2027-01-18 | LLM inference: run and profile vLLM or llama.cpp | track 4 Phase C slice |
| 10 | 2027-01-25 | Linux network receive path and kernel bypass intro | track 0 Networking; track 3 slice |
| 11 | 2027-02-01 | TLA+ on an order state machine | track 2 Verification |
| 12 | 2027-02-08 | Storage and `io_uring` | track 5; track 0 OS `io_uring` |
| 13 | 2027-02-15 | Review and specialization decision | this file |

Month 3 is weeks 9 to 13. The decision is due at the end of week 13 (weekend of 2027-02-21).

---

## Week 1 · Monday 2026-11-23 · Timers and latency histograms

Start [SI-01 benchkit](projects/README.md#si-01-benchkit). Later weeks reuse this harness. A complete histogram library can wait; a pinned loop, two clocks and percentile output is enough this week.

**Study (pick these two):**

1. [`clock_gettime(2)`](https://man7.org/linux/man-pages/man2/clock_gettime.2.html) (also skim [`vdso(7)`](https://man7.org/linux/man-pages/man7/vdso.7.html) so week 6 is not a surprise).
2. Gil Tene, [How NOT to Measure Latency](https://www.youtube.com/watch?v=lJ8ydIuPFeU) (QCon SF 2013). Then the [HdrHistogram](https://github.com/HdrHistogram/HdrHistogram) README (the [C port](https://github.com/HdrHistogram/HdrHistogram_c) is the one to copy ideas from, not to vendor blindly).

**Prototype:** on the Linux machine, a small C++ program (the start of SI-01) that (a) compares `rdtscp` (or `rdtsc` if `rdtscp` is missing) against `clock_gettime(CLOCK_MONOTONIC)` for an empty timed region and for a calibrated busy-spin of a known length, (b) records a histogram and prints p50/p99/p99.9/max. Pin with `taskset`. Check for a constant-rate TSC (`constant_tsc` / `nonstop_tsc` in `/proc/cpuinfo`).

**Home assignment:** measure harness overhead (empty timed region) and show that an average of the same data hides a tail you can see in p99.9 and max. Break it: run without pinning, or with turbo and idle states left free, and watch max move.

**Work assignment:** find one existing latency or timing number on the OMS hot path (order handling, gateway, or market data). Decide whether the measurement is open-loop or closed-loop, and whether the reported figure is an average. Write only the lesson in the weekend report (for example: "we report a mean of N samples from a closed loop, so idle time is invisible"). No employer numbers.

**Knowledge check** (notes closed):

1. What does `constant_tsc` on x86 actually promise, and what still has to be calibrated?
2. Why is `clock_gettime` often much cheaper than a general syscall on Linux?
3. Why is an average the wrong headline for latency?
4. What is coordinated omission, in one sentence?
5. What is the cost of the empty timed region on this machine (from this week's run)?
6. If p50 is fine and max is huge, name two causes that are not "the code got slower".
7. Why pin the benchmark thread before trusting a histogram?

**Done means:** SI-01 repo folder exists with a timer, a histogram printout, overhead documented, weekend report filled, track 1 items "Timers" and "Latency histograms" ticked only if the criteria in "How a week is done" hold.

---

## Week 2 · Monday 2026-11-30 · CPU pipeline and branch prediction

**Study:**

1. Bakhvalov, [ch. 3 CPU Microarchitecture](https://github.com/dendibakh/perf-book/tree/main/chapters/3-CPU-Microarchitecture) (pipelining and ILP). Use [uops.info](https://uops.info/table.html) for the core's `imul` latency and throughput.
2. Bakhvalov, [ch. 10 Optimizing Branch Prediction](https://github.com/dendibakh/perf-book/tree/main/chapters/10-Optimizing-Branch-Prediction), or if short on time the shorter pair [Algorithmica: branching](https://en.algorithmica.org/hpc/pipelining/branching/) and [branchless programming](https://en.algorithmica.org/hpc/pipelining/branchless/).

**Prototype:** two pinned loops counted with `perf stat -e cycles,instructions,branches,branch-misses`: (1) a long `imul` dependency chain versus four independent chains; (2) `if (x[i] < 128) sum += x[i]` over random bytes, sorted versus unsorted. Predict cycles per iteration before running. Reuse SI-01 for the wall-time histogram if it is ready.

**Home assignment:** compute mispredict cost as extra cycles divided by extra misses. Break it: reuse one tiny random array so the predictor learns it.

**Work assignment:** pick one data-dependent branch on the OMS (order type, session state, or checksum path). From memory, is it predictable on the live mix? What would a sorted-versus-unsorted experiment look like on a replay? Lesson only.

**Knowledge check:**

1. What is the difference between instruction latency and throughput (ports) for `imul` on this core?
2. When is a loop latency-bound rather than throughput-bound?
3. What does a branch mispredict cost, in cycles, on this machine?
4. Why can a branchless `cmov` version be slower than a well-predicted branch?
5. What does IPC below 1 often indicate in a tiny loop like these?
6. Why does repeating a small random pattern make a "random branch" benchmark lie?
7. Name one ROB or port fact from this week's reading that showed up (or failed to show up) in `perf stat`.

**Done means:** both experiments measured, mispredict cost in the weekend report, matching track 0 CPU checkboxes ticked only if the week-done bar is met.

---

## Week 3 · Monday 2026-12-07 · Caches and prefetching

Start [SI-02 memlab](projects/README.md#si-02-memlab) as a pointer-chase and stride sweep, not the whole project.

**Study:**

1. Drepper, cache sections: [LWN part 2](https://lwn.net/Articles/252125/) (same material as the [PDF](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf) 3.3.1 to 3.3.3).
2. Bakhvalov, [ch. 8 Optimizing Memory Accesses](https://github.com/dendibakh/perf-book/tree/main/chapters/8-Optimizing-Memory-Accesses) (memory prefetching), or [Algorithmica: prefetching](https://en.algorithmica.org/hpc/cpu-cache/prefetching/).

**Prototype:** pointer chase over a random cyclic permutation, one node per 64-byte line, working sets from 4 KiB to at least 256 MiB (1 GiB if RAM allows). Plot ns and cycles per load. Then the same buffer in sequential, strided and random order.

**Home assignment:** compare the curve steps with `lscpu -C` (or `/sys/devices/system/cpu/cpu0/cache/`). Break it: 4096-byte stride so accesses collide in one set.

**Work assignment:** from memory, sketch the cache-line layout of one hot OMS object (order, book level or session). Where would a prefetch help, and where would random order IDs defeat it? Lesson only, no structs from work.

**Knowledge check:**

1. What are L1, L2, L3 and DRAM load latencies on this machine, in ns?
2. What is a cache line size here, and why do we chase one node per line?
3. Inclusive versus exclusive last-level cache: which does this CPU use, and why would it matter?
4. Which access pattern did the hardware prefetcher hide, and which did it not?
5. Why does a large power-of-two stride sometimes miss as if the cache were tiny?
6. Why is bandwidth (STREAM-style) a different measurement from pointer-chase latency?
7. What would `__builtin_prefetch` need (addresses known early) to help the random-index case?

**Done means:** a latency-versus-size table in the weekend report; track 0 cache and prefetcher items ticked only if week-done holds.

---

## Week 4 · Monday 2026-12-14 · TLB and huge pages

**Study:**

1. Drepper on page tables: [LWN part 3](https://lwn.net/Articles/253361/) (PDF section 4.3).
2. Kernel docs: [transparent huge pages](https://docs.kernel.org/admin-guide/mm/transhuge.html) and [HugeTLB pages](https://docs.kernel.org/admin-guide/mm/hugetlbpage.html). Skim [`mlock(2)`](https://man7.org/linux/man-pages/man2/mlock.2.html).

**Prototype:** rerun the week 3 chase at a large working set (256 MiB to 1 GiB) with 4 KiB pages, `madvise(MADV_HUGEPAGE)`, and `MAP_HUGETLB` if the machine has HugeTLB pages configured. Confirm huge pages in `/proc/meminfo` (`AnonHugePages` / `HugePages_*`). Count dTLB misses with `perf stat` (`perf list` for the event name on this CPU).

**Home assignment:** compare 4 KiB versus 2 MiB at the same working set. Break it: set THP to `never` (and set it back afterwards).

**Work assignment:** does the OMS hot path pre-fault or lock memory, or can a first-touch page fault land in an order handler? What would a TLB-miss spike look like during sequence replay? Lesson only.

**Knowledge check:**

1. What is TLB reach (entries times page size) on this CPU, from vendor docs or `cpuid`?
2. What is a page walk, and which cache can hold page-table entries?
3. THP versus explicit HugeTLB: one operational difference that showed up this week.
4. Measured load latency at 1 GiB (or the largest size run) for 4 KiB versus huge pages.
5. Why can THP promote some pages and not others in a pointer-chase arena?
6. Why do latency-sensitive paths `mlock` after faulting pages in, not instead of faulting them in?
7. Which `perf` event counted dTLB misses on this CPU?

**Done means:** huge-page confirmation in the report (or a documented reason they were unavailable); track 0 TLB checkbox ticked only if week-done holds.

---

## Week 5 · Monday 2026-12-21 · Coherence and false sharing

**Study:**

1. [Mara Bos, ch. 7 Understanding the Processor](https://marabos.nl/atomics/hardware.html) (MESI-style coherence in plain language). Optional backup: McKenney [perfbook](https://mirrors.edge.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.html) Appendix C "Why Memory Barriers?".
2. [Joe Mario on `perf c2c`](https://joemario.github.io/blog/2016/09/01/c2c-blog/) and [`perf-c2c(1)`](https://man7.org/linux/man-pages/man1/perf-c2c.1.html).

**Prototype:** two threads incrementing adjacent counters in one cache line versus padded to 64 (and 128) bytes. Confirm with `perf c2c` if the kernel and CPU support it. Then a two-thread ping-pong of one line; record round-trip ns. Optional cross-check: [core-to-core-latency](https://github.com/nviennot/core-to-core-latency).

**Home assignment:** report the slowdown factor of false sharing and the ping-pong time for SMT siblings versus a different core. Break it: pad 64 versus 128 and see whether adjacent-line prefetch still couples them.

**Work assignment:** from memory, name one shared counter, sequence number or stats field that two OMS threads might hit. Is it padded? What would `perf c2c` on a load-test binary be looking for? Lesson only.

**Knowledge check:**

1. In MESI (or MOESI), what state is a line in when two cores take turns writing it?
2. What is false sharing, in one sentence?
3. What slowdown did false sharing show on this machine?
4. Why can padding of 64 bytes be insufficient on some Intel cores?
5. Why is cross-core ping-pong much more expensive than a local increment?
6. What does `perf c2c` add over "the benchmark got slower"?
7. SMT siblings versus separate cores: which ping-pong was faster here, and is that what you predicted?

**Done means:** false-sharing and ping-pong numbers in the report; track 0 coherence checkbox ticked only if week-done holds.

---

## Week 6 · Monday 2026-12-28 · Syscalls, scheduler and context switches

**Study:**

1. [`sched(7)`](https://man7.org/linux/man-pages/man7/sched.7.html) plus the kernel [EEVDF](https://docs.kernel.org/scheduler/sched-eevdf.html) page (CFS history: [sched-design-CFS](https://docs.kernel.org/scheduler/sched-design-CFS.html)).
2. [`vdso(7)`](https://man7.org/linux/man-pages/man7/vdso.7.html).

**Prototype:** (1) pin a loop of `syscall(SYS_getppid)` versus `clock_gettime` and compare ns using SI-01. (2) `perf bench sched pipe` on one core versus two cores, then a tiny pipe or futex ping-pong of your own. Optional comparison: [Tsuna's context-switch note](https://blog.tsunanet.net/2010/11/how-long-does-it-take-to-make-context.html) (old hardware; the method matters more than the numbers).

**Home assignment:** context-switch cost same core versus cross core. Break it: a `SCHED_FIFO` busy loop on a spare core (`chrt -f 50`) and observe RT throttling via `/proc/sys/kernel/sched_rt_runtime_us` (do not lock up the login core).

**Work assignment:** which OMS threads block on syscalls (gateway, logging, store-and-forward) versus spinning on a ring? What would a context switch between them do to a one-digit-microsecond budget? Lesson only.

**Knowledge check:**

1. ns per `getppid` syscall versus per vDSO `clock_gettime` on this box.
2. What is the vDSO, and which calls typically use it?
3. Why is a same-CPU pipe context switch a different cost from a cross-CPU wakeup?
4. What does `SCHED_FIFO` change, and what is RT throttling for?
5. EEVDF versus CFS: one sentence on what the kernel is trying to keep fair.
6. Why do hot paths avoid syscalls even when each syscall is "only a couple of hundred ns"?
7. What was `perf bench sched pipe` on this machine?

**Done means:** syscall, vDSO and context-switch numbers in the report; track 0 scheduler and syscall checkboxes ticked only if week-done holds. Full `io_uring` waits until week 12.

---

## Week 7 · Monday 2027-01-04 · perf, profiling and flame graphs

**Study:**

1. Brendan Gregg, [perf examples](https://www.brendangregg.com/perf.html), plus [`perf-stat(1)`](https://man7.org/linux/man-pages/man1/perf-stat.1.html) and [`perf-record(1)`](https://man7.org/linux/man-pages/man1/perf-record.1.html).
2. Gregg, [CPU flame graphs](https://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html) (tooling: [FlameGraph](https://github.com/brendangregg/FlameGraph)). Optional: [off-CPU analysis](https://www.brendangregg.com/offcpuanalysis.html).

**Prototype:** pick one earlier prototype that still has source (weeks 2, 3 or 5). `perf record` it, produce a flame graph, and `perf annotate` the hottest function. Record the top three stacks. If `perf` needs extra permissions, document the sysctl or `perf_event_paranoid` change.

**Home assignment:** change the program so the hottest stack moves (for example sort the branch data, or pad the false-sharing counters) and show the flame graph changing. Break it: profile at `-O0` and explain why that graph is not the one to optimize from.

**Work assignment:** if a non-production test binary can be profiled at work, do that and write only the lesson (for example: "the hottest stack was logging, not matching"). If it cannot, write what symbol you would look for on a gateway read loop. No profiles from work in this repo.

**Knowledge check:**

1. What is the difference between `perf stat` and `perf record`?
2. What does each rectangle's width mean on a CPU flame graph?
3. Name the hottest function in this week's prototype and its share of samples.
4. What is a skipped stack (missing frames), and how do you get better stacks on C++?
5. When would an on-CPU flame graph hide the real delay?
6. What does `perf annotate` show that the flame graph does not?
7. Why can a short unpinned run produce a flame graph that blames the wrong thing?

**Done means:** a flame graph described in the weekend report (path under `build/` is fine; do not commit huge SVGs unless they are tiny); track 1 `perf` and flame graph checkboxes ticked only if week-done holds.

---

## Week 8 · Monday 2027-01-11 · GPU and CUDA intro

Light depth: one toy kernel, one timeline of it launching. Not SI-04.

**Study:**

1. NVIDIA, [An Even Easier Introduction to CUDA](https://developer.nvidia.com/blog/even-easier-introduction-cuda/).
2. [CUDA C++ Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html) (execution model: threads, blocks, grids) and a skim of the [Best Practices Guide](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html) on memory.

If there is no NVIDIA GPU this week, install the toolkit if possible and compile; otherwise rent a small GPU by the hour for one evening, or run in a CUDA-capable VM and mark numbers as not production-like. Do not skip the week.

**Prototype:** vector add or SAXPY: host allocate, `cudaMemcpy` up, tiny kernel, copy down, check correctness against a CPU loop. Time (a) the kernel only and (b) kernel plus copies, with SI-01 or CUDA events. Print occupancy-related facts you can get (`nsys` if installed, otherwise just the two times).

**Home assignment:** double the vector length and see whether time scales. Break it: a kernel with a huge block size or a mis-sized grid, or omit the copy and watch the "fast" kernel fail the check.

**Work assignment:** write the analogy, lesson only: GPU latency hiding (lots of warps) versus an OMS hiding I/O with many in-flight orders. Where does the analogy fail?

**Knowledge check:**

1. What is a warp, and how many threads is it on current NVIDIA GPUs?
2. What is the difference between a thread block and the grid?
3. Why was kernel-only time so different from kernel-plus-copy in this run?
4. What does coalesced memory access mean, in one sentence?
5. What is occupancy, and why is 100% occupancy not the goal?
6. Name one way SIMT divergence is like a CPU branch mispredict, and one way it is not.
7. What GPU (or "no GPU") did this week actually run on?

**Done means:** a correct toy kernel and two times in the report; no track 4 phase B checkboxes ticked except the execution-model item if the explanation is solid. SI-04 waits for depth.

---

## Week 9 · Monday 2027-01-18 · LLM inference: run and profile

Run someone else's engine. Do not start SI-09.

**Study:**

1. [PagedAttention / vLLM paper](https://arxiv.org/abs/2309.06180) (abstract, introduction and the KV-cache problem statement are enough this week).
2. One operational guide: [vLLM quickstart](https://docs.vllm.ai/en/latest/getting_started/quickstart.html) **or** the [llama.cpp README](https://github.com/ggml-org/llama.cpp/blob/master/README.md) (ggml-org). Pick the one that matches the hardware (NVIDIA: either; CPU-only: llama.cpp).

**Prototype:** run a small model (Llama-class 1B or a tiny GGUF). Record tokens per second, and if the tool prints them, TTFT and an inter-token time. One short prompt and one long prompt. If `perf` or `nsys` can attach, note whether time is in GPU kernels, CPU sampling or prompt eval.

**Home assignment:** change batch size or context length (whatever the engine exposes in one flag) and record how throughput and latency move. Break it: a context long enough to slow decode, or a CPU run versus GPU run if both exist.

**Work assignment:** continuous batching versus OMS request batching or coalescing. One lesson on what "head of line blocking" means for an inference batch versus an order gateway. No work systems in the repo.

**Knowledge check:**

1. Why is prefill often compute-bound and decode memory-bound?
2. What problem does paged KV cache solve, in one sentence?
3. Rough KV size formula: which dimensions multiply?
4. What tokens/s (or TTFT) did this machine actually get, on which model?
5. What changed when the prompt got longer?
6. Why is batch-1 decode far below peak FLOPs on a big GPU?
7. llama.cpp versus vLLM: which did you run, and what hardware constraint drove the choice?

**Done means:** a real run logged with model name and numbers; track 4 "why prefill vs decode" understood. Building spinfer is a depth-phase choice.

---

## Week 10 · Monday 2027-01-25 · Linux network receive path and kernel bypass intro

**Study:**

1. [packagecloud: Monitoring and Tuning the Linux Networking Stack: Receiving Data](https://blog.packagecloud.io/monitoring-tuning-linux-networking-stack-receiving-data/).
2. Kernel [AF_XDP](https://docs.kernel.org/networking/af_xdp.html) documentation (read for the idea: UMEM, fill/completion/RX/TX rings, zero-copy versus copy). Do not build DPDK this week.

**Prototype:** UDP ping-pong on localhost and, if a second machine or namespace exists, across a real NIC. Histogram with SI-01 of `sendto`/`recvmsg` round trip. Record `ethtool -S` (or `/proc/net/snmp` Udp counters) before and after a short flood. Draw the receive path from NIC to `recvmsg` in `notes/` from memory, then correct it.

**Home assignment:** shrink `SO_RCVBUF` and flood until `RcvbufErrors` (or equivalent) climbs. Break it: pin the NIC IRQ onto the measurement core if you can, and watch the histogram.

**Work assignment:** market-data receive versus order-gateway receive on the OMS. Where does sequence recovery run, and which part would kernel bypass even be legal or useful for? Lesson only. No topology diagrams from work.

**Knowledge check:**

1. Name the stages from NIC DMA to user `recvmsg`.
2. What is NAPI for?
3. Which counter would you check for socket-buffer drops versus NIC-ring drops?
4. What round-trip p50/p99 did localhost UDP show versus (if measured) a real NIC?
5. What problem is AF_XDP trying to avoid, in one sentence?
6. Why is localhost latency not a substitute for wire latency?
7. Why do many market-data feeds use UDP multicast rather than TCP?

**Done means:** a path drawing in `notes/` and a UDP histogram in the report; track 0 receive-path checkbox ticked only if week-done holds. SI-07 stays in the depth catalog.

---

## Week 11 · Monday 2027-02-01 · TLA+ on an order state machine

TLA+ is already started. This week produces one small spec that TLC can check, not a full exchange.

**Study:**

1. [Learn TLA+: conceptual overview](https://learntla.com/intro/conceptual-overview.html), then the start of [the core](https://learntla.com/core/index.html) as far as actions, next-state and invariants.
2. Lamport's [TLA+ page](https://lamport.azurewebsites.net/tla/tla.html) (toolbox and hyperbook links from there). Use [tlaplus/Examples](https://github.com/tlaplus/Examples) only as a shape reference.

**Prototype:** specify a simplified order: states such as `New`, `Acked`, `PartialFill`, `Filled`, `Cancelled`, `Rejected`. Actions: ack, fill (qty remaining), cancel, reject. Invariants: no fill after cancel; remaining quantity never negative; terminal states stay terminal. Model-check with TLC. Then add one bug on purpose (a fill from `Cancelled`) and see TLC catch it.

**Home assignment:** add a sequence number or a gateway retry (at-least-once ack) and an invariant that the client does not double-count fills. Break it: drop the invariant and confirm the bad trace exists.

**Work assignment:** which real OMS transitions did this spec omit (replace, bust, unsolicited cancel, partials on different venues)? Write the omissions as a lesson. Do not transcribe the employer's spec.

**Knowledge check:**

1. What is the difference between an invariant and a temporal property at this stage?
2. What does TLC actually search?
3. What bug did you plant, and how did TLC report it?
4. Why is a spec of the state machine useful even if the C++ already "works"?
5. What is an action, versus a state, in your spec?
6. What did you have to leave out to keep the state space small?
7. Name one invariant you could not check this week that you would want later (liveness of acks, fairness).

**Done means:** `.tla` (and `.cfg` if used) in the repo, TLC result in the weekend report; track 2 "TLA+ basics" ticked if a model actually checked. A libvsr view-change spec is depth, not this week.

---

## Week 12 · Monday 2027-02-08 · Storage and io_uring

**Study:**

1. Jens Axboe, [Efficient IO with io_uring](https://kernel.dk/io_uring.pdf).
2. [`io_uring(7)`](https://man7.org/linux/man-pages/man7/io_uring.7.html). Practical companion if needed: [Lord of the io_uring](https://unixism.net/loti/). [liburing](https://github.com/axboe/liburing) for the API.

**Prototype:** (1) batch `IORING_OP_NOP` with [liburing](https://github.com/axboe/liburing), batch size 1 to 256, ns per operation with SI-01. (2) append a few hundred MB with `write` plus `fsync` versus `io_uring` writes (or `fdatasync` once at the end). Record p50/p99 for the syncs. If liburing is a pain to install, `io_uring` syscalls via a tiny wrapper are acceptable.

**Home assignment:** find the NOP batch size where batched io_uring beats one syscall per op (compare to week 6 `getppid`). Break it: `O_DIRECT` without aligned buffers, or a forgotten `fsync`, and say what is no longer being measured.

**Work assignment:** sequence recovery or store-and-forward on the OMS. Where must a write be durable before ack, and where is group commit enough? Lesson only. No WAL format from work.

**Knowledge check:**

1. What are the SQ and CQ in io_uring?
2. Why can io_uring beat "one syscall per I/O"?
3. NOP ns per op at batch 1 versus the best batch this week.
4. `fsync` versus `fdatasync`: what is the promised difference, and what did you measure?
5. What goes wrong with `O_DIRECT` if buffers are not aligned?
6. Why is a NOP benchmark not a substitute for NVMe durability?
7. Name one crash-consistency question this week did not answer (that SI-11 would).

**Done means:** NOP batch curve and one durability-related number in the report; track 0 io_uring item and a track 5 kernel item ticked only if week-done holds. SI-11 is depth.

---

## Week 13 · Monday 2027-02-15 · Review and specialization decision

No new subject. This is month 3's decision point.

**Study:** re-read your own `notes/` and weekend reports. Skim [`projects/README.md`](projects/README.md) and the specialization table below.

**Prototype:** re-derive two headline numbers on the same machine without looking at old code (examples: empty-timer overhead, L3 pointer-chase step, context-switch ns, UDP p99). If a number disagrees, explain why in the report.

**Home assignment:** score each breadth week 1 to 12: interesting / useful for work / want to go deep (yes or no). Pick **one or two** specializations. Write the choice, the first depth project, and which catalog projects you are explicitly leaving optional.

**Work assignment:** one paragraph on which specialization the current OMS role most needs in the next year, versus which one you would enjoy. If they differ, say which you are picking and why. Lesson only.

**Knowledge check** (sample across the map; swap in your weak spots):

1. Timer overhead and why percentiles.
2. A cache or TLB number from this machine.
3. False sharing in MESI terms.
4. Syscall versus vDSO versus io_uring NOP.
5. What a flame graph width means.
6. Prefill versus decode, or kernel-plus-copy versus kernel-only on GPU.
7. One stage of the Linux receive path.
8. One invariant from the order-state TLA+ spec.

**Done means:** weekend report contains the specialization choice (trading, AI inference/serving, reliability, or a named pair), the first depth project id, and a list of SI projects that are optional. Update the status table in [README.md](README.md). Do not start two depth projects in the same week.

---

## Depth after week 13

Build from the existing catalog. Finish SI-01's remaining pieces (open-loop load generator, coordinated-omission correction, `perf_event_open` around a region) as soon as the first depth project needs them. SI-02 and SI-03 are worth finishing for any specialization that still lacks cache/TLB or jitter numbers on these machines.

Guidance: which projects fit which specialization. Unpicked items stay on the catalog as optional.

| Specialization | Go deep on | Natural project order | Optional (not a failure if skipped) |
|:--|:--|:--|:--|
| **Low-latency trading** | Tracks 0 leftovers, 1, 3 | SI-02 (finish), SI-03 jitterlab, SI-06 ITCH, SI-07 bypass, SI-08 tick-to-trade | SI-04, SI-05, SI-09, SI-12, SI-13, SI-14. SI-10 if replicated matching or gateway failover is the interest. SI-11 if durable sequence store is the interest. |
| **AI inference and serving** | Track 4 | SI-04 SGEMM, SI-05 fused kernels, SI-09 spinfer, SI-12 upstream PR, then SI-13 and SI-14 if serving is the goal | SI-06, SI-07, SI-08. SI-03 if tail latency on the host thread matters. SI-11 if checkpoint or KV durability matters. |
| **Reliability** | Track 2, track 5 | SI-10 Jepsen-style checker (libvsr / exchange), SI-11 storage engine, more TLA+ (view change, recovery) | GPU kernel projects. Trading feed handler unless it is the system under test. |

**Pairing (if two are chosen):**

- Trading plus reliability: SI-03, SI-06 to SI-08, SI-10. Treat SI-07 as the network-fault target.
- AI plus reliability: SI-04/05, SI-09, SI-10, then SI-14 failover. SI-12 still counts as the public artifact.
- Trading plus AI: do not run both catalogs in parallel. Pick a primary for the next 12 weeks and a single secondary project (often SI-04 or SI-06).

**Suggested depth rhythm (same weekly loop):** one subject-sized slice of the current SI project per week, same Monday plan and weekend report. A project is done only when its acceptance list and `RESULTS.md` exist.

**Dependencies that remain true:** SI-01 before serious measurement in any project. SI-04/05 before a serious SI-09. SI-09 before SI-13/14. SI-06 before SI-07 before SI-08.

Rough calendar if week 14 starts Monday 2027-02-22: a single specialization at 8 to 10 hours a week is about 12 to 16 weeks to the first large project (SI-03, SI-06 or SI-04). The old 18-month end date is no longer the success criterion. Published numbers and `RESULTS.md` are.

---

## Adjusting

- Reorder breadth weeks only when a dependency forces it. SI-01 stays first. Profiling (week 7) after there is something to profile. GPU (week 8) before running an inference engine (week 9).
- Slipping is fine. Skipping the weekend report is not.
- After week 13, changing specialization once is allowed; changing every month is not. Write the change in `log/`.
- Review at week 13 and after each finished SI project: update the [README](README.md) status table.
