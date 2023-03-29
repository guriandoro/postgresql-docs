# Install Percona Distribution for PostgreSQL on Red Hat Enterprise Linux and derivatives

This document describes how to install Percona Server for PostgreSQL from Percona repositories on RPM-based distributions such as Red Hat Enterprise Linux and compatible derivatives..

## Platform Specific Notes

If you intend to install Percona Distribution for PostgreSQL on Red Hat Enterprise Linux v8, disable the ``postgresql``  and ``llvm-toolset``modules:

```{.bash data-prompt="$"}
$ sudo dnf module disable postgresql llvm-toolset
```

On CentOS 7, you should install the ``epel-release`` package:

```{.bash data-prompt="$"}
$ sudo yum -y install epel-release
$ sudo yum repolist
```

## Procedure

Run all the commands in the following sections as root or using the `sudo` command:

### Configure the repository

1. Install the `percona-release` repository management tool to subscribe to Percona repositories:

    ```{.bash data-prompt="$"}
    $ sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
    ```

2. Enable the repository

   Percona provides [two repositories](repo-overview.md) for Percona Distribution for PostgreSQL. We recommend enabling the Major release repository to timely receive the latest updates. 

   To enable a repository, we recommend using the `setup` command: 

   ```{.bash data-prompt="$"}
   $ sudo percona-release setup ppg-11
   ```

### Install packages

=== "Install using meta-package"
     
     ```{.bash data-prompt="$"}
     $ sudo yum install percona-ppg-server11
     ```

=== "Install packages individually"

     1. Install the PostgreSQL server package:

         ```{.bash data-prompt="$"}
         $ sudo yum install percona-postgresql11-server
         ```

     2. Install the components:

        Install `pg_repack`:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-pg_repack11
        ```

        Install `pgaudit`:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-pgaudit
        ```

        Install `pgBackRest`:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-pgbackrest
        ```

        Install `Patroni`:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-patroni
        ```

        [Install `pg_stat_monitor`](pg-stat-monitor.md):


        Install `pgBouncer`:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-pgbouncer
        ```

        Install `pgAudit-set_user`:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-pgaudit11_set_user
        ```

        Install `pgBadger`:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-pgbadger
        ```

        Install `wal2json`:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-wal2json11
        ```

        Install PostgreSQL contrib extensions:

        ```{.bash data-prompt="$"}
        $ sudo yum install percona-postgresql11-contrib
        ```

        Install HAProxy
        
        ```{.bash data-prompt="$"}
        $ sudo yum install percona-haproxy
        ```

        Some extensions require additional setup in order to use them with Percona Distribution for PostgreSQL. For more information, refer to [Enabling extensions](enable-extensions.md).

### Start the service

After the installation, the default database storage is not automatically initialized. To complete the installation and start Percona Distribution for PostgreSQL, initialize the database using the following command:

```{.bash data-prompt="$"}
/usr/pgsql-11/bin/postgresql-11-setup initdb
```

Start the PostgreSQL service:

```{.bash data-prompt="$"}
$ sudo systemctl start postgresql-11
```

### Connect to the PostgreSQL server

By default, `postgres` user and `postgres` database are created in PostgreSQL upon its installation and initialization. This allows you to connect to the database as the `postgres` user.

```{.bash data-prompt="$"}
$ sudo su postgres
```

Open the PostgreSQL interactive terminal:

```{.bash data-prompt="$"}
$ psql
```

!!! hint

    You can connect to `psql` as the `postgres` user in one go:

    ```{.bash data-prompt="$"}
    $ sudo su postgres psql
    ```

To exit the `psql` terminal, use the following command:

```{.bash data-prompt="$"}
$ \q
```