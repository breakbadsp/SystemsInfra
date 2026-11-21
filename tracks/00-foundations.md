# Track 0: Foundations

The hardware and OS model everything else rests on. Much of this is already familiar from trading work; the aim here is to **measure** it, not just know it. Measurements land in SI-02 and SI-03.

## CPU

- [ ] Pipeline, out-of-order execution, ROB, ports; reading uops.info and Agner Fog's tables
- [ ] Branch prediction, mispredict cost, branchless techniques
- [ ] SIMD: SSE/AVX2/AVX-512, auto-vectorization and when it fails, intrinsics
- [ ] Top-down microarchitecture analysis (frontend / backend / bad speculation / retiring)

## Memory

- [ ] Cache hierarchy, lines, associativity, inclusive vs exclusive, write-back
- [ ] Hardware prefetchers and access patterns they handle
- [ ] TLB, page walks, huge pages (THP vs explicit), `mlock`
- [ ] Coherence (MESI/MOESI), false sharing, cross-core and cross-socket line transfer cost
- [ ] NUMA: topology, first-touch, `numactl`, remote access penalty
- [ ] Memory ordering: C++ memory model, x86-TSO vs ARM, fences, acquire/release in practice

## Operating system

- [ ] Scheduler (CFS/EEVDF), preemption, context-switch cost, `SCHED_FIFO`
- [ ] Syscall cost, vDSO, `io_uring` as a syscall-batching interface
- [ ] Interrupts, softirqs, NAPI, IRQ affinity
- [ ] Virtual memory: page faults, `mmap`, page cache, writeback
- [ ] Isolation: `isolcpus`, `nohz_full`, `rcu_nocbs`, cgroups, C-states and P-states

## Networking

- [ ] Linux receive path end to end: NIC → DMA ring → IRQ/NAPI → skb → socket → user
- [ ] TCP internals that matter for latency: Nagle, delayed ACK, congestion control, buffer sizing
- [ ] UDP multicast, IGMP, packet loss and gap detection
- [ ] NIC features: RSS, flow steering, timestamping, offloads

## Languages

- [ ] C++: allocation-free hot paths, `std::pmr`, layout control, `[[likely]]`, link-time and profile-guided optimization
- [ ] Rust: same patterns in Rust (`#[repr]`, `unsafe` boundaries, atomics), for side-by-side comparison
- [ ] Reading compiler output: Compiler Explorer, `objdump`, spotting missed optimizations

## Resources

- ★ Ulrich Drepper, *What Every Programmer Should Know About Memory*
- ★ Denis Bakhvalov, *Performance Analysis and Tuning on Modern CPUs* (free, easyperf.net)
- Agner Fog's optimization manuals and instruction tables
- Intel 64 and IA-32 Optimization Reference Manual
- Brendan Gregg, *Systems Performance* (2nd ed.)
- Paul McKenney, *Is Parallel Programming Hard, And, If So, What Can You Do About It?*

## Exit criteria

- [ ] SI-02 and SI-03 published with numbers from at least one desktop and one server-class machine
- [ ] A notes page explaining the Linux receive path, written from memory
