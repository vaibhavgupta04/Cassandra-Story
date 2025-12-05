# CAP Theorem

## **Introduction**

The CAP theorem, also known as Brewer's theorem, is a concept in distributed computing highlighting the trade-offs between three key properties: Consistency, Availability, and Partition Tolerance. It was introduced by computer scientist Eric Brewer in 2000.

**Consistency (C)**: This property ensures that all nodes in a distributed system have a consistent view of the data. In other words, when a distributed system responds to a read request, it provides the most recent write for that data. Consistency implies that all nodes see the same data at the same time.

**Availability (A):** Availability refers to the ability of a distributed system to continue functioning and responding to requests even in the presence of failures or network partitions. A system is considered available if every request gets a response, without guaranteeing that it contains the most recent version of the data.

**Partition Tolerance (P)**: Partition Tolerance is the ability of a distributed system to function and maintain consistency, even when communication between nodes is unreliable or delayed. A network partition occurs when nodes in the system are unable to communicate with each other, leading to the division of the network into isolated components.

The CAP theorem states that in a distributed system, you can achieve at most two out of the three properties simultaneously.

Here are the three scenarios:

**CA** (Consistency and Availability, but no Partition Tolerance): In this scenario, the system sacrifices Partition Tolerance. When a network partition occurs, the system either becomes unavailable or ensures consistency by refusing to respond until the partition is resolved.

**CP** (Consistency and Partition Tolerance, but no Availability): In this case, the system sacrifices Availability. The system remains consistent even in the presence of network partitions, but it may become unavailable during partition scenarios.

**AP** (Availability and Partition Tolerance, but no Consistency): This scenario sacrifices Consistency. The system remains available and can continue to serve requests even during network partitions, but it may provide different views of the data to different nodes, leading to eventual consistency.

## **Comments on CAP theorem**

Looking at the CAP theorem, one must note that while the theorem is flawless in its correctness, the formulation can be misleading about the implications. Two of which have been discussed below.

CA architecture is only feasible in a realistic environment when no partition can happen. A distributed system (database) will always have network partitions in terms of node failures or network failures. One can interpret the fact that the system should discard Partition Tolerance as long as there is no partition, and as soon as a network partition occurs it needs to switch its strategy and choose a tradeoff between Consistency and Availability.

Another aspect to be observed is that the theorem presents the three properties as equal. But while Consistency and Availability can be measured in a spectrum i.e. Consistency-Availability is a tradeoff between one another rather than a choice between the two, Partition Tolerance is rather binary. One can only say whether the system supports Partition Tolerance or it does not.

## **Discussion**

If we imagine the tradeoff between Consistency and Availability as a scale with one extreme meaning sacrificing all consistency for availability and the other extreme meaning sacrificing all availability for consistency, It is important to mention that no implementation of either extreme is feasible;  e.g. while BASE aims for availability it is still eventually consistent, so it is not on the availability extreme. The decision of which tradeoff is the best for a product has to be considered carefully.

## **Reference**

[1] [Report to Brewer’s original presentation of his CAP Theorem at the Symposium on Principles of Distributed Computing (PODC) 2000](https://edisciplinas.usp.br/pluginfile.php/2541318/mod_resource/content/1/TeoremaDeBrewer.pdf)

[2] S. Gilbert and N. Lynch, "Perspectives on the CAP Theorem," in Computer, vol. 45, no. 2, pp. 30-36, Feb. 2012, doi: 10.1109/MC.2011.389.