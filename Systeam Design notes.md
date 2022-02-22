# System design notes for “Grokking the System Design Interview”

系统设计学习笔记：

- 这个东西是什么？-> 具体概念，算法
- 为什么需要？-> 他用来解决什么样的问题，为什么需要它？
- 具体怎么实现？-> 算法
- 这样实现有什么样的问题？-> 不同实现之间，有什么优劣

Everything is a trade-off

# Key Characteristics

## scalability 扩展性/伸缩性

- Horizontal scalling
  - By adding more server into the pool.
- Vertical scaling
  - By adding more resources, RAM, CPU, Storage.

想法：

    scalability是不是在讨论DB的扩展？还是泛指其他的 
    
    横向扩展就是加机器，组成集群？形成拓扑结构?
    
    多机器以后：

    - 怎么实现负载均衡？有的节点老旧，有的节点新。
      - 这种多样性又带来了异质性。那么均匀的负载均衡算法就不适用。
    - 如果要扩展DB，选什么？怎么设计？什么样的表？如何partition DB？如何DB replication?
    - 怎么搭集群，如何设计拓扑结构
    - 这份工作具体是谁完成的？

Scalability并不是针对DB。Scalability是指系统，你的所有设计target for increasing the scalability

再补充一些关于scalability的：

Scalability的Topics covered:

> <https://www.youtube.com/watch?v=-W9F__D3oY4>
>
> - Vertical scaling
> - Horizontal scaling
> - Caching
> - Load balancing
> - Database replication
> - Database partitioning

所以scalability包含LB，Caching，DB replication， DB partition等等一切可以使系统scale up的概念

***

## Reliability 可靠

By definition, reliability is the probability a system will fail **in a given period**. In simple terms, a distributed system is considered reliable if it keeps delivering its services even when one or several of its software or hardware components fail.

redundancy 冗余设计是一个实现reliabled的手段。

***

## Availability 可用

By definition, availability is the time a system remains operational to perform its required function in a specific period

Reliability is **availability over time** considering the full range of possible
real-world conditions that can occur.

可靠性 = 可用性 * 时间

所以可靠性的一定时可用性的。可用的，不一定是可靠的

- availabilty VS reliability
  - reliability --> availabilty
  - availabilty -X-> reliability

有两种方法可以支撑高可用性：

- fail-over
  - active-passive(master-slave failover):
    - there is master server, it regularly send hearbeat signal to the backup server. Once master is down, backup server takes over and resume the service
  - active-active(master-master failover):
    - there are two main servers, and they are all managing the traffic
    - LB
  - failover不好的地方在于：
    - 增加硬件，增加复杂度
    - potential of loss of data

- replication
  - 见replication章节

如何评价和计算availablity：

- 99.99 vs 99.9
  - <https://azure.microsoft.com/en-us/support/legal/sla/summary/>
- parallel vs in sequence
  - if a system has many components and each components has a chance to fail, then the overall availability are:
    - parallel:

      ```
      Availability (Total) = 1 - (1 - Availability (Foo)) * (1 - Availability (Bar))
      ```

    - in sequence :

      ```
      Availability (Total) = Availability (Foo) * Availability (Bar)
      ```

***

## Efficiency

两种衡量方法

- latency： reponse time
- bandwidth: throughput that denotes the number of items delivered in a given time limit.

两种单位：

- 系统节点发出的消息总数
- 发出消息的size。

throught_put = message + cnt * message_size?

Other measurements:

> 通过单位时间所生产的东西来计量，例如内存带宽（memory bandwidth）用来衡量内存系统的吞吐量，而对于Web系统，有这些度量单位：
>
>QPS（Queries Per Second）：用来衡量信息检索系统（如搜索引擎、数据库等）在1秒内的搜索流量
>
>RPS（Requests Per Second）：请求-响应系统（如Web服务器）每秒所能处理的最大请求数量
>
>TPS（Transactions Per Second）：广义上指在1秒内所能执行的原子操作数量，狭义上指DBMS在1秒所能执行的transaction数量

***

## Servicebility and manageability 可维护性

Serviceability or manageability is the simplicity and speed with which a system can be repaired or maintained

比如当出现故障时，自动打电话给相应的维护人员。

## Performance VS scalability

A service is scalable if it results in increased performance in a manner proportional to resources addeds

- If you have a performance problem, your system is slow for a single user.
- If you have a scalability problem, your system is fast for a single user but slow under heavy load.

