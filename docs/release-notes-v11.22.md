# Percona Distribution for PostgreSQL 11.22 (2023-12-13)


| Release date:     | December 13, 2023      |
|:------------------|:----------------------|
| **Installation**: | [Installing Percona Distribution for PostgreSQL](installing.md) |

Percona Distribution for PostgreSQL is a solution with the collection of tools from PostgreSQL community that are tested to work together and serve to assist you in deploying and managing PostgreSQL. The aim of Percona Distribution for PostgreSQL is to address the operational issues like High-Availability, Disaster Recovery, Security, Spatial data handling, Observability, Performance and Scalability and others that enterprises are facing.

This release is based on [PostgreSQL
11.22](https://www.postgresql.org/docs/release/11.22/). 

## Release Highlights

* This is the last minor release of PostgreSQL 11. [Learn more about PostgreSQL 11 end of life implications](https://www.percona.com/blog/postgresql-11-will-soon-reach-end-of-life/).
* Docker images are now available for x86_64 architectures. Their inclusion in the distribution aims to simplify the developers' experience with the Distribution. Refer to the [Docker guide](docker.md) for how to run Percona Distribution for PostgreSQL in Docker.
* Telemetry is now enabled in Percona Distribution for PostgreSQL to fill in the gaps in our understanding of how you use it and help us improve our products. Participation in the anonymous program is optional. You can opt-out if you prefer not to share this information. Find more information in the [on Percona Distribution for PostgreSQL](telemetry.md) document.
* The `percona-postgis33` and `percona-pgaudit` packages on YUM-based operating systems are renamed `percona-postgis33_{{pgversion}}` and `percona-pgaudit{{pgversion}}` respectively.

-----------------------------------------------------------------------------

The following is the list of extensions available in Percona Distribution for PostgreSQL.

| Extension           | Version        | Description                  |
| ------------------- | -------------- | ---------------------------- |
|[HAProxy](http://www.haproxy.org/) | 2.8.3 | a high-availability and load-balancing solution |
| [Patroni](https://patroni.readthedocs.io/en/latest/) | 3.1.0 | a HA (High Availability) solution for PostgreSQL |
| [PgAudit](https://www.pgaudit.org/)             | 1.3.4   | provides detailed session or object audit logging via the standard logging facility provided by PostgreSQL                |
| [pgAudit set_user](https://github.com/pgaudit/set_user)| 4.0.1 | provides an additional layer of logging and control when unprivileged users must escalate themselves to superusers or object owner roles in order to perform needed maintenance tasks.|
| [pgBackRest](https://pgbackrest.org/)           | 2.48    | a backup and restore solution for PostgreSQL       |
|[pgBadger](https://github.com/darold/pgbadger)   | 12.2     | a fast PostgreSQL Log Analyzer.|
|[PgBouncer](https://www.pgbouncer.org/)          |1.21.0    | a lightweight connection pooler for PostgreSQL|
| [pg_gather](https://github.com/jobinau/pg_gather)| v23     | an SQL script for running the diagnostics of the health of PostgreSQL cluster |
| [pgpool2](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=summary) | 4.4.4 | a middleware between PostgreSQL server and client for high availability, connection pooling and load balancing.|
| [pg_repack](https://github.com/reorg/pg_repack) | 1.4.8   | rebuilds PostgreSQL database objects           |
| [pg_stat_monitor](https://github.com/percona/pg_stat_monitor)|2.0.3 | collects and aggregates statistics for PostgreSQL and provides histogram information.|
| [PostGIS](https://github.com/postgis/postgis) | 3.3.4 | a spatial extension for PostgreSQL.|
| [PostgreSQL Common](https://salsa.debian.org/postgresql/postgresql-common)| 256 | PostgreSQL database-cluster manager. It provides a structure under which multiple versions of PostgreSQL may be installed and/or multiple clusters maintained at one time.|
|[wal2json](https://github.com/eulerto/wal2json)  |2.5       | a PostgreSQL logical decoding JSON output plugin|


Percona Distribution for PostgreSQL also includes the following packages:

- `llvm` 12.0.1 packages for Red Hat Enterprise Linux 8  and derivatives. This fixes compatibility issues with LLVM from upstream.
- supplemental ETCD packages which can be used for setting up Patroni clusters. These packages are available for the following operating systems:

|  Operating System   |Package               | Version | Description   |
| ------------------- | ---------------------| --------|---------------|
| RHEL 7             |`python3-python-etcd` | 0.4.5   | A Python client for ETCD     |
| RHEL 8             | `etcd`               | 3.3.11  | A consistent, distributed key-value store|
|                     | `python3-python-etcd`| 0.4.5   | A Python client for ETCD     |


Percona Distribution for PostgreSQL is also shipped with the
[libpq](https://www.postgresql.org/docs/11/libpq.html) library. It
contains "a set of library functions that allow client programs to pass
queries to the PostgreSQL backend server and to receive the results of
these queries."