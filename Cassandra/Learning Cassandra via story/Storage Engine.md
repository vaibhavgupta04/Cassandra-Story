# Storage Engine

> Cassandra uses a storage structure similar to a Log-Structured Merge Tree. Here we explore the intricacies of the storage engine employed by Apache Cassandra. This storage engine is specifically crafted to excel in scenarios with a significant volume of write operations, all the while ensuring commendable performance for read operations.
> 

### **Log-Structured Merge-Tree**

**Cassandra** employs a **two-level Log-Structured Merge-Tree (LSMT)** data structure for storage. At a high level, this structure consists of two tree-like components: an **in-memory cache component (C0)** and an **on-disk component (C1)**.

![LSMT.png](Storage%20Engine/LSMT.png)

When reading and writing data, accessing memory directly is typically faster than accessing the disk. Consequently, all requests first hit the **C0** cache before reaching the **C1** component. Additionally, a synchronization operation periodically persists data from **C0** to **C1**, optimizing network bandwidth usage and reducing I/O operations.

### **MemTable**

**MemTable**, as the name suggests, is a memory-resident data structure similar to a Red-black tree with self-balancing binary search tree properties. This design allows all read and write operations—such as search, insert, update, and delete—to be achieved with **O(log n)** time complexity. 

Being an in-memory mutable data structure, MemTable ensures that writes occur sequentially, enabling fast write operations. However, due to the inherent limitations of physical memory (such as capacity constraints and volatility), we must persist data from MemTable to disk. As the size of the MemTable grows, it eventually reaches a threshold value. At this point, all read and write requests switch to a new MemTable. The old MemTable is then discarded after flushing its contents onto the disk.

![2.png](Storage%20Engine/2.png)

While MemTable efficiently handles a large number of writes, there’s a potential issue. If a node crashes before a flush operation occurs, any data not yet flushed to disk will be lost. Apache Cassandra addresses this challenge using the concept of **Write-Ahead Logs (WAL)**.

### **Commit Log**

**Apache Cassandra** defers the flush operation that persists data from memory to disk. Consequently, an unexpected node or process crash can result in data loss.

However, durability is a critical feature for any modern database system, and Apache Cassandra ensures it by guaranteeing that all writes are stored on disk in an **append-only file called the Commit Log**. Only after this, the data is written into **MemTable** in a write path. The Commit Log’s append-only nature ensures fast operations by avoiding random disk seeks. This way, durability is maintained without compromising write performance.

![3.png](Storage%20Engine/3.png)

Apache Cassandra primarily refers to the Commit Log during crash recovery scenarios. Regular read requests do not directly access the Commit Log.

In this way, Cassandra strikes a balance between durability and performance to provide a robust storage solution.

### **SSTable**

An SSTable (Sorted String Table) is a fundamental on-disk storage structure that efficiently organizes data in a sorted manner. **SSTable** serves as the **disk-resident** part of the **Log-Structured Merge-Tree (LSM tree)** used by Cassandra. The name itself hints at its purpose: data is stored in a **sorted format**.

Let’s try to visualize how an SSTable would look while containing data about the count of various animals kept in a zoo:

![4.png](Storage%20Engine/4.png)

Though the segments are sorted by keys, nevertheless, the same key could be present in multiple segments. So if we’ve to look for a specific key, we need to start our search from the latest segment and return the result as soon as we find it.

With such a strategy, read operations for the recently written keys would be fast. However, in the worst case, the algorithm executes with an O(*N**log(*K*)) time complexity, where *N* is the total number of segments, and *K* is the segment size. As *K* is a constant, we can say that the overall time complexity is O(*N*), which is not efficient.

SSTables are designed to store key-value pairs in a sorted order based on keys, allowing for rapid and efficient range queries during read operations. An important attribute of SSTables is their immutability; once data is written to an SSTable, it becomes a static entity, and any modifications, updates, or deletions result in the creation of new SSTables. This immutable nature simplifies the process of compaction, where obsolete or deleted data is efficiently cleared, ensuring optimal disk space utilization and performance.

Furthermore, SSTables are partitioned based on a consistent hashing algorithm, each responsible for specific ranges of partition key values. This partitioning scheme contributes to the decentralized nature of Cassandra, distributing data across the nodes of the cluster. The inclusion of Bloom filters within SSTables provides an added layer of efficiency by swiftly identifying whether an SSTable might contain relevant data for a given query. This capability helps minimize unnecessary disk reads during read operations, enhancing overall performance.