## Latency vs throughput

见Efficiency

## Availability vs consistency

这里就是CAP theory。详细见下CAP

## Ref

- <https://xie.infoq.cn/article/20c3ad1736d027615b12d6b20>
- <http://www.cyc2018.xyz/%E5%85%B6%E5%AE%83/%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1/%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1%E5%9F%BA%E7%A1%80.html#%E4%B8%80%E3%80%81%E6%80%A7%E8%83%BD>
- <http://www.ayqy.net/blog/scalability_%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1%E7%AC%94%E8%AE%B01/>
- <https://docs.microsoft.com/en-us/azure/architecture/guide/design-principles/redundancy>
- <https://www.techopedia.com/definition/14533/queries-per-second-qps>
- <https://segmentfault.com/a/1190000010844969>
  
# Load Balancing

## Concept

Load Balancer (LB) is another critical component of any distributed system.
It helps to spread the traffic across a cluster of servers to improve
**responsiveness** and **availability** of applications, websites or databases.

> Load balancers can be implemented with hardware (expensive) or with software such as HAProxy.

## Why need?

1.
>Single Point of Failure: If the server goes down or something happens to the server the whole application will be interrupted and it will unavailable for the users for a certain period. It will create a bad experience for users which is unacceptable for service providers.

单点故障，一个点有问题导致的整个系统崩溃。

2.
>Overloaded Servers: There will be a limitation for the number of requests which a web server can handle. If the business grows and the number of requests increases the server will be overloaded. To solve the increasing number of requests we need to add a few more servers and we need to distribute the requests to the cluster of servers.

负载过重的服务器。

## How to 怎么做?

集群:将多台服务器组成集群，使用负载均衡将请求转发到集群中，避免单一服务器的负载压力过大导致性能降低。

LB可以在多层去做:

- client
- client and web
- web and application or cache
- application and DB

### Health Check

To make sure that the server in the pool are healthy. If not healthy, it will be removed
This should regularly happening

### Algorithms

- Least Connection Method
  - when there are a large number of persistent client connections which are **unevenly** distributed between the servers.

- Least Response Time Method > lowest response time
- Least Bandwidth Method > the server who serve the least amount traffic
- Round Robin methond
  - when the servers are of **equal specification** and there are not many persistent connections.

- Weighted Round Robin Method
  - better handle the different specificatioin
  
- IP hash
  - The load balancer routes requests from the same client to the same backend server as long as that server is available.
  - IP Hash ensures that requests from a particular client are always directed to the same backend server, as long as it is available.
  - You cannot add a backend server marked as Backup to a backend set that uses the IP Hash policy.

### L4, L7 load balance

- L4， L7 各自是什么
  - L4:
    - in the transport layer
    - looking at the source, destination IP, ports in the header, but not conent
    - Do NAT
  - L7
    - in the application layer
    - involve content like, header, message, cookies
    - read the message, make decision, like for a video messages, redirect ot the video server; for a biliing traffic, it redirect to sensitive data hosting server.
  
- 各自的好处与坏处
  - L4, simple and quick; less time and conmputing resource
  - L7, more info, more insight, better load-balance but more resource, more expensive?

## LB 怎么好？

LB可以带来：

- 终端用户可以体验更好：更快，无中断的服务
- 更少的downtime的时间。
- SMART的LB可以提供business insight
- 对于Systeam admin，更易于manage

## how bad

> The load balancer can become a performance bottleneck if it does not have enough resources or if it is not configured properly.
>
> Introducing a load balancer to help eliminate a single point of failure results in increased complexity.
>
> A single load balancer is a single point of failure, configuring multiple load balancers further increases complexity.

To resolve, introduce redundant LB

Because LB can also be a SPF(single point of failure). We can add another LB to form a cluster. Each monitor the health of the other. Once the main fail, the second takes over.

> To protect against failures, it's common to set up multiple load balancers, either in active-passive or active-active mode.

## Ref

- <https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/load-balancing-overview>
- <http://www.ayqy.net/blog/load-balancing/>

# Caching

Caches take advantage of the locality of reference principle: **recently requested data is likely to be requested again**

## 缓存的一些概念

- 缓存命中：当某个请求访问缓存得到相应时，称之为缓存命中
- 最大空间：缓存通常位于内存中。因此缓存的空间一般不会特别大。
- 清空策略：因此当缓存存放的数据量超过最大空间时，需要采取一定策略淘汰一些数据

