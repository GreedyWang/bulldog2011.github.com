---
layout: post
title: "Sort and Search 100GB Data on a Single Machine"
date: 2013-01-25 18:01
comments: true
categories: 
keywords: external sorting, merge sort, binary search, big data sort and search, persistent queue, algorithm and data structure
description: show effective algorithm and data structure to sort and search big data on commodity machine with limited memory.
---

How to sort 100GB or more data in effective way? You may tell me to use Hadoop, oh, I know Hadoop can definitely do that, but the cost to build and maintain Hadoop always make me headache. Can we sort 100GB or more data on a single commodity machine with less than 8GB memory? The answer is yes, use a technology called [external sorting](http://en.wikipedia.org/wiki/External_sorting). Since I have just build a big, fast and persistent queue, I want to show you how to use my big queue to sort huge amount of data on a single machine with limited physical memory.

<!--more-->
The algorithm is a typical [merge sort algorithm](http://en.wikipedia.org/wiki/Merge_sort), I adapted it to only use my big queue, here are the detailed steps:

	1. put all your data into a queue called sourceQueue.
	2. build a queue of sorted queues by dividing and sorting the sourceQueue.
		1. build a new queue called queueOfSortedQueues.
		2. extract n items out of the sourceQueue, n is the max num of items that can be sorted 
		   in physical memory in one pass.
		3. sort n items in memory using any effective in memory sorting algorithm like quick sort.
		4. put n sorted items into a new queue and put the queue referece into the queueOfSortedQueues.
		5. repeat 2 - 4 until all items in sourceQueue have been consumed.
	3. merge sort the queueOfSortedQueues
		1. extract n queues out of the queueOfSortedQueues, n must be >= 2 but not be too big, 
		   this is number of ways you want to merge sort in parallel.
		2. mergesort n queues and put the result sorted queue back into the queueOfSortedQueues
				1. build a new result queue for sorted items later.
				2. find out the lowest item in all n input queues by peeking the front of the queue
				3. extract the lowest item out of the queue and put the item into the result queue
				4. repeat 2 & 3 until all items in n input queues have been consumed.
				5. put the result sorted queue back into the queueOfSortedQueues.
		3. repeat 1 & 2 until there is only one left in the queueOfSortedQueues.
	4. The last one left in the queueOfSortedQueues is the final sorted queue.
***Note***: all queues mentioned above refer to my big queue except that queueOfSortedQueues is a normal in memory queue.

Basically, this is a typical divide and conqure algorithm, in order to sort data that is too big to be put into physical memory, you need to first divide the source data into chunks such that each chunk is small enough to be sorted in physical memory in one pass, after sorting all these small chunks, you need to merge and sort these chunks into the final sorted chunk, the merge and sort operation won't consume much memory because it only needs to sequentially read chunks on external storage and sequentially write the final sorted chunk to exertnal storagte, only limited items are kept in memory for comparing and sorting in one turn.

Suppose you have 64GB data to sort, and your machine can only sort 2GB in one pass, then you divide 64GB data into 32 chunks with 2GB each,
then you sort 32 chunks in memory in turn, after in memory sorting finish, if you choose 32 way merge sort, you merge and sort all 32 chunks into one final chunk,
if you choose 4 way merge sort, then you need 8 + 2 + 1 = 11 rounds of merge sort to get the final sorted chunk.

I was able to sort 128GB data(each data item has 100 bytes) in 8.68 hours using my big queue structure and the algorithm above, basically, the sort speed is only limited by disk IO bandwidth, 
if you are intested, you can find my merge sort code [here](https://github.com/bulldog2011/bigqueue/tree/master/samples/sortsearch/src/com/leansoft/bigqueue/sample), the code of multi-thread version is also included. 

Another interesting thing is, after I sorted the big data, by leveraging my [indexed big array](https://github.com/bulldog2011/bigqueue/tree/master/src/main/java/com/leansoft/bigqueue) structure and the typical [binary search alogrithm](http://en.wikipedia.org/wiki/Binary_search_algorithm) I can search more than 100GB sorted data in constant time(far less than 1 second on average), this is really amazing. If you are interested, find the source in the link mentioned above.

Any feedback to further improve my big queue sturucture and the merge sort alogrithm is welcome!
