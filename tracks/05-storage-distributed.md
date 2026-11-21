# Track 5: Storage and distributed systems

Durable state: how bytes reach the disk safely and quickly, and how state stays consistent across machines.

## Storage hardware and kernel

- [ ] NVMe: queues, namespaces, write amplification, FTL behaviour, power-loss protection
- [ ] Page cache vs `O_DIRECT`; `fsync`/`fdatasync`/`sync_file_range` semantics and their costs
- [ ] `io_uring` in depth: SQPOLL, registered buffers and files, linked ops, fixed-file I/O
- [ ] File system guarantees (ext4, XFS) and what is and isn't atomic

## Storage engines

- [ ] Write-ahead logging, group commit, checksums, recovery
- [ ] B-trees vs LSM trees: read, write and space amplification
- [ ] Compaction strategies (leveled, tiered), bloom filters, block cache
- [ ] Why DBMSs avoid `mmap` for buffer management
- [ ] → SI-11

## Distributed systems

- [ ] Replication: VSR/Raft (libvsr), chain replication, quorum systems
- [ ] Partitioning and rebalancing, consistent hashing
- [ ] Transactions: 2PC, Percolator-style, deterministic databases (Calvin)
- [ ] Clocks: Lamport, vector clocks, hybrid logical clocks, TrueTime
- [ ] Case studies: TigerBeetle, FoundationDB, Kafka, Spanner

## Resources

- ★ Alex Petrov, *Database Internals*
- ★ "High-Performance DBMSs with io_uring" (VLDB '26)
- Crotty, Leis, Pavlo, "Are You Sure You Want to Use MMAP in Your DBMS?" (CIDR '22)
- O'Neil et al., the LSM-tree paper; RocksDB wiki
- Dan Luu, "Files are hard"
- TigerBeetle source (`src/io`, `src/vsr`, storage layer)
- CMU 15-445 / 15-721 lecture videos

## Exit criteria

- [ ] SI-11 done, including crash-consistency tests
- [ ] Notes on fsync semantics and io_uring, with measured numbers from my NVMe drives
