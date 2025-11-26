# TPC-DS Toolkit for Cloudberry Database

[![TPC-DS](https://img.shields.io/badge/TPC--DS-v4.0.0-blue)](http://www.tpc.org/tpcds/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](LICENSE)

A comprehensive and automated tool for running TPC-DS benchmarks on Cloudberry Database, HashData, Greenplum, and PostgreSQL. Originally derived from [Pivotal TPC-DS](https://github.com/pivotal/TPC-DS).

## Overview

This toolkit provides:
- End-to-end automated TPC-DS benchmark execution
- Support for both local and cloud deployment models
- Configurable data generation (1GB to 100TB scale factors)
- Flexible query execution with customizable parameters
- Detailed performance reporting and scoring
- Support for multiple database products and versions
- Optimized for MPP architectures
- Easy-to-use configuration system

## Table of Contents
- [TPC-DS Toolkit for Cloudberry Database](#tpc-ds-toolkit-for-cloudberry-database)
  - [Overview](#overview)
  - [Table of Contents](#table-of-contents)
  - [Quick Start](#quick-start)
    - [Guides to run test on MPP Architecture with "local" mode](#guides-to-run-test-on-mpp-architecture-with-local-mode)
    - [Guides to run test on Postgresql compatible database with "Cloud" mode](#guides-to-run-test-on-postgresql-compatible-database-with-cloud-mode)
  - [Supported TPC-DS Versions](#supported-tpc-ds-versions)
  - [Prerequisites](#prerequisites)
    - [Tested Products](#tested-products)
    - [Local Cluster Setup](#local-cluster-setup)
    - [Remote Client Setup](#remote-client-setup)
    - [TPC-DS Toolkit Process](#tpc-ds-toolkit-process)
    - [TPC-DS Tools Dependencies](#tpc-ds-tools-dependencies)
  - [Installation](#installation)
  - [Usage](#usage)
  - [Configuration](#configuration)
    - [Environment Options](#environment-options)
    - [Benchmark Options](#benchmark-options)
    - [Storage Options](#storage-options)
    - [Step Control Options](#step-control-options)
    - [Miscellaneous Options](#miscellaneous-options)
  - [Performance Tuning](#performance-tuning)
  - [Benchmark Modifications](#benchmark-modifications)
    - [1. Date Interval Syntax Changes](#1-date-interval-syntax-changes)
    - [2. ORDER BY Column Alias Fixes](#2-order-by-column-alias-fixes)
    - [3. Column Reference Corrections](#3-column-reference-corrections)
    - [4. Table Alias Additions](#4-table-alias-additions)
    - [5. Result Limiting](#5-result-limiting)
  - [Troubleshooting](#troubleshooting)
    - [Common Issues and Solutions](#common-issues-and-solutions)
    - [Logs and Diagnostics](#logs-and-diagnostics)
  - [Contributing](#contributing)
  - [License](#license)

## Quick Start

### Basic Usage

```bash
# 1. Clone the repository
git clone https://github.com/cloudberry-contrib/TPC-DS-Toolkit.git
cd TPC-DS-Toolkit

# 2. Configure your environment
vim tpcds_variables.sh

# 3. Run the benchmark
./run.sh
```

### Detailed Guides

#### Local Mode (for MPP Architecture)

**Best for**: Cloudberry Database, Greenplum, HashData Lightning

This mode leverages the MPP architecture by generating data directly on segment nodes and loading it using the `gpfdist` protocol, maximizing resource utilization for faster benchmark execution.

**Key Configuration**:
```bash
export RUN_MODEL="local"
export TABLE_ACCESS_METHOD="USING ao_column"
export TABLE_USE_PARTITION="true"
```

**Requirements**:
- Running Cloudberry Database or Greenplum cluster
- `gpadmin` access to the coordinator node
- Password-less SSH between coordinator and segment nodes

Please refer to the [QuickStartLocal.md](tpcds_tools/QuickStartLocal.md) for detailed instructions.

#### Cloud Mode (for PostgreSQL Compatible Databases)

**Best for**: HashData Cloud, Cloudberry Database, Greenplum, PostgreSQL

This mode generates data on the client machine and imports it into the database using the `COPY` command, making it suitable for cloud deployments and remote clients.

**Key Configuration**:
```bash
export RUN_MODEL="cloud"
export PSQL_OPTIONS="-h <host> -p <port> -U <user>"
export RANDOM_DISTRIBUTION="true"
export TABLE_USE_PARTITION="true"
```

**Requirements**:
- `psql` client installed on the client machine
- Passwordless database access (via `.pgpass` file)
- Sufficient disk space for data generation

Please refer to the [QuickStartCloud.md](tpcds_tools/QuickStartCloud.md) for detailed instructions.


## Supported TPC-DS Versions

| Version | Date | Specification |
|---------|------|---------------|
| 4.0.0 | 2023/12/15 | [PDF](http://www.tpc.org/tpc_documents_current_versions/pdf/tpc-ds_v4.0.0.pdf) |
| 3.2.0 | 2021/06/15 | [PDF](http://www.tpc.org/tpc_documents_current_versions/pdf/tpc-ds_v3.2.0.pdf) |
| 2.1.0 | 2015/11/12 | [PDF](http://www.tpc.org/tpc_documents_current_versions/pdf/tpc-ds_v2.1.0.pdf) |
| 1.3.1 | 2015/02/19 | [PDF](http://www.tpc.org/tpc_documents_current_versions/pdf/tpc-ds_v1.3.1.pdf) |

This tool uses TPC-DS 4.0.0 as of the latest version.

## Prerequisites

This tool is built with shell scripts and has been tested primarily on CentOS-based operating systems. To accommodate different products, various options are available to choose storage types, partitions, optimizer settings, and distribution policies. Please review the `tpcds_variables.sh` for detailed configuration options for different models and products.

### Tested Products
- Cloudberry Database 1.x / 2.x
- HashData Enterprise / HashData Lightning
- Greenplum Database 4.x / 5.x / 6.x / 7.x
- PostgreSQL 15.x / 16.x / 17.x

### Local Cluster Setup
For running tests on the coordinator host:

This mode leverages the MPP architecture to use data directories of segment nodes to generate data and load data using the 'gpfdist' protocol. More resources will be utilized for data generation and loading to accelerate the test process.

1. Set `RUN_MODEL="local"` in `tpcds_variables.sh`
2. Ensure you have a running Cloudberry Database with `gpadmin` access
3. Create a `gpadmin` database
4. Configure password-less `ssh` between `mdw` (coordinator) and segment nodes (`sdw1..n`)

### Remote Client Setup  
For running tests from a remote client:

With this mode, all data will be generated on the client machine, and data will be imported into the database using the `copy` command. This mode works for HashData Cloud, Cloudberry, Greenplum, HashData Lightning, and should work for other PostgreSQL-compatible products. However, it is recommended to use `local` mode for non-Cloud MPP products.

1. Set `RUN_MODEL="cloud"` in `tpcds_variables.sh`
2. Install `psql` client with passwordless access (`.pgpass`)
3. Create `gpadmin` database with:
   ```sql
   ALTER ROLE gpadmin SET warehouse=testforcloud;
   ```
4. Configure required variables in `tpcds_variables.sh`:
   ```bash
   export RANDOM_DISTRIBUTION="true"
   export TABLE_STORAGE_OPTIONS="compresstype=zstd, compresslevel=5"
   export CUSTOM_GEN_PATH="/tmp/dsbenchmark" 
   export GEN_DATA_PARALLEL="2"
   ```
> The following conventions are used in this document: mdw for the coordinator node, and sdw1..n for segment nodes.

### TPC-DS Toolkit Process

The TPC-DS benchmark execution follows these sequential steps:

1. **Compile TPC-DS Tools** (`00_compile_tpcds`)
   - Builds the `dsdgen` (data generation) and `dsqgen` (query generation) tools from source
   - Only needs to be run once unless the toolkit is updated

2. **Generate Test Data** (`01_gen_data`)
   - Creates datasets using `dsdgen` based on the specified scale factor
   - Supports parallel data generation for faster execution
   - Data is generated either locally on segment nodes or on a remote client depending on the run model

3. **Initialize Cluster** (`02_init`)
   - Configures database settings and GUCs for optimal benchmark performance
   - Records segment configurations for data generation and loading

4. **Create Database Objects** (`03_ddl`)
   - Creates schemas, tables, and indexes according to TPC-DS specifications
   - Supports different storage types, partitioning, and distribution policies
   - Can drop and recreate existing objects if needed

5. **Load Data** (`04_load`)
   - Imports generated datasets into the database
   - Uses `gpfdist` protocol for local mode or `COPY` command for cloud mode
   - Supports parallel loading for faster data ingestion

6. **Analyze Tables** (`05_analyze`)
   - Computes table statistics for optimal query planning
   - Improves query performance by providing accurate cardinality estimates

7. **Generate Queries** (`06_sql`)
   - Creates the 99 TPC-DS benchmark queries using `dsqgen`
   - Supports custom query generation seeds for reproducible results

8. **Single User Test (Power Test)**
   - Executes all 99 queries sequentially to measure single-threaded performance
   - Measures the time to complete all queries (power metric)

9. **Single User Reports** (`07_single_user_reports`)
   - Generates detailed reports for the single user test results
   - Stores results in the database for further analysis

10. **Multi User Test (Throughput Test)** (`08_multi_user`)
    - Executes multiple query streams concurrently to measure system throughput
    - Simulates real-world workloads with multiple users accessing the system

11. **Multi User Reports** (`09_multi_user_reports`)
    - Generates detailed reports for the multi user test results
    - Analyzes query performance under concurrent workloads

12. **Final Score Calculation** (`10_score`)
    - Computes the final QphDS (Queries per Hour TPC-DS) score
    - Combines power and throughput metrics according to TPC-DS specifications


### TPC-DS Tools Dependencies

Install the dependencies on `mdw` for compiling the `dsdgen` (data generation) and `dsqgen` (query generation) tools:

```bash
ssh root@mdw
yum -y install gcc make byacc flex unzip
```

The original source code is from the [TPC website](http://tpc.org/tpc_documents_current_versions/current_specifications5.asp).

## Installation

Simply clone the repository with Git or download the source code from GitHub:

```bash
ssh gpadmin@mdw
git clone https://github.com/cloudberry-contrib/TPC-DS-Toolkit.git
```

Place the folder under `/home/gpadmin/` and change ownership to gpadmin:

```bash
chown -R gpadmin:gpadmin TPC-DS-Toolkit
```

## Usage

To run the benchmark, login as `gpadmin` on the coordinator node (`mdw`):

```bash
ssh gpadmin@mdw
cd ~/TPC-DS-Toolkit
./run.sh
```

By default, this will run a scale 1 (1GB) benchmark with 1 concurrent user, from data generation through to score computation, in the background. Logs will be stored with the name `tpcds_<timestamp>.log` in the `~/TPC-DS-Toolkit` directory.

## Configuration

The benchmark is controlled through the `tpcds_variables.sh` file, which is organized into the following modules:

### Environment Options

These options define the core environment settings for the benchmark:

```bash
# Core settings
export ADMIN_USER="gpadmin"        # OS user that executes this toolkit
export BENCH_ROLE="dsbench"         # Database user for running the benchmark
export DB_SCHEMA_NAME="tpcds"       # Database schema for TPC-DS tables
export RUN_MODEL="local"            # "local" or "cloud" run mode

# Remote cluster connection
export PSQL_OPTIONS=""              # Database connection options (host, port, user)
```

### Benchmark Options

These options control the scale and concurrency of the benchmark:

```bash
export GEN_DATA_SCALE="1"           # Scale factor (1 = 1GB, 1000 = 1TB)
export MULTI_USER_COUNT="2"         # Number of concurrent users for throughput tests

```

### Step Options

These options control the execution of each benchmark step:

```bash
## Step 00_compile_tpcds: Compile TPC-DS tools
export RUN_COMPILE_TPCDS="true"  # Compile data/query generators (one-time setup)

## Step 01_gen_data: Generate test data
export RUN_GEN_DATA="true"       # Generate test data
export GEN_NEW_DATA="true"       # Generate new data vs reusing existing data
### Default path to store the generated benchmark data, separated by space for multiple paths.
export CUSTOM_GEN_PATH="/tmp/dsbenchmark"
### How many parallel processes to run on each data path to generate data in all modes
export GEN_DATA_PARALLEL="2"
### The following variables only take effect when RUN_MODEL is set to "local".
export USING_CUSTOM_GEN_PATH_IN_LOCAL_MODE="false"  # Use custom data generation path in local mode

## Step 02_init: Initialize cluster
export RUN_INIT="true"           # Initialize cluster settings and GUCs

## Step 03_ddl: Create database objects
export RUN_DDL="true"            # Create database schemas/tables
export DROP_EXISTING_TABLES="true" # Drop existing tables before creating new ones

## Step 04_load: Load generated data
export RUN_LOAD="true"           # Load generated data
export LOAD_PARALLEL="2"         # Number of parallel processes to load data (max 24)
export TRUNCATE_TABLES="true"    # Truncate existing tables before loading data

## Step 05_analyze: Compute table statistics
export RUN_ANALYZE="true"        # Compute table statistics for query optimization
export RUN_ANALYZE_PARALLEL="5"  # Number of parallel processes for analyze (max 24)

## Step 06_sql: Generate and run queries
export RUN_SQL="true"                 # Run power test queries
export RUN_QGEN="true"                # Generate queries for TPC-DS benchmark
export UNIFY_QGEN_SEED="true"         # Use unified seed for query generation
export QUERY_INTERVAL="0"             # Wait time between each query execution
export ON_ERROR_STOP="0"              # Stop on error flag (1 to stop)

## Step 07_single_user_reports: Generate single user reports
export RUN_SINGLE_USER_REPORTS="true" # Generate single-user test results

## Step 08_multi_user: Run multi-user test
export RUN_MULTI_USER="false"         # Run throughput test queries
export RUN_MULTI_USER_QGEN="true"     # Generate queries for multi-user test

## Step 09_multi_user_reports: Generate multi-user reports
export RUN_MULTI_USER_REPORTS="false" # Generate multi-user test results

## Step 10_score: Calculate final score
export RUN_SCORE="false"              # Compute final benchmark score
```

### Misc Options

These options control various miscellaneous settings:

```bash
export LOG_DEBUG="false"                # Enable debug logging
export SINGLE_USER_ITERATIONS="1"      # Number of times to run the power test
export EXPLAIN_ANALYZE="false"         # Set to true for query plan analysis
export RANDOM_DISTRIBUTION="false"     # Use random distribution for fact tables
export ENABLE_VECTORIZATION="off"      # Set to on/off to enable vectorization
export STATEMENT_MEM="1GB"             # Memory per statement for single-user test
export STATEMENT_MEM_MULTI_USER="1GB"  # Memory per statement for multi-user test
export GPFDIST_LOCATION="p"            # Where gpfdist will run: p (primary) or m (mirror)
export OSVERSION=$(uname)
export ADMIN_USER=$(whoami)
export ADMIN_HOME=$(eval echo ${HOME}/${ADMIN_USER})
export MASTER_HOST=$(hostname -s)
export DB_SCHEMA_NAME="$(echo "${DB_SCHEMA_NAME}" | tr '[:upper:]' '[:lower:]')"
export DB_EXT_SCHEMA_NAME="ext_${DB_SCHEMA_NAME}"
export GEN_PATH_NAME="dsgendata_${DB_SCHEMA_NAME}"
export BENCH_ROLE="$(echo "${BENCH_ROLE}" | tr '[:upper:]' '[:lower:]')"
export DB_CURRENT_USER=$(psql ${PSQL_OPTIONS} -t -c "SELECT current_user;" 2>/dev/null | tr -d '[:space:]')
```

Key options explained:

- `LOG_DEBUG`: When set to `true`, enables detailed debug logging for troubleshooting.
- `EXPLAIN_ANALYZE`: When set to `true`, executes queries with `EXPLAIN ANALYZE` to see query plans, costs, and memory usage. For debugging only, as it affects benchmark results.
- `RANDOM_DISTRIBUTION`: When set to `true`, fact tables are distributed randomly rather than using pre-defined distribution columns. Recommended for cloud products.
- `SINGLE_USER_ITERATIONS`: Controls how many times the power test runs. The fastest query time from multiple runs is used for final scoring.
- `STATEMENT_MEM`: Sets memory per statement for single-user tests. Should be less than `gp_vmem_protect_limit`.
- `STATEMENT_MEM_MULTI_USER`: Sets memory per statement for multi-user tests. Note: `STATEMENT_MEM_MULTI_USER` × `MULTI_USER_COUNT` should be less than `gp_vmem_protect_limit`.
- `ENABLE_VECTORIZATION`: Set to `on` to enable vectorized computing for better performance (supported in Cloudberry Database 2.0+ and HashData Lightning 1.5.3+). Only works with AO column and PAX table formats.
- `GPFDIST_LOCATION`: Specifies where `gpfdist` will run: `p` for primary segment nodes or `m` for mirror segment nodes.

### Storage Options

These options control the storage settings for tables:

```bash
## Support TABLE_ACCESS_METHOD as ao_row / ao_column / heap in both GPDB 7 / CBDB
## Support TABLE_ACCESS_METHOD as "PAX" for PAX table format for CBDB 2.0 only.
export TABLE_ACCESS_METHOD="USING ao_column"  # Uncomment to enable

## Set to use partition for the following tables:
## catalog_returns / catalog_sales / inventory / store_returns / store_sales / web_returns / web_sales
export TABLE_USE_PARTITION="true"

## SET TABLE_STORAGE_OPTIONS with different options in GP/CBDB/Cloud
export TABLE_STORAGE_OPTIONS="WITH (appendoptimized=true, orientation=column, compresstype=zstd, compresslevel=5)"
```

**Note**: 
- `TABLE_ACCESS_METHOD`: Default is commented out for compatibility with all products. For Cloudberry Database and Greenplum 7.0+, set to `USING ao_column` for best performance. `USING PAX` is available exclusively for Cloudberry Database 2.0.
- For earlier Greenplum versions without `TABLE_ACCESS_METHOD` support, use full storage options in `TABLE_STORAGE_OPTIONS`:
  ```bash
  export TABLE_STORAGE_OPTIONS="appendoptimized=true, orientation=column, compresstype=zlib, compresslevel=5, blocksize=1048576"
  ```
- Distribution policies are defined in `TPC-DS-Toolkit/03_ddl/distribution.txt`. For products supporting `REPLICATED` distribution, 14 dimension tables use `REPLICATED` distribution by default. For early Greenplum versions without `REPLICATED` support, see `TPC-DS-Toolkit/03_ddl/distribution_original.txt`.
- Table partition definitions are in `TPC-DS-Toolkit/03_ddl/*.sql.partition`. When using partitioning with column-oriented tables, large block sizes may cause high memory consumption. Reduce block size or partition count if encountering out-of-memory errors.

## Performance Tuning

For optimal performance, consider the following tuning recommendations:

1. **Memory Settings**
   ```bash
   # Recommended for 8GB+ RAM per segment node cluster
   export STATEMENT_MEM="8GB"
   # Should be set to STATEMENT_MEM / MULTI_USER_COUNT for balanced performance
   export STATEMENT_MEM_MULTI_USER="4GB"
   ```

2. **Storage Optimization**
   ```bash
   # For best compression and query performance
   export TABLE_ACCESS_METHOD="USING ao_column"
   export TABLE_STORAGE_OPTIONS="WITH (appendoptimized=true, orientation=column, compresstype=zstd, compresslevel=9)"
   # Use partitioned tables for better query performance on large datasets
   export TABLE_USE_PARTITION="true"
   ```

3. **Concurrency Tuning**
   ```bash
   # Adjust based on available CPU cores
   export GEN_DATA_PARALLEL="$(nproc)"
   # Set based on available system resources and expected workload
   export MULTI_USER_COUNT="$(( $(nproc) / 2 ))"
   # Increase parallel loading for faster data ingestion
   export LOAD_PARALLEL="4"
   export RUN_ANALYZE_PARALLEL="8"
   ```

4. **Enable Vectorization** (for supported systems)
   ```bash
   export ENABLE_VECTORIZATION="on"
   ```

5. **Distribution Policy**
   ```bash
   # Use random distribution for cloud products or when unsure about data distribution
   export RANDOM_DISTRIBUTION="true"
   ```

6. **Optimizer Settings**
   ```bash
   # Adjust optimizer settings in 01_gen_data/optimizer.txt
   # Configure ORCA vs Planner usage for each query
   # After changing settings, regenerate queries with RUN_QGEN="true"
   ```

7. **Query Generation**
   ```bash
   # Use unified seed for reproducible results
   export UNIFY_QGEN_SEED="true"
   ```

8. **Power Test Iterations**
   ```bash
   # Run power test multiple times and use the fastest result
   export SINGLE_USER_ITERATIONS="3"
   ```

## Benchmark Modifications

The TPC-DS queries were modified in the following ways to ensure compatibility:

### 1. Date Interval Syntax Changes
Changed date addition syntax from:
```sql
and (cast('2000-02-28' as date) + 30 days)
```
To:
```sql
and (cast('2000-02-28' as date) + '30 days'::interval)
```
Affected queries: 5, 12, 16, 20, 21, 32, 37, 40, 77, 80, 82, 92, 94, 95, and 98.

### 2. ORDER BY Column Alias Fixes
Added subqueries for ORDER BY clauses with column aliases:
```sql
-- New version with subquery
select * from (
  -- Original query
) AS sub
order by
  lochierarchy desc
  ,case when lochierarchy = 0 then s_state end
  ,rank_within_parent
limit 100;
```
Affected queries: 36 and 70.

### 3. Column Reference Corrections
Modified query templates to exclude columns not found in the query, specifically in common table expressions where alias columns were used in dynamic filters.
Affected query: 86.

### 4. Table Alias Additions
Added table aliases to improve query parser compatibility.
Affected queries: 2, 14, and 23.

### 5. Result Limiting
Added `LIMIT 100` to queries that could produce very large result sets.
Affected queries: 64, 34, and 71.

## Troubleshooting

### Common Issues and Solutions

1. **Missing or Invalid Environment Variables**  
   Ensure all required environment variables in `tpcds_variables.sh` are set correctly. If any variable is missing or invalid, the script will abort and display the problematic variable name. Double-check the following key variables:
   - `RUN_MODEL`
   - `GEN_DATA_SCALE`
   - `TABLE_ACCESS_METHOD`
   - `PSQL_OPTIONS`

2. **Permission Errors**  
   - Verify ownership: `chown -R gpadmin:gpadmin /home/gpadmin/TPC-DS-Toolkit`
   - Ensure `gpadmin` has proper database access permissions

3. **Data Generation Failures**  
   - Confirm successful compilation of `dsdgen`
   - Verify `CUSTOM_GEN_PATH` points to a valid, writable directory
   - Check available disk space

4. **Query Execution Errors**  
   - Ensure tables and schemas exist (set `RUN_DDL=true` on first run)
   - Look for syntax errors in modified queries
   - Verify database connectivity

5. **Performance Issues**  
   - Adjust memory settings based on system resources
   - Enable vectorization if supported
   - Use appropriate storage options for your workload
   - Consider partitioning for large tables
   - Adjust database parameters as referred to: [set_gucs](tpcds_tools/set_gucs.sh)

### Logs and Diagnostics

For detailed diagnostics, examine:
- Main log file: `tpcds_<timestamp>.log` in `~/TPC-DS-Toolkit`
- Database server logs
- System resource utilization during test runs

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.