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

- [6.Goal Implementation Details](#6-goal-implementation-details)

- [7.Team Roles and Contributions](#7-team-roles-and-contributions)

- [8. Self-Evaluation](#8-self-evaluation)
  - [Efficiency and Performance Discussion](#efficiency-and-performance-discussion)
  - [Limitations and Identified Shortcomings](#limitations-and-identified-shortcomings)
  - [Ideas for Future Improvements](#ideas-for-future-improvements)


## 1. Choice of Technology

### Programming Language(s)

The project is written in **Rust** for its performance and handling low-level tasks like bulk data import. Its ecosystem has a library called orientdb-client, which directly interacts with OrientDB’s binary protocol.

We have no code written in *C*,but due to *Rust* dependencies that require *C*, it has to be installed.

Since OrientDB is Java-based, it requires a *Java* to run. We recommend *Java 11*, because OrientDB is does not work properly with newer versions due to changes in internal Java APIs, causing a lot of runtime issues.


### Database System

As mentioned above, we chose **OrientDB** as our database system. It is a multi-model NoSQL database, which supports graphs, documents, and key/value data structures natively, which fits our dataset.

We selected OrientDB because:

- **Native graph support** with vertices and edges, ideal for our data.
- **Bulk loading optimizations** like cache tuning to boost insert performance.
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

## 6. Goal Implementation Details

This section explains how each project goal was addressed through database queries. It includes query execution details, logic, and the results retrieved during knowledge graph exploration.

- **Successors of `/c/en/steam_locomotive`**  
  - **Query 1** : SELECT expand(out().asSet().label) FROM `Concept` WHERE id = :node_id
    - **Execution time:** 1.970208ms 
    - **Result:**
	1 locomotive
	2 steam
	3 steam locomotive
    - **Explanation:** The system attempted to retrieve all concepts that are successors, meaning "directly connected in the outgoing direction", of the node representing “steam locomotive”.

- **Count Successors of `/c/en/value`**  
  - **Query 2** : SELECT out().asSet().size() as value FROM `Concept` WHERE id = :node_id
    - **Execution time:** 1.790208ms  
    - **Result:** *177*  
    - **Explanation:** The system retrieved all concepts that are successors (i.e., directly connected in the outgoing direction) of the node representing “value”.

- **Predecessors of `Q40157`**  
  - **Query 3** : SELECT expand(in().asSet().label) FROM `Concept` WHERE id = :node_id
    - **Execution time:** 792.708µs 
    - **Result:**  
      1. the floor is lava  
      2. lava lamp  
      3. lava lake  
      4. primary magma  
      5. magma  
    - **Explanation:** Identifies all incoming nodes connected to `Q40157`.

- **Predecessors of `/c/en/country`**  
  - **Query 4** : SELECT in().asSet().size() as value FROM `Concept` WHERE id = :node_id
    - **Execution time:** 4.915084ms  
    - **Result:** *735*  
    - **Explanation:** The system retrieved all concepts that are predecessors (i.e., directly connected in the incoming direction) of the node representing “country”.

- **Neighbors of `/c/en/spectrogram`**  
  - **Query 5** : SELECT expand(both().asSet().label) FROM `Concept` WHERE id = :node_id
    - **Execution time:** 337.958µs  
    - **Result:** *None returned*  
    - **Explanation:** A neighbor query returns all directly connected nodes, regardless of direction, so both directions.

- **Count of Neighbors of `/c/en/jar`**  
  - **Query 6** : SELECT both().asSet().size() as value FROM `Concept` WHERE id = :node_id
    - **Execution time:** 2.678291ms  
    - **Result:** 373  
    - **Explanation:** Counts all neighboring nodes directly connected to "jar".

- **Grandchildren of `Q676`**  
  - **Query 7** : SELECT expand(out().out().asSet().label) FROM `Concept` WHERE id = :node_id
    - **Execution time:** 545.791µs  
    - **Result:**  
      - genre fiction, literary form, literary class, art genre, genre, literary genre, form, written work, creative work, group of literary works, serial  
    - **Explanation:** Explores two levels of hierarchy to find concepts that are two steps down from `Q676`.

- **Grandparents of `/c/en/ms_dos`**  
  - **Query 8** : SELECT expand(in().in().asSet().label) FROM `Concept` WHERE id = :node_id
    - **Execution time:** 513.084µs  
    - **Result:**
	1. dr dos
	2. ms dos
	3. pc dos
	4. 8.3
	5. dos
	6. drive letter
	7. print screen
    - **Explanation:** Retrieves nodes two levels above ms_dos inward.

- **Total Nodes in Graph**  
  - **Query 9** : SELECT COUNT(*) as value FROM `Concept`
    - **Execution time:** 4.1337995s  
    - **Result:** 2,160,968  
    - **Explanation:** Counts all concept nodes in the dataset.

- **Nodes with No Successors**  
  - **Query 10** : SELECT count(*) as value FROM `Concept` WHERE out().asSet().size() = 0
    - **Execution time:** 34.456729s  
    - **Result:** 649,184  
    - **Explanation:** Identifies nodes with no outgoing edges.

- **Nodes with No Predecessors**  
  - **Query 11** : SELECT count(*) as value FROM `Concept` WHERE in().asSet().size() = 0
    - **Execution time:** 30.909017417s 
    - **Result:** 1,129,781  
    - **Explanation:** Finds nodes with no incoming edges.

- **Nodes with the Most Neighbors**  
  - **Query 12** : SELECT label FROM `Concept` WHERE both().asSet().size() = (SELECT max(both().asSet().size()) FROM `Concept`)
    - **Execution time:** 127.836150583s 
    - **Result:** `slang`  
    - **Explanation:** Node with the highest degree of connectivity in both directions. Make sure to not count them twice.

- **Nodes with a Single Neighbor**  
  - **Query 13** : SELECT count(*) as value FROM `Concept` WHERE both().asSet().size() = 1
    - **Execution time:** 63.250132291s  
    - **Result:** 1,276,217  
    - **Explanation:** Indicates sparse or leaf nodes in the graph.

- **Rename /c/en/transportation_topic/n**
  - **Query 14** : UPDATE `Concept` SET id = :new_id, label = :new_label WHERE id = :node_id
    - **Parameters:**
	node_id = /c/en/transportation_topic/n
	new_id = new_id
	new_label = new_label
    - **Execution time:** 2.347ms
    - **Result:**
	OK - Node renamed successfully
    - **Explanation:** Indicates a structural update to the knowledge graph.

- **Similar Nodes to** `/c/en/emission_nebula`  
  - **Query 15**:
    ```sql
    SELECT expand(
      unionall(
        (
          MATCH {
            class: Concept,
            where: (id = :node_id)
          }.outE() {as: e1}
           .in      {as: child}
           .inE()   {as: e2, where: (@class = $matched.e1.@class)}
           .out     {as: similar_node, where: (id <> :node_id)}
          RETURN DISTINCT similar_node.label as value
        ),
        (
          MATCH {
            class: Concept,
            where: (id = :node_id)
          }.inE()  {as: e1}
           .out    {as: parent}
           .outE() {as: e2, where: (@class = $matched.e1.@class)}
           .in     {as: similar_node, where: (id <> :node_id)}
          RETURN DISTINCT similar_node.label as value
        )
      ).asSet()
    )
    ```
    - **Parameters:**
	node_id = /c/en/emission_nebula
    - **Execution time:** `67.373917ms`  
    - **Result (sample):**
	1. emit
	2. emissions test
	3. solar nebula
	4. preplanetary nebula
	5. diffuse nebula
	6. emission line
	7. protoplanetary nebula
	8. stellar nebula
	9. dark nebula
	10. act
	11. reflection nebula
	12. planetary nebula
	13. issue
    - **Explanation:** Uses semantic or lexical similarity via shared parent/child edge types to identify related concepts in the graph.

- **Relatedness Between Two Nodes**  
  - **Query 16 (Instance 1)** :
    ```sql
    SELECT expand(shortestPath(
            (SELECT FROM `Concept` WHERE id = :node1_id),
            (SELECT FROM `Concept` WHERE id = :node2_id),
            'BOTH'
    ).label) as value
    ´´´
    - **Parameters:**
	node1_id = /c/en/flower
	node2_id = /c/en/spacepower   
    - **Execution time:** 92.495209ms  
    - **Result:**
	1. flower
	2. plant
	3. power
	4. spacepower
  
  - **Query 16 (Instance 2)** :
    ```sql
    SELECT expand(shortestPath(
            (SELECT FROM `Concept` WHERE id = :node1_id),
            (SELECT FROM `Concept` WHERE id = :node2_id),
            'BOTH'
    ).label) as value
    ´´´
    - **Parameters:**
	node1_id = /c/en/uchuva
	node2_id = /c/en/square_sails/n 
    - **Execution time:** 129.800916ms  
    - **Result:**
	1. uchuva
	2. tomatillo
	3. tomato
	4. sauce
	5. slang
	6. square
	7. square sail
	8. square sails  
    - **Explanation:** Measures conceptual proximity via shared neighbors or embeddings.

- **Distant Synonyms (e.g., `/c/en/defeatable` at distance 2)**  
  - **Query 17 (Instance 1)** :
    ```sql
    SELECT DISTINCT result.label
FROM (
  MATCH
    {class: Concept, where: (id = '/c/en/defeatable'), as: start}
      .bothE('$r$Synonym', '$r$Antonym') {as: e1}
      .bothV() {as: v1}
      .bothE('$r$Synonym', '$r$Antonym') {as: e2}
      .bothV() {as: result}
  RETURN
    start,
    result,
    [e1, e2] AS path
)
WHERE
  (path[(@class = '$r$Antonym')].size() % 2) = 0
  AND shortestPath(start, result, "BOTH", ['$r$Synonym', '$r$Antonym']).size() = 3

    ´´´
    - **Execution time:** 27.036625ms  
    - **Result:**
	1 conquerable
	2 weak
	3 attainable
	4 beatable
	5 possible
	6 subduable
	7 subjugable
	8 superable
	9 surmountable
	10 vanquishable
	11 vincible
    - **Explanation:** Traverses `Synonym` relations across multiple hops to identify conceptually close terms.
  - **Query 17 (Instance 2): `/c/en/telegraphically` at distance 5**
    - **Execution time:** 481.602458ms  
    - **Result:**
	1. momentarily
	2. concisely
	3. fleetingly
	4. hastily
	5. quickly
	6. shortly
	7. succinctly
	8. summarily
	9. temporarily
	10. transiently
	11. awhile
	12. early
	13. in nutshell
    - **Explanation:** Traverses `Synonym` relations across multiple hops to identify conceptually close terms.

- **Distant Antonyms**  
  - **Query 18 (Instance 1): `/c/en/automate` at distance 3** :
    ```sql
    SELECT DISTINCT result.label
FROM (
  MATCH
    {class: Concept, where: (id = '/c/en/automate'), as: start}
      .bothE('$r$Synonym', '$r$Antonym') {as: e1}
      .bothV() {as: v1}
      .bothE('$r$Synonym', '$r$Antonym') {as: e2}
      .bothV() {as: v2}
      .bothE('$r$Synonym', '$r$Antonym') {as: e3}
      .bothV() {as: result}
  RETURN
    start,
    result,
    [e1, e2, e3] AS path
)
WHERE
  (path[(@class = '$r$Antonym')].size() % 2) = 1
  AND shortestPath(start, result, "BOTH", ['$r$Synonym', '$r$Antonym']).size() = 4
    ´´´
    - **Execution time:** 35.141417ms  
    - **Result:**
	1. civilize
	2. cultivate
	3. refine
	4. tame
	5. teach
	6. temper
	7. personify
  - **Query 18 (Instance 2):** `/c/en/rollercoaster` at distance 6  
    - **Execution time:** 41.286930208s  
    - **Result:**
	1. fill
	2. driving
	3. driving straight
	4. go
	5. go straight
	6. going
	7. going straight
	8. stand
	9. stand still
	10. stay
	11. stay straight
	12. still
	13. straight
	14. twist
	15. unbend
	16. advance
	17. level  
    - **Explanation:** Alternates through `Synonym` and `Antonym` edges; ensures path ends in opposite polarity using modulo logic on antonym edge count.

## 7. Team Roles and Contributions

We are **Michał Rawski** and **Alexander Rosol**. We both collaborated to solve the problems, choosing whichever solutions we felt were best. In the end, most of **Michał Rawski**'s contributions were included in the final version, as he has experience with Rust.

**Alexander Rosol** handled testing on a virtual machine and wrote the majority of the documentation with much guidance from **Michał Rawski**, since **Alexander Rosol** is not familiar with Rust.

## 8. Self-Evaluation

### Efficiency and performance discussion

We are happy with how the project went, as we met most milestones—often before they were even checked. Based on feedback and observed behavior, the performance appears to be good, and we believe our implementation is quite efficient.

### Limitations and Identified Shortcomings

One major limitation was running OrientDB on Windows, which led to a number of OS-specific errors. Almost all official documentation and community resources primarily target macOS, and the overall ecosystem is small. This made fixing issues nearly impossible at times. OrientDB itself was frequently the bottleneck in our workflow—for example, using its native seeding process proved so cumbersome that we opted to implement our own solution.

### Ideas for Future Improvements

Looking forward, we would like to configurable cache sizing. However, this depends heavily on the version of OrientDB, and modifying the version can break compatibility with other parts of the system.
