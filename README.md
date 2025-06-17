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