## 为什么需要缓存?

提高效率，不浪费资源。利用缓存层来吸收不均匀的负载和流量高峰：

> Popular items can skew the distribution, causing bottlenecks. Putting a cache in front of a database can help absorb uneven loads and spikes in traffic.

## 怎么做？--缓存的位置

缓存基本处于数据层前的任何一层。减少最终抵达数据库的请求。缓存通常用于最接近front end的层面。这样更快
缓存的位置可以有（从进到远）：

- 客户端缓存：浏览器缓存。HTTP缓存
- 反向代理
- Web 缓存：CDN
- 应用层的本地缓存
- 应用层的分布式缓存：Redis这种“键值存储”
- 数据库的缓存：一些数据库有一些查询缓存。

### Application cache

What happens when you expand this to many nodes? If the request layer is
expanded to multiple nodes, it’s still quite possible to have each node host its
own cache. However, if your load balancer randomly distributes requests
across the nodes, the same request will go to different nodes, thus increasing
cache misses. Two choices for overcoming this hurdle are **global caches and
distributed caches.**

这里讲的就是本地缓存和分布式缓存

>本地缓存：指的是在应用中的缓存组件，其最大的优点是应用和cache是在同一个进程内部，请求缓存非常快速，没有过多的网络开销等，在单应用不需要集群支持或者集群情况下各节点无需互相通知的场景下使用本地缓存较合适；同时，它的缺点也是应为缓存跟应用程序耦合，**多个应用程序无法直接的共享缓存**，**各应用或集群的各节点都需要维护自己的单独缓存，对内存是一种浪费**。

>分布式缓存：指的是与**应用分离**的缓存组件或服务，其最大的优点是**自身就是一个独立的应用**，与本地应用隔离，多个应用可直接的共享缓存。

## 缓存存在的问题

### Cache invalidation

Consistency of Cache <=> DB

- cache-aside
- write-through cache
- write-around cache
- write-back cache
  
### Cache **eviction** policies

- FIFO
- LIFO
- LRU
- LFU
- MRU
- RR(Random Replacement)

## Code sample: LRU, LFU

- LRU: <https://leetcode.com/problems/lru-cache/>
- LFU

## Ref

- <https://tech.meituan.com/2017/03/17/cache-about.html>
- <http://www.ayqy.net/blog/caching/>
- <https://docs.microsoft.com/en-us/azure/architecture/best-practices/caching>
- <https://docs.microsoft.com/en-us/azure/architecture/best-practices/cdn>
- <http://www.cyc2018.xyz/%E5%85%B6%E5%AE%83/%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1/%E7%BC%93%E5%AD%98.html#%E4%B8%80%E3%80%81%E7%BC%93%E5%AD%98%E7%89%B9%E5%BE%81>

# CDN

## what is CDN

CDN is distributed content server network. Serving content to nearest service end-users.
It generally store static data, like html, js, css file.

## Push CDNs vs Pull CDNs

> Push CDNs receive new content whenever changes occur on your server. You take full responsibility for providing content, uploading directly to the CDN and rewriting URLs to point to the CDN. You can configure when content expires and when it is updated. Content is uploaded only when it is new or changed, minimizing traffic, but maximizing storage.

> Sites with a small amount of traffic or sites with content that isn't often updated work well with push CDNs. Content is placed on the CDNs once, instead of being re-pulled at regular intervals.

> Pull CDNs grab new content from your server when the first user requests the content. You leave the content on your server and rewrite URLs to point to the CDN. This results in a slower request until the content is cached on the CDN.

> A time-to-live (TTL) determines how long content is cached. Pull CDNs minimize storage space on the CDN, but can create redundant traffic if files expire and are pulled before they have actually changed.

> Sites with heavy traffic work well with pull CDNs, as traffic is spread out more evenly with only recently-requested content remaining on the CDN.

##

# DNS

## What is it

translate domain names to IP address.

## how it work

- client contact DNS server(maybe according to outbound IP) to get
- if there is no cache, this DNS server just look up until root server to find the relevant map.
- Once find, this DNS server will cache this in itself and also send result to the client
- client just make request.

## Who maintain DNS

- 13 root server managed mainly by verisign, US army, US research party.
- These are the secondary servers maintained by Governments, ISP's, and private companies like Google, OpenDNS. These servers get stuff from the Root Servers and cache them and they feed it to us the users. These are faster than the root server because results are cached from the Root Servers.

