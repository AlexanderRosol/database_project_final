# Michał Rawski, Alexander Rosół
# Project Documentation – Table of Contents

1. [Choice of Technology](#choice-of-technology)
   - Programming language(s)
   - Database system
   - Libraries and tools

2. [System Architecture](#system-architecture)
   - Description of components and interactions
   - (Optional) Architecture diagram

3. [Prerequisites](#prerequisites)
   - Software requirements
   - Installation of dependencies
   - Hardware or environment assumptions

4. [Installation and Setup Instructions](#installation-and-setup-instructions)
   - Step-by-step setup guide
   - How to run and test the system

5. [Design and Implementation Process](#design-and-implementation-process)
   - Overview of design decisions
   - Implementation steps and rationale

6. [Goal Implementation Details](#goal-implementation-details)
   - Explanation of how each project goal was addressed
   - Database queries and underlying logic

7. [Team Roles and Contributions](#team-roles-and-contributions)
   - Names of team members
   - Who did what (development, design, documentation, testing, etc.)

8. [Results](#results)
   - Example outputs and use cases
   - Performance metrics (e.g. timings)
   - Screenshots or sample runs

9. [User Manual](#user-manual)
   - How to use the `dbcli` command-line utility
   - How to reproduce results step-by-step

10. [Self-Evaluation](#self-evaluation)
    - Efficiency and performance discussion
    - Limitations and identified shortcomings
    - Ideas for future improvements

## 1. Choice of Technology

### Programming Language(s)

The project is implemented in **Rust**. Rust was chosen due to its strong emphasis on memory safety, performance, and concurrency—qualities that are particularly beneficial for high-throughput, low-level tasks such as bulk data import and transformation.

Additionally, Rust’s ecosystem provides convenient libraries (like `orientdb-client`) that allow direct interaction with the OrientDB server over its binary protocol.

However, because OrientDB is itself written in Java, a **Java Runtime Environment (JRE)** is required to run the OrientDB server. We recommend **Java 11**, as OrientDB has compatibility issues with newer Java versions (Java 17+). Specifically, OrientDB relies on certain internal Java APIs and behaviors that were changed or removed in later versions, leading to potential runtime failures or unstable behavior.

### Database System

As teased above, the chosen database is **OrientDB**, a multi-model NoSQL database that supports graphs, documents, and key/value data structures natively. It is particularly well-suited for this project, which involves storing and querying a large-scale knowledge graph.

OrientDB was selected for several reasons:

- **Graph model support**: Natively supports vertices and edges, simplifying conceptual mapping of entities and relationships.
- **Bulk loading optimization**: Supports specific commands (e.g., disabling WAL, altering cache size) for improving performance during high-volume inserts.
- **Schema flexibility**: Allows for dynamically creating classes and properties on-the-fly, which suits the variability in edge types.
- **Lightweight setup**: Can run as an embedded server or remote process, with minimal dependencies.

### Libraries and Tools

The project uses the following libraries and tools:

- **[orientdb-client (v0.6.0)](https://crates.io/crates/orientdb-client)**: A Rust client library that provides an interface to interact with OrientDB using its binary protocol. It enables database creation, session management, and running commands like creating vertices, edges, and indexes.

- **[clap (v4.5.38)](https://crates.io/crates/clap)**: Used for command-line parsing, although not prominently used in the current seeding script.

- **[num_cpus (v1.16.0)](https://crates.io/crates/num_cpus)**: Provides access to the number of logical CPU cores for potential thread-based optimizations.

- **[sysinfo (v0.35.1)](https://crates.io/crates/sysinfo)**: Used for accessing system-related information such as memory usage and process load (optional but useful during performance benchmarking).

- **Standard Rust libraries**:
  - `std::fs`, `std::io` for file reading.
  - `std::collections::{HashMap, HashSet}` for data management.
  - `std::thread` and `std::time::Instant` for parallelism and performance tracking.

Together, these components enable a performant, scalable, and maintainable way to seed a graph database from TSV data and efficiently support complex graph queries.

## 2. System Architecture

### Description of Components and Interactions

The system is designed to ingest and structure a large-scale knowledge graph into an OrientDB graph database. It is composed of the following main components:

#### 1. **OrientDB Server**
- Acts as the central persistent storage engine.
- Handles graph modeling, indexing, and querying of vertices (`Concept`) and edges (typed relationships).
- Must be run with **Java 11** due to compatibility issues with newer JDK versions.
- Exposed via its native binary protocol on port `2480` (http://localhost:2480).

#### 2. **Rust Data Seeder**
- A command-line Rust application responsible for:
  - Connecting to the OrientDB server.
  - Creating and configuring the database.
  - Parsing and ingesting data from a `.tsv` file.
  - Dynamically generating schema (classes, properties, edge types).
  - Performing high-performance, (optionally) multithreaded bulk inserts of both vertices and edges.
  - Applying post-insert optimizations (e.g., deferred index creation).
  - Resetting OrientDB's write-ahead log (WAL) and cache settings to production-safe values after import.

Key optimization strategies:
- **Deferred indexing**: Indexes are created *after* data is inserted to avoid overhead during bulk insertions.
- **WAL and cache tuning**: Write-ahead log is disabled and cache size is increased during import to improve speed.
- **Threaded execution**: Although the current configuration uses a single thread, the architecture supports multi-threaded seeding, where each thread processes a chunk of the data.

#### 3. **Input Data File (`cskg.tsv`)**
- TSV-formatted file containing concepts and relationships.
- Each row represents a labeled edge between two concepts.
- Parsed twice:
  - First pass: Extracts unique concepts and relationship types.
  - Second pass: Creates edges between concepts.

---

### Architecture Diagram

Here's a conceptual diagram of how the components interact:

![System Architecture Diagram](./images/Architecture_diagram.svg)

This modular, decoupled architecture allows for:
- Easy re-running or modification of the seeding process.
- Clean separation between data processing logic (Rust) and data storage (OrientDB).
- Scalability via future parallelism and data batching.

## 3. Prerequisites

This section describes the necessary software, libraries, and system assumptions required to use the database-backed knowledge graph and execute the data seeding process.

### 3.1 Software Requirements

To successfully build and run the loading pipeline, ensure the following software is installed:

- **Java Development Kit (JDK) 11**  
  Required for running the OrientDB server.

  > ⚠️ Note: Newer Java versions (e.g., 17 or above) may not be compatible with OrientDB 3.x.  
  > OrientDB relies on certain internal Java APIs that have been deprecated or removed in later Java versions. Java 11 is a stable, LTS version known to work reliably.

- **Rust (edition 2024)**  
  Required to compile and run the seeding logic. Install it via [https://rustup.rs](https://rustup.rs).

- **OrientDB Server (v3.x)**  
  The graph database used in this project. Available from [https://orientdb.com/download/](https://orientdb.com/download/).  
  The database should be accessible on `localhost:2424`.

- **Rust Crates / Dependencies** (included in `Cargo.toml`):
  - `orientdb-client = "0.6.0"` — Client library for OrientDB.
  - `clap = "4.5.38"` (with `derive` feature) — Command-line argument parsing.
  - `num_cpus = "1.16.0"` — Determines available CPU cores.
  - `sysinfo = "0.35.1"` — System information and diagnostics.

### 3.2 Installation of Dependencies

1. **Install Java 11**

   Using SDKMAN (recommended on Unix systems):
   ```bash
   sdk install java 11.0.20-tem
   sdk use java 11.0.20-tem
