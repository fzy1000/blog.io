---
title: kafka设计与原理详解
date: 2021-06-09 18:51:19
tags:
- Kafka
- mq 
category: Kafka
---


<h2>Kafka技术概览</h2>

<h3>Kafka的特性</h3>
1. 高吞吐量、低延迟：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒</br>
</br>
2. 可扩展性：kafka集群支持热扩展</br>
</br>
3. 持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失</br>
</br>
4. 容错性：允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）</br>
   </br>
5. 高并发：支持数千个客户端同时读写</br>

<h3>Kafka一些重要设计思想</h3>
下面介绍先大体介绍一下Kafka的主要设计思想，可以让相关人员在短时间内了解到kafka相关特性，如果想深入研究，后面会对其中每一个特性都做详细介绍。

+ Consumergroup：各个consumer可以组成一个组，每个消息只能被组中的一个consumer消费，如果一个消息可以被多个consumer消费的话，那么这些consumer必须在不同的组。

+ 消息状态：在Kafka中，消息的状态被保存在consumer中，broker不会关心哪个消息被消费了被谁消费了，只记录一个offset值（指向partition中下一个要被消费的消息位置），这就意味着如果consumer处理不好的话，broker上的一个消息可能会被消费多次。

+ 消息持久化：Kafka中会把消息持久化到本地文件系统中，并且保持极高的效率。

+ 消息有效期：Kafka会长久保留其中的消息，以便consumer可以多次消费，当然其中很多细节是可配置的。

+ 批量发送：Kafka支持以消息集合为单位进行批量发送，以提高push效率。

+ push-and-pull : Kafka中的Producer和consumer采用的是push-and-pull模式，即Producer只管向broker push消息，consumer只管从broker pull消息，两者对消息的生产和消费是异步的。

+ Kafka集群中broker之间的关系：不是主从关系，各个broker在集群中地位一样，我们可以随意的增加或删除任何一个broker节点。

+ 负载均衡方面： Kafka提供了一个 metadata API来管理broker之间的负载（对Kafka0.8.x而言，对于0.7.x主要靠zookeeper来实现负载均衡）。

+ 同步异步：Producer采用异步push方式，极大提高Kafka系统的吞吐率（可以通过参数控制是采用同步还是异步方式）。

+ 分区机制partition：Kafka的broker端支持消息分区，Producer可以决定把消息发到哪个分区，在一个分区中消息的顺序就是Producer发送消息的顺序，一个主题中可以有多个分区，具体分区的数量是可配置的。分区的意义很重大，后面的内容会逐渐体现。

+ 离线数据装载：Kafka由于对可拓展的数据持久化的支持，它也非常适合向Hadoop或者数据仓库中进行数据装载。

+ 插件支持：现在不少活跃的社区已经开发出不少插件来拓展Kafka的功能，如用来配合Storm、Hadoop、flume相关的插件。


<h3>kafka 应用场景</h3>
1. 日志收集：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等</br>
</br>
2. 消息系统：解耦和生产者和消费者、缓存消息等。</br>
   </br>
3. 用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。</br>
   </br>
4. 运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。</br>
   </br>
5. 流式处理：比如spark streaming和storm</br>
   </br>
6. 事件源</br>
   </br>

<h3>kafka Kafka架构组件</h3>

Kafka中发布订阅的对象是topic。我们可以为每类数据创建一个topic，把向topic发布消息的客户端称作producer，从topic订阅消息的客户端称作consumer。Producers和consumers可以同时从多个topic读写数据。一个kafka集群由一个或多个broker服务器组成，它负责持久化和备份具体的kafka消息。

+ topic：消息存放的目录即主题</br>
  </br>
+ Producer：生产消息到topic的一方</br>
  </br>
+ Consumer：订阅topic消费消息的一方</br>
  </br>
+ Broker：Kafka的服务实例就是一个broker</br>
  </br>

<img src="/blog.io/img/kafka结构图.png">

</br>
<img src="/blog.io/img/kafka架构.png">

<h3>Kafka Topic&Partition</h3>

消息发送时都被发送到一个topic，其本质就是一个目录，而topic由是由一些Partition Logs(分区日志)组成,其组织结构如下图所示：

<img src="/blog.io/img/Topic结构组织.png">

我们可以看到，每个Partition中的消息都是有序的，生产的消息被不断追加到Partition log上，其中的每一个消息都被赋予了一个唯一的offset值。
Kafka集群会保存所有的消息，不管消息有没有被消费；我们可以设定消息的过期时间，只有过期的数据才会被自动清除以释放磁盘空间。比如我们设置消息过期时间为2天，那么这2天内的所有消息都会被保存到集群中，数据只有超过了两天才会被清除。
Kafka需要维持的元数据只有一个–消费消息在Partition中的offset值，Consumer每消费一个消息，offset就会加1。其实消息的状态完全是由Consumer控制的，Consumer可以跟踪和重设这个offset值，这样的话Consumer就可以读取任意位置的消息。
把消息日志以Partition的形式存放有多重考虑，第一，方便在集群中扩展，每个Partition可以通过调整以适应它所在的机器，而一个topic又可以有多个Partition组成，因此整个集群就可以适应任意大小的数据了；第二就是可以提高并发，因为可以以Partition为单位读写了。

