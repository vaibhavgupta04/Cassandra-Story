# Read Path in Cassandra

The read path in Cassandra involves several steps, and understanding it is crucial for optimizing performance in read-heavy workloads. Below is a comprehensive description of the read path in Cassandra.
In our database let’s say we want to fetch student information of a particular student.

```jsx
SELECT * FROM student WHERE institute_id = 'IIIT_G' AND year = 2023 AND student_id = '12345';
```

<aside>
💡 The read path at the initial steps are same as the write path i.e.
Client request → coordinator node → Partitioner  → relevant nodes 
so I’m not going on details about these terms in this page, if you have any confusion you can refer back to the “Write Path in Cassandra” page.

</aside>

### Client Request

A client sends a read request to the Cassandra cluster.

In our case let’s a SELECT query has been fired.

### Coordinator Node

The node receiving the request becomes the coordinator node and will act as a mediator between the client and the nodes that contain the required data.

Let’s assume the node we are requesting to happens to be the IIITD node. So the IIITD node becomes the coordinator.

![coordinator.drawio.png](Write%20Path%20in%20Cassandra/coordinator.drawio.png)

### Partitioner

with the help of the partitioner and snitches coordinator, the node determines the required nodes to be queried. In our case, the partitioner uses our partition key ("institute_id") and generates a token. The coordinator, using this token, determines the node responsible for handling this read request. The coordinator will determine the whereabouts of the IIITG node through [snitch](https://docs.datastax.com/en/cassandra-oss/2.1/cassandra/architecture/architectureSnitchesAbout_c.html?hl=snitches) and establish communication with a proprietary binary protocol specified here:  [cassandra/doc/native_protocol_v3.spec at trunk · apache/cassandra · GitHub](https://github.com/apache/cassandra/blob/trunk/doc/native_protocol_v3.spec) 

![iiit-replica.drawio (1).png](Write%20Path%20in%20Cassandra/iiit-replica.drawio_(1).png)

### Read Operation

There are three types of read requests that a coordinator can send to a replica:

- A direct read request
- A digest request
- A background read repair request

In a direct read request, the coordinator node contacts one replica node. Then the coordinator sends a digest request to a number of replicas determined by the consistency level specified by the client. The coordinator sends these requests to the replicas that currently respond the fastest (through [dynamic snitching](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/architecture/archSnitchDynamic.html)). The contacted nodes respond with a digest of the requested data; if multiple nodes are contacted, the rows from each replica are compared in memory for consistency. If they are not consistent, the replica having the most recent data (based on the timestamp) is used by the coordinator to forward the result back to the client. To ensure that all replicas have the most recent version of the data, a blocking read repair is carried out to update out-of-date replicas before sending the data back to the client. After responding to the client, the coordinator sends a digest request to all remaining replicas and triggers read repair (if required) to ensure that the requested row is made consistent on all replicas involved in a read query.

![read-cluster.drawio.png](Read%20Path%20in%20Cassandra/read-cluster.drawio.png)

Let’s consider in our case the read operation has been made with READ_CONSISTENCY_LEVEL of QUORAM. If you remember from the “Write Path” page our quorum value is 2. That would mean we need to check data consistency in two nodes before sending out the data back to the client. Let’s say IIITD gets the request but the data is present in replicas of IIITG, IIITH, and IIITB. Through dynamic snitching, we found out IIITG is the node with the lowest read latency so we made a data request from IIITG. Then we send a message digest request to one other node (matching our quorum value of 2) based on the read latency (let’s say the node turned out to be IIITH node). A message digest is simply a hash value of the data we are querying for. Checking the message digest instead of the entire data saves us some time. Now at the coordinator’s end, we compute the message digest of the data we got from IIITG and check if it’s the same as that of IIITH. In case of inconsistency data with the latest timestamp i.e. the latest data will be sent to the client. But before that, a blocking read repair is conducted between these two nodes so that now both nodes (all involved nodes in case of more than 2 nodes) have consistent data. After the data is sent to the client a second round of message digest requests is sent to all the remaining replicas (in this case IIITB) to check consistency at all nodes. The coordinator upon receiving the message digest compares whichever response has the latest data using timestamp and triggers a read repair on all those nodes with stale data. This will mark the end of the read path in Cassandra.

Side note: A question might arise, while we were sending out data to the client we only checked for data in 2 nodes, in our case IIITH and IIITG, but what if the IIITB node had the latest data? Does that mean we are sending out-of-date data to the client? 
The answer is yes, here’s when the “eventual consistency” property of the Cassandra database kicks in. However, we can avoid this scenario as Cassandra also provides a “tunable consistency” feature to our database. If we had chosen READ_CONSISTENCY_LEVEL of ALL then all the nodes would have been consulted during the read process before sending out the data to the client. This however would increase the read time as more nodes would be involved in the process we just discussed. Hence the “tunable consistency” is fine-tuning between consistency and performance (speed). 

**Rapid read protection using speculative_retry**

Rapid read protection allows Cassandra to still deliver read requests when the originally selected replica nodes are either down or taking too long to respond. If the table has been configured with the speculative_retry property, the coordinator node for the read request will retry the request with another replica node if the original replica node exceeds a configurable timeout value to complete the read request.

*Recovering from replica node failure with rapid read protection*

![Untitled](Read%20Path%20in%20Cassandra/Untitled.png)

For more details on read consistency level follow this:

[DataStax Apache Cassandra - How is the consistency level configured?](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/dml/dmlConfigConsistency.html)

> Reference
> 

 [Apache Cassandra Documentation: How are read requests accomplished?](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/dml/dmlClientRequestsRead.html)