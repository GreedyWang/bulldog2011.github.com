---
layout: post
title: "Turn Big Queue into a Thrift based Queue Serivce"
date: 2013-01-27 21:01
comments: true
categories: [big queue]
keywords: persistent queue, thrift queue, apache thrift, light-weight RPC
description: show how to turn big queue into a thrift based queue service
---

In this post, I will show you how to add a Thrift RPC component to my [big queue](https://github.com/bulldog2011/bigqueue) library to turn it into a light-weigth queue service.

###Why I choose Thrift?
I love Thrift so much that I can't help recommending and dumpling all of its good parts here:  

<!--more-->  

>1. Thrift is ***stable and mature***, it is created by Facebook, now it's an Apache project, it has been successfully used by famous projects like Cassandra and HBase.  
2. Thrift is ***simple and light-weight***, you just need to define a simple interface using its light-weight IDL(interface definition language), then you can auto-generate basic server and client proxy code without much effort, this can not only minimize development effort, but later upgrading effort - you just need to update the IDL then re-generate.  
3. Thrift has ***high performance***, it provides highly effective serialization protocols like TBinaryProtocol and service model like TNonBlockingServer, so you won't get troubled with building your own NIO server which is very tricky.  
4. Thrift has good ***cross-language*** support, such as java, csharp, php, ruby, just name a few. One big factor I choose Thrift is - everytime I build a Thrift service, clients for different language platforms are basically ready, If I need a client for languge X, I can easily generate one using its universal code generator.  
5. Thrift is ***flexible***, Thrift has a pluggable architecture, transport protocols(like tcp or http), serialization protocol(like TBinaryProtocol, TJSONProtocol) and server models(like TNonBlockingServer, TThreadPoolServer) are all changeable according to your real needs.  
  
Basically, I think guys at Facebook have made a really cool RPC framework, greatly simplified service development.

###The Basic Steps to Turn Big Queue into Thrift Serivce
I won't show much details about thrift and its development here, since there are already many reference and turtorils out there, If you need more information about Thrift, visit its [official site](http://thrift.apache.org) and [wiki](http://wiki.apache.org/thrift/), I just list the basic steps to turn my big queue into a Thrift service here, in case you need to do similar things later, the whole source is [here](https://github.com/bulldog2011/bigqueue/tree/master/samples/thriftqueue).  


1. ***Define serivce interface using Thrift IDL***   
Below is the IDL I defined for my big queue, basically, this interface mirrors the big queue interface, one new thing I added is "topic" support, with topic, client can enqueue into or dequeue from different topics(queues), just like topic semantics in messaging system, in fact, topic corresponds to queue name, if a topic does not exist when client enqueue, server will create a new queue for this topic before enqueue.  
	
		namespace java com.leansoft.bigqueue.thrift
		namespace csharp Leansoft.BigQueue.Thrift
		
		struct QueueRequest {
		    1: required binary data
		}
		
		enum ResultCode
		{
		  SUCCESS,
		  FAILURE
		}
		
		struct QueueResponse {
		    1: required ResultCode resultCode,
		    2: binary data
		}
		
		service BigQueueService {
		    QueueResponse enqueue(1:string topic, 2:QueueRequest req);
		    QueueResponse dequeue(1:string topic);
		    QueueResponse peek(1:string topic);
		    i64 getSize(1:string topic);
		    bool isEmpty(1:string topic);
		}



2. ***Genenerate proxy code using Thrift code generator***  
This is quite easy by using the command line tool,   
to generate java code, use:  

		thrift-0.6.1.exe --gen java queue.idl
to generate csharp code, use:  

		thrift-0.6.1.exe --gen csharp queue.idl
After generation, copy or link generated code into project as source.

3. ***Implement serivce interface on server side***
Still quite easy, just delegate all queue operations to the real big queue, a topic to queue map is added to support topic semantics, source [here](https://github.com/bulldog2011/bigqueue/blob/master/samples/thriftqueue/src/com/leansoft/thriftqueue/server/ThriftQueueServiceImpl.java).

4. ***Implement server***
Using Thrift, a server can be built with less than 20 lines of code, source [here](https://github.com/bulldog2011/bigqueue/blob/master/samples/thriftqueue/src/com/leansoft/thriftqueue/server/ThriftQueueServer.java), I chosed TBinaryProtocol(which has good serialization performance) as serialization protocol, and TNonblockingServer(which has good performance even with large amount of concurrent connections) as server model, you may try different ones according to your real needs.

5. ***Implement client***
On client side, you just need to create a thrift client which also implements the BigQueueService interface defined above, then call methods on the serivce client according to your real needs, and close the client finally. That's it. Java client code is [here](https://github.com/bulldog2011/bigqueue/blob/master/samples/thriftqueue/src/com/leansoft/thriftqueue/client/ThriftQueueClientDemo.java). How about client for other languages? quite easy, I just built a csharp client within 30 minutes, code [here](https://github.com/bulldog2011/bigqueue/tree/master/samples/thriftqueue/CSharpClient). 


Now let's check the working of the queue service, start the server first, than run the client demo, if everything works fine, the client can interact with the queue on server smoothly, if not, do some troubleshooting, or ask me for help by replying this post. 

***Note:*** this is just the bare minimum queue service I introduced to you, in real world, you need to enhance and adapt it according to your real needs before putting it into production. 

###Final Words

Cool! I turned my big queue into a Thrift based queue service with less than 2 hours, this is agile development I am pursuing!

When compared with heavy-weight enterprise level messaging products like ActiveMQ or RabbitMQ, BigQueue with Thrift support is a quite light-weight, fast and persistent messaging framework, this is the ***just enough queue*** which can slove your business problem in agile and effective way, if you find this queue useful, don't foget to let me know by leaving feedback at this post.






