In this post, I will show you how to add a Thrift RPC component to my [big queue](https://github.com/bulldog2011/bigqueue) library to turn it into a light-weigth queue service.

###Why I choose Thrift?
Here are the reasons why I love Thrift very much:
<!--more-->  
>1. Thrift is ***stable and mature***, it is created by Facebook, now it's an Apache project, it has been used by famous projects like Cassandra and HBase.  
2. Thrift is ***simple and light-weight***, you just need to define a simple interface using its light-weight IDL(interface definition language), then you can auto-generate basic server and client proxy code without much effort, this can not only minimize development effort, but minimize later upgrading effort, you just need to update the IDL then re-generate.  
3. Thrift has ***high performance***, it provides highly effective serialization protocols like TBinaryProtocol and service model like TNonBlockingServer, so you don't need to be bothered with building your own NIO server which is very tricky.  
4. Thrift has good ***cross-language*** support, java, csharp, php, ruby, just name a few, one big factor I choose Thrift is - after I build a Thrift based queue service, clients for different language platforms are basically ready, If I need a client for languge X, I can easily generate one using its universal code generator.  
5. Thrift is ***flexible***, Thrift has a pluggable design, transport protocols(like tcp or http), serialization protocol(like TBinaryProtocol, TJSONProtocol) and server models(like TNonBlockingServer, TThreadPoolServer) are all changable according to your real requirements.  
Basically, I think guys at Facebook have made a really cool RPC framework, greatly simplified service development.

###The Basic Steps to Develop Thrift Serivce
I won't show much details about thrift and it's development here, since there are already many references and turtorils out there, If you need more information about Thrift, visit its [official site](http://thrift.apache.org) and [wiki](http://wiki.apache.org/thrift/), I just list the basic steps to turn my big queue into a Thrift service here, in case you need to do similar things later, the whole source is [here](https://github.com/bulldog2011/bigqueue/tree/master/samples/thriftqueue).  


1. Define serivce interface using Thrift IDL   
below is the IDL I defined for my big queue, basically, this interface mirrors the big queue interface, I just added 'topic' support, in fact, topic is queue name, with topic, client can enqueue into or dequeue from different topics(queues), if a topic does not exist yet when client enqueue, then server will create a new queue for this topic before enqueue.  
	
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



2. Genenerate proxy code using Thrift code generator  
This is quite easy by using the command line tool,   
to generate java code, use:  

		thrift-0.6.1.exe --gen java queue.idl
to generate csharp code, use:  

		thrift-0.6.1.exe --gen csharp queue.idl

3. Implement serivce interface on server side.
Still quite easy, just delage the queue operations to the real big queue, a topic to queue map is added to support topic semantics, source [here](https://github.com/bulldog2011/bigqueue/blob/master/samples/thriftqueue/src/com/leansoft/thriftqueue/server/ThriftQueueServiceImpl.java).

4. Implement server  
A server can be built with less than 20 lines of code with thrift, source [here](https://github.com/bulldog2011/bigqueue/blob/master/samples/thriftqueue/src/com/leansoft/thriftqueue/server/ThriftQueueServer.java), I just choose TBinaryProtocol since it has good serialization performance, and TNonblockingServer which has good performance even with large amount of concurrent connections.

5. Implement client
On client side, you just need to create a thrift client which also implements the BigQueueService interface defined above, then call methods on the serivce client according to your real requirements, and close the client finally, that's it, Java client code is [here](https://github.com/bulldog2011/bigqueue/blob/master/samples/thriftqueue/src/com/leansoft/thriftqueue/client/ThriftQueueClientDemo.java). How about client for other languages, quite easy, I just built a csharp client within 30 minutes, code [here](https://github.com/bulldog2011/bigqueue/tree/master/samples/thriftqueue/CSharpClient). 

###Finally
Start the server first, than run the client demo, now the client can interact with the queue on server smoothly. Cool! I turned my big queue into a Thrift service with less than 2 hours, this is agile development I am pursuing!





