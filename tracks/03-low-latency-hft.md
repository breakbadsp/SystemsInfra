# Track 3: Low-latency and HFT infra

The full path from a packet on the wire to an order on the wire, and every microsecond in between. Builds on modern_exchng_cpp (exchange side) by now building the participant side.

The live week-by-week plan is [`ROADMAP.md`](../ROADMAP.md). Week 10 is the Linux receive path and an AF_XDP reading, not SI-07. SI-06, SI-07 and SI-08 are low-latency trading depth after week 13, if that specialization is chosen. Keep ticking checkboxes here.

## Market structure

- [ ] Order types, matching rules, auctions, market data levels (L1/L2/L3)
- [ ] Exchange protocols: NASDAQ ITCH/OUCH, CME MDP 3.0 (SBE), FIX/FAST
- [ ] Colocation, cross-connects, microwave vs fibre, why venues publish via multicast A/B feeds

## Market data

- [ ] Feed handlers: decoding, sequence gaps, A/B line arbitration, snapshot recovery
- [ ] Order book building at L3 for thousands of symbols; cache-friendly book layouts
- [ ] Normalization and fan-out to strategies (shared memory, SPSC rings)
- [ ] → SI-06

## Network path

- [ ] Kernel tuning for UDP receive: busy polling (`SO_BUSY_POLL`), IRQ affinity, ring sizes, coalescing off
- [ ] Kernel bypass: AF_XDP, DPDK, OpenOnload / ef_vi, RDMA verbs
- [ ] Zero-copy, `SO_TIMESTAMPING` software vs hardware timestamps
- [ ] → SI-07

## Time

- [ ] PTP (IEEE 1588), linuxptp (`ptp4l`, `phc2sys`), boundary and transparent clocks
- [ ] Clock accuracy budget; MiFID II timestamping requirements as a reference point
- [ ] TSC vs PHC; correlating timestamps across hosts

## The order path

- [ ] Strategy → risk checks → order gateway; pre-trade risk without adding latency
- [ ] Order entry protocols: OUCH, FIX, binary venue protocols
- [ ] Tick-to-trade measurement with timestamps at every hop → SI-08

## Host and hardware

- [ ] Server tuning: BIOS (C-states, turbo, SMT, power profiles), `isolcpus`/`nohz_full`, SMI awareness
- [ ] NIC selection: what matters for latency (Solarflare/AMD, NVIDIA/Mellanox, Exablaze)
- [ ] FPGA in trading: what moves to hardware (feed decoding, risk checks, order triggers) and why; read-level understanding, not building

## Resources

- ★ Larry Harris, *Trading and Exchanges*
- ★ Carl Cook, "When a Microsecond Is an Eternity" (CppCon 2017)
- ★ Erik Rigtorp's low-latency tuning guide (rigtorp.se)
- NASDAQ TotalView-ITCH 5.0 and OUCH specifications (NASDAQ publishes sample ITCH files)
- DPDK documentation; Linux kernel AF_XDP documentation
- linuxptp documentation
- LMAX Disruptor paper (Martin Thompson et al.)
- David Gross, "Trading at light speed" (Meeting C++ 2022)
- Red Hat low-latency performance tuning guide for RHEL

## Hardware note

AF_XDP works on most modern NICs with driver support. DPDK and ef_vi need a supported NIC (or a VM with virtio for functional work, though latency numbers from a VM aren't meaningful). Hardware timestamping needs a NIC with a PHC. Rent bare metal for final numbers if the home setup can't do it.

## Exit criteria

- [ ] SI-06, SI-07, SI-08 done, with a per-hop latency breakdown
- [ ] A notes page on PTP and timestamp correlation
