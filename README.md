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

- **C Compiler (e.g., GCC)**  
  Rust requires a system C compiler to build native dependencies and run correctly.  

- **OrientDB Server (v3.2.x)**  
  The graph database used in this project. Available from [https://orientdb.com/download/](https://orientdb.com/download/).  
  The database should be accessible on localhost:2480.

- **Rust Crates / Dependencies** (included in Cargo.toml):
  - orientdb-client = "0.6.0" — Client library for OrientDB.
  - clap = "4.5.38" (with derive feature) — Command-line argument parsing.
  - num_cpus = "1.16.0" — Determines available CPU cores.
  - sysinfo = "0.35.1" — System information and diagnostics.

### 3.2 Installation Instructions (Ubuntu/Debian)

Follow these steps to install all required software on a Debian-based system:

---

### Java (JDK 11)

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
```

Ensure Java 11 is the default version:

```bash
sudo update-alternatives --config java
java -version
```

Verify with:

```bash
java -version
```
Expected output should begin with `openjdk version "11"`.

---

### Rust (Edition 2024)

Install Rust using the official installer:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Follow the prompts, then restart your terminal. Verify with:

```bash
rustc --version
```

---

### C Compiler (Build Essentials)

Install system C compiler and related tools:

```bash
sudo apt install build-essential -y
```

Verify with:

```bash
gcc --version
```

---

### OrientDB Server (v3.2.38)

1. Download from: [https://github.com/orientechnologies/orientdb/releases](https://github.com/orientechnologies/orientdb/releases/) to choose a different version or just use:

```bash
wget -O ./orientdb-community-3.2.38.tar.gz https://repo1.maven.org/maven2/com/orientechnologies/orientdb-community/3.2.38/orientdb-community-3.2.38.tar.gz
```

2. Extract the archive (adjust for version):

```bash
tar -xzf orientdb-community-3.2.38.tar.gz
```

3. (Optional) Set environment variables:

```bash
export ORIENTDB_HOME="$(pwd)"
export PATH="$ORIENTDB_HOME/bin:$PATH"
```

To persist changes:

```bash
echo 'export ORIENTDB_HOME="$(pwd)"' >> ~/.bashrc
echo 'export PATH="$ORIENTDB_HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

4. Test the server:

```bash
cd orientdb-community-3.2.38/bin/
./server.sh
```

When prompted set the username to `root` and password also to `root`.
OrientDB should now be running at `localhost:2480`.

---

### Rust Crates (Dependencies)

Ensure your `Cargo.toml` contains:

```toml
[dependencies]
orientdb-client = "0.6.0"
clap = { version = "4.5.38", features = ["derive"] }
num_cpus = "1.16.0"
sysinfo = "0.35.1"
```

Install dependencies with:

```bash
cargo build
```

This will fetch and compile all required Rust crates.

---

Now everything should be installed that is needed to start the seeding process and running the dbcli.

### 3.3 System Assumptions

- A 64-bit Linux system (Ubuntu/Debian recommended) for best compatibility and performance.
- At least 8 GB RAM (the more the better).
- To allow access from the OrientDB server at localhost port 2480 (HTTP), make sure the port is not blocked.
- Multi-core CPU (4 cores or more) to benefit from parallel vertex and edge insertion, if possible.
- SSD storage preferred for faster database and file I/O operations, if possible.

### 4. Project Setup and Execution

Presuming all required software has been successfully installed, here’s how to correctly set up and run the system.

---

### 4.1 Directory Layout

Below is the expected file structure for the project:

```
/Rawski_Rosol
├── build/
│   └── cskg.tsv
├── orient-rs/
│   ├── src/
│   └── Cargo.toml
└── orientdb-community-3.2.38/
    └── bin/
        └── server.sh
```

---

### 4.2 Running the System

1. **Start the OrientDB Server**

Navigate to the `orientdb-community-3.2.38/bin/` directory and run the server:

```bash
cd /path/to/Project_file/orientdb-community-3.2.38/bin
./server.sh
```

When prompted set the username to `root` and password also to `root`.

2. **Run the Rust Loading Logic**

In a new terminal, navigate to the `orient-rs/` directory:

```bash
cd /path/to/Project_file/orient-rs
cargo run
```

This will execute the data loading logic and seed the OrientDB instance with data from `build/cskg.tsv`.

---

## 5. Design and Implementation Process

### Overview of Design Decisions

For our knowledge graph project, we chose **OrientDB** as the database because it naturally supports graph structures, which makes querying and managing linked data more efficient. 

We used **Java 11** as the preferred runtime environment because newer Java versions have compatibility issues with OrientDB. Specifically, some OrientDB versions rely on internal Java APIs or specific JVM behaviors that changed or were deprecated in later Java releases, which can cause unexpected failures or degraded performance.

The key design decision was to optimize data import performance by carefully managing database settings and operations during bulk loading. Instead of maintaining indexes during vertex insertions—which slows down inserts significantly—we defer index creation until after all data is loaded. This approach improves bulk import speed dramatically.

### Implementation Steps and Rationale

1. **Database Setup**  
   We start by connecting to the local OrientDB server, dropping any existing database with the same name, and creating a fresh database. This ensures a clean slate for loading the knowledge graph data.

2. **Schema Definition and Configuration**  
   We define a `Concept` class (vertex) with properties `id` and `label`.  
   To optimize bulk insertions, we temporarily disable database features like validation, write-ahead logging (WAL) flushing, and reduce index maintenance overhead. These changes improve write throughput during import.

3. **Data Collection (First Pass)**  
   We read the knowledge graph TSV file to gather unique vertices and edge classes. For each vertex, we store its best label based on length and alphabetical order. This preprocessing ensures data quality and consistency.

4. **Edge Classes Creation**  
   After collecting all unique edge classes, we create corresponding edge classes extending from the OrientDB edge type `E`.

5. **Bulk Vertex Creation**  
   We insert all vertices in bulk without maintaining indexes during insertion. This is achieved by batching the inserts inside a transaction, significantly speeding up the process since index updates are expensive.

6. **Index Creation Post-Insert**  
   Once all vertices are loaded, we create a unique index on the `id` property of the `Concept` class. Building the index after the bulk insert is faster than updating it incrementally during insertion.

7. **Edge Data Collection and Creation (Second Pass)**  
   We perform a second pass through the TSV file to collect all edges. Then, edges are created in batches with smaller batch sizes due to the more complex operations and index lookups involved. With the index in place, lookups during edge creation are efficient.

8. **Restoring Production Settings**  
   After data import, we revert database settings like WAL flushing and cache size back to production-appropriate values to ensure durability and performance for normal operations.

