# HerdDB

HerdDB is a SQL distributed database implemented in Java. It has been designed to be embeddable in any Java Virtual Machine. It is optimized for fast "writes" and primary key read/update access patterns.

## Architecture

HerdDB leverages Apache Zookeeper and Apache Bookkeeper to build a fully replicated, shared-nothing architecture without any single point of failure.

The main unit of work for an HerdDB cluster is the tablespace, a tablespace is a group of tables. In the context of a tablespace you can run transactions, joins and subqueries which span multiple tables.

In a cluster for each tablespace a leader node is designated by the administrator (with some kind of auto-healing and auto leader reassignment in case of failure) and all the transactions on its tables are run on that node. This system scales well by having many tablespaces and so the load can be spread among all the machines in the cluster.

Indexes are supported by using an implementation of the Block Range Index pattern (BRIN indexes), adapted to the way the HerdDB uses to store data.

![Architecture about HerdDB](../images/nosql/01_herddb_architecture.md)

### Write Operation

The leader of the tablespace keeps in memory a data structure which holds all the PKs for a table in an hash table, and an in-memory buffer which contains all the dirty and/or recently accessed records.

When an INSERT reachs the the server the write is first logged to the log, then the map of valid PKs gets updated and the new record is stored in the memory buffer. If an UPDATE is issued on the same PK (and this is our primary use case) the update is directly performed in memory, without hitting “data pages” disks, we only write to the log in order to achieve durability. If a GET comes for the same PK we can read directly the “dirty” record from the buffer. After a configurable timeout or when the system is running out of memory a checkpoint is performed and buffers are flushed to disk, creating immutable data pages, so usually all the work is in memory, writes are performed serially on the transaction log and when flushing to disk complete data pages are written, without ever modifiing existing files. This kind of write pattern is very suitable of our use case: data files are always written or read entirely, leveraging the most of OS caches.

### Replicator

HerdDB leverages Apache BookKeeper ability to provide a distributed write ahead log, when a node is running as leader it writes each state change to BookKeeper, working as a replicated state machine. Some features:
- each write is guaranteed to be “durable” after the ack from BookKeeeper

- each replica is guaranteed to read only entries for which the ack has been received from the writer (the Last-Add-Confirmed protocol)

- each ledger (the basic storage unit of BookKeeper) can be written only once

For each tablespace you can add a virtually unlimited number of replicas, each ‘replica’ node will ‘tail’ the transaction log and replay each data manipulation activity to its local copy of the data. If a “new leader” comes in, BookKeeper will fence out the “old leader”, preventing any further write to the ledger, this way the old leader will not be able to carry on its activity and change its local state: this will guarantee that every node will converge to the same consistent view of the system.

Apache BookKeeper servers, called Bookies, can be run standalone but the preferred way is to run them inside the same JVM of the database, leveraging the ability to talk to the Bookie without passing from the network stack.

## Deployment

```shell
(foreground mode)
docker run --rm -it  -e server.port=7000 -p 7000:7000 -e server.advertised.host=$(hostname)  --name herddb herd:latest

(daemon mode)
docker run --restart=always -d -e server.port=7000 -p 7000:7000 -e server.advertised.host=$(hostname)  --name herddb herd:latest

# in order to access local cli (inside the container)
docker exec -it herddb /bin/bash bin/herddb-cli.sh -x jdbc:herddb:server:0.0.0.0:7000 -q "select * from sysnodes"

# in order to access the cli from outside the container using herddb-cli.sh from a HerdDB downloaded package
bin/herddb-cli.sh -x jdbc:herddb:server:$(hostname):7000 -q "select * from sysnodes"

# use the cli with a temporary container
docker run --network host -it --entrypoint /bin/bash herd /herddb/bin/herddb-cli.sh -x jdbc:herddb:server:$(hostname):7000 -q "select * from sysnodes"


Running multiple HerdDB using docker

# start ZooKeeper
docker run -p 2181:2181 --name zookeeper --restart always -d zookeeper

# start as many 
# HerdDB inside the container will by default bind to port 7000, you have to pass server.port=0 in order to let the server choose a random port
docker run -d  --restart always --network host -e server.mode=cluster -e server.bookkeeper.start=true -e server.port=0 -e server.zookeeper.address=$(hostname):2181 herd:latest
docker run -d  --restart always --network host -e server.mode=cluster -e server.bookkeeper.start=true -e server.port=0 -e server.zookeeper.address=$(hostname):2181 herd:latest
docker run -d  --restart always --network host -e server.mode=cluster -e server.bookkeeper.start=true -e server.port=0 -e server.zookeeper.address=$(hostname):2181 herd:latest
docker run -d  --restart always --network host -e server.mode=cluster -e server.bookkeeper.start=true -e server.port=0 -e server.zookeeper.address=$(hostname):2181 herd:latest
docker run -d  --restart always --network host -e server.mode=cluster -e server.bookkeeper.start=true -e server.port=0 -e server.zookeeper.address=$(hostname):2181 herd:latest
docker run -d  --restart always --network host -e server.mode=cluster -e server.bookkeeper.start=true -e server.port=0 -e server.zookeeper.address=$(hostname):2181 herd:latest

Remember that the 'default' tablespace will be served by the first instance, so you should keep it running. If the leader of 'default' tablespace is lost you can recover by issuing ALTER TABLESPACE commands directly to other nodes and change the leader

# in order to see your hosts
docker run --network host -it --entrypoint /bin/bash herd /herddb/bin/herddb-cli.sh -x jdbc:herddb:zookeeper:$(hostname):2181 -q "select * from sysnodes"

# scale up the 'default' tablespace
docker run --network host -it --entrypoint /bin/bash herd /herddb/bin/herddb-cli.sh -x jdbc:herddb:zookeeper:$(hostname):2181 -q "ALTER TABLESPACE 'default','expectedreplicacount:2','maxleaderinactivitytime:10000"
docker run --network host -it --entrypoint /bin/bash herd /herddb/bin/herddb-cli.sh -x jdbc:herddb:zookeeper:$(hostname):2181 -q "select tablespace_name, leader, replica from systablespaces" 
docker run --network host -it --entrypoint /bin/bash herd /herddb/bin/herddb-cli.sh -x jdbc:herddb:zookeeper:$(hostname):2181 -q "select * from systablespacereplicastate"

```

## Reference

- https://herddb.org/
- https://github.com/diennea/herddb/tree/master/herddb-bench