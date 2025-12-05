# Query Access Patterns

### Single Row Retrieval:

```jsx
SELECT * FROM student WHERE institute_id = 'IIIT_A' AND year = 2023 AND student_id = '12345';
```

The query involves retrieving a specific student's details based on their institute, year, and unique student ID. 

Access Pattern: 

- The coordinator node identifies the token ranges corresponding to the specified institute and sends a read request to the nodes that hold this token range.
- Since the rows within the partition are sorted according to year and student_id Cassandra efficiently retrieves the data involving minimum partition scans

### **Range Query by Institute:**

```sql
SELECT * FROM student WHERE institute = 'IIIT_H' AND year = 2023;
```

This query retrieves all students from a specific institute who belong to a particular academic year.

**Access Pattern:**

- The coordinator node identifies the token ranges corresponding to the specified institute and then scatters the query to the relevant replica node(s).
- Within a node, since data is sorted by year rows belonging to the same year are grouped together making the retrieval efficient as Cassandra can directly find the offset of the year 2023 and fetch data sequentially, and avoid reading the entire partition.
- In case the partition is too large and the data for the specified year has been split on multiple nodes then the coordinator aggregates the data from all the required nodes and sends the data to the client. This will cause more latency as it involves reading data from multiple nodes.

### **Aggregated Statistics by Year:**

let's assume the partition for IIIT_D has been fragmented into 2 nodes due to a high number of students in IIIT Delhi. We would like to retrieve aggregated statistics, counting the number of students in each academic year for the specified institute.

```sql
SELECT year, COUNT(*) FROM student WHERE institute = 'IIIT_D' GROUP BY year;
```

**Access Pattern:**

- The coordinator node scatters the query to nodes responsible for the relevant token ranges based on the institute partition key.
- Each node processes the local query and sends back partial count results, which the coordinator aggregates and returns.

### **Filtering by Multiple Attributes:**

```sql
SELECT * FROM student WHERE institute = 'IIIT_G' AND year = 2022 AND gender = 'Female';
```

This query filters students based on multiple attributes (institute, year, and gender).

**Access Pattern:**

- This query would throw an error as in Cassandra we can’t query on a non-primary key attribute.
- One of the ways to work around this could be to introduce a secondary index on “gender” attributes in our “student” column family. However, you need to be aware of the implications of creating a secondary index in Cassandra. You can read more about it here:
[DataStax Apache Cassandra - when to use an index](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_using/useWhenIndex.html)

### **Batch Retrieval by Student IDs:**

```sql
SELECT * FROM student WHERE student_id IN ('12345', '67890', '54321');
```

This query retrieves details for multiple students identified by their student IDs.

**Access Pattern:**

- the code above would throw an error because we haven’t specified the partition key and the first clustering key (ordering of keys matters while defining primary key)
- the correct code would be

```jsx
SELECT * FROM student WHERE institute_id = "IIIT_A" AND year = 2022 AND student_id IN ('12345', '67890', '54321');
```

- this would request in the whole partition specified by IIIT_A and year 2022 being scanned. Hence we need to be careful when we use the “IN” keyword in CQL as it might involve unnecessary read of the entire partition.