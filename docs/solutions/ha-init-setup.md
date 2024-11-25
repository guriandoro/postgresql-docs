# Initial setup for high availability

This guide provides instructions on how to set up a highly available PostgreSQL cluster with Patroni. This guide relies on the provided [architecture](ha-architecture.md) for high-availability.

## Preconditions

1. This is an example deployment where etcd runs on the same host machines as the Patroni and PostgreSQL and there is a single dedicated HAProxy host. Alternatively etcd can run on different set of nodes. 

    If etcd is deployed on the same host machine as Patroni and PostgreSQL, separate disk system for etcd and PostgreSQL is recommended due to performance reasons.

2. For this setup, we will use the nodes that have the following IP addresses:


    | Node name     | Public IP address | Internal IP address
    |---------------|-------------------|--------------------
    | node1         | 157.230.42.174    | 10.104.0.7
    | node2         | 68.183.177.183    | 10.104.0.2
    | node3         | 165.22.62.167     | 10.104.0.8
    | HAProxy-demo  | 134.209.111.138   | 10.104.0.6


!!! note

    We recommend not to expose the hosts/nodes where Patroni / etcd / PostgreSQL are running to public networks due to security risks.  Use Firewalls, Virtual networks, subnets or the like to protect the database hosts from any kind of attack. 

## Initial setup 

It’s not necessary to have name resolution, but it makes the whole setup more readable and less error prone. Here, instead of configuring a DNS, we use a local name resolution by updating the file `/etc/hosts`. By resolving their hostnames to their IP addresses, we make the nodes aware of each other’s names and allow their seamless communication.

1. Set the hostname for nodes. Run the following command on each node. Change the node name to `node1`, `node2` and `node3` respectively:

    ```{.bash data-prompt="$"}
    $ sudo hostnamectl set-hostname node1
    ```

2. Modify the `/etc/hosts` file of each PostgreSQL node to include the hostnames and IP addresses of the remaining nodes. Add the following at the end of the `/etc/hosts` file on all nodes:

    === "node1"    

        ```text hl_lines="3 4"
        # Cluster IP and names 
        10.104.0.1 node1 
        10.104.0.2 node2 
        10.104.0.3 node3
        ```    

    === "node2"    

        ```text hl_lines="2 4"
        # Cluster IP and names 
        10.104.0.1 node1 
        10.104.0.2 node2 
        10.104.0.3 node3
        ```    

    === "node3"    

        ```text hl_lines="2 3"
        # Cluster IP and names 
        10.104.0.1 node1 
        10.104.0.2 node2 
        10.104.0.3 node3
        ```    

    === "HAproxy-demo"    

        The HAProxy instance should have the name resolution for all the three nodes in its `/etc/hosts` file. Add the following lines at the end of the file:    

        ```text hl_lines="4 5 6"
        # Cluster IP and names
        10.104.0.6 HAProxy-demo
        10.104.0.1 node1
        10.104.0.2 node2
        10.104.0.3 node3
        ```