## disadvantage

> Accessing a DNS server introduces a **slight delay**, although mitigated by caching described above.
>
> DNS server management could be complex and is generally managed by governments,
> ISPs, and large companies.
>
> DNS services have recently come under DDoS attack, preventing users from
> accessing websites such as Twitter without knowing Twitter's IP address(es).

# Message Queue

# Application Server

## what is

In the application layer, there could be component like microservice, service discovery, etc.

## Web server vs application server

Application server handle logic, while web server only handle http requests back and forth.

<https://stackoverflow.com/questions/936197/what-is-the-difference-between-application-server-and-web-server>

## Disadvantage

- Increase complexity. Need a well configuration and design from the "architectural, operations, and process viewpoint" Perspective
- deployment and operations
- So still depend on how your business is. The scale, the DAU, sth like that.

## Ref

<https://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones>

every server contains exactly the same codebase and does not store any user-related data, like sessions or profile pictures, on local disc or memory
<https://lethain.com/introduction-to-architecting-systems-for-scale/#platform_layer>


# Proxy

## 是什么？

Typically, proxies are used to:

- Filter requests
- Log requests
- Transform requests
  - (by adding/removing headers, encrypting/decrypting, or compressing a resource).

Another advantage of a proxy server is that its cache can serve a lot of requests. If multiple clients access a particular resource, the proxy server can cache it and serve it to all the clients without going to the remote server

## 为什么？有啥作用

应该说，放在不同位置的代理，或者不同类别的代理作用应该都各不相同。
正向的：

- log request: fiddler
- 保护客户端或者保护服务器
- 访问内网
- 加密：v2ray
- 代理翻墙

## 不同的类别

- Open proxy: 面向公众的，任何人都可以访问的正向代理
- Anonymous proxy: 匿名代理, 不公开客户端原始IP地址的代理服务
- 透明代理: 不作任何修改，一般用作网关，路由器。

### Reverse Proxy

> retrieves resources on behalf of a client from one or more servers. These resources are then returned to the client, appearing as if they originated from the proxy server itself.

如何理解反向搭理？

- 反向代理是和服务器密切相关的。
- 正向代理是其关联的客户端和外界服务器的中介。而反向代理则是其关联服务器和外界客户端的中介
- 正向代理知道客户端，不了解服务器；反向代理知道服务器，不了解客户端
- 反向代理可以保护内网服务器，不为外界所知
- 就可以把反向代理理解成“服务器作为客户端的正向代理”

反向代理的作用：

- 加密
- 负载均衡
- 缓存
- 安全防护
- 访问控制
- 托管静态内容

## Reverse proxy VS load balancer

- mulitple server -> load balancer
- single application server, web server architecture -> Reverse proxy
- Current software like nginx can support both

## Ref

- <http://www.ayqy.net/blog/reverse-proxy/>
- <https://docs.microsoft.com/en-us/azure/architecture/guide/multitenant/considerations/map-requests#reverse-proxies>
- <https://serverfault.com/questions/127021/what-is-the-difference-between-load-balancer-and-reverse-proxy>
- <https://stackoverflow.com/questions/59782057/what-is-the-difference-between-reverse-proxy-and-load-balancer>


# SQL and NoSQL

## 是什么

### SQL

关系型数据库，提供的数据存储，检索机制是基于表关系建模的。

> store data in rows and columns. Each row contains all
the information about one entity and each column contains all the separate
data points.

MySQL, Oracle, MS SQL Server, SQLite, Postgres and MariaDB

### No-SQL

- 非关系型数据库。提供的数据存储，检索机制并不是基于表关系建模的。
- 在关系型数据库之外的广阔世界里，数据不一定非要打平存放到二维表格里，关系也不是只能用**主键、外键、关系表**来描述
- 从数据库类型类型来看，有很多no-sql的DB
- 从实践角度来看，用NoSQL的方式使用MySQL数据库也算
  - 比如在数据表中存一列 JSON 字符串，把这一列当作键值数据库来用

#### No-Sql比较