SSTables incorporate multiple indexes, including a partition index and a row index. The partition index facilitates quick localization of the starting position of a partition during read operations, while the row index aids in efficiently finding the position of individual rows within the SSTable. Optional compression of SSTables further optimizes storage space usage and I/O efficiency.

The immutable nature simplifies compaction processes, contributing to Cassandra's scalability, fault tolerance, and ability to handle large datasets with high read and write throughput

### Partition Index (Index Table)

Cassandra employs indexing to minimize the number of segments it must scan when searching for a specific key. Each entry in the index includes the **first key** of a segment and its corresponding **page offset position** on the actual data file. The index is maintained in memory as a **B-Tree data structure**, allowing efficient searches for an offset with **O(log(K))** time complexity.

Here’s how the process works:

1. Suppose we want to find the key “**beer**.”
2. We begin by searching for all keys in the index that come before “**beer**.”
3. Using the offset value obtained from the index, we focus on a **limited number of segments**. For example, we might examine the fourth segment, where the first key is “**alligator**.”

![5.png](Storage%20Engine/5.png)

On the other hand, if we had to search for an absent key such as “kangaroo,” we’d have to look through all the segments in vain. So, we realize that using a sparse index optimizes the search to a limited extent.

Moreover, we should note that SSTable allows the same key to be present in different segments. Therefore, with time, more and more updates will happen for the same key, thereby creating duplicate keys in a sparse index too.

In the following sections, we’ll learn how Cassandra addresses these two problems with the help of bloom filters and compaction.

### **Bloom Filter**

**Apache Cassandra** enhances read queries using a **probabilistic data structure** called a **bloom filter**. In simple terms, it optimizes searches by first checking if a key is a member of a set using the bloom filter.

By attaching a bloom filter to each segment of the **SSTable** (Sorted String Table), we achieve significant optimizations for read queries, especially for keys that do not exist in a segment. Bloom filters provide probabilistic answers, which means we might receive a “Maybe” response even for missing keys. However, if the response is “No,” we can be certain that the key is definitely missing.

### Additional Components

We have discussed some major storage components of the LSM tree. On top of these Cassandra makes use of some additional components which are discussed below.
A complete storage component of Cassandra looks something like this:

![c_-storage-components.drawio.png](Storage%20Engine/c_-storage-components.drawio.png)

### Row Cache

Row cache is a feature in Cassandra that stores frequently accessed rows of data in memory, allowing for faster access and retrieval of data. This helps to reduce the number of disk reads and improve overall database performance. The row cache is configurable and can be enabled on a per-table basis, allowing for fine-tuning of caching behavior based on specific data access patterns.

### Key Cache

The key cache in Apache Cassandra is a memory-resident cache that stores the locations of frequently accessed keys in the database. This cache helps to reduce the number of disk seeks when accessing data by keeping the most commonly accessed keys readily available in memory. By caching the key locations, Cassandra can quickly locate the corresponding data on disk, resulting in improved read performance. The key cache is also configurable and can be set at the table level.

### Index Summary (Partition Summary)

The index summary can be thought of as an "index of an index." It is a data structure that provides a summarized representation of the index data for the SSTable files. Instead of directly mapping to the actual data locations, the index summary acts as a high-level guide to quickly locate the position of the index entries within the SSTable. This allows Cassandra to efficiently navigate and access the index entries without needing to scan the entire index, improving the performance of range queries.

### Column Bloom Filter

A column bloom filter is a compact, probabilistic data structure associated with each SSTable. It efficiently filters out unnecessary disk reads during queries by indicating whether a requested column is likely present in the SSTable. This optimization enhances read performance by skipping unnecessary I/O operations, though it may have occasional false positives. The size and parameters of the bloom filter are configurable to balance memory usage and the probability of false positives.

### Data File

Data file is the main section containing the actual data, organized as sorted key-value pairs. Each key corresponds to a row or a column, and the data is sorted based on these keys to optimize read performance.

> Reference
> 

[**Guide to the Storage Engine in Apache Cassandra](https://www.baeldung.com/cassandra-storage-engine#:~:text=Sorted%20String%20Table%20(SSTable)%20is,available%20in%20a%20sorted%20format.)
[Apache Cassandra Documentation - How is data written?](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/dml/dmlHowDataWritten.html)**