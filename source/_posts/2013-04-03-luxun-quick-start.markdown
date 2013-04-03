---
layout: post
title: "luxun quick start"
date: 2013-04-03 17:01
comments: true
categories: [luxun]
keywords: distributed, persistent queue, kafka
description: luxun quick start
---

## Step 0 : Prerequisite
To use Luxun, you need to have JDK 1.6 or above installed on your operating system, and Luxun supports ***Windows*** and ***Linux*** operation systems.

## Step 1 : Download Luxun package
Download a recent stable release by following instructions on [Luxun github site](https://github.com/bulldog2011/luxun), then extract zip file to a local folder.  Current(when this post is written) stable release is [0.6.0](https://github.com/bulldog2011/bulldog-repo/raw/master/repo/releases/com/leansoft/luxun/0.6.0/luxun-0.6.0-bin.zip).

Luxun distribution provides scripts(in `bin` folder) for both Windows and Linux environments, the demo below is made in a windows environment, if you are using Linux environment, please change script accordingly.

## Step 2 : Start the server
{% codeblock %}

D:\test\luxun-0.6.0>bin\server.bat conf\server.properties
2013-04-03 16:36:18.856 INFO  [LuxunServer] Starting luxun server 0.6
2013-04-03 16:36:18.872 INFO  [LogManager] starting log cleaner every 600000 ms.
2013-04-03 16:36:18.888 INFO  [ThriftServer] Wating server to start, time waited : 0 s.
2013-04-03 16:36:19.902 INFO  [ThriftServer] Thrift server started on port : 9092
2013-04-03 16:36:19.902 INFO  [LuxunServer] Server started.

{% endcodeblock %}

## Step 3 : Send some messages
Luxun comes with a command line client that will take input from standard input and send it out as messages to the Luxun server. Each line will be sent as a separate message. The topic `demo` is created automatically when messages are sent to it. You should see something like this:

{% codeblock %}

D:\test\luxun-0.6.0>bin\producer-console.bat --broker-list 0:localhost:9092 --topic demo
Enter your message and exit with empty string.
> Welcome to Luxun
Message sent : Welcome to Luxun
> 鲁迅是一个伟大的中国作家
Message sent : 鲁迅是一个伟大的中国作家
>

{% endcodeblock %}

## Step 4 : Start a consumer
Luxun also has a command line consumer that will dump out messages to standard output.

{% codeblock %}
 
D:\test\luxun-0.6.0>bin\simple-consumer-console --topic demo --server luxun://localhost:9092

[1] 0: Welcome to Luxun
[2] 1: 鲁迅是一个伟大的中国作家

{% endcodeblock %}

If you have each of the above commands running in a different terminal then you should now be able to type messages into the producer terminal and see them appear in the consumer terminal.

Both of these command line tools have additional options. Running the command with no arguments will display usage information documenting them in more detail. 

`simple-consumer-console` is built on Luxun's simple consumer, there is also a similar `consumer-console` which is built on Luxun's advanced consumer. Both can be used for demo and testing.
