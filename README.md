# Michał Rawski, Alexander Rosół
# Project Documentation – Table of Contents

- [1. Choice of Technology](#1-choice-of-technology)
  - [Programming Language(s)](#programming-languages)
  - [Database System](#database-system)
  - [Extra](#extra)

- [2. System Architecture](#2-system-architecture)
  - [Description of Components and Interactions](#description-of-components-and-interactions)
  - [Architecture Diagram (Optional)](#architecture-diagram-optional)

- [3. Prerequisites](#3-prerequisites)
  - [Software Requirements](#software-requirements)
  - [Installation of Dependencies (Ubuntu/Debian)](#installation-of-dependencies-ubuntudebian)
  - [Hardware or Environment Assumptions](#hardware-or-environment-assumptions)

- [4. Project Setup and Execution](#4-project-setup-and-execution)
  - [Step-by-Step Guide](#step-by-step-guide)

- [5. Design and Implementation Process](#5-design-and-implementation-process)
  - [Overview of Design Decisions](#overview-of-design-decisions)
  - [Implementation Steps and Rationale](#implementation-steps-and-rationale)

## 1. Choice of Technology

### Programming Language(s)

The project is written in **Rust** for its performance and handling low-level tasks like bulk data import. Its ecosystem has a library called orientdb-client, which directly interacts with OrientDB’s binary protocol.

We have no code written in *C*,but due to *Rust* dependencies that require *C*, it has to be installed.

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

Make sure the **OrientDB** server is running, so that data processing possible.

## 3. Prerequisites

This chapter outlines which software is used and how to install it, so that is works for sure. We ran into a lot of OS and version incompatibility issues, so we stress that using these specific instructions will lead to the program working, while any change may break it.

### Software Requirements

This list only outlines the software used. In the following chapter it will be covered how to install them:

- **Java Development Kit (JDK) 11**  
  Required for running the OrientDB server.

  > Newer Java versions may not be compatible with OrientDB 3.2.38.  

- **Rust**  
  Required to compile and run the seeding logic.

- **C Compiler**  
  Rust requires a system C compiler to build native dependencies and to work. That is why we could not test not the machines given to us, as the was no C and no sudo privileges to install it. 

- **OrientDB Server (v3.2.38)**  
  The graph database used in this project. We use v3.2.38. Choosing another version might break the process of our project.

- **Rust Crates / Dependencies** (included in Cargo.toml):
  - orientdb-client = "0.6.0" — Library for OrientDB's binary protocol.

### Installation of Dependencies (Ubuntu/Debian)

Follow these steps to install all required software. Make sure the versions match to dodge unforeseen errors:

---

#### Java (JDK 11)

```bash
sudo apt update
sudo apt install openjdk-11-jdk -y
```

Make Java 11 the default version:

```bash
sudo update-alternatives --config java
java -version
```

Verify with:

```bash
java -version
```

---

#### Rust

Install Rust:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Follow the standard prompts and then restart your terminal. Verify with:

```bash
rustc --version
```

---

#### C Compiler

Install system C compiler and related tools:

```bash
sudo apt install build-essential -y
```

Verify with:

```bash
gcc --version
```

---

#### OrientDB Server (v3.2.38)

1. Either download from: [https://github.com/orientechnologies/orientdb/releases](https://github.com/orientechnologies/orientdb/releases/) to choose a different version or just use:

```bash
wget -O ~/orientdb-community-3.2.38.tar.gz https://repo1.maven.org/maven2/com/orientechnologies/orientdb-community/3.2.38/orientdb-community-3.2.38.tar.gz
```

2. Extract the archive (not adjusted for version):

```bash
tar -xzf orientdb-community-3.2.38.tar.gz
```

3. (Optional) Set environment variables:

```bash
export ORIENTDB_HOME="$(pwd)"
export PATH="$ORIENTDB_HOME/bin:$PATH"
```

(Optional)To persist changes:

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

When prompted, set the username to `root` and password also to `root`.
OrientDB should now be running at `localhost:2480`.
To exit, just press ctrl + c.

---

####Rust Crates / Dependencies
Building the dependencies at the right place will be covered later.

Now everything should be installed that is needed to start the seeding process and running the dbcli.

#### Hardware or Environment Assumptions

- 64-bit Linux system (Ubuntu/Debian).
- At least 8 GB RAM (the more the better).
- Allow access from the OrientDB server at port 2480 (HTTP). Make sure the port is not blocked manually.

## 4. Project Setup and Execution

Presuming all required software has been successfully installed, here is how to correctly set up and run the system.

---

#### Directory Layout

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

### Step-by-Step Guide

#### Start the OrientDB Server

Navigate to the OrientDB `bin` directory and run the server:

```bash
cd /path/to/Project_file/orientdb-community-3.2.38/bin
./server.sh
```

When prompted, set the username to `root` and the password also to `root`.

#### Run the Rust Data Loading Logic (`seed.rs`)

In a new terminal, navigate to the `orient-rs` directory and run:

```bash
cd /path/to/Project_file/orient-rs
cargo run
```

This will execute the logic inside `src/seed/seed.rs` (assuming `main.rs` calls the seed logic) and load data from `build/cskg.tsv` into the running OrientDB instance.

#### Using the Database CLI (`dbcli.rs`)

The CLI code is inside `src/dbcli/dbcli.rs` and depends on the OrientDB server running.

##### Add Required Dependency

Make sure your `Cargo.toml` contains:

```toml
[dependencies]
orientdb-client = "0.6.0"
```

#### Prepare the CLI for Running

To run or build the CLI, you can temporarily replace the content of `src/main.rs` with that of `src/dbcli/dbcli.rs` by copying:

```bash
# Backup original main.rs
cp src/main.rs src/main_backup.rs

# Replace main.rs with dbcli.rs
cp src/dbcli/dbcli.rs src/main.rs
```

#### Compile the CLI in Release Mode

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

We used **Java 11** as the preferred runtime environment because newer Java versions have compatibility issues with OrientDB. Specifically, some OrientDB versions rely on internal Java APIs or specific JVM behaviors that changed or were deprecated in later Java releases, which can cause unexpected failures.

The key design decision was to optimize data import performance by carefully managing database settings and operations during bulk loading, which is why we used **Rust**. Instead of maintaining indexes during vertex insertions, which slows down inserts, we do not do the index creation until after all data is loaded. This approach improved bulk import speed by a lot.

### Implementation Steps and Rationale

1. **Database Setup**  
   We start by connecting to the local OrientDB server, dropping any existing database with the same name, and creating a fresh database.

2. **Schema Definition and Configuration**  
   We define a `Concept` class (vertex) with properties `id` and `label`.  
   To optimize bulk insertions, we temporarily disable database features like validation and reduce index maintenance overhead.

3. **Data Collection (First Pass)**  
   We read the knowledge graph TSV file to gather unique vertices and edge classes. For each vertex, we store its best label based on length and alphabetical order. This pre-processing ensures data quality and consistency.

4. **Edge Classes Creation**  
   After collecting all unique edge classes, we create corresponding edge classes extending from the OrientDB edge type `E`.

5. **Bulk Vertex Creation**  
   We insert all vertices in bulk without maintaining indexes during insertion. This is achieved by batching the inserts inside a transaction, significantly speeding up the process since index updates are expensive.

6. **Index Creation Post-Insert**  
   Once all vertices are loaded, we create a unique index on the `id` property of the `Concept` class. Building the index after the bulk insert is faster than updating it incrementally during insertion.

7. **Edge Data Collection and Creation (Second Pass)**  
   We perform a second pass through the TSV file to collect all edges. Then, edges are created in batches with smaller batch sizes due to the more complex operations and index lookups involved. With the index in place, lookups during edge creation are efficient.

8. **Restoring Production Settings**  
   After data import, we revert database settings like cache size back to production-appropriate values to ensure durability and performance for normal operations.
