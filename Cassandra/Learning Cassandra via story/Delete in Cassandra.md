# Delete in Cassandra

Cassandra treats a delete as an insert or [upsert](https://docs.datastax.com/en/glossary/doc/glossary/gloss_upsert.html). The data being added to the partition in the DELETE command is a deletion marker called a tombstone. The tombstones go through Cassandra's write path and are written to SSTables on one or more nodes. The key difference feature of a tombstone: it has a built-in expiration date/time. At the end of its expiration period (for details see below) the tombstone is deleted as part of Cassandra's normal compaction process. You can also mark a Cassandra record (row or column) with a time-to-live value. After this amount of time has ended, Cassandra marks the record with a tombstone and handles it like other tombstoned records.

### Why Tombstones?

Deletion in Cassandra differs from traditional relational databases primarily due to the distributed and decentralized nature of Cassandra's architecture. In relational databases, the delete operation is typically straightforward, involving the immediate removal of the specified data. However, Cassandra by design allows any node in the cluster to process a write operation, and every write is sent to all replica nodes for that data. Now consider a situation when one of the replica nodes was down when a delete operation was performed; the node would simply miss the delete request. Once this downed node comes back online, it would mistakenly think that all the other nodes where the delete was applied had missed a write and would start repairing all the other nodes by sending the deleted data. This would lead to data resurrection issues (zombie records) in the cluster. Therefore in-place deletes will not work with a distributed system like Cassandra and would need a more sophisticated mechanism to handle deletes, hence, Tombstones.

---

To prevent the resurgence of deleted data, Cassandra incorporates a grace period for each tombstone. This period allows unresponsive nodes to recover and process tombstones normally. If a client initiates a new update to a tombstone’ed record during this grace period, Cassandra replaces the tombstone with the updated information. Similarly, if a client requests a read for the record within the grace period, Cassandra disregards the tombstone, retrieving the information from alternative replicas if available.

Upon recovery of an unresponsive node, Cassandra leverages hinted handoff to replay missed database mutations during the downtime. Notably, Cassandra refrains from replaying mutations for tombstoned records still within their grace period. However, if the node remains unresponsive beyond the grace period, there is a risk of missing the deletion.

Once the grace period concludes, Cassandra proceeds to delete the tombstone during the compaction process. This strategy ensures a systematic approach to tombstone management, balancing data consistency and recovery in distributed environments.

To learn more about tombstones and deletion in Cassandra follow these links  

[Tombstones in Apache Cassandra](https://medium.com/walmartglobaltech/tombstones-in-apache-cassandra-d0a068a72dcc)

[DataStax Apache Cassandra - how data is deleted?](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/dml/dmlAboutDeletes.html?hl=how%2Cdata%2Cdeleted%3F)