---
layout: post
title: "Luxun Performance Test"
date: 2013-04-20 14:41
comments: true
categories: [luxun] 
keywords: log collector, Apache Kafka, messaging, persistent queue
description: Luxun Performance Test Result Analysis & Conclusion
---

[Luxun](https://github.com/bulldog2011/luxun) is a high-throughput, persistent, distributed, publish-subscribe messaging system tailored for big data collecting and analytics. I did an intensive performance test to Luxun these days, detailed reports can be found [here](https://github.com/bulldog2011/luxun/wiki/Luxun-Performance), below is a list of findings:

<!--more-->

>1. The producing performance is quite good, on single sever grade machine, in one topic case, the average producing throughput is ***> 100MBps***.
2. The consuming performance is also quite good, on single server grade machine, in one topic case, the average consuming throughput is ***> 100MBps***. 
3. On single machine case, Luxun performance is only limited by disk IO bandwidth, it's recommended to use Luxun with ultra fast disk for better performance.
4. In networking case, Luxun performance is only limited by network bandwidth, it's very easy for Luxun to saturate a 1Gbps network, it's recommended to use Luxun in 10Gbps network, or enable compression for better network usage.
5. Luxun is ***not sensitive to*** JVM heap memory setting, our tests show there is not much difference whether additional heap memory is added or not, this is because Luxun queue is based on memory mapped file which does not use JVM heap memory directly, also Luxun uses automatic on-demand paging and swapping algorithm internally for efficient memory usage. However, this is not to say Luxun can work well on machine with very few memory, because memory mapped file will still use much off-heap memory, it's recommend to use Luxun broker on machine with at least ***4GB*** physical memory, more is better.
6. Even on a normal notebook, both producing and consuming throughput of Luxun is ***> 50MBps***.
7. The Luxun performance is similar on both Windows and Linux platforms.
8. Async batch producing performance is much better than sync producing, in some case, there is order of magnitude difference. Async batch mode has much less network roundtrip overhead, hence better performance. It's recommended to use async batch producing mode whenever possible.
9. Flush on broker side has much negative impact on Luxun performance, especially in sync producing case, it's recommended to ***disable flush*** on broker because of the unique nature of [Memory Mapped File](http://en.wikipedia.org/wiki/Memory-mapped_file) used internally by Luxun.
10. Message size has much impact on Luxun performance, bigger message, better performance, this is apparent since small message has more overhead caused by network roundtrip.
11. The one way not confirmed produce interface has much better performance than the two way confirmed produce interface, our tests show there are differences between the three times, so in the Luxun high level producer implementation, we chose one way not confirmed produce interface as underlying produce interface.
12. A performance comparison between Luxun and [Apache Kafka](http://kafka.apache.org/) shows Luxun has much better performance than Kafka, see detail in [this post](http://bulldog2011.github.io/blog/2013/04/19/performance-benchmark-luxun-vs-apache-kafka/). 
13. The overall performance of Luxun will not change much when the number of topics increase, the throughput will be shared among different topics.
14. Compression, when used appropriately, can improve the overall performance of Luxun a lot, our tests show Snappy is more efficient than GZip, although this depends on particular applications.


