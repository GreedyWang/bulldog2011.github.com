---
layout: post
title: "Big Array Tutorial"
date: 2013-01-24 21:37
comments: true
categories: 
keywords: memory mapped array
description: This is a tutorial to show the basic API usage of big array.
---

This is a tutorial to show the basic API usage of big array, the source of this tutorial is [here](https://github.com/bulldog2011/bigqueue/blob/master/src/test/java/com/leansoft/bigqueue/tutorial/BigArrayTutorial.java).

Big array is a building block of big queue, since big array is also a valuable data struture with unique feature, I decided to write a separate tutorial for it.

The interface of big array is [here](https://github.com/bulldog2011/bigqueue/blob/master/src/main/java/com/leansoft/bigqueue/IBigArray.java), basically, it's simple and similar to a typical indexed array except that big array is an append only array, you can't randomly append data in big array like typical array after allocation, you can only append data on head of the array, although you can randomly read data by index in big array after you append data to it.

You can create(initialize) a new big array in just on statement:

{% codeblock lang:java %}
// create a new big array
IBigArray bigArray = new BigArrayImpl("d:/bigarray/tutorial", "demo");
{% endcodeblock %}

the first parameter is a directory you want to store the array data file, the second parameter is the array name. Now you have a reference to an empty array.

<!--more-->

To append items into the array, you just call the ***append*** method on the array reference, here we append 10 numbers into the array:

{% codeblock lang:java %}
// append some items into the array
for(int i = 0; i < 10; i++) {
	String item = String.valueOf(i);
	long index = bigArray.append(item.getBytes());
}
{% endcodeblock %}

The append operation will return an index of type long, this index can be used to retrive the appended data later, just like the index of normal array.  
Big array index is incremental, starting from 0, upon successful append, it will be incremented to point to the next to be appended slot, internally, big array has two pointers, one is tail pointer, pointing to the first index of the array, the other is head pointer, pointing to the next to be appended index, to learn about the current tail index and head index of array, use ***getTailIndex*** and ***getHeadIndex*** methods on the interface.

***Note:*** the append method only accept byte array data as input, if your object is not byte array data, you are responsible to convert your object into byte array first before append, this is called serialization, when you get data later, you are also response to de-serialize the byte array data into your object format.

Now there are 10 items in the array, it's not empty anymore, to find out the total number of items in the array, call the ***size*** method:

{% codeblock lang:java %}
// get current size of the array
int size = bigArray.size();
{% endcodeblock %}

Big array support random read by index, just call ***get*** method and provide the index to the data, just like normal array indexing operation, here we get 3 items from the array by index:

{% codeblock lang:java %}
// randomly read items in the array
String item0 = new String(bigArray.get(0)); // item0 equals to 0

String item3 = new String(bigArray.get(3)); // item3 equals to 3

String item9 = new String(bigArray.get(9)); // item9 equals to 9
{% endcodeblock %}

Note: you can only get(retrive) data which has been appended before, if you want to get an index which has no data yet, you will get ArrayIndexOutOfBoundsException, this behavior is the same as typical array.

If you want to make empty the whole array, just call ***removeAll*** method like this:

{% codeblock lang:java %}
// empty the big array
bigArray.removeAll();
{% endcodeblock %}
Now all items in the array are removed, array tail and head has been be reset to 0, which means you can start to append data start from index 0 again.

When you finish with the array, just call ***close*** method to release resource used by the array, this is not mandatory, just a best practice, call close will release part of used memory immediately. Usually, you initialize big array in a try block and close it in the finally block, here is the usage paradigm:

{% codeblock lang:java %}
// typical array initialization and closing paradigm
IBigArray bigArray = null;
try {
    bigArray = new BigArrayImpl("d:/bigarray/tutorial", "demo");
} finally {
    bigArray.close();
}
{% endcodeblock %}


Now its your turn to play with the big array, in this tutorial, I just used very small number of data, but actually the big array can hold lage amount of data, normal array has index of type int, but big array has index of type long, the total size is only limit by your avaiable disk space, so just try to append as much data as you can imagine, then randomly read data to find out how fast big array can be used to randomly access huge amout of data on disk. Also, please review test code of big array to learn more advanced usage like [multi-threads producing and consuming cases](https://github.com/bulldog2011/bigqueue/tree/master/src/test/java/com/leansoft/bigqueue/load).