| Type      | Concept | Example | Apply to
| ----------- | ----------- | ----------- | ----------- |
| key-Value   | Data is stored in an array of key-value pairs <br> 类似一个 hash表  |   Redis, Voldemort, and Dynamo  | 用于简单、或者频繁更改的数据，经常用作**内存缓存**
| Document Database  | 1. data is stored in documents，比如JSON，XML (instead of rows and columns in a table) and these documents are grouped together in collections。<br> 2. **可以理解为增强型的键值存储** <br> 3. 与键值存储最大的区别在于数据库能够理解并处理所存储的值（即文档）  | MongoDB， CouchDB | 适用于持久化存储，用来存放**不经常更改**的数据，作为关系型数据库的一般替代方案 |
| Wide-Column Databases  |  1. Concept: column, super column, column families, super column families. <br> 2. 本质是二维MAP。<br> 3. 高性能以及良好的扩展性 | Cassandra, HBase   | 适用于非常大的数据集，被 Twitter、Facebook 等社交网络用来存储海量用户所产生的数据 |
| Graph | 1. These databases are used to store data whose **relations** are best represented in a graph <br> 2. 数据基于图来建模 <br> 3. 图中每个节点代表一条记录，每条边表示节点之间的关系   | Neo4J  | 描述复杂关系的场景

## 怎么用？

使用no-sql的话，

- 就把一部分的工作从数据库转移到了应用层面。
- 应用层更容易横向扩展，这种转移有助于提升系统的可扩展性

## 有什么问题

### basic difference

| Type      | SQL | No-SQL |
| ----------- | ----------- | ----------- |
| Schema | fixed  | Dynamic  |
| Querying | SQL | Different syntax for using UnSQL(Unstructured Query Language) |
| Scalability  | mostly Vertically  | horizontally |
| Reliability/ACID? | Good | Not good |

### 用哪种

| Pro & Cons | SQL | No-SQL |
| ----------- | ----------- | ----------- |
| Advantage | 1. ensure ACID compliance <br> 2. 明确的扩展模式<br> 3. community support | 1. 易于扩展 <br> 2. 无需复杂的连表查询 <br> 3. 与OOP一致，易于使用 <br> 4. 无需预先定义，修改成本低 <br> 5. 读写性能高， 大数据 |
| Disadvantage | 1.复杂的连表查询导致数据读取性能不佳 <br> 2. 不方便扩展 <br> 3. 关系模型和OOP有差异 <br> 4. 只支持存取结构化数据，关系模式（如表结构）必须预先定义，并且修改成本高 | 1.缺乏强一致性的保证 <br> 2. less community support |

NoSQL 数据库适用于：

> - **快速变化的**数据，如点击流（click stream）数据或日志数据
> - 排行榜或评分数据
> - 临时数据，如购物车数据
> - 频繁访问的热点数据
> - 元数据（metadata），以及查找表（lookup tables）

快速增长的数据？

## Ref

- <https://www.acodersjourney.com/>
- <http://www.ayqy.net/blog/nosql/10-questions-to-ask-yourself-before-choosing-a-nosql-database/>>
- <https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/data-store-overview>
- <https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/data-store-decision-tree>
- <https://docs.microsoft.com/en-us/azure/architecture/guide/technology-choices/data-store-considerations>
- <http://www.ayqy.net/blog/nosql/>
- <https://blog.mlab.com/2012/08/why-is-mongodb-wildly-popular/>
- <https://stackoverflow.com/questions/8729779/why-nosql-is-better-at-scaling-out-than-rdbms>

# Index

Catalog 目录，
一个B树。m阶多叉树
写一个B树就基本可以理解index是什么，底层怎么做，以及好处坏处。

## Index decrease write performance

When CRUD happen, index are also need to be updated.

    If the goal of the database is to provide a data store that is often written to and rarely read from, in that case, decreasing the performance of the more common operation, which is writing, is probably not worth the increase in performance we get from reading.

## Ref

- <https://en.wikipedia.org/wiki/B-tree>
- <https://docs.microsoft.com/en-us/sql/t-sql/statements/create-index-transact-sql?view=sql-server-ver15>
- <https://docs.microsoft.com/en-us/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described?view=sql-server-ver15>

# Redundancy and Replication

## What is?

redundancy

- duplication of critical components or functions of a system
- with the intention of increasing the reliability of the system,
- usually in the form of a backup or fail-safe, or to improve actual system performance

Replication

sharing information to ensure **consistency** between redundant resources

复制不是单纯的复制数据库，更重要的是，这里面存在的数据同步机制，来确保数据库的一致性。

> 数据库与应用服务最大的区别在于，应用服务可以是无状态的（或者可以将共享状态抽离出去，比如放到数据库），而数据库操作一定是有状态的，在扩展数据库时必须要考虑数据的一致性

