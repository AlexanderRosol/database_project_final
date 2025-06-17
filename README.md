# Michał Rawski, Alexander Rosół
# Project Documentation – Table of Contents

- [1. Choice of Technology](#1-choice-of-technology)
  - [Programming Language(s)](#programming-languages)
  - [Database System](#database-system)
  - [Extra](#extra)

- [2. System Architecture](#2-system-architecture)
  - [Description of Components and Interactions](#description-of-components-and-interactions)
  - [Architecture Diagram (Optional)](#architecture-diagram-optional)

## 1. Choice of Technology

### Programming Language(s)

The project is written in **Rust** for its performance and handling low-level tasks like bulk data import. Its ecosystem has a library called orientdb-client, which directly interacts with OrientDB’s binary protocol.

We have no code written in *C*, but due to *Rust* dependencies that require *C*, it has to be installed.

Since OrientDB is Java-based, it requires a *Java* to run. We recommend *Java 11*, because OrientDB is does not work properly with newer versions due to changes in internal Java APIs, causing a lot of runtime issues.


### Database System

As mentioned above, we chose **OrientDB** as our database system. It is a multi-model NoSQL database, which supports graphs, documents, and key/value data structures natively, which fits our dataset.

We selected OrientDB because:

- **Native graph support** with vertices and edges, ideal for our data.
- **Bulk loading optimizations** (e.g., disabling WAL, cache tuning) to boost insert performance (we did not know at the time that this will cause us headaches).
- **Flexible schema** allowing dynamic creation of classes and properties.
- **Lightweight setup**, running embedded or remotely with minimal dependencies.
- We would have chosen *neo4j*, but it was not allowed, due to it being used in our laboratory classes.

### Extra

- **[orientdb-client (v0.6.0)]** is the dependency that allows us to interact with the OrientDB binary protocol. Through it, we can create vertices and relationships, run queries and more.

## 2. System Architecture

### Description of Components and Interactions

The system seeds a large-scale knowledge graph from our given `cskg.tsv` file into an OrientDB database. These are the following components of the architecture:

#### 1. **OrientDB Server**
- Stores and queries graph data.
- Exposed via binary protocol at `http://localhost:2480`.

#### 2. **Rust Data Seeder**
A CLI tool that:
- Connects to OrientDB and initializes the database.
- Parses the `.tsv` file and dynamically generates schema.
- Inserts data efficiently using:
  - **Deferred indexing**
  - (Optional) **cache tuning**
  - (Optional) **multithreading**
- Resets cache to safe values after import.

#### 3. **Input File (`cskg.tsv`)**
- TSV file that is our dataset.
- Parsed in two passes:
  1. Extract unique concepts and edge types.
  2. Insert edges between concepts.

---

### Architecture Diagram

![System Architecture Diagram](./images/Architecture_diagram.svg)

Make sure the **OrientDB** server is running, so that data processing is possible.

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

# 4. Project Setup and Execution

Presuming all required software has been successfully installed, here’s how to correctly set up and run the system.

---

## 4.1 Directory Layout

Below is the expected file structure for the project:

```
/Rawski_Rosol
├── build/
│   └── cskg.tsv
├── orient-rs/
│   ├── Cargo.toml
│   ├── src/
│   │   ├── main.rs
│   │   ├── seed/
│   │   │   └── seed.rs
│   │   └── dbcli/
│   │       └── dbcli.rs
└── orientdb-community-3.2.38/
    └── bin/
        └── server.sh
```

---

## 4.2 Running the System

### Step 1: Start the OrientDB Server

Navigate to the OrientDB `bin` directory and run the server:

```bash
cd /path/to/Project_file/orientdb-community-3.2.38/bin
./server.sh
```

When prompted, set the username to `root` and the password also to `root`.

### Step 2: Run the Rust Data Loading Logic (`seed.rs`)

In a new terminal, navigate to the `orient-rs` directory and run:

```bash
cd /path/to/Project_file/orient-rs
cargo run
```

This will execute the logic inside `src/seed/seed.rs` (assuming `main.rs` calls the seed logic) and load data from `build/cskg.tsv` into the running OrientDB instance.

### Step 3: Using the Database CLI (`dbcli.rs`)

The CLI code is inside `src/dbcli/dbcli.rs` and depends on the OrientDB server running.

#### 3.1 Add Required Dependency

Make sure your `Cargo.toml` contains:

```toml
[dependencies]
orientdb-client = "0.6.0"
```

#### 3.2 Prepare the CLI for Running

To run or build the CLI, you can temporarily replace the content of `src/main.rs` with that of `src/dbcli/dbcli.rs` by copying:

```bash
# Backup original main.rs
cp src/main.rs src/main_backup.rs

# Replace main.rs with dbcli.rs
cp src/dbcli/dbcli.rs src/main.rs
```

#### 3.3 Compile the CLI in Release Mode

```bash
cargo build --release
```

This will generate a binary at:

```
target/release/orient-rs
```

You can rename this binary to `dbcli` if you like:

```bash
mv target/release/orient-rs target/release/dbcli
```


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

# 6. Goal Implementation Details

This section explains how each project goal was addressed through database queries. It includes query execution details, logic, and the results retrieved during knowledge graph exploration.

- **Successors of `/c/en/steam_locomotive`**  
  - **Query 1**  
    - **Execution time:** 490.5 µs  
    - **Result:** *None returned*  
    - **Explanation:** The system attempted to retrieve all concepts that are successors (i.e., directly connected in the outgoing direction) of the node representing “steam locomotive”.

- **Count Successors of `/c/en/value`**  
  - **Query 2**  
    - **Execution time:** 4.75 ms  
    - **Result:** *177*  
    - **Explanation:** The system retrieved all concepts that are successors (i.e., directly connected in the outgoing direction) of the node representing “value”. A total of 177 such successor nodes were found.

- **Predecessors of `Q40157`**  
  - **Query 3**  
    - **Execution time:** 1.02 ms  
    - **Result:**  
      1. the floor is lava  
      2. lava lamp  
      3. lava lake  
      4. primary magma  
      5. magma  
    - **Explanation:** Identifies all incoming nodes connected to `Q40157`, which likely represents “lava”.

- **Predecessors of `/c/en/country`**  
  - **Query 4**  
    - **Execution time:** 4.997 ms  
    - **Result:** *735*  
    - **Explanation:** The system retrieved all concepts that are predecessors (i.e., directly connected in the incoming direction) of the node representing “country”. A total of 735 such predecessor nodes were found.

- **Neighbors of `/c/en/spectrogram`**  
  - **Query 5**  
    - **Execution time:** 595.75 µs  
    - **Result:** *None returned*  
    - **Explanation:** A neighbor query returns all directly connected nodes, regardless of direction, but no results were found.

- **Count of Neighbors of `/c/en/jar`**  
  - **Query 6**  
    - **Execution time:** 10.26 ms  
    - **Result:** 373  
    - **Explanation:** Counts all neighboring nodes directly connected to "jar".

- **Grandchildren of `Q676`**  
  - **Query 7**  
    - **Execution time:** 1.81 ms  
    - **Result:**  
      - genre fiction, literary form, literary class, art genre, genre, literary genre, form, written work, creative work, group of literary works, serial  
    - **Explanation:** Explores two levels of hierarchy to find concepts that are two steps down from `Q676` (e.g., genre or form types).

- **Grandparents of `/c/en/ms_dos`**  
  - **Query 8**  
    - **Execution time:** 1.06 ms  
    - **Result:**  
      - dr dos, ms dos, pc dos, 8.3, dos, drive letter, print screen  
    - **Explanation:** Retrieves nodes two levels above MS-DOS to understand its higher-level classifications.

- **Total Nodes in Graph**  
  - **Query 9**  
    - **Execution time:** 2.96 s  
    - **Result:** 2,160,968  
    - **Explanation:** Counts all concept nodes in the dataset.

- **Nodes with No Successors**  
  - **Query 10**  
    - **Execution time:** 32.79 s  
    - **Result:** 649,184  
    - **Explanation:** Identifies terminal nodes with no outgoing edges.

- **Nodes with No Predecessors**  
  - **Query 11**  
    - **Execution time:** 30.90 s  
    - **Result:** 1,129,781  
    - **Explanation:** Finds root nodes with no incoming edges.

- **Nodes with the Most Neighbors**  
  - **Query 12**  
    - **Execution time:** 115.41 s  
    - **Result:** `slang`  
    - **Explanation:** Node with the highest degree of connectivity.

- **Nodes with a Single Neighbor**  
  - **Query 13**  
    - **Execution time:** 56.52 s  
    - **Result:** 1,276,217  
    - **Explanation:** Indicates sparse or leaf nodes in the graph.

- **Query 14: Rename /c/en/transportation_topic/n**  
  - Renamed `/c/en/transportation_topic/n`  
  - **New ID:** new_id  
  - **New Label:** new_label  
  - **Explanation:** Indicates a structural update to the knowledge graph, such as normalization or renaming of ambiguous or incorrectly labeled nodes.

- **Similar Nodes to `/c/en/emission_nebula`**  
  - **Query 15**  
    - **Execution time:** 13.10 ms  
    - **Result (sample):** solar nebula, diffuse nebula, protoplanetary nebula, planetary nebula, etc.  
    - **Explanation:** Uses semantic or lexical similarity to find related nodes.

- **Relatedness Between Two Nodes**  
  - **Query 16 (Instance 1)**  
    - Nodes: `/c/en/flower` and `/c/en/spacepower`  
    - **Execution time:** 15.73 ms  
    - **Result:** flower, plant, power, spacepower  
  - **Query 16 (Instance 2)**  
    - Nodes: `/c/en/uchuva` and `/c/en/square_sails/n`  
    - **Execution time:** 68.28 ms  
    - **Result:** uchuva, tomatillo, tomato, square, square sail, etc.  
    - **Explanation:** Measures conceptual proximity via shared neighbors or embeddings.

- **Distant Synonyms (e.g., `/c/en/defeatable` at distance 2)**  
  - **Query 17 (Instance 1)**  
    - **Execution time:** 46.90 ms  
    - **Result:** conquerable, beatable, surmountable, etc.  
  - **Query 17 (Instance 2):** `/c/en/telegraphically` at distance 5  
    - **Execution time:** 346.96 ms  
    - **Result:** concisely, hastily, quickly, etc.  
    - **Explanation:** Traverses `Synonym` relations across multiple hops to identify conceptually close terms.

- **Distant Antonyms**  
  - **Query 18 (Instance 1):** `/c/en/automate` at distance 3  
    - **Execution time:** 47.44 ms  
    - **Result:** civilize, tame, personify, etc.  
  - **Query 18 (Instance 2):** `/c/en/rollercoaster` at distance 6  
    - **Execution time:** 39.84 s  
    - **Result:** stand still, stay, unbend, straight, etc.  
    - **Explanation:** Alternates through `Synonym` and `Antonym` edges; ensures path ends in opposite polarity using modulo logic on antonym edge count.

## 7. [Team Roles and Contributions](#team-roles-and-contributions)

We are **Michał Rawski** and **Alexander Rosol**. We both collaborated to solve the problems, choosing whichever solutions we felt were best. In the end, most of **Michał Rawski**'s contributions were included in the final version, as he has experience with Rust.

**Alexander Rosol** handled testing on a virtual machine and wrote the majority of the documentation with much guidance from **Michał Rawski**, since **Alexander Rosol** is not familiar with Rust.

## 10. [Self-Evaluation](#self-evaluation)

We are happy with how the project went, as we met most milestones—often before they were even checked. Based on feedback and observed behavior, the performance appears to be good, and we believe our implementation is quite efficient.

### Limitations and Identified Shortcomings

One major limitation was running OrientDB on Windows, which led to a number of OS-specific errors. Almost all official documentation and community resources primarily target macOS, and the overall ecosystem is small. This made fixing issues nearly impossible at times. OrientDB itself was frequently the bottleneck in our workflow—for example, using its native seeding process proved so cumbersome that we opted to implement our own solution.

### Ideas for Future Improvements

Looking forward, we would like to implement Write-Ahead Logging (WAL) and configurable cache sizing. However, this depends heavily on the version of OrientDB, and modifying the version can break compatibility with other parts of the system.

