# Track 0: Foundations

The hardware and OS model everything else rests on. Much of this is already familiar from trading work; the aim here is to **measure** it, not just know it. Measurements land in SI-02 and SI-03.

The live week-by-week plan is [`ROADMAP.md`](../ROADMAP.md). This file is the topic catalog. Keep ticking checkboxes here. Weeks 1 to 12 cover a slice of these topics at light depth; the rest wait until after the week 13 specialization decision.

Each checkbox below has a short block: what to **read** (primary sources first), what to **do** on my own Linux box, how to **break** it on purpose, and how I'll **know** I have it. The checkboxes are unchanged, so progress tracking works as before.

---

## How to learn this track

Use the [weekly workflow](../ROADMAP.md#weekly-workflow) for each depth lab. Tick a topic checkbox when its **Know** condition is supported by the evidence in your entry. Use older topics for optional recall practice.

### Using AI without outsourcing understanding

AI is fast at generating code and explanations, and confidently wrong about specifics: latencies, event names, kernel defaults, chapter numbers. The skill being built here is my own model of the machine, so AI is used to sharpen that model, never to replace it.

**Use AI for:**

- **Socratic quizzing.** Have it ask me questions and grade my answers, rather than explain the topic to me.
- **Benchmark scaffolds that I then audit.** Let it write the boring parts (thread pinning, argument parsing, CSV output). I read every line of the timed region and can explain it before running it.
- **Explaining tool output.** Paste `perf stat`, `perf annotate`, `objdump` or Compiler Explorer output and ask what each hot instruction or counter means, then confirm against the manual.
- **Critiquing my notes.** After I have written them from memory, ask it to find errors, vague claims and claims without numbers.
- **Spotting benchmark mistakes.** Dead-code elimination, missing warm-up, frequency scaling, timer overhead, page faults inside the timed region, coordinated omission.

**Do not use AI for:**

- **Numbers.** Any latency, bandwidth or cost it states is a hypothesis until I measure it on my hardware. My measurement beats its claim, and if they disagree I find out why.
- **Writing my notes.** If I did not reason it through, it does not go in `notes/`. Polishing grammar afterwards is fine.
- **Skipping the prediction.** Write my own prediction before asking; otherwise I only learn to agree with it.
- **Final word on correctness.** Concurrency and memory-ordering code it produces gets the same review as code from a junior engineer, plus a test that tries to break it.

**Verification habits:**

- Ask for the source: "Which manual section, kernel doc or man page says this?" Then open it. If it cannot name one, treat the claim as unverified.
- Cross-check against at least one primary source from the topic block and against my own measurement.
- Turn a corrected AI answer into a recall question if it helps you practice.
- Prefer asking it to explain *why my result differs from the source* over asking *what the answer is*.

**Prompts to reuse** (fill in the brackets):

```text
Quiz me on [topic]. Do not explain anything first. Ask one question at a time,
easy to hard. After each answer, tell me only what was right, what was wrong
or missing, and the primary source I should reread. Stop after 8 questions
and list my weak spots.
```

```text
Write a minimal C++20 microbenchmark for [effect] on Linux x86-64, pinned to
one core, reporting p50/p99/p99.9/max. Then list every way this benchmark
could measure the wrong thing (compiler, CPU, OS, timer). Keep the timed
region under 15 lines so I can audit it.
```

```text
Here is my source, the compiler and flags, and the perf annotate output.
Explain what each of the hottest 10 instructions does and which hypothesis
the profile supports. Mark anything you are guessing, and name the manual
section or uops.info entry I should check.
```

```text
Here are my notes on [topic], written from memory. Find factual errors,
vague statements and claims without a number. For each error, cite the
primary source (manual section, kernel doc, man page) I can check. Do not
rewrite the notes.
```

```text
Review this benchmark for: dead-code elimination, warm-up, CPU frequency
and C-states, core pinning, timer overhead, page faults or allocation inside
the timed region, and run-to-run variance. Say which issue would change the
result most and how I would prove it.
```

### Optional recall practice

Answer questions from older notes and check your answers against the sources. To revisit a measurement, predict the result again and compare it with a fresh run.

### Tools used throughout

- `perf`: [Brendan Gregg's perf examples](https://www.brendangregg.com/perf.html), [perf-stat(1)](https://man7.org/linux/man-pages/man1/perf-stat.1.html), [perf-annotate(1)](https://man7.org/linux/man-pages/man1/perf-annotate.1.html)
- Instruction costs: [uops.info](https://uops.info/table.html), [Agner Fog's instruction tables](https://www.agner.org/optimize/instruction_tables.pdf)
- Compiler output: [Compiler Explorer](https://godbolt.org/), [llvm-mca](https://llvm.org/docs/CommandGuide/llvm-mca.html)
- Tracing: [bpftrace](https://github.com/bpftrace/bpftrace), [bcc](https://github.com/iovisor/bcc)

Book references below: "Bakhvalov ch. N" is a chapter of *Performance Analysis and Tuning on Modern CPUs* ([source and chapters on GitHub](https://github.com/dendibakh/perf-book)). "Drepper N.N" is a section of [*What Every Programmer Should Know About Memory*](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf). "Gregg ch. N" is *Systems Performance* (2nd ed.). Agner Fog's manuals change chapter numbers between versions, so look up the chapter by title.

---

## CPU

- [ ] Pipeline, out-of-order execution, ROB, ports; reading uops.info and Agner Fog's tables
  - **Read:** [Bakhvalov ch. 3, CPU Microarchitecture](https://github.com/dendibakh/perf-book/tree/main/chapters/3-CPU-Microarchitecture) (pipelining, ILP sections). [Agner Fog, microarchitecture manual](https://www.agner.org/optimize/microarchitecture.pdf): "Out-of-order execution" and the chapter for my core (Zen or the matching Intel "Lake").
  - **Do:** time two loops with `perf stat -e cycles,instructions`: one long dependency chain of `imul`, and four independent chains. Before running, predict cycles per iteration from [uops.info](https://uops.info/table.html) latency and throughput, and run the loop through `llvm-mca`.
  - **Break:** make the chain go through memory (store, then reload the same address) and watch cycles per iteration jump to the store-forwarding latency.
  - **Know:** I can predict a small loop's cycles per iteration within about 20% before running it, and say whether it is latency-bound or throughput-bound.

- [ ] Branch prediction, mispredict cost, branchless techniques
  - **Read:** [Bakhvalov ch. 10, Optimizing Branch Prediction](https://github.com/dendibakh/perf-book/tree/main/chapters/10-Optimizing-Branch-Prediction). [Algorithmica: branching](https://en.algorithmica.org/hpc/pipelining/branching/) and [branchless programming](https://en.algorithmica.org/hpc/pipelining/branchless/). Intro: [Dan Luu, branch prediction](https://danluu.com/branch-prediction/).
  - **Do:** `if (x[i] < 128) sum += x[i]` over random bytes, sorted vs unsorted, with `perf stat -e cycles,branches,branch-misses`. Mispredict cost = extra cycles / extra misses. Then a branchless version; confirm in Compiler Explorer that it really is `cmov` or SIMD.
  - **Break:** reuse one small random array for every iteration and watch the predictor learn it ([Lemire, "Benchmarking is hard: processors learn to predict branches"](https://lemire.me/blog/2019/10/16/benchmarking-is-hard-processors-learn-to-predict-branches/)).
  - **Know:** my machine's measured mispredict penalty in cycles, and a case where branchless is slower.

- [ ] SIMD: SSE/AVX2/AVX-512, auto-vectorization and when it fails, intrinsics
  - **Read:** [Algorithmica: SIMD](https://en.algorithmica.org/hpc/simd/), especially [auto-vectorization](https://en.algorithmica.org/hpc/simd/auto-vectorization/). [Bakhvalov ch. 9](https://github.com/dendibakh/perf-book/tree/main/chapters/9-Optimizing-Computations) (Vectorization, Compiler Intrinsics). Agner Fog [optimizing_cpp.pdf](https://www.agner.org/optimize/optimizing_cpp.pdf), "Using vector operations". Reference: [Intel Intrinsics Guide](https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html).
  - **Do:** sum and dot product over `float`: scalar `-O2`, `-O3 -march=native`, then AVX2 intrinsics. Read GCC's `-fopt-info-vec-missed` ([developer options](https://gcc.gnu.org/onlinedocs/gcc/Developer-Options.html)) or Clang's `-Rpass-missed=loop-vectorize` ([LLVM vectorizers](https://llvm.org/docs/Vectorizers.html)). Measure GB/s at L1-sized and DRAM-sized arrays.
  - **Break:** add a possibly-aliasing output pointer, or a float reduction without `-ffast-math`, and read the missed-vectorization reason.
  - **Know:** three reasons a loop fails to vectorize, and the vector vs scalar speedup in L1 vs DRAM on my machine, with why it shrinks. Optional: [Travis Downs on AVX-512 frequency](https://travisdowns.github.io/blog/2020/08/19/icl-avx512-freq.html).

- [ ] Top-down microarchitecture analysis (frontend / backend / bad speculation / retiring)
  - **Read:** [Bakhvalov ch. 6](https://github.com/dendibakh/perf-book/tree/main/chapters/6-CPU-Features-For-Performance-Analysis) (Top-Down Microarchitecture Analysis, TMA on Intel and on AMD). [easyperf: Top-Down performance analysis methodology](https://easyperf.net/blog/2019/02/09/Top-Down-performance-analysis-methodology). [toplev manual](https://github.com/andikleen/pmu-tools/wiki/toplev-manual) on Intel.
  - **Do:** run top-down level 1 (`perf stat --topdown` where supported, `toplev` on Intel, or the AMD method from Bakhvalov) on four earlier programs: dependent `imul` chain, random-branch filter, in-L1 SIMD sum, DRAM pointer chase. Predict the dominant bucket for each first.
  - **Break:** shrink the pointer chase to fit in L1 and watch the program move from backend-bound to retiring.
  - **Know:** I predicted the top-level category correctly for all four, and can say which counters drive each bucket.

## Memory

- [ ] Cache hierarchy, lines, associativity, inclusive vs exclusive, write-back
  - **Read:** Drepper 3.3.1 to 3.3.3 (Associativity, Measurements of Cache Effects, Write Behavior), also as [LWN part 2](https://lwn.net/Articles/252125/). [Algorithmica: associativity](https://en.algorithmica.org/hpc/cpu-cache/associativity/). Light intro: [Igor Ostrovsky, Gallery of Processor Cache Effects](http://igoro.com/archive/gallery-of-processor-cache-effects/).
  - **Do (SI-02):** pointer chase over a random cyclic permutation, one node per 64-byte line, working sets from 4 KiB to 1 GiB. Plot ns and cycles per load. Compare the steps with `lscpu -C` and the vendor's cache sizes.
  - **Break:** access with a 4096-byte stride so every access maps to the same set; watch the effective cache size collapse.
  - **Know:** L1, L2, L3 and DRAM load latency for my machine in ns and cycles, and which curve step matches which cache.

- [ ] Hardware prefetchers and access patterns they handle
  - **Read:** Drepper 6.3 (Prefetching, especially 6.3.1 Hardware Prefetching and 6.3.2 Software Prefetching). [Bakhvalov ch. 8](https://github.com/dendibakh/perf-book/tree/main/chapters/8-Optimizing-Memory-Accesses) (Memory Prefetching). [Algorithmica: prefetching](https://en.algorithmica.org/hpc/cpu-cache/prefetching/).
  - **Do (SI-02):** same 1 GiB buffer, ns per access for sequential, fixed strides (64 B to 4 KiB) and random order. Then add `__builtin_prefetch` at a few distances to the random-index version (addresses known in advance) and find the best distance.
  - **Break:** strides that cross a 4 KiB page each step; check Drepper's statement about prefetchers and page boundaries against my numbers.
  - **Know:** which patterns my prefetcher hides, and the sequential vs random ns-per-access ratio at DRAM size.

- [ ] TLB, page walks, huge pages (THP vs explicit), `mlock`
  - **Read:** Drepper 4.3 (Optimizing Page Table Access), also as [LWN part 3](https://lwn.net/Articles/253361/). Kernel docs: [transparent huge pages](https://docs.kernel.org/admin-guide/mm/transhuge.html), [HugeTLB pages](https://docs.kernel.org/admin-guide/mm/hugetlbpage.html). [Algorithmica: paging](https://en.algorithmica.org/hpc/cpu-cache/paging/). [mlock(2)](https://man7.org/linux/man-pages/man2/mlock.2.html).
  - **Do (SI-02):** rerun the pointer chase at 256 MiB to 1 GiB with 4 KiB pages, `madvise(MADV_HUGEPAGE)`, and `MAP_HUGETLB`. Verify the benchmark mapping in `/proc/PID/smaps`. Check `AnonHugePages` for PMD-sized THP and the HugeTLB fields for explicit HugeTLB. Count dTLB misses with `perf stat` (check `perf list` for the event names on my CPU).
  - **Break:** Repeat with `MADV_NOHUGEPAGE` on a new anonymous mapping before touching it. Verify the backing and compare the curve.
  - **Know:** my TLB reach (entries times page size) from the vendor docs, and the measured latency difference between 4 KiB and 2 MiB pages at 1 GiB.

- [ ] Coherence (MESI/MOESI), false sharing, cross-core and cross-socket line transfer cost
  - **Read:** [McKenney's perfbook](https://mirrors.edge.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.html), Appendix C "Why Memory Barriers?" (MESI) and ch. 3 "Hardware and its Habits". Drepper 3.3.4 (Multi-Processor Support) and 6.4.1 (Concurrency Optimizations). [Mara Bos, ch. 7 Understanding the Processor](https://marabos.nl/atomics/hardware.html).
  - **Do (SI-02):** two threads incrementing counters in one cache line vs padded apart; confirm with `perf c2c` ([Joe Mario's guide](https://joemario.github.io/blog/2016/09/01/c2c-blog/), [perf-c2c(1)](https://man7.org/linux/man-pages/man1/perf-c2c.1.html)). Then a ping-pong core-to-core latency matrix; cross-check with [core-to-core-latency](https://github.com/nviennot/core-to-core-latency).
  - **Break:** pad to 64 bytes vs 128 bytes and check whether adjacent-line prefetch still causes interference.
  - **Know:** round-trip line transfer latency for SMT siblings, same cluster or CCX, and cross cluster (cross socket on the server), and the false-sharing slowdown factor, explained in MESI terms.

- [ ] NUMA: topology, first-touch, `numactl`, remote access penalty
  - **Read:** Drepper 5 (NUMA Support, especially 5.4 Remote Access Costs) and 6.5 (NUMA Programming), also as [LWN part 4](https://lwn.net/Articles/254445/). [Kernel NUMA doc](https://docs.kernel.org/mm/numa.html). [numactl(8)](https://man7.org/linux/man-pages/man8/numactl.8.html), [numa(7)](https://man7.org/linux/man-pages/man7/numa.7.html).
  - **Do (SI-02, server-class machine):** `numactl --hardware`, then pointer-chase latency and streaming bandwidth with `--cpunodebind=0 --membind=0` vs `--membind=1`. A single-node desktop only shows one row; the exit criterion needs a server anyway.
  - **Break:** allocate and initialize a buffer from a thread on node 0, then read it from node 1 (first touch); compare with initializing on node 1.
  - **Know:** local vs remote latency and bandwidth ratio on the server, and a one-paragraph explanation of first-touch placement.

- [ ] Memory ordering: C++ memory model, x86-TSO vs ARM, fences, acquire/release in practice
  - **Read:** Preshing: [Memory Reordering Caught in the Act](https://preshing.com/20120515/memory-reordering-caught-in-the-act/) and [Acquire and Release Semantics](https://preshing.com/20120913/acquire-and-release-semantics/). [cppreference: memory_order](https://en.cppreference.com/w/cpp/atomic/memory_order). [x86-TSO paper (Sewell et al., CACM)](https://www.cl.cam.ac.uk/~pes20/weakmemory/cacm.pdf). perfbook ch. 15 "Advanced Synchronization: Memory Ordering". [Mara Bos, ch. 3 Memory Ordering](https://marabos.nl/atomics/memory-ordering.html).
  - **Do:** reproduce Preshing's store-load reordering test on x86 and count how often both threads read 0. Add a `seq_cst` fence and confirm the count drops to zero. In Compiler Explorer, compare a `seq_cst` store, a `release` store and a `relaxed` store on x86-64 and ARM64, using the [C++ to hardware mappings](https://www.cl.cam.ac.uk/~pes20/cpp/cpp0xmappings.html).
  - **Break:** make the SPSC ring indices `relaxed`. Explain why it may pass tests on x86 and is still wrong (compiler reordering, ARM).
  - **Know:** the one reordering x86-TSO allows, why acquire/release costs nothing extra on x86 but not on ARM, and the measured cost of a `seq_cst` store vs a `release` store on my machine.

## Operating system

- [ ] Scheduler (CFS/EEVDF), preemption, context-switch cost, `SCHED_FIFO`
  - **Read:** Kernel docs: [EEVDF](https://docs.kernel.org/scheduler/sched-eevdf.html), [CFS](https://docs.kernel.org/scheduler/sched-design-CFS.html), [RT group scheduling and throttling](https://docs.kernel.org/scheduler/sched-rt-group.html). [LWN: An EEVDF CPU scheduler for Linux](https://lwn.net/Articles/925371/). [sched(7)](https://man7.org/linux/man-pages/man7/sched.7.html). Gregg ch. 6 (CPUs).
  - **Do (SI-03):** context-switch cost with `perf bench sched pipe` ([perf-bench(1)](https://man7.org/linux/man-pages/man1/perf-bench.1.html)) pinned to one core vs two cores, then my own pipe or futex ping-pong timed with benchkit. Compare with [Tsuna's measurements](https://blog.tsunanet.net/2010/11/how-long-does-it-take-to-make-context.html).
  - **Break:** run a `SCHED_FIFO` busy loop (`chrt -f 50`, on a spare core) and see RT throttling as regular gaps in the jitter tool; read `/proc/sys/kernel/sched_rt_runtime_us` to explain them.
  - **Know:** context-switch cost in ns, same core vs cross core, and why a cross-core wakeup costs more.

- [ ] Syscall cost, vDSO, `io_uring` as a syscall-batching interface
  - **Read:** [vdso(7)](https://man7.org/linux/man-pages/man7/vdso.7.html). [io_uring(7)](https://man7.org/linux/man-pages/man7/io_uring.7.html). Jens Axboe, [Efficient IO with io_uring](https://kernel.dk/io_uring.pdf). [LWN: Ringing in a new asynchronous I/O API](https://lwn.net/Articles/776703/). Practical guide: [Lord of the io_uring](https://unixism.net/loti/).
  - **Do:** time `syscall(SYS_getppid)` (a real syscall) vs `clock_gettime` (vDSO) in a pinned loop. Then submit N `IORING_OP_NOP` requests per `io_uring_enter` with [liburing](https://github.com/axboe/liburing), N from 1 to 256, and plot ns per operation.
  - **Break:** switch the clocksource to one the vDSO cannot read (for example `hpet`, if available) in `/sys/devices/system/clocksource/clocksource0/current_clocksource` and watch `clock_gettime` become a syscall. Switch it back afterwards.
  - **Know:** ns per syscall and per vDSO call on my box, and the batch size at which io_uring beats one syscall per operation.

- [ ] Interrupts, softirqs, NAPI, IRQ affinity
  - **Read:** Kernel docs: [NAPI](https://docs.kernel.org/networking/napi.html), [IRQ affinity](https://docs.kernel.org/core-api/irq/irq-affinity.html), [/proc filesystem](https://docs.kernel.org/filesystems/proc.html) (`/proc/interrupts`, `/proc/softirqs`). The softirq and NAPI parts of the [packagecloud receive-path guide](https://blog.packagecloud.io/monitoring-tuning-linux-networking-stack-receiving-data/).
  - **Do (SI-03):** diff `/proc/interrupts` and `/proc/softirqs` over 10 seconds, idle vs under network load. Count interrupts per CPU with `bpftrace` on the `irq:irq_handler_entry` tracepoint. Move NIC IRQs off the measurement core with `/proc/irq/N/smp_affinity_list`.
  - **Break:** pin the NIC IRQ onto the measured core and watch the jitter histogram change.
  - **Know:** which interrupts hit my isolated core, how often, and the gap size each one causes.

- [ ] Virtual memory: page faults, `mmap`, page cache, writeback
  - **Read:** [Kernel memory management concepts](https://docs.kernel.org/admin-guide/mm/concepts.html). Drepper 7.5 (Page Fault Optimization). [mmap(2)](https://man7.org/linux/man-pages/man2/mmap.2.html). [vm sysctls](https://docs.kernel.org/admin-guide/sysctl/vm.html) (the `dirty_*` writeback knobs). Gregg ch. 7 (Memory) and ch. 8 (File Systems).
  - **Do:** `mmap` 1 GiB anonymous memory, touch one byte per page, and time per page; check with `perf stat -e page-faults`. Repeat with `MAP_POPULATE` and with huge pages. For major faults, read an `mmap`ed file after dropping caches (`echo 3 | sudo tee /proc/sys/vm/drop_caches`) vs with a warm cache.
  - **Break:** write a multi-GiB file with buffered writes while a small latency-sensitive loop runs; watch `Dirty` and `Writeback` in `/proc/meminfo` and the loop's p99.
  - **Know:** minor-fault cost per 4 KiB page on my box, and why hot paths pre-fault and `mlock` their memory.

- [ ] Isolation: `isolcpus`, `nohz_full`, `rcu_nocbs`, cgroups, C-states and P-states
  - **Read:** [Kernel parameters](https://docs.kernel.org/admin-guide/kernel-parameters.html) (`isolcpus`, `nohz_full`, `rcu_nocbs`). [NO_HZ doc](https://docs.kernel.org/timers/no_hz.html). [cgroup v2](https://docs.kernel.org/admin-guide/cgroup-v2.html) (cpuset). [cpuidle](https://docs.kernel.org/admin-guide/pm/cpuidle.html) and [cpufreq](https://docs.kernel.org/admin-guide/pm/cpufreq.html). [Erik Rigtorp, low-latency tuning guide](https://rigtorp.se/low-latency-guide/).
  - **Do (SI-03):** this is jitterlab itself. A tight loop on one core records gaps above a threshold; apply one knob at a time and keep a gap histogram for each. Cross-check with [sysjitter](https://github.com/Xilinx-CNS/cns-sysjitter). Read C-state exit latencies from `/sys/devices/system/cpu/cpu*/cpuidle/state*/latency`.
  - **Break:** after tuning, start a periodic timer or second task on the isolated core and confirm the tick and the gaps come back.
  - **Know:** the knobs ranked by measured effect on p99.99 gap and max, and what `nohz_full` needs (a single runnable task) to stop the tick.

## Networking

- [ ] Linux receive path end to end: NIC → DMA ring → IRQ/NAPI → skb → socket → user
  - **Read:** [packagecloud: Monitoring and Tuning the Linux Networking Stack: Receiving Data](https://blog.packagecloud.io/monitoring-tuning-linux-networking-stack-receiving-data/). [Kernel scaling doc](https://docs.kernel.org/networking/scaling.html). [Cloudflare: How to receive a million packets per second](https://blog.cloudflare.com/how-to-receive-a-million-packets/). Gregg ch. 10 (Network).
  - **Do:** send UDP between two machines and record the counter at each stage: `ethtool -S` (NIC and ring), `/proc/net/softnet_stat` (softirq backlog), `/proc/net/snmp` (`Udp: InErrors`, `RcvbufErrors`). Measure kernel-to-user latency by comparing a software receive timestamp (`SO_TIMESTAMPNS`, see [socket(7)](https://man7.org/linux/man-pages/man7/socket.7.html)) with `clock_gettime` right after `recvmsg`.
  - **Break:** set a tiny `SO_RCVBUF`, flood the socket and watch `RcvbufErrors` climb.
  - **Know:** I can draw the path from memory with the counter I would check at each stage (this is the exit-criteria notes page).

- [ ] TCP internals that matter for latency: Nagle, delayed ACK, congestion control, buffer sizing
  - **Read:** [tcp(7)](https://man7.org/linux/man-pages/man7/tcp.7.html) (`TCP_NODELAY`, `TCP_QUICKACK`, `tcp_rmem`, `tcp_wmem`). [ip-sysctl doc](https://docs.kernel.org/networking/ip-sysctl.html). [Marc Brooker, "It's always TCP_NODELAY"](https://brooker.co.za/blog/2024/05/09/nagle.html). [Cloudflare: The story of one latency spike](https://blog.cloudflare.com/the-story-of-one-latency-spike/).
  - **Do:** request/response ping-pong over the LAN where each message is sent as two small writes (header, then body), with and without `TCP_NODELAY`. Look for the Nagle plus delayed-ACK stall in the histogram. Use `ss -ti` to read cwnd and RTT, and `sysctl net.ipv4.tcp_congestion_control`.
  - **Break:** add delay and loss with [netem](https://man7.org/linux/man-pages/man8/tc-netem.8.html) (`tc qdisc add dev X root netem delay 1ms loss 0.1%`) and watch p99 and retransmits. Remove it afterwards.
  - **Know:** the stall size I measured and why it happens, and why order entry uses TCP with `TCP_NODELAY` while market data uses UDP.

- [ ] UDP multicast, IGMP, packet loss and gap detection
  - **Read:** [ip(7)](https://man7.org/linux/man-pages/man7/ip.7.html) (`IP_ADD_MEMBERSHIP`, `IP_MULTICAST_LOOP`, `IP_MULTICAST_TTL`). [RFC 3376, IGMPv3](https://www.rfc-editor.org/rfc/rfc3376) (membership reports). The Cloudflare million-packets post above for receive-side limits.
  - **Do (prep for SI-06):** a sender publishes sequence-numbered packets to a multicast group; a receiver detects and logs gaps. Watch the join with `tcpdump -i X igmp`.
  - **Break:** inject loss and reordering with netem on the sender, and separately make the consumer slow with a small `SO_RCVBUF`. The gap detector must report exactly the injected loss; cross-check with `/proc/net/snmp`.
  - **Know:** where loss can happen (network, NIC ring, socket buffer), which counter shows each, and a detector proven against injected faults.

- [ ] NIC features: RSS, flow steering, timestamping, offloads
  - **Read:** [Kernel scaling doc](https://docs.kernel.org/networking/scaling.html) (RSS, RPS, RFS, aRFS, XPS). [Timestamping](https://docs.kernel.org/networking/timestamping.html). [Segmentation offloads](https://docs.kernel.org/networking/segmentation-offloads.html). [ethtool(8)](https://man7.org/linux/man-pages/man8/ethtool.8.html).
  - **Do:** on each machine record `ethtool -i`, `-k` (offloads), `-l` (channels), `-x` (RSS indirection) and `-T` (timestamping capabilities) in the log. Measure small-packet UDP latency and CPU use with GRO on vs off.
  - **Break:** reduce to one channel (`ethtool -L`) and watch all receive IRQs land on one core.
  - **Know:** my NICs' queue counts, whether they support hardware RX timestamps, and the latency vs throughput trade of GRO.

## Languages

- [ ] C++: allocation-free hot paths, `std::pmr`, layout control, `[[likely]]`, link-time and profile-guided optimization
  - **Read:** cppreference: [polymorphic_allocator](https://en.cppreference.com/w/cpp/memory/polymorphic_allocator), [monotonic_buffer_resource](https://en.cppreference.com/w/cpp/memory/monotonic_buffer_resource), [`[[likely]]`](https://en.cppreference.com/w/cpp/language/attributes/likely). GCC [optimize options](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html) (`-flto`, `-fprofile-generate`, `-fprofile-use`) and the [Clang users manual](https://clang.llvm.org/docs/UsersManual.html) (profile-guided optimization). Talk: [Carl Cook, "When a Microsecond Is an Eternity" (CppCon 2017)](https://www.youtube.com/watch?v=NH1Tta7purM).
  - **Do:** pick one hot path in modern_exchng_cpp (for example order add/cancel), count allocations in it with a counting `operator new`, then remove them with pmr containers over a pre-sized buffer. Measure p50/p99 with benchkit. Then build with LTO, then PGO, and record each delta.
  - **Break:** give the monotonic resource `std::pmr::null_memory_resource()` as upstream and shrink the buffer, so any hidden allocation throws instead of silently hitting the heap.
  - **Know:** proof that the hot path allocates zero times, and the measured LTO and PGO deltas.

- [ ] Rust: same patterns in Rust (`#[repr]`, `unsafe` boundaries, atomics), for side-by-side comparison
  - **Read:** [Rust reference: type layout](https://doc.rust-lang.org/reference/type-layout.html) (`repr(C)`, `repr(align)`). [Rustonomicon](https://doc.rust-lang.org/nomicon/). [Mara Bos, *Rust Atomics and Locks*](https://marabos.nl/atomics/) (free online). [std::sync::atomic](https://doc.rust-lang.org/std/sync/atomic/).
  - **Do:** port the false-sharing demo and the SPSC ring to Rust with `#[repr(C, align(64))]`. Benchmark with the same pinning and percentiles as the C++ version ([criterion](https://github.com/bheisler/criterion.rs) is fine for throughput, but compare percentiles from the same harness). Compare the generated assembly with [cargo-show-asm](https://github.com/pacak/cargo-show-asm) and Compiler Explorer.
  - **Break:** use `Ordering::Relaxed` on the ring indices and write down why it is wrong even if tests pass.
  - **Know:** the invariant every `unsafe` block relies on, written next to it, and C++ vs Rust numbers that are within noise or explained.

- [ ] Reading compiler output: Compiler Explorer, `objdump`, spotting missed optimizations
  - **Read:** [Compiler Explorer](https://godbolt.org/). Talk: [Matt Godbolt, "What Has My Compiler Done for Me Lately?" (CppCon 2017)](https://www.youtube.com/watch?v=bSkpMdDe4g4). [Algorithmica: compilation](https://en.algorithmica.org/hpc/compilation/). [objdump(1)](https://man7.org/linux/man-pages/man1/objdump.1.html), [perf-annotate(1)](https://man7.org/linux/man-pages/man1/perf-annotate.1.html), [llvm-mca](https://llvm.org/docs/CommandGuide/llvm-mca.html).
  - **Do:** for each exercise above, put the hot loop in Compiler Explorer with my exact compiler and flags, then `perf annotate` the real binary and check that the hot instructions match. Find one missed optimization (aliasing, a bounds check, an indirect call through `std::function`) and fix it with a source change only.
  - **Break:** compile the same loop at `-O0`, `-O2` and `-O3 -march=native` and explain each difference that matters for speed.
  - **Know:** I can read the assembly of a 20-line hot loop and explain each instruction, and I have one documented missed optimization with before/after numbers.

## Resources

- ★ Ulrich Drepper, *What Every Programmer Should Know About Memory*: [PDF](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf), or the LWN series starting at [part 1](https://lwn.net/Articles/250967/) ([2 caches](https://lwn.net/Articles/252125/), [3 virtual memory](https://lwn.net/Articles/253361/), [4 NUMA](https://lwn.net/Articles/254445/), [5](https://lwn.net/Articles/255364/) and [6 what programmers can do](https://lwn.net/Articles/256433/), [7 tools](https://lwn.net/Articles/257209/))
- ★ Denis Bakhvalov, *Performance Analysis and Tuning on Modern CPUs* (free, [easyperf.net](https://easyperf.net/); [source and chapters](https://github.com/dendibakh/perf-book))
- Agner Fog's [optimization manuals and instruction tables](https://www.agner.org/optimize/)
- Intel 64 and IA-32 Optimization Reference Manual ([Intel optimization page](https://www.intel.com/content/www/us/en/developer/articles/technical/intel64-and-ia32-architectures-optimization.html); the [SDM](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html) for instruction and PMU details)
- Brendan Gregg, *Systems Performance* (2nd ed.) ([book page](https://www.brendangregg.com/systems-performance-2nd-edition-book.html); [Linux performance](https://www.brendangregg.com/linuxperf.html))
- Paul McKenney, *Is Parallel Programming Hard, And, If So, What Can You Do About It?* ([free PDF](https://mirrors.edge.kernel.org/pub/linux/kernel/people/paulmck/perfbook/perfbook.html))

Also used above:

- Sergey Slotin, [*Algorithms for Modern Hardware* (Algorithmica)](https://en.algorithmica.org/hpc/): short, measurement-driven chapters on pipelining, caches and SIMD
- Mara Bos, [*Rust Atomics and Locks*](https://marabos.nl/atomics/): clearest treatment of memory ordering and what atomics compile to, useful for C++ too
- [uops.info](https://uops.info/table.html): measured latency, throughput and ports per instruction and microarchitecture
- Kernel documentation at [docs.kernel.org](https://docs.kernel.org/admin-guide/kernel-parameters.html) and man pages at [man7.org](https://man7.org/linux/man-pages/man7/sched.7.html): the primary source for every OS and networking topic
- [Erik Rigtorp, low-latency tuning guide](https://rigtorp.se/low-latency-guide/): checklist to compare against my own SI-03 measurements, not a substitute for them

## Exit criteria

- [ ] SI-02 and SI-03 published with numbers from at least one desktop and one server-class machine
- [ ] A notes page explaining the Linux receive path, written from memory
