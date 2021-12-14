
Consistent Hashing
CAP Theorem,
Load Balancing, Caching, **Data Partitioning**, **Indexes**, Proxies,
Queues, Replication,  SQL vs. NoSQL.

# Key Characteristics

## scalability

- Horizontal scalling
- vertical scaling

## reliability

- By definition, reliability is the probability a system will fail in a given period In simple terms, a distributed system is considered reliable if it keeps delivering its services even when one or several of its software or hardware components fail.
- redundancy
高可用

## availability

- By definition, availability is the time a system remains operational to perform its required function in a specific period
- availabilty VS reliability
  - reliability --> availabilty
  - reliability <X- availabilty

## efficiency

- latency
- bandwidth

## servicebility and manageability

- Serviceability or manageability is the simplicity and speed with which a system can be repaired or maintained

并发用户数 高并发？

# Load Balancing

## concept

Load Balancer (LB) is another critical component of any distributed system.
It helps to spread the traffic across a cluster of servers to improve
**responsiveness** and **availability** of applications, websites or databases.

集群:将多台服务器组成集群，使用负载均衡将请求转发到集群中，避免单一服务器的负载压力过大导致性能降低。

## Algorithm

### Health Check

to make sure that the server in the pool are healthy. If not healthy, it will be removed
This should regularly happening

### algorithms

- Least Connection Method
  - when there are a large number of persistent client connections which are **unevenly** distributed between the servers.

- Least Response Time Method
- Least Bandwidth Method
- Round Robin methond
  - when the servers are of equal specification and there are not many persistent connections.

- Weighted Round Robin Method
- IP hash

### redundant LB

### Code and implementation

<https://www.lintcode.com/problem/526/>]

# Caching

Caches take advantage of the locality of reference principle: recently requested data is likely to be requested again

## Application cache

What happens when you expand this to many nodes? If the request layer is
expanded to multiple nodes, it’s still quite possible to have each node host its
own cache. However, if your load balancer randomly distributes requests
across the nodes, the same request will go to different nodes, thus increasing
cache misses. Two choices for overcoming this hurdle are global caches and
distributed caches.

## CDN

static media

## Cache invalidation

Consistency of Cache <=> DB

- write-through cache
- write-around cache
- write-back cache
  
## Cache **eviction** policies

- FIFO
- LIFO
- LRU
- LFU
- MRU
- RR(Random Replacement)

## Code

LRU
LFU

# Data partitioning 不太会

## Partitioning Methods

### Horizaontal Partitioning

- put different rows into different tables.
- Also Known as data sharding
- not even distrubuted range

### Vertical Partitioning

- partitioning by columns
- not be possible for a single server to handle all the metadata queries for 10
billion photos by 140 million users

### Directory-Based partitioning

lookup service that knows your current partitioning scheme and abstracts it away from the DB access code

## Partitioning Criteria

when insert data, based on which criteria that we should follow to insert this record into relevant partitioning

### Key or hash-based Partitioning

- hashing: hash(ID) % DB_count
- consistant hashing

### List Partitioning

each partition is assigned a list of
values, so whenever we want to insert a new record, we will see which
partition contains our key and then store it there.

### Round-Robin

i mod n

### Composite Partitoning

Under this scheme, we combine any of the above partitioning schemes to devise a new scheme.

## Common Problems of Data Partitioning ??

### join and denormalization

反范式化
<http://www.ayqy.net/blog/database-denormalization/>

### Referential Integrity

### Rebalancing

# Index 不太会

seem like a hash to a specific column for improving the query performance

## Index decrease write performance

When CRUD happen, index are also need to be updated.

    If the goal of the database is to provide a data store that is often written to and rarely read from, in that case, decreasing the performance of the more common operation, which is writing, is probably not worth the increase in performance we get from reading.

# Proxy

Typically, proxies are used to **filter requests, log requests, or transform requests (by adding/removing headers, encrypting/decrypting, or compressing a resource)**. Another advantage of a proxy server is that its cache can serve a lot of requests. If multiple clients access a particular resource, the proxy server can cache it and serve it to all the clients without going to the remote server

## Types

### Open proxy

### Reverse Proxy

retrieves resources on behalf of a client from one or more servers. These resources are then returned to the client, appearing as if they originated from the proxy server itself.

# Redundancy and Replication

redundancy

- duplication of critical components or functions of a system
- with the intention of increasing the reliability of the system,
- usually in the form of a backup or fail-safe, or to improve actual system performance

Replication

sharing information to ensure consistency between redundant resources

# SQL and NoSQL

## SQL

store data in rows and columns. Each row contains all
the information about one entity and each column contains all the separate
data points.

MySQL, Oracle, MS SQL Server, SQLite, Postgres and MariaDB

## No-SQL

| Type      | Concept | Example |
| ----------- | ----------- | ----------- |
| key-Value   | Data is stored in an array of key-value pairs  | Redis, Voldemort, and Dynamo  |
| Document Database  | data is stored in documents (instead of rows and columns in a table) and these documents are grouped together in collections        | MongoDB       |
| Wide-Column Databases  |  columnar databases we have column families, which are containers for rows.    | Cassandra   |
| Graph | These databases are used to store data whose relations are best represented in a graph   | Neo4J  |

## Difference

| Type      | SQL | No-SQL |
| ----------- | ----------- | ----------- |
| Schema | fixed  | Dynamic  |
| Querying | SQL | Different syntax for using UnQL(Unstructured Query Language) |
| Scalability  | mostly Vertically  | horizontally |
| Reliability/ACID? | Good | Not good |

## Which one to use

| SQL | No-SQL |
| ----------- | ----------- |
| ensure ACID compliance  | Storing large volumes of data that often have little to no structure  |
| data is structured and unchanging | Making the most of cloud computing and storage |
| | Rapid development |

## Ref

https://www.acodersjourney.com/10-questions-to-ask-yourself-before-choosing-a-nosql-database/

<https://azure.microsoft.com/en-us/overview/nosql-database/>
<https://cloud.netapp.com/blog azure-cvo-blg-azure-nosql-types-services-and-a-quick-tutorial>

# CAP Theorem

C: consistency -> all nodes see the same data at same time
A: Availability -> every request receive by non-failing node must result in a response
P: Partition -> communication break (or a network failure) between any two nodes in the system

CAP is a "three choices, pick two" scenario.
In the presence of
a network partition, a distributed system must choose either
Consistency or Availability.

# PACELC Theorem

P(partition) + A(Availability) + C(Consistency) + Else + L(Latency) + C(Consistency)


# Consistant hashing 

# Long-Polling vs WebSocket vs Server-Sent events

# Bloom filter

# Quorum


# Leader and Follower


# Heartbeat

# Checksum

# System Design Interviews

## Requirement Clarification

Scope.

## Back-of-the-envelope estimation

Back-of-the-envelope: roughly

- What scale is expected from the system ?
- How much storage will we need?
- What network bandwidth usage are we expecting?

## System interface definition

Define API

## Define data model

- Schema
- NoSQL vs SQL

## High-level design

draw the diagram

## Detailed design

Focus on some single part. Need more practice

## Identifying and resolving bottlenecks


# Example: Design URL shorten service

##  Requirement Clarification

##  Estimation 

read-heavy or write heavy?

