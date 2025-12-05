# Read Anatomy: read inside a node

To satisfy a read, Cassandra must combine results from the active memtable and potentially multiple SSTables. Cassandra processes data at several stages on the read path to discover where the data is stored, starting with the data in the memtable and finishing with SSTables:

- Check the memtable
- Check row cache, if enabled
- Checks Bloom filter
- Checks partition key cache, if enabled
- Goes directly to the compression offset map if a partition key is found in the partition key cache, or checks the partition summary if not
    
    If the partition summary is checked, then the partition index is accessed
    
- Locates the data on disk using the compression offset map
- Fetches the data from the SSTable on disk

![read_anatomy.drawio.png](Read%20Anatomy%20read%20inside%20a%20node/read_anatomy.drawio.png)

> Reference
> 

 [Apache Cassandra Documentation: How is data read?](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/dml/dmlAboutReads.html)