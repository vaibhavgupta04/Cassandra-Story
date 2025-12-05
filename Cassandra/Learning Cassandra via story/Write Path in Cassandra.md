# Write Path in Cassandra

Inserting data into a Cassandra column family involves a well-defined process. Let’s say we want to insert a student's information into the “student” column family.

| institute_id | student_id | name | roll | year |
| --- | --- | --- | --- | --- |
| IIIT_G | uuid() | Ravi Teja | iit2023001 | 2023 |

Here's a comprehensive description of the write path in Cassandra.

### Client Request

The write process begins when a client sends a write request to the Cassandra cluster. Clients can be any application or service that interacts with Cassandra, using one of the available drivers or APIs. 

In our case let’s assume we have executed INSERT command through our application.

### Coordinator Node

When a request is sent to any Cassandra node it acts as a mediator between the client and the nodes that store the data and it’s called the coordinator. The coordinator node is responsible for coordinating the entire write operation and responding back to the client.

At times, when the coordinator forwards a write request to the replica nodes, they might be unavailable. In such situations, the coordinator plays a crucial role in implementing a mechanism known as Hinted Handoff. We’ll dive into the details of this mechanism later.

Let’s assume the node we are requesting to happens to be the IIITD node. So the IIITD node becomes the coordinator.

![coordinator.drawio.png](Write%20Path%20in%20Cassandra/coordinator.drawio.png)

### Partitioner

The **partitioner** is responsible for deciding how to distribute data across the nodes in a Cassandra cluster based on the partition key of a row. Essentially, it acts as a hash function, computing a token(resultant hash value) from the partition key. Once the partitioner obtains the token, it precisely determines which node will handle the request. Every node in Cassandra has precise knowledge of which nodes are responsible for a certain range of tokens as nodes are in constant periodic communication about their state through [gossip protocol](https://docs.datastax.com/en/cassandra-oss/2.1/cassandra/architecture/architectureGossipAbout_c.html). According to the replication factor and replication strategy, the coordinator also determines the replica nodes responsible for the specified data.

Cassandra provides three types of partitioners: **Murmur3Partitioner**, **RandomPartitioner**, and **ByteOrderedPartitioner**.

> [Read more about these partitioner from the offical documentation page here.](https://docs.datastax.com/en/cassandra-oss/2.1/cassandra/architecture/architecturePartitionerAbout_c.html)
> 

In our case, the partitioner uses our partition key ("institute_id") and generate a token. The coordinator, using this token, determines the node responsible for handling this write request should be the “IIITG” node. The coordinator will determine the whereabouts of the IIITG node through [snitch](https://docs.datastax.com/en/cassandra-oss/2.1/cassandra/architecture/architectureSnitchesAbout_c.html?hl=snitches) and establish a TCP connection for data transfer and communicate with a proprietary binary protocol specified here:  [cassandra/doc/native_protocol_v3.spec at trunk · apache/cassandra · GitHub](https://github.com/apache/cassandra/blob/trunk/doc/native_protocol_v3.spec) 

![requriednode.drawio.png](Write%20Path%20in%20Cassandra/requriednode.drawio.png)

### **Write Consistency Level**

When a client interacts with a Cassandra node, known as the coordinator, for write operations, such as inserting data, it can specify a write consistency level. This level determines the number of replica nodes that must confirm the success of the local insert to the coordinator before the coordinator signals success back to the client. The success confirmation involves the data being added to the commit log and written to the memtable.

For instance, if an application issues an insert request with a WRITE_CONSISTENCY_LEVEL of TWO on a table configured with a REPLICATION_FACTOR of 3, the coordinator will respond to the application's success only when two out of the three replicas acknowledge the successful write. This doesn't mean the third replica won't eventually store the data; it will. However, at the point of coordinator acknowledgment, the client already receives a success response.

Cassandra offers various write consistency levels, ranging from less strict to full consistency. These levels include ANY, ONE, TWO, THREE, QUORUM, LOCAL_QUORUM, EACH_QUORUM, and ALL, each providing different trade-offs between consistency and performance.

For more details on write consistency level follow this:

[DataStax Apache Cassandra - How is the consistency level configured?](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/dml/dmlConfigConsistency.html)

For simplicity, suppose the write request we issued had WRITE_CONSISTENCY_LEVEL set to QUORAM. 
quorum is calculated as: quorum = (sum_of_replication_factors / 2) + 1

since we had configured our replication factor as 3 and had used Simple_Strategy as the replication strategy

$$

quorum = (3/2)+1 = 2
$$

![iiit-replica.drawio (1).png](Write%20Path%20in%20Cassandra/iiit-replica.drawio_(1).png)

Now, before the coordinator forwards the write request to all the 3 replica nodes, it will ask to the [Failure Detector](https://docs.datastax.com/en/cassandra-oss/2.2/cassandra/architecture/archDataDistributeFailDetect.html#:~:text=Failure%20detection%20is%20a%20method,to%20unreachable%20nodes%20whenever%20possible.) component how many of these replica nodes are available and compare it to the WRITE_CONSISTENCY_LEVEL provided in the request. If the number of replica nodes available is less than the WRITE_CONSISTENCY_LEVEL provided, the Failure Detector will immediately throw an Exception.

The coordinator sends a write request to *all* replicas that own the row being written. As long as all replica nodes are up and available, they will get the write regardless of the consistency level specified by the client. 

The coordinator node responds to the client once it receives write acknowledgments from the number of nodes specified by the consistency level. Exceptions:

- If the coordinator cannot write to enough replicas to meet the requested consistency level, it throws an `Unavailable` Exception and does not perform any writes.
- If there are enough replicas available but the required writes don't finish within the timeout window, the coordinator throws a `Timeout` Exception.

![sucess.drawio.png](Write%20Path%20in%20Cassandra/sucess.drawio.png)

If the WRITE_CONSISTENCY_LEVEL for this request was THREE (or ALL), the coordinator would have to wait until node3 acknowledges success too, and of course this write request would be slower.

### **Hinted Handoff**

Suppose in the last example that only 2 of 3 replica nodes were available. In this case, the Failure Detector would still allow the request to continue as the number of available replica nodes is not less than the WRITE_CONSISTENCY_LEVEL provided. In this case, the coordinator would behave exactly as described before but there would be one additional step. The coordinator would write locally the hint (the write request blob along with some metadata) in the disk (hints directory) and would keep the hint there for 3 hours (by default) waiting for the replica node to become available again. If the replica node recovers within this period, the coordinator will send the hint to the replica node so that it can update itself and become consistent with the other replicas. If the replica node is offline for more than 3 hours, then a read repair is needed. This process is referred as Hinted Handoff.

The coordinator thus returns the write success message to the client.

---

This is how a write request is executed in a multi-node database in Cassandra. 
To understand how a write request is handled in a multi-datacentre environment you can follow this link which contains a detailed explanation of the subject matter.

[DataStax Apache Cassandra - **Multiple datacenter write requests**](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/dml/dmlClientRequestsMultiDCWrites.html)

Now that we have a bird’s eye view of the entire write path in Cassandra, let’s zoom in and see how data is written inside a node on the next page.