# Compaction

In Cassandra, compaction is the process of organizing and consolidating data files to improve efficiency, reclaim disk space, and optimize performance. It is an essential maintenance operation to manage the storage and prevent the accumulation of obsolete or deleted data. The compaction process involves merging multiple SSTables (Sorted String Tables), which are the on-disk data structures used by Cassandra.

Here's a high-level overview of the compaction process in Cassandra:

1. **SSTables:**
    
    Cassandra stores data in SSTables, which are immutable data files sorted by key. As data is continuously written and updated, new SSTables are created.
    
2. **Compaction Trigger:**
    
    Compaction is triggered based on configurable conditions, such as the size of SSTables or the number of SSTables per table. When these conditions are met, compaction is initiated to merge and optimize SSTables.
    
3. **Compaction Strategies:**
    
    Cassandra supports different compaction strategies, including SizeTieredCompactionStrategy, LeveledCompactionStrategy, and DateTieredCompactionStrategy. Each strategy has its own characteristics and trade-offs in terms of performance and disk space utilization.
    
4. **Compaction Types:**
    
    There are two primary types of compaction: minor compaction and major compaction.
    
    - **Minor Compaction:** Involves compacting a small number of SSTables, typically within the same level or tier. It is triggered more frequently and helps to optimize small sets of data.
    - **Major Compaction:** Involves compacting a larger number of SSTables across multiple levels or tiers. It is less frequent and aims to optimize a larger portion of the data.
5. **Tombstone Removal:**
    
    During compaction, tombstones (markers indicating deleted data) and obsolete data are identified and removed. This helps in reclaiming disk space and ensures that deleted data is not retained indefinitely.
    
6. **Merging SSTables:**
    
    The compaction process involves merging overlapping SSTables to create a new, consolidated SSTable. This new SSTable may contain only the latest version of each piece of data, eliminating duplicates and outdated information.
    
7. **Compaction Strategies Impact:**
    
    The choice of compaction strategy impacts the trade-off between read and write performance, disk space usage, and compaction overhead. Administrators can select a strategy based on their specific use cases and requirements.
    
8. **Compaction during Read Operations:**
    
    In some cases, compaction might be triggered during read operations to ensure that the most recent and relevant data is provided. This process is known as read-time compaction.
    
9. **Compaction and Tombstone Grace Period:**
    
    Compaction plays a role in managing tombstones. After the grace period for tombstones ends, compaction is responsible for permanently removing tombstones from the SSTables.
    

By periodically running compaction, Cassandra maintains efficient storage, reduces read amplification, and ensures optimal performance in distributed and scalable environments. The choice of compaction strategy and tuning parameters depends on the specific requirements and workload characteristics of the Cassandra deployment.

> References
> 

[DataStax Apache Cassandra - How is data maintained?](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/dml/dmlHowDataMaintain.html#dmlHowDataMaintain__dml-compaction)