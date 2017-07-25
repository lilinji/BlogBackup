# 第三届阿里中间件性能挑战赛总结

## 喜讯

rapids团队(右二：王立鹏，右三：车煜林)获得季军。

![third place photo](../Pictures/CompSummary/third_place.jpg)

rapids团队(左一：王立鹏，左二：车煜林)与评测环境负责人(左三：万少)合影。

![photo](../Pictures/CompSummary/rapids_and_wanshao.jpg)

## 比赛介绍

比赛分为初赛(有关消息引擎)，复赛(有关数据库重放和范围查询)，最终答辩。其中，top100队伍进入复赛,复赛top10队伍进入最终答辩。

初赛和复赛考察了选手算法设计, 工程实现，性能调优的能力，要求选手对消息引擎和数据库重放和范围查询有比较深入的理解，并且能够分析出系统的性能瓶颈进行针对性的优化。

比赛的奖金非常诱人。

![stimulus](../Pictures/CompSummary/contest_stimulus.png)

比赛的评委非常权威, 对中间件，数据库有深入理解。

![researchers](../Pictures/CompSummary/alibaba_researchers.png)
![experts](../Pictures/CompSummary/alibaba_experts.png)

最终进入答辩top10队伍与评委以及组织者进行了合影。

![members](../Pictures/CompSummary/members_photo0.jpg)

此外，此次比赛引入了额外的24小时极客pk赛(考察分布式数据库的`top k,n`查询)。极客pk赛的激励也非常诱人，top10队伍的选手们在24h内，竞争3个西行游学名额。

![extra stimulus](../Pictures/CompSummary/extra_stimulus.png)

## 初赛介绍

### 赛题要求

在单机环境下，实现带有持久化的消息队列引擎，由第一个生产进程产生消息队列，并在最终调用`Producer`的`flush()`接口时候，保证安全地持久化到磁盘；之后由另一个消费进程进行消息队列的消费。消息会对应到一个命名空间，例如`Topic0`，`Queue0`这样的字符串。对于同一个`Producer`产生的命名空间的消息需要保证顺序性，也就是说由`(Producer_j, Name_i)`唯一标识了一个队列，这个队列中的消息在消费的时候需要保证顺序性。

题目要求实现四个`Producer`有关的接口:`createBytesMessageToTopic(topic, body)`, `createBytesMessageToQueue(queue, body)`, `send(message)` , `flush()`和两个`Consumer`有关的接口`attachQueue(queue, topics)`，`poll()`。并且，如果不同`Consumer`绑定到了同一个命名空间，这些`Consumer`看到是对应命名空间的视图，需要全部消费完命名空间中的消息，并且大家消费的内容是相同的。题目中，产生出的消息总量为40,000,000条。

### 解题思路

题目中，产生出的消息总量为40,000,000条。如果在文件中不经过压缩，存所有信息所需的磁盘空间是4GB左右。但是测试环境的IO速度很不理想，因此选手需要充分利用linux 的 pagecache来解决IO瓶颈。

根据 https://lonesysadmin.net/2013/12/22/better-linux-disk-caching-performance-vm-dirty_ratio/ 和 https://stackoverflow.com/questions/27900221/difference-between-vm-dirty-ratio-and-vm-dirty-background-ratio ，我们可以知道，在linux中，pagecache sync相关有两个主要参数 `vm.dirty_background_ratio`(默认为10%)，`vm.dirty_ratio`(默认为20%)，当dirty page的大小达到了总物理内存大小的10%时，linux操作系统会进行刷盘但不阻塞`fwrite`系统调用的写线程，但若dirty page的大小达到了物理内存大小的20%的时候，写线程就会被阻塞。

基于此，我们可以想到的优化策略是减小最终消息队列序列化到文件的大小，这个可以通过压缩算法达到，例如jdk中带有的`GZIPOutputStream`，`DeflaterOutputStream`, 和其他的压缩方法实现`SnappyFramedOutputStream`, `LZ4BlockOutputStream`。其他的选手也有通过统计消息的特征进行非通用的存储设计来减小最终文件的大小，利用的是相同的思路。

然后针对本题目，文件的存储方式对于最终的成绩影响并不大，我们选择的存储方式是：每个名字一个文件夹，例如`Topic0`一个文件夹；在文件夹下面，每个文件是每个线程的名字(因为一个线程绑定一个`Producer`)，例如`Thread-0`。这样的存储设计较为简单，也不需要频繁加锁，读写锁的使用当且仅当文件夹创建的时候；并且所有文件夹创建之后(命名空间确定之后)就不会有读者-写者冲突。`(Producer_j, Name_i)`唯一标识的队列对应了一个唯一的文件，该文件访问不需要加锁。

当使用压缩算法时候，我们会发现jdk中带有的`GZIPOutputStream`，`DeflaterOutputStream`的压缩速率不太理想，因此最终我们选择了`SnappyFramedOutputStream`这个广泛应用的压缩算法，做一个压缩率和CPU占用的权衡。

对于读取的反序列化，我们做了一个实现上的优化，只要进行一次线性访问就可以获取出对应的属性，而不是通过正则表达式的`split`进行三次线性扫描。具体实现可见 https://github.com/CheYulin/OpenMessageShaping/blob/master/src/main/java/io/openmessaging/demo/DefaultBytesMessage.java 中 `public String toString()`和`static public DefaultBytesMessage valueOf(String myString)`这两个序列化和反序列化的方法。

