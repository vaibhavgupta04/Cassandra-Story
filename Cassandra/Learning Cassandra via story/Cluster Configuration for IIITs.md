# Cluster Configuration for IIITs

Let's consider how Cassandra's decentralized architecture caters to the distributed nature of our IIIT network, ensuring data availability and fault tolerance.

Storing all the data in a single node, especially in a scenario involving multiple IIIT colleges and a substantial amount of student information, is not a recommended approach. The centralization of data on a single node can lead to several challenges and limitations that hinder the efficiency and scalability of the database system. 

Cassandra's distributed architecture offers compelling solutions to address these concerns. Distributing data across multiple nodes in Cassandra offers key advantages, including enhanced scalability through horizontal growth, improved fault tolerance with data replication, high availability by eliminating single points of failure, efficient load distribution for optimal performance, and the ability to geographically distribute data for faster access.

In our scenario with five IIIT colleges, we optimize Cassandra's geographic distribution capabilities by dedicating nodes to each college. This strategic setup enhances load balancing, minimizes latency, and ensures faster data access for users affiliated with each institution. By leveraging this decentralized approach, we not only improve overall system responsiveness but also bolster fault tolerance and high availability. This tailored solution effectively manages diverse student information across the IIIT network. 

![IIIT-cluster.drawio.png](Cluster%20Configuration%20for%20IIITs/IIIT-cluster.drawio.png)

Hence we create a cluster of 5 nodes dedicated to each college.

## Data Distribution

Cassandra employs a partitioning mechanism to distribute data across nodes in a cluster. Each row in a table is uniquely identified by a primary key, and the partition key determines the distribution of data among nodes (more on this later). The objective is to evenly distribute data to prevent hotspots and ensure efficient utilization of resources.

- **Partition Key:** The choice of a partition key is paramount. It is the attribute used to determine the node responsible for storing and managing a specific piece of data. A well-selected partition key promotes even data distribution, avoiding scenarios where certain nodes become overloaded with data.
- **Token Ring:** Cassandra utilizes a token ring to map partition keys to specific nodes. The ring structure allows for a consistent and predictable method of distributing data, making it scalable as nodes are added or removed from the cluster.

The mechanism behind this distribution is called “consistent hashing”. We highly recommend checking this article on consistent hashing to have a sound knowledge of working on consistent hashing.

 [DataStax Apache Cassandra - consistent hashing](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/architecture/archDataDistributeHashing.html)

### **Virtual nodes**

The concept of virtual nodes was introduced with Cassandra 1.2. virtual nodes (vnodes) are a feature designed to enhance the distribution and manageability of data across a cluster of nodes. Instead of assigning a single token range to each node, virtual nodes allow each physical node to be responsible for multiple token ranges. This approach brings several advantages to the system:

- Even Data Distribution
- Efficient Repair and Recovery
- Improved Load Balancing
- Simplified node addition and removal

to understand how data is distributed across a cluster using virtual nodes go through the below article: 
 [DataStax Apache Cassandra - How data is distributed across a cluster (using virtual nodes)](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/architecture/archDataDistributeDistribute.html)

Let’s pause now, before we go on to discuss our configuration further let us see how this information of geographically distributed nodes will be used to model our database and understand why it’s an extremely crucial part of designing a Cassandra database.