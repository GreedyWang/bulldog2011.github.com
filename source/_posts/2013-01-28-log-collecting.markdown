---
layout: post
title: "Producing and consuming 4TB log daily on one commodity machine"
date: 2013-01-28 13:10
comments: true
categories: 
keywords: log collector, log collecting, persistent queue
description: show how to collect logs using big queue service and show performance test numbers.
---

I have built a [big queue](https://github.com/bulldog2011/bigqueue), and I have [turned it into a Thrift based queue service](http://bulldog2011.github.com/blog/2013/01/27/thrift-queue/), in this post, I will show you how to collect logs using this queue service, also, I will show some performance number to let you know the capability of big queue as a log collecting tool.

###Collecting and Consuming Logs using Thrift based Queue Serivce

Big queue was originally designed for log collecting and analysis scenario, it's very simple to collecting logs using big queue, all you need to do are:  

<!--more-->

>At producing/collecting side:  
	
		1. Generate Log accoding to your app logic  
		2. Serialize it into binary data  
        3. Put the data into big queue through Thrift RPC call. 

>At consuming/analysis side:  
	
		1. Pull data from big queue through Thrift RPC call  
        2. Deserialize data into log object  
        3. Analyze log according your real needs.

That's it.

Big queue only accepts binary data as input and will return binary data as output, so you are responsible for the serialization/deserialization part, I would recommend Thrift as as a serialization protocol since it is light-weight, high performance and cross-language, here is a simple log format I defined using Thrift IDL(Note: this is the bare minimum, you need to adapt and enhance according to your real needs):

		namespace java com.leansoft.logging.thrift
		
		// The level of log event
		enum LogLevel {
		  DEBUG = 0,
		  INFO = 1,
		  WARN = 2,
		  ERROR = 3,
		  FATAL = 4
		}
		
		struct LogEvent
		{
		    1: i64 createdTime,
		    2: string hostId,
		    3: LogLevel logLevel,
		    4: string message
		}


With log format defined, I can easily generate class for my target lanague, like Java or CSharp, then with TSerializer and TDeserializer helpers from Thrift, I can easilty serialize log object into binary data or deserialize binary data back into log object, this is typcial serialization use case of Thrift.

You many choose any other serialization framework(like protocol buffer, Avro, etc) you like, big queue is serialization independent, it only sees bytes without caring about content inside the bytes.

[Here](https://github.com/bulldog2011/bigqueue/tree/master/samples/thriftqueue/src/com/leansoft/thriftqueue/load) are the code I used to test the log collecting capability of big queue, you may find it useful.

By the way, this is just the bare minimum, in real world, you may build 'Agent' which encapsulates all details above and only expose simple interface to outside user for log collecting, inside the 'Agent', some optimizations like async batching and compression can further improve the performance of the log collector.

###Performance Test
I've did some basic performance test to validate the performance of my Thrift based queue service, the full hardware spec for server is [here](http://bulldog2011.github.com/lab/), on the client side, I just used my ordinary notebook(Intel i5 2.5GHz CPU, 10G memory and Win7 OS) to simulate multiple clients producing and consuming scenario, full source is [here](https://github.com/bulldog2011/bigqueue/tree/master/samples/thriftqueue/src/com/leansoft/thriftqueue/load), I used a cheap 1Gbps switch between server and clients to simulate reald world scenarion, below are the performance numbers:

1. Sequential Test - Producing then Consuming
{% codeblock %}
*******************************************************
Producer Report:
*******************************************************
Log producer thread number : 50
Test duration(s) : 279.144
Total logs sent : 10000000
Log sending success count : 10000000
Log sending failure count : 0
Log sending exception count : 0
Total byes produced : 20105400000
Average log size(byte) : 2010.54
Throughput(MB/s) : 68.68857507621235
Average log sending delay(ms) : 1.2961799

*******************************************************
Consumer Report:
*******************************************************
Log consumer thread number : 50
Test duration(s) : 274.157
Total logs received : 10000000
Log receiving success count : 10000000
Log receiving failure count : 0
Log receiving exception count : 0
Total byes received : 20105400000
Average log size(byte) : 2010.54
Throughput(MB/s) : 69.93804134519353
Average log receiving delay(ms) : 1.3691828
{% endcodeblock %}

2. Concurrent Test - Producing and Consuming currently
{% codeblock %}
*******************************************************
Concurrency Test Report:
*******************************************************
Producer Report:
*******************************************************
Log producer thread number : 50
Test duration(s) : 395.692
Total logs sent : 10000000
Log sending success count : 10000000
Log sending failure count : 0
Log sending exception count : 0
Total byes produced : 21588200000
Average log size(byte) : 2158.82
Throughput(MB/s) : 52.03064979186187
Average log sending delay(ms) : 1.8724801

*******************************************************
Consumer Report:
*******************************************************
Log consumer thread number : 50
Test duration(s) : 395.692
Total logs received : 10000000
Log receiving success count : 10000000
Log receiving failure count : 0
Log receiving exception count : 0
Total byes received : 21588200000
Average log size(byte) : 2158.82
Throughput(MB/s) : 52.03064979186187
Average log receiving delay(ms) : 1.8693821
{% endcodeblock %}

Even in concurrent scenario, big queue can concurrently producing and consuming around 50MBbs log data, this is equal to 50 * 3600 * 24 / (1024 * 1024) = 4.11 TB daily, amazing!

Regarding resource usage, CPU and memory usage on both server side and client side are under normal load, while network and disk IO are quite high on server side.
According to my observation, big queue is extremely fast, only limited by network and disk IO bandwidth.


###Final Word
Big queue is extremely fast, only limited by network and disk IO bandwidth, while at the same time it's persistent and reliable, this makes big queue a nature fit for log data collecting and analysis, I would recommend you to try big queue as a log collecting tool and let me know your real world experience with big queue by leaving feedback on this post.

