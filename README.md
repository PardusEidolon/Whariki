# Whāriki

> *"To cover with a mat, spread out on the ground. (woven flax mat, often metaphorically foundational)"*

This repository explores the integration of `irmin-pack` as a highly efficient, on-disk storage solution for the Cardano Node. The primary goal is to significantly reduce the node's memory footprint and enhance its overall performance, enabling support for multiple Cardano networks (e.g., testnet, devnet, mainnet) on a single instance.

## Goals

* **Reduce Memory Footprint**: Transition critical chain data to disk to lower RAM consumption.
* **Improve Performance**: Achieve faster data access and higher transaction throughput.
* **Enable Multi-Network Support**: Facilitate running multiple Cardano network instances concurrently.

## Focus

This project draws upon concepts from:

* **Irmin-pack**: A Git-like, content-addressable storage system, utilising `irmin-rs` bindings for Rust.
* **Wodan Filesystem**: Principles for flash-aware storage, atomic writes, and data integrity.
* **LSM-tree Optimisations**: Strategies from the "Monkey" paper for efficient disk I/O and memory management.

For am understanding of the project's overview, goals, and detailed approach, please refer to the [Project Overview Document](./Doc/Whakari/Overview.md).