# Track 2: Reliability and correctness

Systems that stay correct when processes crash, disks lie, clocks drift and networks partition. The breadth labs cover overload, packet loss, model checking, and restart recovery. The depth projects are SI-10 and SI-11.

The live week-by-week plan is [`ROADMAP.md`](../ROADMAP.md).

## Failure models

- [ ] Crash-stop, crash-recovery, omission, Byzantine; which one each system assumes
- [ ] Gray failures, fail-slow disks and NICs, metastable failures
- [ ] Storage faults: torn writes, misdirected writes, latent sector errors, fsync failure semantics
- [ ] Clock faults: drift, steps, leap seconds, NTP vs PTP behaviour under faults

## Consensus and replication

- [ ] Viewstamped Replication Revisited (already implemented in libvsr: revisit view change and recovery)
- [ ] Raft, Multi-Paxos; compare against VSR
- [ ] Primary-backup and chain replication
- [ ] Linearizability vs serializability vs sequential consistency
- [ ] Leases and fencing tokens

## Testing

- [ ] Deterministic simulation testing (FoundationDB, TigerBeetle VOPR, libvsr's simulator)
- [ ] Linearizability checking (Wing & Gong, Knossos, Porcupine) → SI-10
- [ ] Fault injection: network namespaces, `tc netem`, iptables partitions, `dm-flakey`, process kill, clock skew
- [ ] Property-based testing and fuzzing (libFuzzer, AFL++), sanitizers in CI
- [ ] Crash-consistency testing of storage (ALICE-style) → SI-11

## Verification

- [ ] TLA+ basics; model one protocol (e.g. libvsr view change) and model-check it
- [ ] Assertions as design: TigerStyle, pre/postconditions, C++26 contracts

## Operations

- [ ] SLOs, error budgets, alerting on symptoms not causes
- [ ] Graceful degradation, load shedding, backpressure, retry budgets
- [ ] Postmortems: write one for a real incident (work or project)

## Resources

- ★ Martin Kleppmann, *Designing Data-Intensive Applications*
- ★ Liskov and Cowling, "Viewstamped Replication Revisited" (2012)
- ★ Will Wilson, "Testing Distributed Systems with Deterministic Simulation" (Strange Loop 2014)
- Jepsen analyses (jepsen.io)
- Yuan et al., "Simple Testing Can Prevent Most Critical Failures" (OSDI '14)
- Pillai et al., "All File Systems Are Not Created Equal" (OSDI '14)
- Ongaro and Ousterhout, Raft paper
- TigerBeetle docs and TigerStyle
- Google, *Site Reliability Engineering*

## Exit criteria

- [ ] Every project in this repo has a fault or crash test, listed in its `RESULTS.md`
- [ ] SI-10 and SI-11 done
- [ ] One TLA+ model checked
