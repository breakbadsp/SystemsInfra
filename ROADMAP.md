# Roadmap

Spend 8 to 10 hours a week on the shared core. Use weeks 1 to 12 to try performance, robustness, and domain-specific work. In week 13, choose a specialization and one depth project from the [project catalog](projects/README.md#specializations). Keep the other projects optional.

## Weekly workflow

Read the assigned material, write a prediction, and run one lab. Record the run command, environment, evidence, and outcome in one [weekly entry](templates/weekly-report.md). Apply the lesson to an order management system (OMS) if useful. Follow the [repository rules](README.md#rules) when you record that lesson.

Each lab has a pass condition. Record `passed`, `needs rework`, or `blocked`. If the lab cannot run, record the blocker and what would resolve it. Tick a [track](README.md#tracks) item when your evidence covers that item. Leave unfinished assessments open in the log.

Answer the two questions with your notes closed. Check your answers against the sources and results. For performance labs, keep the workload fixed, repeat a baseline and one controlled variant, and explain any difference or lack of a difference.

## Week 1 · 2026-11-23 · Timers and latency histograms

**Read** [`clock_gettime(2)`](https://man7.org/linux/man-pages/man2/clock_gettime.2.html) and [How NOT to Measure Latency](https://www.youtube.com/watch?v=lJ8ydIuPFeU). The week's sources, in reading order, are in [`resources/week-01-timers-and-histograms.md`](resources/week-01-timers-and-histograms.md).

**Lab** Start [SI-01 benchkit](projects/README.md#si-01-benchkit) with a pinned `CLOCK_MONOTONIC` timing loop and percentile output. Measure the empty region and a short repeatable workload. Add a known delay to a small fraction of samples. Keep TSC calibration and a reusable histogram library for depth.

**Pass** Record the sample count, timer overhead, and baseline and delayed distributions. Explain which statistics show the injected delay.

**Recall** How does timer overhead affect a short measurement? How can a closed-loop driver hide the latency of requests that would have arrived during a stall?

## Week 2 · 2026-11-30 · Branch prediction

**Read** [CPU microarchitecture](https://github.com/dendibakh/perf-book/tree/main/chapters/3-CPU-Microarchitecture) and [branch prediction](https://github.com/dendibakh/perf-book/tree/main/chapters/10-Optimizing-Branch-Prediction).

**Lab** Run the same branch-heavy loop over sorted and shuffled copies of the same input. Check the optimized assembly for a conditional branch. Verify that both runs produce the same result. Compare cycles and branch misses with `perf stat` where supported.

**Pass** Record the relative timing and the evidence for your explanation. If hardware counters are unavailable, report the timing and assembly evidence and mark the branch-miss claim unverified.

**Recall** Why can input order affect the timing when the output is identical? When can branchless code cost more?

## Week 3 · 2026-12-07 · Caches, prefetching, and TLB

**Read** Drepper on [caches](https://lwn.net/Articles/252125/) and the kernel guide to [transparent huge pages](https://docs.kernel.org/admin-guide/mm/transhuge.html#monitoring-usage).

**Lab** Start [SI-02 memlab](projects/README.md#si-02-memlab). Sweep a randomized pointer chase across working-set sizes that fit in cache and sizes that exceed it. Compare with ordered traversal of the same nodes. At one large size, repeat with base pages and transparent huge pages where available. Use per-mapping advice and verify the benchmark mapping in `/proc/PID/smaps`.

**Pass** Record time per load by working-set size and access order. Relate the results to cache sizes and access order. Report the page-size comparison only if you verify the backing for both runs. Record the reason if that comparison is unavailable.

**Recall** Why does a randomized dependent chain limit prefetching? What must you hold fixed to attribute a timing difference to page size?

## Week 4 · 2026-12-14 · Overload and bounded queues

**Read** [Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/), sections on queue management and load shedding.

**Lab** Build a single-worker request processor with a fixed queue capacity and explicit rejection when the queue is full. Drive it on a fixed schedule below capacity, above capacity, and below capacity again. Compare two queue capacities using the same arrival schedule and service time. Measure p99 accepted-request latency, peak queue depth, and rejects for each phase.

**Pass** Check that queue depth stays within capacity, every submitted request completes or receives an explicit rejection, and the queue drains after the offered rate drops. Explain the latency and rejection trade-off.

**Recall** Why can a larger queue increase tail latency? How would a closed-loop load driver change the overload test?

## Week 5 · 2026-12-21 · Coherence and false sharing

**Read** [Understanding the Processor](https://marabos.nl/atomics/hardware.html) and [`perf-c2c(1)`](https://man7.org/linux/man-pages/man1/perf-c2c.1.html).

**Lab** Put two counters on the same cache line, then on separate cache lines. Have two threads on separate cores update one counter each. Hold the iteration count and core placement fixed.

**Pass** Verify the counter totals and layouts. Compare time per update across repeated runs. Use `perf c2c` if supported, and record the limits of your evidence.

**Recall** How can writes to different counters contend? What evidence would distinguish false sharing from contention on one counter?

## Week 6 · 2026-12-28 · System calls and scheduling

**Read** [`sched(7)`](https://man7.org/linux/man-pages/man7/sched.7.html) and [`vdso(7)`](https://man7.org/linux/man-pages/man7/vdso.7.html).

**Lab** Run `perf bench sched pipe --threaded` with both threads restricted to one CPU, then with two CPUs available. Record affinity and check thread placement before attributing any difference to cross-CPU wakeups.

**Pass** Record the command, affinity, and repeated timings. Explain which work the end-to-end timing includes and what the experiment can establish about placement.

**Recall** What does the vDSO avoid? Why does a pipe round trip include more than a context switch?

## Week 7 · 2027-01-04 · Profiling

**Read** [`perf-record(1)`](https://man7.org/linux/man-pages/man1/perf-record.1.html) and [CPU flame graphs](https://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html).

**Lab** Profile one earlier lab. Identify the largest source of CPU samples. Change one implementation detail and collect a second profile. Use a user-space sampling profiler if `perf` is unavailable.

**Pass** Save both profiles or record the top stacks and sample counts. Explain the change in samples and compare elapsed time for the same amount of work.

**Recall** What does width represent on a flame graph? When can elapsed time rise without a matching rise in CPU samples?

## Week 8 · 2027-01-11 · GPU execution

**Read** [An Even Easier Introduction to CUDA](https://developer.nvidia.com/blog/even-easier-introduction-cuda/) and the CUDA guide to [timing](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html#timing).

**Lab** Build a vector-add kernel and check its output against a CPU reference. Use CUDA events for kernel time. Measure copy and kernel time together with a host timer that waits for the work to finish. Repeat at a second vector size.

**Pass** Verify the output and report both timings for each size. If you can only compile, record what built and mark the lab blocked on GPU execution. Continue to week 9.

**Recall** Which work can the host timer miss without synchronization? When can copies dominate the total time?

## Week 9 · 2027-01-18 · Inference performance

**Read** the introduction to [PagedAttention](https://arxiv.org/abs/2309.06180) and the guide for [vLLM](https://docs.vllm.ai/en/latest/getting_started/quickstart.html) or [llama.cpp](https://github.com/ggml-org/llama.cpp/blob/master/README.md).

**Lab** Run one small model with a short and a long prompt. Hold the model, backend, and generation settings fixed. Record prompt and output token counts. Use llama.cpp on a CPU if a GPU is unavailable.

**Pass** Report the engine version, model, prompt lengths, and measured prefill and decode rates or timings. Explain what changed between runs.

**Recall** Why do prefill and single-token decode use the hardware differently? How does a longer context affect the KV cache?

## Week 10 · 2027-01-25 · Network receive and packet loss

**Read** [Linux network receive path](https://blog.packagecloud.io/monitoring-tuning-linux-networking-stack-receiving-data/) and the [AF_XDP overview](https://docs.kernel.org/networking/af_xdp.html).

**Lab** Send numbered UDP packets to a local receiver at a fixed rate. Repeat with a smaller receive buffer and a slow reader. Compare sequence gaps and socket drop counters. Keep kernel bypass for [SI-07](projects/README.md#si-07-kernel-bypass-receive).

**Pass** Show a run with no gaps and a run with induced drops. Explain where the observed drops occurred and how an application can detect the loss.

**Recall** How can a receiver distinguish packet loss from reordering? Which parts of the physical receive path does a loopback test omit?

## Week 11 · 2027-02-01 · Model checking an order state machine

**Read** the [Learn TLA+ core guide](https://learntla.com/core/index.html) on actions and invariants.

**Lab** Model a small order state machine with fills and cancellation. Check that quantity never goes negative and a cancelled order cannot fill. To check the latter as an invariant, record whether a fill occurred after cancellation in the model state. Add an invalid fill transition to the model.

**Pass** Save the model, TLC configuration, and commands. Show a complete check for the bounded model and a counterexample for the planted failure.

**Recall** What does a passing bounded check establish? Which state in the counterexample first violates the invariant?

## Week 12 · 2027-02-08 · Storage and restart recovery

**Read** [`fsync(2)`](https://man7.org/linux/man-pages/man2/fsync.2.html) and [Efficient IO with io_uring](https://kernel.dk/io_uring.pdf).

**Lab** Build a small append log with numbered records. Acknowledge a batch only after its sync succeeds. Have a separate driver record the acknowledgments, terminate the writer mid-run, restart it, and check the recovered records. Compare synchronous writes with `io_uring` if available. Keep record sizes, batch sizes, synchronization cadence, and filesystem fixed. Submit the sync after the writes complete and check every completion.

**Pass** Recover every acknowledged record and reject or ignore an incomplete final record. Use a test hook to pause after writing a record prefix, then terminate the writer. In a separate run, terminate it after an acknowledgment. If both I/O paths run, compare throughput and time to acknowledgment. Record any unavailable path. This failure test covers writer termination.

**Recall** What can go wrong if you acknowledge before syncing? What additional fault would you inject to test power-loss durability?

## Week 13 · 2027-02-15 · Review and specialization

Rerun one earlier benchmark and one failure test. Compare the new evidence with the earlier result and explain meaningful differences.

Choose one specialization and one first project from the [project catalog](projects/README.md#specializations). Add the choice to the weekly entry. Define the first milestone with a target workload, a correctness property, and an observable pass condition. Record what evidence from the breadth labs led to the choice.

During depth work, use the same weekly entry for one project milestone at a time. Use the project's acceptance checks and `RESULTS.md` to close it.
