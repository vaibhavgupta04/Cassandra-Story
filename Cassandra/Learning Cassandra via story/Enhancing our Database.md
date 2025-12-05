# Enhancing our Database

With our "student" column family and a distributed cluster of five nodes, the next critical step in optimizing our Cassandra database is replication. Replication involves copying and storing data across multiple nodes, providing fault tolerance and high availability. This strategic approach mitigates the risk of data loss and ensures that the system remains robust even in the face of node failures.

To strike a balance between fault tolerance, availability, and efficient use of resources, we have chosen a replication factor of 3 for our Cassandra nodes. This means that each piece of data in our "student" column family will be replicated and stored on three different nodes within the cluster.

## Replication Strategy

In Cassandra, the replication strategy is a crucial aspect of the database configuration that determines how data is replicated across nodes in a cluster. The replication strategy defines the number of replicas (copies) of each piece of data and dictates their distribution throughout the cluster. Cassandra provides two replication strategies ("Simple Strategy" and "Network Topology Strategy.”), and the choice depends on the specific requirements and goals of the system.

Read more about these replication strategies from here: 
[DataStax Apache Cassandra - data replication](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/architecture/archDataDistributeReplication.html)

Having considered the requirements of our IIIT student database and the simplicity of our network setup, we have opted for the "SimpleStrategy" as our replication strategy in Cassandra.