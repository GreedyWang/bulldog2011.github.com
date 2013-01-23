---
layout: post
title: "The Design of A Big Queue"
date: 2013-01-23 20:55
comments: true
categories: 
---

#Why a Big Queue?

This is a big data era, we are always facing challenge to find insights in big data. Last time, I have worked on the architecture and design of a large scale logging, tracing, monitoring and analytics platform, the core of the platform is a log collecting system and the core of the collector is a big queue, see figure below:

{% img center /images/bigqueue/log_collector.png 400 400 %}
 

The figure above looks like a typical producing and consuming scenario, the big queue works like a broker, at the left side of the queue, there are many agents deployed on application servers, they continuously collect log data on application servers and push them to the big queue, the agents work just like producers, on the right side of the big queue, there are several analysis systems which continuously pull log data from the queue, analyze the data and store them into the backend, the analysis systems work just like consumers. If you are interested in an industrial log collecting product built on this architecture, please refer to [apache kafka](http://kafka.apache.org/).

#The Requirements

Basically, we need the big queue to be:

+ **FAST & THREAD SAFE**      
The system needs to collect logs from more than 1000 production machines, they may produce more than 100,000 logs per second(average size is 1KB), this is equal to more than 100MB per second, if the big queue can’t keep up, logs will be lost. To further improve the throughout, we want all producers and consumers to work concurrently, so the queue needs to work in a thread safe manner, otherwise, there will be data lose or corruption.
+ **BIG & PERSISTENT**   
Daily logs will be at TB level, the queue should have the capacity to store up to one week’s logs, 
In case any of the analysis system is down(for example, periodical maintenance or even crash), the queue should continue to store logs for backend system to consume later when they are up again. Also, if the big queue itself is down, the logs already stored should survive since there are persistent, when the queue is up again, it will continue to serve the consumers.
+ **MEMORY EFFICIENT**  
Compared with big disk storage, current computer system is still shortage of physical memory, usually, memory will be less than 32GB on a commodity machine. We need the queue to use memory efficiently although it needs to handle logs more than 100MB per second and at TB level daily. 


#The Design Thinking Flow

Below is a simple and elegant design of the big queue I come up with to resolve the requirements and challenges above:

{% img center /images/bigqueue/bigqueue_abstraction_layers.png 300 300 %}
 
Usually, when I design something, I follow a top-down abstract thinking flow: as I learned in data structure course in college long ago, a queue data structure is usually built on an array data structure, so before I can build a queue I need to build an array first, the array I need should have following characters: first, it should be as fast as in memory access; second, it should be disk backed(hence it will be big and persistent). Seems there is contradiction between these two characters: if you need something fast, you need to put it in memory which is volatile and only has limited capacity, if you need something big and persistent, you need to put it on disk which has much slower access speed than physical memory, is there a technology to resolve these two contradictions? After an intensive research, I finally found memory mapped technology which seems a natural bridge between psychical memory and disk, if you need background about memory mapped technology, [here](http://www.kdgregory.com/index.php?page=java.byteBuffer) and [here](http://vanillajava.blogspot.co.uk/2012/03/presentation-on-using-shared-memory-in.html) are two good references. Now let’s continue the top-down thinking flow, before I can build a big array, I need to build a data structure called memory mapped page which can bridge the gap between speed and capacity, at the same time, I need some auxiliary structures to manage mapped pages in a memory efficient way, in the design figure above, these auxiliary structures are called mapped page factory and LRU cache. Whenever big array needs a mapped page, it requests one from the mapped page factory and returns it to the factory when it has done with the page. Mapped page factory encapsulates algorithms to allocate, cache and recycle mapped pages in a memory efficient and thread safe way by leveraging LUR cache structure.

Now, with the design in mind, I can implement these abstract structures in a bottom up approach, you can find the implementation details by studying the open source java code on [github](https://github.com/bulldog2011/bigqueue) if you are interested.


#Additional Design Notes:  
+ Although I learned some people used to map a single big file into memory, like [here](http://kdgcommons.svn.sourceforge.net/viewvc/kdgcommons/trunk/src/main/java/net/sf/kdgcommons/buffer/MappedFileBuffer.java?revision=HEAD&view=markup) and [here](http://vanillajava.blogspot.com/2011/12/using-memory-mapped-file-for-huge.html), I have memory leak concern with such approach(although I am not sure), instead, I came with up a novel pagging and swapping algorithm which only map fixed size(for example, 128M) page file into memory on demand and swap it out when it is not accessed within a fixed time to live(TTL) period. Which such design, I can not only use memory safer and more efficient, but can delete some used pages files to save disk space whenever necessary.

+ As we know, queue is a rear append and front read structure, so as long as I can put the queue front page and rear page(technically, this is called working set) in memory, then read and append operations can always happen in memory, that means the enqueue and dequeue operations are always an O(1) memory access.

+ The big queue is based on a **big array** structure, the big array itself is also an interesting data structure with some interesting features(I plan to write some use cases of this structure in the near future), the big array supports sequential append(called append only), sequential and random read. Sequential append and read are both O(1) memory access, while random read is O(1) memory access if the corresponding page is in cache, and is O(1) disk access if the corresponding page is not in cache. The big array is index based, just like normal in memory array, starting with index 0, when a new item is appended, the head index will be incremented, index is the pointer to the appended data, later you can use the index to read back the data. The index is of type long, this is a really very big range, I guess the big array won’t be used up before the world is end:)

+ Serialization is outside of the consideration of the big queue framework, the enqueue operation only accepts byte array as input(the dequeue operation only returns byte array), I left out serialization deliberately since I think it should not be the responsibility of the big queue framework, there are many existing and well known serialization frameworks(like protobuf, thrift, etc) which can do serialization work very well.

+ To ensure thread safe, some multi-threading technologies like read-write lock and thread local buffer are leveraged, the queue can not only work in read/write separation way – consumer and producer can work concurrently, but consumers and producers can all work together concurrently, no one will block other, this tremendously improved the throughout of the queue.

+ The queue has interface to delete used page files(for example, if data in these page files have been consumed by consumers) to save disk space, this is called garbage collection on disk files, much like GC in memory.

+ Abstractly, the whole queue looks like a huge FIFO circle buffer, disk backed and memory mapped.


#Performance Test  
[TODO]

#Conclusion  
To resolve a big data challenge I designed and implemented a simple while elegant big queue that are:  
>1. **Fast** : close to the speed of direct memory access, both enqueue and dequeue are close to O(1) memory access.  
>2. **Big** : the total size of the queue data is only limited by the available disk space.  
>3. **Persistent** : all data in the queue is persisted on disk, and is crash resistant.  
>4. **Memory-efficient** : automatic paging & swapping algorithm, only most-recently accessed data is kept in memory.  
>5. **Thread-safe**: multiple threads can concurrently enqueue and dequeue without data corruption.  
>6. **Simple and Light-weight**: current number of source files is 12 and the library jar is less than 20K.

Log data collecting is just use case of the big queue, I can anticipate the big queue will be used in more scenarios since big data challenges are becoming common these days.