其他选手还考虑了使用一些 hard coding的trick来进一步减小磁盘IO,因为理论上分析 `4G * 0.2 = 0.8G`，只要能够使得最终的输出达到 `0.8G` 的大小的话，磁盘IO对于写线程来说总是异步的，因为写线程只是进行了内存的拷贝。我们最终没有选择进行hard coding，需要写磁盘的大小为 `2.4G`，最终的成绩为: 生产耗时 `73.952s`， 消费耗时 `35.995s`， TPS为 `363811`， 对应代码的版本为 https://github.com/CheYulin/OpenMessageShaping/tree/296f94e3b63f5d286d421a4ae180dd3af63be3b1 。

## 复赛介绍

### 赛题要求

进行数据库(从空开始)的主从增量同步，输入为10G顺序append的日志文件(只能单线程顺序读取1次，模拟真实流式处理场景)。日志文件中包含insert key(带有全部属性)/delete key/update property(单个)/update key这四种类型的数据变化操作(通过I/D/U以及其他日志中的字段标识)。其中，10G数据在server端，最后的查询结果需要落盘在client端，server和client是两台配置相同的16逻辑cpu核数的阿里云虚拟机。

赛题数据为单表的日志，并且有字段的长度有确定的范围，选手可以基于此进行优化。赛题最后的符合要求range在(1,000,000, 8,000,000)，最后产生符合的条数占比约为1/7，byte[]大小大约38MB,因此网络带宽不会成为系统瓶颈。比赛测试环境为16个logical CPUs，因此需要解决并发的一些问题，充分利用并行，来获取极致的性能。所有输入文件在ramfs中，顺序单线程访问所需理论时间2.5s，因此需要选手能够比较好地通过计算流水线，overlap处理，计算和IO(通过ramfs)。

#### 解题思路

文件读取通过mmap，减少内核态和用户态拷贝。建立好的流水线，处理好重放计算时候需要保证的顺序性，并且需要充分将IO和计算overlap在一起。并行eval产生byte[]，网络传输和落盘采用Zero-Copy，以减少内核态和用户态拷贝的开销。

数据库重放时候如果利用数据update key不会带来原来属性的特征，可以使用数组表示符合范围的数据库记录，否则可以通过hashmap加hashset进行表示。使用通用策略的时候，重放计算的内存占用为数据库的大小；使用针对题目专用策略时候，重放计算的内存占用为数组大小。数据库重放时候，若采用通用策略，则重放计算瓶颈在于内存占用和hashmap的index probing计算。

## 比赛感想和收获

### 比赛感想

本次比赛相当地激烈刺激。在初赛的比赛过程中，我们rapids团队开始做得比较晚，并且开始阶段一直没抓出比赛考察pagecache的这个点，做了许多对初赛题目无用的文件布局的优化。但我们没有放弃，在比赛的最后一天抓到了性能优化要点：即通过压缩，减少写盘大小，利用起pagecache来。在最后一天从100名开外到80名，再通过工程上的性能调优以及选择更高效的压缩算法snappy，最终升至32名，成功进入复赛。

复赛的过程相比初赛要顺利一些。但是整个过程一样惊心动魄，我们经历了好几次进入top10又被超越的情况。但是，我们顶住了压力，在最后一晚上保住了top10（复赛成绩第6名），成功晋级答辩环节。

### 技术总结

经过初赛，我们明白了在rocketmq设计中pagecache的重要性，只有把文件读写的size压缩到pagecache对应大小才可以取得比较好的性能，学习到了一些快速的压缩算法，例如snappy和lz4。

复赛中，我们明白了内存访问的pattern的重要性，通过`byte[][]`的方式访问会比较慢，而通过扁平化的存储会比较访存友好。在实际应用中，需要针对具体的数据库表字段进行相应的优化。并且，我们认识到了在java中，默认的`extends Object`会带来将近8byte的开销，并且jvm会进行8byte内存对齐，在设计小对象的时候要格外小心。

复赛中，另一个考察点是计算和IO的overlap，我们需要设计出来合适的并行计算流水线，解决一些相关的同步和并发控制问题，来取得比较好的性能。另外hashmap和hashset的具体内存layout对于性能的影响也非常大，java自带的基于拉链和转化拉链为红黑树的组织形式开销会比较大，hashmap的probing方式对于性能影响也很大。我们最终选择了从trove hashmap开地址的方式进行修改，利用了trove hashmap中基于knuth书本上的probing方式，可以取得不错的性能。对应不作任何取巧的通用版本也可以在线上跑到8.9s的成绩。

### 技术展望

在之后的开发中，也需要注意操作系统和语言底层vm实现相关的内容，才能取得比较理想的性能。我们通过阿里中间件的博客学习到了许多新的知识，之后需要多多关注业界的技术热点，从而自己提高工程能力，并且也可能从中找出数据库领域值得研究并且未被研究透的问题。

## 相关补充资料

比赛攻略可见

https://github.com/CheYulin/IncrementalSyncShaping/tree/master/comp_summary

初赛相关代码可见

https://github.com/CheYulin/OpenMessageShaping/tree/master/src/main/java/io/openmessaging/demo

复赛相关代码可见下表

版本 | 链接
--- | ---
trick统计信息获取版本 | https://github.com/CheYulin/IncrementalSyncShaping/tree/specialStatistics
trick版本 |  https://github.com/CheYulin/IncrementalSyncShaping
通用版本 | https://github.com/CheYulin/IncrementalSyncShaping/tree/generalImpl8.9s
