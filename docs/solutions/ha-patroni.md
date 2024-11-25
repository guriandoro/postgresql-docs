# Configure Patroni

Run the following commands on all nodes. You can do this in parallel:

1. Export and create environment variables to simplify the config file creation:

    * Node name:

       ```{.bash data-prompt="$"}
       $ export NODE_NAME=`hostname -f`
       ```

    * Node IP:

       ```{.bash data-prompt="$"}
       $ export NODE_IP=`hostname -i | awk '{print $1}'`
       ```
   
    * Create variables to store the PATH:

       ```{.bash data-prompt="$"}
       $ DATA_DIR="/var/lib/postgresql/{{pgversion}}/main"
       $ PG_BIN_DIR="/usr/lib/postgresql/{{pgversion}}/bin"
       ```

        **NOTE**: Check the path to the data and bin folders on your operating system and change it for the variables accordingly.

    * Patroni information:

       ```{.bash data-prompt="$"}
       $ NAMESPACE="percona_lab"
       $ SCOPE="cluster_1"
       ```

2. Use the following command to create the `/etc/patroni/patroni.yml` configuration file and add the following configuration for `node1`:

    ```bash
    echo "
    namespace: ${NAMESPACE}
    scope: ${SCOPE}
    name: ${NODE_NAME}

    restapi:
        listen: 0.0.0.0:8008
        connect_address: ${NODE_IP}:8008

    etcd3:
        host: ${NODE_IP}:2379

    bootstrap:
      # this section will be written into Etcd:/<namespace>/<scope>/config after initializing new cluster
      dcs:
          ttl: 30
          loop_wait: 10
          retry_timeout: 10
          maximum_lag_on_failover: 1048576

          postgresql:
              use_pg_rewind: true
              use_slots: true
              parameters:
                  wal_level: replica
                  hot_standby: "on"
                  wal_keep_segments: 10
                  max_wal_senders: 5
                  max_replication_slots: 10
                  wal_log_hints: "on"
                  logging_collector: 'on'
                  max_wal_size: '10GB'
                  archive_mode: "on"
                  archive_timeout: 600s
                  archive_command: "cp -f %p /home/postgres/archived/%f"

      # some desired options for 'initdb'
      initdb: # Note: It needs to be a list (some options need values, others are switches)
          - encoding: UTF8
          - data-checksums

      pg_hba: # Add following lines to pg_hba.conf after running 'initdb'
          - host replication replicator 127.0.0.1/32 trust
          - host replication replicator 0.0.0.0/0 md5
          - host all all 0.0.0.0/0 md5
          - host all all ::0/0 md5

      # Some additional users which needs to be created after initializing new cluster
      users:
          admin:
              password: qaz123
              options:
                  - createrole
                  - createdb
          percona:
              password: qaz123
              options:
                  - createrole
                  - createdb 
    
    postgresql:
        cluster_name: cluster_1
        listen: 0.0.0.0:5432
        connect_address: ${NODE_IP}:5432
        data_dir: ${DATA_DIR}
        bin_dir: ${PG_BIN_DIR}
        pgpass: /tmp/pgpass0
        authentication:
            replication:
                username: replicator
                password: replPasswd
            superuser:
                username: postgres
                password: qaz123
        parameters:
            unix_socket_directories: "/var/run/postgresql/"
        create_replica_methods:
            - basebackup
        basebackup:
            checkpoint: 'fast'

    tags:
        nofailover: false
        noloadbalance: false
        clonefrom: false
        nosync: false
    " | sudo tee -a /etc/patroni/patroni.yml
    ```

    ??? admonition "Patroni configuration file"

        Let's take a moment to understand the contents of the `patroni.yml` file. 

        The first section provides the details of the node and its connection ports. After that, we have the `etcd` service and its port details.

        Following these, there is a `bootstrap` section that contains the PostgreSQL configurations and the steps to run once the database is initialized. The `pg_hba.conf` entries specify all the other nodes that can connect to this node and their authentication mechanism. 


3. Check that the systemd unit file `percona-patroni.service` is created in `/etc/systemd/system`. If it is created, skip this step. 

    If it's **not created**, create it manually and specify the following contents within:

    ```ini title="/etc/systemd/system/percona-patroni.service"
     [Unit]
     Description=Runners to orchestrate a high-availability PostgreSQL
     After=syslog.target network.target 

     [Service]
     Type=simple 

     User=postgres
     Group=postgres 

     # Start the patroni process
     ExecStart=/bin/patroni /etc/patroni/patroni.yml 

     # Send HUP to reload from patroni.yml
     ExecReload=/bin/kill -s HUP $MAINPID 

     # only kill the patroni process, not its children, so it will gracefully stop postgres
     KillMode=process 

     # Give a reasonable amount of time for the server to start up/shut down
     TimeoutSec=30 

     # Do not restart the service if it crashes, we want to manually inspect database on failure
     Restart=no 

     [Install]
     WantedBy=multi-user.target
    ```

4. Make systemd aware of the new service:

    ```{.bash data-prompt="$"}
    $ sudo systemctl daemon-reload
    ```

5. Repeat steps 1-4 on the remaining nodes. In the end you must have the configuration file and the systemd unit file created on every node. 
6. Now it's time to start Patroni. You need the following commands on all nodes but not in parallel. Start with the `node1` first, wait for the service to come to live, and then proceed with the other nodes one-by-one, always waiting for them to sync with the primary node:

    ```{.bash data-prompt="$"}
    $ sudo systemctl enable --now patroni
    $ sudo systemctl restart patroni
    ```
   
    When Patroni starts, it initializes PostgreSQL (because the service is not currently running and the data directory is empty) following the directives in the bootstrap section of the configuration file. 

6. Check the service to see if there are errors:

    ```{.bash data-prompt="$"}
    $ sudo journalctl -fu patroni
    ```

    A common error is Patroni complaining about the lack of proper entries in the pg_hba.conf file. If you see such errors, you must manually add or fix the entries in that file and then restart the service.

    Changing the patroni.yml file and restarting the service will not have any effect here because the bootstrap section specifies the configuration to apply when PostgreSQL is first started in the node. It will not repeat the process even if the Patroni configuration file is modified and the service is restarted.

    Proceed with staring Patroni on the remaining nodes. 

7. Check the cluster. Run the following command on any node:
 
    ```{.bash data-prompt="$"}
    $ patronictl -c /etc/patroni/patroni.yml list $SCOPE
    ```

    The output resembles the following:

    ```{.text .no-copy}
    + Cluster: cluster_1 (7440127629342136675) -----+----+-------+
    | Member | Host       | Role    | State     | TL | Lag in MB |
    +--------+------------+---------+-----------+----+-----------+
    | node1  | 10.0.100.1 | Leader  | running   |  1 |           |
    | node2  | 10.0.100.2 | Replica | streaming |  1 |         0 |
    | node3  | 10.0.100.3 | Replica | streaming |  1 |         0 |
    +--------+------------+---------+-----------+----+-----------+
    ```

If Patroni has started properly, you should be able to locally connect to a PostgreSQL node using the following command:

```{.bash data-prompt="$"}
$ sudo psql -U postgres
```

The command output is the following:

```
psql ({{dockertag}})
Type "help" for help.

postgres=#
```