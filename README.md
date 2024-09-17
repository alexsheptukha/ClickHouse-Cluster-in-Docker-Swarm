# ClickHouse cluster with Zookeeper and Docker Swarm

## Prerequisites

To create this cluster you need to have Docker Swarm with at least 2 nodes to have true replication. Before deploying you need to create docker configs to spread them across cluster

```
docker config create zookeeper.xml ./zookeeper.xml

docker config create cluster.xml ./cluster.xml

docker config create macros_replica1.xml ./macros_replica1.xml

docker config create macros_replica2.xml ./macros_replica2.xml

```

Replace hostname values in deployment configuration in docker-compose.yml with your desired server values.

Refer to the [official guide](https://clickhouse.com/docs/en/architecture/replication) to learn about shards and replicas, configure your .xml configs for a desired number of shards and replicas.


## Deployment


To start the cluster run this command: 

```
docker stack deploy -c docker-compose.yml clickhouse_stack
```

To stop the cluster:

```
docker stack rm clickhouse_stack
```

Check [this guide](https://github.com/alex-kabin/clickhouse-cluster-docker/tree/master) to update your configuration and to add Clickhouse Proxy and Tabix  
  
## Usage

Check if everything is set up correctly using these queries:

```
SELECT * FROM system.clusters;

SELECT * FROM system.zookeeper where path = '/';
```

They should return your configured nodes and Zookeeper.

To create replicated table run a command like this:

```
CREATE TABLE my_replicated_table ON CLUSTER mycluster
(
    id UInt64,
    name String,
    created_at DateTime
)
ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/my_replicated_table', '{replica}')
ORDER BY id;
```

After that run this query to see your table:

```
SELECT * FROM system.replicas;
```