<h2>Kafka 核心组件</h2>

 <h3>Replications、Partitions 和Leaders</h3>

通过上面介绍的我们可以知道，kafka中的数据是持久化的并且能够容错的。Kafka允许用户为每个topic设置副本数量，副本数量决定了有几个broker来存放写入的数据。如果你的副本数量设置为3，那么一份数据就会被存放在3台不同的机器上，那么就允许有2个机器失败。一般推荐副本数量至少为2，这样就可以保证增减、重启机器时不会影响到数据消费。如果对数据持久化有更高的要求，可以把副本数量设置为3或者更多。
Kafka中的topic是以partition的形式存放的，每一个topic都可以设置它的partition数量，Partition的数量决定了组成topic的log的数量。Producer在生产数据时，会按照一定规则（这个规则是可以自定义的）把消息发布到topic的各个partition中。上面将的副本都是以partition为单位的，不过只有一个partition的副本会被选举成leader作为读写用。
关于如何设置partition值需要考虑的因素。一个partition只能被一个消费者消费（一个消费者可以同时消费多个partition），因此，如果设置的partition的数量小于consumer的数量，就会有消费者消费不到数据。所以，推荐partition的数量一定要大于同时运行的consumer的数量。另外一方面，建议partition的数量大于集群broker的数量，这样leader partition就可以均匀的分布在各个broker中，最终使得集群负载均衡。在Cloudera,每个topic都有上百个partition。需要注意的是，kafka需要为每个partition分配一些内存来缓存消息数据，如果partition数量越大，就要为kafka分配更大的heap space。


<h3>Producers</h3>

Producers直接发送消息到broker上的leader partition，不需要经过任何中介一系列的路由转发。为了实现这个特性，kafka集群中的每个broker都可以响应producer的请求，并返回topic的一些元信息，这些元信息包括哪些机器是存活的，topic的leader partition都在哪，现阶段哪些leader partition是可以直接被访问的。
Producer客户端自己控制着消息被推送到哪些partition。实现的方式可以是随机分配、实现一类随机负载均衡算法，或者指定一些分区算法。Kafka提供了接口供用户实现自定义的分区，用户可以为每个消息指定一个partitionKey，通过这个key来实现一些hash分区算法。比如，把userid作为partitionkey的话，相同userid的消息将会被推送到同一个分区。
以Batch的方式推送数据可以极大的提高处理效率，kafka Producer 可以将消息在内存中累计到一定数量后作为一个batch发送请求。Batch的数量大小可以通过Producer的参数控制，参数值可以设置为累计的消息的数量（如500条）、累计的时间间隔（如100ms）或者累计的数据大小(64KB)。通过增加batch的大小，可以减少网络请求和磁盘IO的次数，当然具体参数设置需要在效率和时效性方面做一个权衡。
Producers可以异步的并行的向kafka发送消息，但是通常producer在发送完消息之后会得到一个future响应，返回的是offset值或者发送过程中遇到的错误。这其中有个非常重要的参数“acks”,这个参数决定了producer要求leader partition 收到确认的副本个数，如果acks设置数量为0，表示producer不会等待broker的响应，所以，producer无法知道消息是否发送成功，这样有可能会导致数据丢失，但同时，acks值为0会得到最大的系统吞吐量。


若acks设置为1，表示producer会在leader partition收到消息时得到broker的一个确认，这样会有更好的可靠性，因为客户端会等待直到broker确认收到消息。若设置为-1，producer会在所有备份的partition收到消息时得到broker的确认，这个设置可以得到最高的可靠性保证。
Kafka 消息有一个定长的header和变长的字节数组组成。因为kafka消息支持字节数组，也就使得kafka可以支持任何用户自定义的序列号格式或者其它已有的格式如Apache Avro、protobuf等。Kafka没有限定单个消息的大小，但我们推荐消息大小不要超过1MB,通常一般消息大小都在1~10kB之前。

<h3>Consumers</h3>

Kafka提供了两套consumer api，分为high-level api和sample-api。Sample-api 是一个底层的API，它维持了一个和单一broker的连接，并且这个API是完全无状态的，每次请求都需要指定offset值，因此，这套API也是最灵活的。
在kafka中，当前读到消息的offset值是由consumer来维护的，因此，consumer可以自己决定如何读取kafka中的数据。比如，consumer可以通过重设offset值来重新消费已消费过的数据。不管有没有被消费，kafka会保存数据一段时间，这个时间周期是可配置的，只有到了过期时间，kafka才会删除这些数据。
High-level API封装了对集群中一系列broker的访问，可以透明的消费一个topic。它自己维持了已消费消息的状态，即每次消费的都是下一个消息。
High-level API还支持以组的形式消费topic，如果consumers有同一个组名，那么kafka就相当于一个队列消息服务，而各个consumer均衡的消费相应partition中的数据。若consumers有不同的组名，那么此时kafka就相当与一个广播服务，会把topic中的所有消息广播到每个consumer。

<img src="/blog.io/img/consumer-groups.png">






