# README


## About 
DozerDb enhances Neo4j core / AKA Neo4j Community Edition with enterprise features.

This project builds the actual plugin which is used to enhance Neo4j Community Edition. 

See https://dozerdb.org for installation instructions.

## Versioning
The plugin combines the dozerdb-core and dozerdb-browser artifacts into an uber jar which is dropped into the Neo4j lib directory and takes control of the bootstrap process.

The version uses the current Neo4j full version number and appends the final number anytime the dozerdb-core or dozerdb-browser projects have an update.

Example - for Neo4j version 5.26.26 - the first release would be 5.26.26.0.
If a change occurs within the dozerdb-browser or dozerdb-core package - then the version would become:  5.26.26.1 and so on.

## Development
Please ensure you use java 17 or above when working on the plugin.

If you would like to use an open source java version manager - please check out https://sdkman.io/
For those using sdkman - you can use the following command to switch to the favor of openjdk that we use.



## Building

The plugin is a shaded uber-jar that bundles two sibling projects. To build it locally
you need all three repositories checked out side-by-side:

- [`dozerdb-plugin`](https://github.com/dozerdb/dozerdb-plugin) (this repo) — produces the uber-jar
- [`dozerdb-core`](https://github.com/dozerdb/dozerdb-core) — Java/Scala enterprise enhancements for Neo4j
- [`dozerdb-browser`](https://github.com/dozerdb/dozerdb-browser) — the embedded Neo4j Browser fork

### Prerequisites

- JDK 17+ (see [sdkman.io](https://sdkman.io/) to manage JDK installations)
- Node.js 18+ and Yarn 1.x (Classic) — for the browser build
- Maven 3.6+ on your `PATH` — `dozerdb-browser` uses a system Maven; `dozerdb-core` and `dozerdb-plugin` ship `./mvnw`
- Git

### 1. Clone the three repositories as siblings

```bash
mkdir dozerdb && cd dozerdb
git clone https://github.com/dozerdb/dozerdb-core.git
git clone https://github.com/dozerdb/dozerdb-browser.git
git clone https://github.com/dozerdb/dozerdb-plugin.git
```

In each repo, check out the branch matching your target Neo4j version. For Neo4j 5.26.x:

| Repo              | Branch                                   |
| ----------------- | ---------------------------------------- |
| `dozerdb-core`    | `neo4j-5.26.X` (e.g. `neo4j-5.26.26`)    |
| `dozerdb-browser` | `dozerdb-5.26.X` (e.g. `dozerdb-5.26.26`)|
| `dozerdb-plugin`  | `neo4j-lts-5.x`                          |

### 2. Install `dozerdb-core` into your local Maven repository

```bash
cd dozerdb-core
./mvnw -DskipTests install
cd ..
```

### 3. Build and install `dozerdb-browser`

```bash
cd dozerdb-browser
yarn install
NODE_OPTIONS="--openssl-legacy-provider" yarn build
yarn prepare-jar
mvn -DskipTests install
cd ..
```

`--openssl-legacy-provider` is required on Node.js 17+ because the embedded webpack
version relies on a hashing API removed from newer OpenSSL.

### 4. Build the plugin

```bash
cd dozerdb-plugin
./mvnw clean verify
```

The shaded uber-jar will be at `target/dozerdb-plugin-<version>.jar`. Copy it into
your Neo4j installation's `lib/` directory and restart the database.

## Configuring the maximum number of databases

DozerDB lifts Neo4j Community's single-user-database restriction. The cap is
controlled by `dbms.max_databases` in `conf/neo4j.conf`:

```properties
# Default is 100. Bump as needed.
dbms.max_databases=500
```

Restart Neo4j for the change to take effect. Creating a database past the limit
fails with a clear error message.

### Tuning when running many databases

Each Neo4j database — even idle — owns its own page-cache region, open file
handles, transaction-log files, schema caches, and bolt/transaction-manager
state. The defaults that ship with Neo4j are sized for a handful of databases,
not hundreds. Review these tunables when you raise `dbms.max_databases`:

| Concern              | Tunable                                                | Notes                                                                                                                                       |
| -------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------- |
| JVM heap             | `server.memory.heap.max_size`, `server.memory.heap.initial_size` (in `neo4j.conf`) | Per-database metadata, schema caches, and transaction state live on heap. Allow comfortable headroom. Set initial = max to avoid resize pauses. |
| Page cache           | `server.memory.pagecache.size` (in `neo4j.conf`)        | Size for your **active** working set, not linearly with database count. Idle databases barely touch the page cache.                          |
| Open file descriptors| OS-level `ulimit -n` (interactive) or `LimitNOFILE` in the systemd unit | Each database keeps multiple store/index/log files open even when idle. Default 1024 is too low — raise to 40000+ for hundreds of databases. |
| Heap-vs-page-cache   | Total ≤ ~70% of system RAM                              | Leave room for the OS file cache and per-process overhead. The Neo4j memory recommendation tool (`neo4j-admin server memory-recommendation`) is a good starting point. |

If a database refuses to open after raising `dbms.max_databases`, check the OS
file-descriptor limit first — it's almost always the bottleneck.