## 用来做什么？

提升可靠性，

包括性能？性能提高体现在哪里？

## 怎么做？

### 结构

- 一主多从
- 多主多从
- 无主多从

### 方法

- 异步复制
  - 优势：无需等待复制完成，性能没有太大影响
  - 劣势:
    - 无法保证强一致性
    - 数据丢失
  
- 同步复制
  - 优势
    - 强一致性
  - 劣势
    - 一挂全挂
    - 性能问题

- 半同步复制



## Quorum

# Data partitioning

## 是什么？

将单库分区。

## 为什么？

- 单库存在读写查找上的性能瓶颈。
- 即使复制方案下，基于每一个数据库都有一份完整数据的情况，性能瓶颈依然存在。还在限制系统的扩展性和性能。
  - 容量有限。数据量大到单库无法容纳
  - 性能有限。查询更新慢

## 怎么做？如何分区？

### Partitioning Methods

#### Horizaontal Partitioning -> **AKA Sharding(分片)**

它是：

- put different rows into different tables.
- Also Known as **data sharding**
- 每一片都是原数据的一个子集

它怎么好：

- schema保持不变，每一片的表结构完全一致
- 减少了资源争用，有助于提升性能

它怎么不好：

- 要合理地选择分区的key --> shared key, 来确保流量可以被尽可能**均匀**的分散到每一个片上。
- 这里并不是数据量，而是流量

#### Vertical Partitioning

它是：

- partitioning by columns

它怎么好：

- 根据不同的业务需求，可以把信息分的更细。
  - 比如，不太经常变化的数据，放到缓存里。
  - 照片这种大型内容拆出去单独存放：instagram? 等等
  - 或者对部分敏感的列拆出来，做更有针对性的安全控制。
  - 降低访问量。

它怎么不好：

- 即使是垂直分区，当业务量不断增长的时候，可能也总有一天要再次把这种“feature specific DB”再次分到多个不同的server上。
  - E.g it is not be possible for a single server to handle all the metadata queries for 10 billion photos by 140 million users

#### 按功能分区

When it's possible to identify a **bounded context** for each distinct business area in an application, functional partitioning is a way to improve isolation and data access performance.

Another common use for functional partitioning is to separate **read-write data** from **read-only data**.

#### 补充：Directory-Based partitioning

This is lookup service that knows your current partitioning scheme and abstracts it away from the DB access code

### 如何insert Data? -> Partitioning Criteria

When insert data, based on which criteria that we should follow to insert this record into relevant partitioning

#### Key or hash-based Partitioning

- hashing: hash(ID) % DB_count
- consistant hashing

#### List Partitioning

each partition is assigned a list of
values, so whenever we want to insert a new record, we will see which
partition contains our key and then store it there.

#### Round-Robin

i mod n

#### Composite Partitoning

Under this scheme, we combine any of the above partitioning schemes to devise a new scheme.

## 为什么好？为什么不好？Common Problems of Data Partitioning ?

### 怎么好？

- Improve scalability
- Performance
- Security
- Operational flexibility
- Match the data store to the pattern of use
  - or example, large binary data can be stored in blob storage, while more structured data can be held in a document database
- availability
  
### 怎么不好？

大多数都是由于跨分区查询操作造成的

#### join and denormalization

什么是范式？三大总则。

>1NF：第一范式（First normal form）要求数据表中每个字段的值都不可再分

>2NF：第二范式（Second normal form）在满足 1NF 的基础上，要求所有非主属性都完全依赖于其主键

>3NF：第三范式（Third normal form）在满足 2NF 的基础上，要求所有非主属性都不传递依赖于任何主键

<https://segmentfault.com/a/1190000013695030>

以及什么是反范式化

<http://www.ayqy.net/blog/database-denormalization/>

### Referential Integrity

Enforce data intergirty constraints such as foreign keys in a partitioned database can be extremely difficult

### Rebalancing

when there is more DB partitions, we sometimes have to consider rebalancing issues.
This require downtime. A directory-based partitioning could help, but the node of this one can also be a Single point of failure.

ref:

- <https://docs.microsoft.com/en-us/azure/architecture/best-practices/data-partitioning>
- <http://www.ayqy.net/blog/database-partitioning/>

# Consistant hashing

# Consistency  

## 是什么？

什么是一致性？

