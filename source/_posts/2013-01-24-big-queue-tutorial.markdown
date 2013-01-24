---
layout: post
title: "Big Queue Tutorial"
date: 2013-01-24 20:17
comments: true
categories: 
keywords: memory mapped queue
description: a tutorial to show the basic API usage of big queue.
---

This is a tutorial to show the basic API usage of big queue, the source of this tutorial is [here]().

The interface of big queue is [here](https://github.com/bulldog2011/bigqueue/blob/master/src/main/java/com/leansoft/bigqueue/IBigQueue.java), basically, it's as simple as the queue interface we learned in data structure course in college, if you want to refresh the concept of queue ADT, [here is the wikipeida link](http://en.wikipedia.org/wiki/Queue_%28abstract_data_type%29).


You can create(initialize) a new big queue in just one statement:

{% codeblock lang:java %}
// create a new big queue
IBigQueue bigQueue = new BigQueueImpl("d:/bigqueue/tutorial", "demo");
{% endcodeblock %}

the first parameter is a directory you want to store the queue data file, the second parameter is the queue name. Now you have a reference to an empty queue.

<!--more-->


To add or produce item into the queue, you just call the ***enqueue*** method on the queue reference, here we enqueue 10 numbers into the queue:

{% codeblock lang:java %}
// enqueue some items
for(int i = 0; i < 10; i++) {
	String item = String.valueOf(i);
	bigQueue.enqueue(item.getBytes());
}
{% endcodeblock %}

***Note:*** the enqueue method only accept byte array data as input, if your object is not byte array data, you are responsible to convert your object into byte array first before enqueue, this is called serialization, when you dequeue later, you are also response to de-serialize the byte array data into your object format.


Now there are 10 items in the queue, and it's not empty anymore, to find out the total number of items in the queue, call the ***size*** method:

{% codeblock lang:java %}
// get current size of the queue
int size = bigQueue.size();
{% endcodeblock %}

The ***peek*** method just let you peek item at the front of the queue without removing the item from the queue:

{% codeblock lang:java %}
// peek the front of the queue
String item = new String(bigQueue.peek());
{% endcodeblock %}

To remove or consume item from the queue, just call the ***dequeue*** method, here we dequeue 5 items from the queue:

{% codeblock lang:java %}
// dequeue some items 
for(int i = 0; i < 5; i++) {
	String item = new String(bigQueue.dequeue());
}
{% endcodeblock %}

since the queue is a FIFO queue, the number dequeued will be in FIFO order: 0, 1, 2, 3, 4.

To remove all remaining items from the queue, just do like this:

{% codeblock lang:java %}
// dequeue all remaining items
while(true) {
	byte[] data = bigQueue.dequeue();
	if (data == null) break;
}
{% endcodeblock %}

Now the queue is empty again, to check it's empty, just call ***isEmpty*** method:

{% codeblock lang:java %}
boolean isEmpty = bigQueue.isEmpty();
{% endcodeblock %}

Finally, when you finish with the queue, just call ***close*** method to release resource used by the queue, this is not mandatory, just a best practice, call close will release part of used memory immediately. Usually, you initialize big queue in a try block and close it in the finally block, here is the usage paradigm:

{% codeblock lang:java %}
// typical queue initialization and closing paradigm
IBigQueue bigQueue = null;
try {
    bigQueue = new BigQueueImpl("d:/bigqueue/tutorial", "demo");
} finally {
    bigQueue.close();
}
{% endcodeblock %}

Now its your turn to play with the big queue, in this tutorial, I just used very small number of data, but actually the big queue can hold lage amount of data, the total size is only limit by your avaiable disk space, so just try to enqueue and dequeue as much data as you can imagine, also, please review test code of big queue to learn more advanced usage like [multi-threads producing and consuming cases](https://github.com/bulldog2011/bigqueue/tree/master/src/test/java/com/leansoft/bigqueue/load).

