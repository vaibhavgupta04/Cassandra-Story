# Modeling our Student data

Our journey begins with the intricacies of designing the "student" column family, exploring the nuances of Cassandra's schema design and how it aligns with the specific needs of academic institutions.

Student column family will have essential attributes such as name, roll number, academic year, student ID, institute affiliation, and other relevant details. By creating this unified database structure, we aim to demonstrate the power of Cassandra in handling the data requirements of multiple IIIT colleges.

---

| Attribute | Data Type | Description |
| --- | --- | --- |
| student_id | UUID | Unique identifier for each student |
| name | Text | Full name of the student |
| roll | Text | Roll number assigned to the student |
| year | Int | Academic year of enrollment |
| institute_id  | Text | Name of the IIIT institute the student belongs to |
| *other attributes* | *Appropriate Data Types* | *Description of other relevant attributes* |

---

![IIIT-cluster.drawio.png](Cluster%20Configuration%20for%20IIITs/IIIT-cluster.drawio.png)

## Design consideration

The principles of designing distributed databases, particularly in systems like Cassandra emphasize the need for a well-thought-out data model that considers both the distribution of data across the cluster and the patterns of data access during read-and-write operations (queries). The main goal of our database design is to minimize read and write latency with high availability rather than enforcing consistency. A high-level goal while modeling our data should be: 

1. Spread data evenly (and logically) around the cluster.
2. Minimize the number of partitions read.

## Primary Key

In Cassandra, the choice of the primary key is pivotal for efficient data retrieval and distribution across the cluster. The primary key (also referred to as row key) consists of two components: the partition key and the clustering key.

> Primary key = Partition key + clustering key
> 

The **partition key** determines the distribution of data across nodes in the cluster. Choosing a suitable partition key is crucial for achieving even data distribution and avoiding hotspots. In our case, making "institute_id" the partition key will be a good choice. Assigning each IIIT college a unique "institute_id" allows the data for each institution to be stored on dedicated nodes, promoting effective load balancing and ensuring that queries related to a specific college are served efficiently.

**Clustering** in Cassandra is a storage engine process of sorting the data within a partition and is based on the columns defined as the clustering keys. In our case, storing the data in sorted order according to year might be a good option as this allows student records belonging to the same year to reside in close proximity to one another in a sorted manner enabling range queries (queries with multiple rows to be fetched) based on years to be fetched efficiently. We have added student_id as another clustering key making the primary key combination unique for each row in the column family.  Hence clustering key, comprising "year" and "student_id," defines the order of data within each partition. This choice is particularly advantageous for our "student" column family as sorting data based on the academic year and student ID facilitates queries related to specific years or individual students within an institute, optimizing the retrieval process.

By choosing “institute_id” as the partition key and “year” and “student_id” as the clustering key, we achieve a harmonious balance between efficient data distribution and structured retrieval. This deliberate choice for the primary key ensures that Cassandra’s decentralized architecture seamlessly aligns with the specific needs of our IIIT database, resulting in optimal performance and responsiveness.