- 弱一致性（最弱）
  - 写完不一定能读到, 比如游戏，视频，网络电话。
- 最终一致性
  - 写完，异步复制，最终可以读到，比如Email, DNS
  - 通常在highly available的系统中
- 强一致性 最强
  - 写完，立刻同步，立即能读到，文件系统。

# CAP Theorem

## 是什么？

- C: consistency -> all nodes see the same data at same time or error
  
  每次读取都能得到最新写入的结果，**抑或出错**
- A: Availability -> every request receive by non-failing node must result in a response, but without guarantee that it contains the most recnet version of the information
  
  每个请求都能收到正常响应，但不保证返回的是最新信息
- P: Partition -> communication break (or a network failure) between any two nodes in the system
  
  即便有一部分由于网络故障down掉了，系统仍能继续运行

## 为什么？

系统设计中也面临许多权衡取舍(与第一节相对)：

- 性能与可扩展性 Performance and scalability
  - 服务可以通过加资源的方式成比例的提升性能
  - 但资源也会引入多样性。新旧节点的不同引起的异质性会影响性能
- 延迟与吞吐量 latency and throughput
  - 吞吐量是指单位时间内所能处理的操作数
  - 无法兼具低延迟和高吞吐量，所以可能权衡的原则是：在一定延迟的情况下，尽可能追求最大的吞吐量
- 可用性与一致性 availiblity and consistency
  - 即本节要讨论的内容

## 怎么做

CAP is a "three choices, pick two" scenario. In the presence of a network partition,a distributed system must choose either Consistency or Availability:

- choose consistency: 当网络不完全可靠，出现故障，取消操作，保证一致性，但会降低可用性，用户可能会收到超时错误
  - > Waiting for a response from the partitioned node might result in a timeout error. CP is a good choice if your business needs require **atomic reads and writes**.
  - > An atomic operation is, by definition, a single operation that either succeeds completely or fails completely
- choose availability: 保证可用，返回旧信息。但是这不是一致的
  - > Responses return the most readily available version of the data available on any node

## 有什么问题？

当没有network partitions的时候怎么选？
其次，保证可用的情况下，如果需要很长时间返回，虽然可用，但是业务可能无法接受。
大部分情况下，分区是平稳运行的，并不会出错，这时候该怎么选？

## ref

<https://www.donnywals.com/what-does-atomic-mean-in-programming/>

# PACELC Theorem

## 是什么

P(partition) + A(Availability) + C(Consistency) + Else(no partition) + L(Latency) + C(Consistency)

## 为什么

实践指导理论，理论指导实践

## 怎么做？

> Dynamo and Cassandra are PA/EL systems:
> They choose availability over consistency when a partition occurs; otherwise, they choose lower latency.
>
> BigTable and HBase are PC/EC systems:
> They will always choose consistency, giving up availability and lower latency
>
> MongoDB can be considered PA/EC (default configuration)

不同的数据库可以实现不同的配置

Ref:
<https://cloud.tencent.com/developer/article/1811555?from=article.detail.1585052>

# Leader and Follower

> HeartBeat mechanism is used to detect if an existing leader has failed, so that new leader
> election can be started

> The server which receives votes from the majority of the servers, transitions to leader
> state. The majority is determined as discussed in Quorum. Once elected, the leader
> continuously sends HeartBeat to all the followers. If followers do not get a heartbeat in
> specified time interval, a new leader election is triggered.

# Heartbeat

## What is?

Each server periodically sends a hearbeat message to a central monitoring server or other servers in the system to show that it is still alive and functioning

## how

> Heartbeating is one of the mechanisms for detecting failures in a distributed
system. If there is a central server, all servers periodically send a heartbeat
message to it.

> If there is no central server, all servers randomly choose a set
of servers and send them a heartbeat message every few seconds. This way,
if no heartbeat message is received from a server for a while, the system can
suspect that the server might have crashed.
If there is no heartbeat within a
configured timeout period, the system can conclude that the server is not
alive anymore and stop sending requests to it and start working on its
replacement.

## Why

# Client-Sever Communication --> Long-Polling vs WebSocket vs Server-Sent events

## What is

A standard HTTP web request:

- Client open a connection and request data from the server
- The server calculate the response
- The sever sends the response back to the client on the opened request

## Why

## How and What's good and What's bad?

### Ajax Polling

What is Akax?

