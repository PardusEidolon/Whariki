
# Overview

This project aims to be an alternative choice to the Cardano Node's storage layer by integrating `irmin-pack`, a highly efficient, on-disk storage engine with git's design principles in mind. The primary goal is to  reduce the node's memory footprint and improve its overall performance, thereby enabling a single node to efficiently manage and validate transactions on machines with limited resources and on single-board computers for example a raspberry pi.

## Description

The Cardano Node currently stores a significant portion of its chain data in-memory, which leads to high memory consumption. While the node (currently maintained by intersectMBO another being built by Pragma in Rust) have introduced a Log-Structured Merge-tree (LSM-tree) implementation using an LMDB backend, the pursuit of optimal performance and resource utilisation for a rapidly growing blockchain necessitates further inquiry for alternatives as the problem stems beyond storing digital distributed ledgers. This project proposes leveraging `irmin-pack`, known for its compact storage and efficient data management, as an alternative backend.
## Brief/Why

The decision to explore `irmin-pack` stems from several key motivations:

- **Memory Reduction**: The existing in-memory storage of chain data in the Cardano Node is a significant bottleneck. `irmin-pack` has demonstrated the ability to dramatically reduce the size of large objects on disk, such as bringing down a 250GB `Tezos` chain to just 20GB, while maintaining low memory usage (e.g., 1GB) and low latency for reads and writes. This is a critical factor for scalability and operational efficiency.
- **Performance**: `irmin-pack`'s design, inspired by Git, offers efficient content-addressable storage. It utilises functional data structures and optimises for non-volatile memory, which aligns well with modern hardware characteristics.
### Addressing Current Limitations

While the `UTxO-HD` feature has been introduced as of `cardano-node-v10.4.1`, `irmin-pack` offers a potentially more effective approach to persistent, low-footprint storage for complex data structures like a blockchain ledger but also has properties that could extend to areas that involve large data-sets with no locality. Projects like `Amaru`, a Rust-based Cardano node, currently rely on RocksDB for storage; this project could offer a highly effective alternative.
## Current Problems

The current Cardano Node architecture faces several challenges related to data storage:

- **High Memory Consumption**: Storing all chain data in-memory is resource-intensive, limiting the node's scalability and increasing operational costs.
- **Suboptimal Disk Utilisation**: While LSM-trees are being adopted, existing implementations may not be fully optimised for different types of hardware, leading to suboptimal performance trade-offs between lookup and update costs.
- **Complexity of Historical States**: Efficiently reconstructing historical ledger states can be expensive if not all past states are kept in memory. The current approach of keeping 'k' past ledger states in memory, while memory-efficient, still needs robust disk-based solutions for persistence across very large data sets.
- **Lack of Unified Multi-Network Support**: Running multiple Cardano networks on a single node is currently impractical due to it's current resource constraints.

## Goals / Outcomes

Upon successful completion, this project will deliver:

- **Proof of Concept**: A working integration of `irmin-pack` with a basic Cardano Node implementation to test feasibility and performance.
- **Reduced Memory**: A significant decrease in the Cardano Node's RAM usage by offloading cold chain data to highly optimised on-disk storage.
- **Improved Performance**: Enhanced read and write throughput, and reduced lookup latency for blockchain data, crucial for block validation and synchronisation.
- **A Reliable Storage Layer**: A durable and reliable on-disk storage layer leveraging `irmin-pack`'s atomic write guarantees and integrity checks.
- **Multi-Network Capability**: A single Cardano Node capable of supporting multiple network instances (`testnet`, `devnet`, `mainnet`) simultaneously and efficiently.

## Recommended Approach

The recommended approach involves, drawing inspiration and techniques from various sources:

1. **`Irmin-pack` Integration**:
    - Utilise `irmin-pack` as the core on-disk storage engine.
    - Develop Rust bindings (`irmin-rs`) or adapt existing ones to interface the Cardano Node's core logic with `irmin-pack`.
    - Focus on `irmin-pack`'s ability to handle fixed-length keys and bounded-length values, which can be extended for variable-length keys using layering.
2. **Flash-Aware Design Principles (from `Wodan`)**:
    - Incorporate concepts from "`Wodan`: a pure OCaml, flash-aware filesystem library".
    - Align block sizes with flash erase blocks to prevent write amplification.
    - Implement checksums (e.g., CRC32C) for every block to prevent torn writes and detect data corruption.
    - Adopt strategies for fast mounting and recovery by logging in-use bitmaps periodically.
    - Leverage "hitchhiker trees" (a functional variant of Bepsilon-trees) for atomic and efficient write operations, especially on flash media.
3. **Cardano Consensus and Storage Layer Context**:
    - Understand the existing "Immutable DB" and "Volatile DB" split within Cardano's storage layer. The new `irmin-pack` based solution might replace or integrate with these components.
    - Address the challenges of maintaining consistency and historical ledger states, including efficient reconstruction of past ledger states.
    - Consider the 'k' parameter and its implications on maximum rollback and data stability.
    - Evaluate how the new storage impacts non-functional requirements such as predictable performance and resource requirements.

## Scope

The initial phase of this project will focus on:

- **Core `irmin-pack` Integration**: Implementing the fundamental read and write operations to `irmin-pack` for core blockchain data (blocks, headers, ledger states etc).
- **Benchmarking**: Establishing baseline metrics for memory usage, read/write latency, and throughput compared to the existing storage solutions.
- **Addressing Critical Data Structures Layouts**: Focusing on key-value storage of ledger state and transactions.

Out of scope for the initial phase:

- Full integration with all existing Cardano Node components (e.g., specific mini-protocols beyond what's strictly necessary for core functionality).
- Demonstrating the ability to run two distinct Cardano networks on a single node instance with reduced resource contention.
- Advanced caching strategies within `irmin-pack` beyond its native capabilities.
- Migration tools for existing Cardano Node databases to the new `irmin-pack` format.
- Comprehensive UI or command-line tooling for managing the `irmin-pack` backend.

Subsequent phases will expand on full integration, advanced features, and testing across diverse workloads and hardware configurations.

## References

- [High level overview of UTxO-HD | Ouroboros Consensus](https://ouroboros-consensus.cardano.intersectmbo.org/docs/for-developers/utxo-hd/Overview)
- [pragma-org/amaru](https://github.com/pragma-org/amaru?tab=readme-ov-file)
- [Cardano Consensus and Storage Layer](https://ouroboros-consensus.cardano.intersectmbo.org/pdfs/report.pdf)
- [intersectMBO/lsm-tree](https://github.com/IntersectMBO/lsm-tree)
- [intersectMBO/cardano-node](https://github.com/IntersectMBO/cardano-node)
- [mirage/irmin-rs](https://github.com/mirage/irmin-rs/tree/master)
- [Introducing irmin-pack](https://tarides.com/blog/2020-09-01-introducing-irmin-pack/)
- [Niv Dayan, Manos Athanassoulis, and Stratos Idreos. 2017. "Monkey: Optimal Navigable Key-Value Store."](https://doi.org/10.1145/3035918.3064054)
- [Chris Okasaki. 1998. "Purely Functional Data Structures" doi:10.1017/CBO9780511530104](https://doi.org/10.1017/CBO9780511530104)
- [Gabriel de Perthuis. 2017 "Wodan: a pure OCaml, flash-aware filesystem library"](https://g2p.github.io/research/wodan.pdf)