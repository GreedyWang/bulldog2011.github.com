---
layout: post
title: "Performance Benchmark - Luxun vs Apache Kafka"
date: 2013-04-19 18:11
comments: true
categories: [luxun] 
keywords: log collector, Apache Kafka, messaging, persistent queue
description: Performance Benchmark - Luxun vs Apache Kafka
---

[Luxun](https://github.com/bulldog2011/luxun) is a high-throughput, distributed, pub-sub messaging system tailored for big data collecting and analytics. Luxun is inspired by [Apache Kafka](http://kafka.apache.org/), both have a similar architecture. In order to compare the performance of Luxun and Kafka, I did a round of performance testing, below are the testing results:

<!--more-->

```

	Producing Throughput Comparison:
	
	  Async, Flush Disabled:    Luxun >> Kafka
	  Async, Flush Enabled:     Luxun >> Kafka
	  Sync, Flush Disabled:     Luxun >> Kafka
	  Sync, Flush Enabled:      Luxun < Kafak
	
	Consuming Throughput Comparison:
	
	  Luxun > Kafka

```


##Analysis & Conclusion
The overall performance of Luxun is ***much better*** than Kafka:
>1. In async producing mode, no matter whether flush is enabled or not, the throughput of Luxun is ***at least twice*** the throughput of Kafka.
2.  In sync producing mode, if flush is disabled, Luxun performs ***much better*** than Kafka.
3.  In sync producing mode, if flush is enabled, Luxun performs ***worse*** than Kafka.
4.  In all consuming tests, Luxun performs ***much better*** than Kafka.

The only lose case of Luxun is in sync producing mode when flush is enabled, Luxun uses [Memory Mapped File](http://en.wikipedia.org/wiki/Memory-mapped_file) internally, we are still not sure the cause of poor performance to explicitly flush memory mapped buffer in Java, this will be a future optimization of Luxun. However, following unique feature of memory mapped file makes explicit flush unnecessary(or not recommended) on Luxun system:

>* OS will ensure the message persistence even the process crashes and there is no explicit flush before the crash.
* Message appended by producer thread will be immediately visible to consumer threads, even producer thread hasn’t flushed the message explicitly.   

Also, the inner paging and swapping mechanism of Luxun will automatically flush a cached page when it is replaced out, making explicit flush unnecessary on Luxun system.

Regrading inner implementation, Luxun and Kafka have two main differences:   
>* Luxun queue is based on [Memory Mapped File](http://en.wikipedia.org/wiki/Memory-mapped_file) while Kafka queue is based on filesystem and OS page cache.
* Luxun leveraged [Thrift RPC](http://thrift.apache.org/) as communication layer while Kafka built its custom NIO communication layer.   

We believe the performance difference between Luxun and Kafak is mainly caused by memory mapped file, while Thrift RPC or custom NIO communication layer does not make much difference here.


Detailed performance test results can be found [here](https://github.com/bulldog2011/luxun/wiki/Performance---Luxun-vs.-Apache-Kafka)