- Ajax is AsynchronousJavascript + XML/json
- 只要是JS调用异步通讯组件并使用格式化的数据来更新web页面上的内容或操作过程，那么我们用的方法就可算是AJAX
- Ajax is the technique concept. not a framework or somethinge else

Ajax polling is the client periodically/repeatly sending request to server, asking for new Data.

This way will create **HTTP** overhead since it just keep asking server for any new data

### **HTTP** long-polling

Connection is initiate by client, and it is there untill server has an update or timeout

Hanging Get:

> - If the server does not have any data available for the client, instead of
> sending an empty response, the server holds the request and waits until
> some data becomes available or timeout period reached.
>
> - Once the data becomes available, a full response is sent to the client.
> The client then immediately **re-request** information from the server so
> that the server will almost always have an available waiting request
> that it can use to deliver data in response to an event

### Web Socket

A full duplex(双工) communication**channel over a single TCP connection**. Both **server and client can send data at anytime.**
  
- Bi-direction
- Without ask
- low communication overhead and real-time data transfer

Scenario(mupltple user communication & server-side data constantly changing):

- Social feed
- Chat apps
- Tacking APP
- live audience

### SSE

Initiated by client, once established, it will be there. And **direction is from server to client.**

If client wants to send data to server, it would require to use another tech.

- The client requests data from a server using regular **HTTP**.
- The requested webpage opens a connection to the server.
- The server sends the data to the client whenever there’s new information available.

Scenario(best when we need real-time traffic from the server):

- Facebook update. News feed. Push Notification

## ref

<https://medium.com/must-know-computer-science/system-design-client-server-communication-674818ca448d>

# Bloom filter

It is a data strcutre.

A Bloom filter is a data structure designed to tell you, rapidly and memory-efficiently, whether an element is present in a set.

The price paid for this efficiency is that a Bloom filter is a probabilistic data structure: it tells us that the element **either definitely is not in the set** or **may be** in the set.

<https://llimllib.github.io/bloomfilter-tutorial/>

# Checksum

## what is?

An error-detection method

## why?

When moving data between components(across network, storage, software), it is possible that the data fetched from a node may arrived corrupted.

This is to ensure data integrity.

## how to do?

- Hash fuction: with some algorithm, take the input data and produce a string
- When a system is storing some data, it computes a checksum of the data and store the checksum **along with the data.**

# Step that should be follow for System Design Interviews

## Requirement Clarification

- Who is going to use it?
- How are they going to use it?
- How many users are there?
- What does the system do?
- What are the inputs and outputs of the system?
- How much data do we expect to handle?
- How many requests per second do we expect?
- What is the expected read to write ratio?

- 从你的认知展开问
- 从你所掌握的system design的知识点展开（有点倒推的意思）

可以总结出一个cheatsheet：

- 不同的指标下，有什么样的固定模式
  - 比如 highly available 应该从哪些细节设计实现
  - 比如 latency要是少 应该从哪些细节设计实现

明确出：

- 功能
- 存什么
- 前后端？还是只有后端？

## Back-of-the-envelope(粗略的) estimation

- What **scale** is expected from the system ?
- How much **storage** will we need?
  - Are photos videos included?
- What network bandwidth usage are we expecting? This is critical for deciding how we will manage:
  - traffic
  - balance load

## System interface definition

- Define API according to function you want to achieve(This is something that you should have figure out in the step: Requirement Clarification)

## Define data model

- identify various system entities,
  - 在这个系统中有什么样的主体，有点像是面向对象，比如Twitter：有用户，tweets。
- how they will interact with each other,
- and different aspects of data management like storage, transportation, encryption, etc.

## High-level design

draw the diagram, We should identify enough components that are needed to solve
the actual problem from **end to end.**

本质是读写

## Detailed design

Design core part. For example, for url shortening service, discuss:

- Generating and storing a hash of the full url
  - MD5 and Base62
  - Hash collisions
  - SQL or NoSQL
  - Database schema
- Translating a hashed url to the full url
  - Database lookup
- API and **object-oriented design**

Algorithm, DB, Corner case. 注意tradeOFF.

## Identifying and resolving bottlenecks

- Load balancer
- Horizontal scaling
- Caching
- Database sharding

这部分涉及什么？

# System design blog I follow

- <https://www.educative.io/courses/grokking-the-system-design-interview>
- <https://medium.com/must-know-computer-science>
- <http://www.ayqy.net/blog/category/back-end/>
- <https://github.com/donnemartin/system-design-primer>
