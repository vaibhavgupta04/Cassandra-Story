# Learning Cassandra via story

<aside>
💡 A complete guide to Cassandra database.

</aside>

Welcome to the fascinating realm of Cassandra, a distributed NoSQL database system designed for handling massive amounts of data across multiple nodes and ensuring high availability and fault tolerance. In this tutorial, we will embark on a journey to unravel the intricate internal workings of Cassandra by delving into the design and architecture of a hypothetical database for a network of five Indian Institutes of Information Technology (IIIT) colleges.

We will focus on the design of a Cassandra database tailored for the collaborative efforts of five IIIT colleges namely IIIT Allahabad(IIIT A), IIIT Delhi(IIIT D), IIIT Banglore(IIIT B), IIIT Hyderabad(IIIT H), and IIIT Gwalior(IIIT G) aiming to manage student information efficiently. Our demo table, aptly named "student," will serve as a tangible example to illustrate key concepts and practices in Cassandra database development.

Throughout this journey, we will refer to and reference various online sources and tutorials to provide you with an opportunity to delve deeper into specific concepts. These resources will be valuable for those seeking a more detailed understanding of Cassandra's architecture and principles.

## **Prerequisites**

---

Before diving into the details, it's recommended that you have a basic understanding of the architecture of the Cassandra database as well as data modeling in Cassandra. If you're new to these topics, here’s a complete guide to help you familiarize yourself with the fundamentals:

[Getting started with Cassandra](Learning%20Cassandra%20via%20story/Getting%20started%20with%20Cassandra%205b608606165e4be5952f79f385231579.md)

[**Academy DS220 Data Modeling with Apache Cassandra**](https://youtube.com/playlist?list=PL2g2h-wyI4SqIigskyJNAeL2vSTJZU_Qp&si=XXyY47uu10a3tV-b)

## Setting up the story

---

Imagine a network of five prestigious Indian Institutes of Information Technology (IIIT) colleges, each contributing to the collective pursuit of knowledge and technological innovation. At the heart of this educational consortium lies the need for an efficient and scalable database system to manage student information seamlessly across the institutes.

To streamline the data management process, we've chosen to focus on a single-column family named "student." We will be designing and understanding the workings of the Cassandra database around this column family.

## Cluster Configuration and Data Modeling

---

Let’s start our story by configuring our cluster 

[Cluster Configuration for IIITs](Learning%20Cassandra%20via%20story/Cluster%20Configuration%20for%20IIITs%205322b49b5cfc4b10b3347ab08f635849.md)

So we march on to model our “student” column family 

[Modeling our Student data](Learning%20Cassandra%20via%20story/Modeling%20our%20Student%20data%2064ec0c832bfd45efa94a09b8109ed756.md)

Now that we have a basic configuration and data model ready let’s move on to make our database resilient, fault-tolerant, and increase availability by introducing replication.

[Enhancing our Database](Learning%20Cassandra%20via%20story/Enhancing%20our%20Database%2016db987f63f444e7ad71914167bf3aff.md)

## Storage Engine

---

Now we have our nodes and column family ready but before we start inserting data into it let’s take a look under the hood and see what is a node even made up of and understand how it stores data inside it.

[Storage Engine](Learning%20Cassandra%20via%20story/Storage%20Engine%20873768bbdae94698a008a8946ced7c5d.md)

## Write in Cassandra

---

Now that we have an idea of how data is stored and persisted inside a Cassandra node let’s start inserting data into our database and travel along the write path our data will follow behind the scenes.

[Write Path in Cassandra](Learning%20Cassandra%20via%20story/Write%20Path%20in%20Cassandra%2003d41a5bfbd64a47adb4f02966813345.md)

[Write Anatomy: write inside a node](Learning%20Cassandra%20via%20story/Write%20Anatomy%20write%20inside%20a%20node%20168983d0dd7e42e0a5ef8b48304400fb.md)

## Read in Cassandra

---

Alright, time to get that data back. Let’s read 

[Read Path in Cassandra](Learning%20Cassandra%20via%20story/Read%20Path%20in%20Cassandra%20db50118520814d19a1a66bcf50c80c9c.md)

[Read Anatomy: read inside a node](Learning%20Cassandra%20via%20story/Read%20Anatomy%20read%20inside%20a%20node%202215cada3e0a4a458feb357cbf079591.md)

## Delete in Cassandra

---

The delete path in Cassandra is a bit different from traditional relational databases due to the distributed nature of the system. 

[Delete in Cassandra ](Learning%20Cassandra%20via%20story/Delete%20in%20Cassandra%20a5a2b6007daa43e1b1f00e1e667b0ea1.md)

[Compaction](Learning%20Cassandra%20via%20story/Compaction%2003abd051f8b940cabf85f20a627ca6a6.md)

## Example queries

---

Let’s take a few queries and understand their interaction with the database.

[Query Access Patterns ](Learning%20Cassandra%20via%20story/Query%20Access%20Patterns%20c0ffb1e4e09f47eb9fc7ce1b9e93b8cf.md)

## References ****

---

[CAP Theorem](Learning%20Cassandra%20via%20story/CAP%20Theorem%20fb0dcf3ebe7840bea73c028bae8bc928.md)

[DataStax Apache Cassandra 2.1](https://docs.datastax.com/en/cassandra-oss/2.1/index.html)