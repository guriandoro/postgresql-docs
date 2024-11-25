# Configure etcd distributed store  

The distributed configuration store provides a reliable way to store data that needs to be accessed by large scale distributed systems. The most popular implementation of the distributed configuration store is etcd. etcd is deployed as a cluster for fault-tolerance and requires an odd number of members (n/2+1) to agree on updates to the cluster state. An etcd cluster helps establish a consensus among nodes during a failover and manages the configuration for the three PostgreSQL instances.

This document provides configuration for etcd version 3.5.x. For how to configure etcd cluster with earlier versions of etcd, read the blog post by _Fernando Laudares Camargos_ and _Jobin Augustine_ [PostgreSQL HA with Patroni: Your Turn to Test Failure Scenarios](https://www.percona.com/blog/postgresql-ha-with-patroni-your-turn-to-test-failure-scenarios/)

If you [installed the software from tarballs](../tarball.md), check how you [enable etcd](../enable-extensions.md#etcd).

The `etcd` cluster is first started in one node and then the subsequent nodes are added to the first node using the `add `command. 

!!! note

    Users with deeper understanding of how etcd works can configure and start all etcd nodes at a time and bootstrap the cluster using one of the following methods:

    * Static in the case when the IP addresses of the cluster nodes are known
    * Discovery  service - for cases when the IP addresses of the cluster are not known ahead of time.

    See the [How to configure etcd nodes simultaneously](../how-to.md#how-to-configure-etcd-nodes-simultaneously) section for details.

### Configure `node1`

1. Create the configuration file. You can edit the sample configuration file `/etc/etcd/etcd.conf.yaml` or create your own one. Replace the node name and IP address with the actual name and IP address of your node.

    ```yaml title="/etc/etcd/etcd.conf.yaml"
    name: 'node1'
    initial-cluster-token: PostgreSQL_HA_Cluster_1
    initial-cluster-state: new
    initial-cluster: node1=http://10.104.0.1:2380
    data-dir: /var/lib/etcd
    initial-advertise-peer-urls: http://10.104.0.1:2380 
    listen-peer-urls: http://10.104.0.1:2380
    advertise-client-urls: http://10.104.0.1:2379
    listen-client-urls: http://10.104.0.1:2379
    ```

2. Start the `etcd` service to apply the changes on `node1`.

    ```{.bash data-prompt="$"}
    $ sudo systemctl enable --now etcd
    $ sudo systemctl start etcd
    $ sudo systemctl status etcd
    ```

3. Check the etcd cluster members on `node1`:

    ```{.bash data-prompt="$"}
    $ sudo etcdctl member list --write-out=table --endpoints=http://10.104.0.1:2379
    ```
    
    Sample output:

    ```{.text .no-copy}
    +------------------+---------+-------+----------------------------+----------------------------+------------+
    |        ID        | STATUS  | NAME  |         PEER ADDRS         |        CLIENT ADDRS        | IS LEARNER |
    +------------------+---------+-------+----------------------------+----------------------------+------------+
    | 9d2e318af9306c67 | started | node1 | http://10.104.0.1:2380     | http://10.104.0.1:2379     |      false |
    +------------------+---------+-------+----------------------------+----------------------------+------------+
    ```

4. Add the `node2` to the cluster. Run the following command on `node1`:

    ```{.bash data-prompt="$"}
    $ sudo etcdctl member add node2 --peer-ulrs=http://10.104.0.2:2380
    ```

    ??? example "Sample output"
    
        ```{.text .no-copy}
        Added member named node2 with ID 10042578c504d052 to cluster

        etcd_NAME="node2"
        etcd_INITIAL_CLUSTER="node2=http://10.104.0.2:2380,node1=http://10.104.0.1:2380"
        etcd_INITIAL_CLUSTER_STATE="existing"
        ```

### Configure `node2`

1. Create the configuration file. You can edit the sample configuration file `/etc/etcd/etcd.conf.yaml` or create your own one. Replace the node names and IP addresses with the actual names and IP addresses of your nodes.

    ```yaml title="/etc/etcd/etcd.conf.yaml"
    name: 'node2'
    initial-cluster-token: PostgreSQL_HA_Cluster_1
    initial-cluster-state: existing
    initial-cluster: node1=http://10.104.0.1:2380,node2=http://10.104.0.2:2380
    data-dir: /var/lib/etcd
    initial-advertise-peer-urls: http://10.104.0.2:2380 
    listen-peer-urls: http://10.104.0.2:2380
    advertise-client-urls: http://10.104.0.2:2379
    listen-client-urls: http://10.104.0.2:2379
    ```

3. Start the `etcd` service to apply the changes on `node2`:

    ```{.bash data-prompt="$"}
    $ sudo systemctl enable --now etcd
    $ sudo systemctl start etcd
    $ sudo systemctl status etcd
    ```

### Configure `node3`

1. Add `node3` to the cluster. **Run the following command on `node1`**

    ```{.bash data-prompt="$"}
    $ sudo etcdctl member add node3 http://10.104.0.3:2380
    ```

2. On `node3`, create the configuration file. You can edit the sample configuration file `/etc/etcd/etcd.conf.yaml` or create your own one. Replace the node names and IP addresses with the actual names and IP addresses of your nodes.

    ```yaml title="/etc/etcd/etcd.conf.yaml"
    name: 'node3'
    initial-cluster-token: PostgreSQL_HA_Cluster_1
    initial-cluster-state: existing
    initial-cluster: node1=http://10.104.0.1:2380,node2=http://10.104.0.2:2380,node3=http://10.104.0.3:2380
    data-dir: /var/lib/etcd
    initial-advertise-peer-urls: http://10.104.0.3:2380 
    listen-peer-urls: http://10.104.0.3:2380
    advertise-client-urls: http://10.104.0.3:2379
    listen-client-urls: http://10.104.0.3:2379
    ```

3. Start the `etcd` service to apply the changes.

    ```{.bash data-prompt="$"}
    $ sudo systemctl enable --now etcd
    $ sudo systemctl start etcd
    $ sudo systemctl status etcd
    ```

4. Check the etcd cluster members. 

    ```{.bash data-prompt="$"}
    $ sudo etcdctl member list
    ```

    ??? example "Sample output"

        ```
        2d346bd3ae7f07c4: name=node2 peerURLs=http://10.104.0.2:2380 clientURLs=http://10.104.0.2:2379     isLeader=false
        8bacb519ebdee8db: name=node3 peerURLs=http://10.104.0.3:2380 clientURLs=http://10.104.0.3:2379     isLeader=false
        c5f52ea2ade25e1b: name=node1 peerURLs=http://10.104.0.1:2380 clientURLs=http://10.104.0.1:2379     isLeader=true
        ``` 
