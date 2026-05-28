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

This build is tested on Ubuntu 24.04
