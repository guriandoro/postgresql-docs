# High Availability in PostgreSQL with Patroni

Regardless whether you are a small startup or a big enterprise, a downtime of your services may cause severe consequences like loss of customers, impact on your reputation, and penalties for not meeting the  Service Level Agreements (SLAs). That’s why ensuring that your deployment operates without disruption is crucial.

In this solution document you will find the following:

* the [technical overview](#technical-overview) of high availability; 
* the [reference architecture](ha-architecture.md) that we recommend to achieve high availability
* the [step-by-step deployment guide](ha-setup-apt.md). The guide focuses on the minimalist approach to HA. It also gives instructions how to deploy additional components that you can add when your infrastructure grows.
* The [testing guidelines](ha-test.md) on how to verify that your high availability works as expected, providing replication and failover.

## Technical overview

High availability is the ability of the system to operate continuously without the interruption of services. During the outage, the system must be able to fail over the services from a primary database node that is down to one of the standby nodes within a cluster. For the standby nodes to always be up to date with the primary, there must be a replication mechanism between them.

To break it down, achieving high availability means to put these principles in practice:

* **Single point of failure (SPOF)** – Eliminate any single point of failure in the database environment, including the physical or virtual hardware the database system relies on and which would cause it to fail.
* **Redundancy** – Ensure sufficient redundancy of all components within the database environment and reliable crossover to these components in the event of failure.
* **Failure detection** – Monitor the entire database environment for failures.

??? information "Measuring high availability"

    The need for high availability is determined by the business requirements, potential risks, and operational limitations. The level of high availability depends on how much downtime you can bear without negatively impacting your users and how much data loss you can tolerate during the system outage.

    The measurement of availability is done by establishing a measurement time frame and dividing it by the time that it was available. This ratio will rarely be one, which is equal to 100% availability. At Percona, we don’t consider a solution to be highly available if it is not at least 99% or two nines available.

    The following table shows the amount of downtime for each level of availability from two to five nines.

    | Availability %       | Downtime per year | Downtime per month | Downtime per week | Downtime per day  |
    |----------------------|-------------------|--------------------|-------------------|-------------------|
    | 99% (“two nines”)    | 3.65 days         | 7.31 hours         | 1.68 hours        | 14.40 minutes     |
    | 99.5% (“two nines five”) | 1.83 days         | 3.65 hours         | 50.40 minutes     | 7.20 minutes      |
    | 99.9% (“three nines”) | 8.77 hours        | 43.83 minutes      | 10.08 minutes     | 1.44 minutes      |
    | 99.95% (“three nines five”) | 4.38 hours        | 21.92 minutes      | 5.04 minutes      | 43.20 seconds     |
    | 99.99% (“four nines”) | 52.60 minutes     | 4.38 minutes       | 1.01 minutes      | 8.64 seconds      |
    | 99.995% (“four nines five”) | 26.30 minutes     | 2.19 minutes       | 30.24 seconds     | 4.32 seconds      |
    | 99.999% (“five nines”) | 5.26 minutes      | 26.30 seconds      | 6.05 seconds      | 864.00 milliseconds |
    

## Ways to achieve high availability in PostgreSQL

To achieve high availability you should have the following:

* **Replication** to ensure the standby nodes continuously receive updates from the primary and are in sync with it. Consider using **streaming replication** where a standby node connects to its primary and continuously receives a stream of WAL records as they’re generated. The configuration of both primary and standbys must be the same. 
* **Failover** to eliminate the single point of failure by promoting one of the standbys to become primary when the initial primary node is down. To minimize the time it takes to promote a new primary, the failover must be automatic. 

    The tool of choice for failover is **Patroni** as it monitors the state and health of the cluster and takes care of the failover procedure if there’s an outage. Patroni relies on the distributed configuration store (DCS) that stores the cluster configuration, health and status. We recommend and use etcd as the DCS for Patroni due to its simplicity, consistency and reliability. Etcd not only stores the cluster data, it also handles the election of a new primary node (a leader in ETCD terminology).

* **Backup and recovery solution** to protect your environment against data loss and ensure quick service restoration. **pgBackRest** is a robust, reliable solution for Backup and Continuous WAL Archiving. 
* **Monitoring** to ensure that each node and the whole PostgreSQL cluster perform effectively. We suggest
using Percona Monitoring and Management (PMM), a fast, customizable, and reliable monitoring tool.

In the [reference architecture](ha-architecture.md) section we give the recommended combination of open-source tools to achieve high availability in PostgreSQL. We focus on the minimalist deployment with three-node PostgreSQL cluster.



## Next steps

[Architecture](ha-architecture.md){.md-button}


