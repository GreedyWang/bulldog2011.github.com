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

<!--more-->

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

## Step 5 : Write some code
Below is some very simple examples of using Luxun for message producing and consuming, for consuming part,  `SimpleConsumer` is used, both `consume by index` and `consume by fanout id` are demonstrated in every case.

####Setup & Cleanup
The demos are showed as unit test cases, before every test cases, we set up two brokers and two simple consumers for testing, after every test cases, we do some cleanup:

{% codeblock SimpleDemo.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/SimpleDemo.java source %}

	private int brokerId1 = 0;
	private int brokerId2 = 1;
	private int port1 = 9092;
	private int port2 = 9093;
	private LuxunServer server1 = null;
	private LuxunServer server2 = null;
	private String brokerList = brokerId1 + ":localhost:" + port1 + "," + brokerId2 + ":localhost:" + port2;
	private String broker1 = brokerId1 + ":localhost:" + port1;
	
	private SimpleConsumer simpleConsumer1 = null;
	private SimpleConsumer simpleConsumer2 = null;
	
	@Before
	public void setup() {
		// set up 2 brokers
		Properties props1 = new Properties();
	    props1.put("brokerid", String.valueOf(brokerId1));
	    props1.put("port", String.valueOf(port1));
	    props1.put("log.dir", TestUtils.createTempDir().getAbsolutePath());
		ServerConfig config1 = new ServerConfig(props1);
		server1 = new LuxunServer(config1);
		server1.startup();
		
		Properties props2 = new Properties();
	    props2.put("brokerid", String.valueOf(brokerId2));
	    props2.put("port", String.valueOf(port2));
	    props2.put("log.dir", TestUtils.createTempDir().getAbsolutePath());
		ServerConfig config2 = new ServerConfig(props2);
		server2 = new LuxunServer(config2);
		server2.startup();
		
		// set up two simple consumers
		// create a consumer 1 to connect to the Luxun server running on localhost, port 9092, socket timeout of 60 secs
		simpleConsumer1 = new SimpleConsumer("localhost", port1, 60000);
		// create a consumer 2 to connect to the Luxun server running on localhost, port 9093, socket timeout of 60 secs
		simpleConsumer2 = new SimpleConsumer("localhost", port2, 60000);
	}


	@After
	public void cleanup() throws Exception {
		server1.close();
		server2.close();
		
		simpleConsumer1.close();
		simpleConsumer2.close();
		
		Utils.deleteDirectory(new File(server1.config.getLogDir()));
		Utils.deleteDirectory(new File(server2.config.getLogDir()));
		Thread.sleep(500);
	}
{% endcodeblock %}


####Send a single message

{% codeblock SimpleDemo.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/SimpleDemo.java source %}

	@Test
	public void sendSingleMessage() throws Exception {
		Properties props = new Properties();
		props.put("serializer.class", StringEncoder.class.getName());
		props.put("broker.list", broker1);
		
		ProducerConfig config = new ProducerConfig(props);
		Producer<String, String> producer = new Producer<String, String>(config);
		
		ProducerData<String, String> data = new ProducerData<String, String>("test-topic", "test-message");
		producer.send(data);
		
		producer.close(); // finish with the producer
		
		// consume by index
		List<MessageList> listOfMessageList = simpleConsumer1.consume("test-topic", 0, 10000);
		assertTrue(listOfMessageList.size() == 1);
		MessageList messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		Message message = messageList.get(0);
		assertEquals("test-message", new String(message.getBytes()));
		
		// consume by fanout id
		String fanoutId = "demo";
		listOfMessageList = simpleConsumer1.consume("test-topic", fanoutId, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		message = messageList.get(0);
		assertEquals("test-message", new String(message.getBytes()));
	}

{% endcodeblock %}

####Send multiple messages in one request

{% codeblock SimpleDemo.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/SimpleDemo.java source %}

	@Test
	public void sendMultipleMessages() throws Exception {
		Properties props = new Properties();
		props.put("serializer.class", StringEncoder.class.getName());
		props.put("broker.list", broker1);
		
		ProducerConfig config = new ProducerConfig(props);
		Producer<String, String> producer = new Producer<String, String>(config);
		
		List<String> messages = new ArrayList<String>();
		messages.add("test-message1");
		messages.add("test-message2");
		messages.add("test-message3");
		ProducerData<String, String> data = new ProducerData<String, String>("test-topic", messages);
		producer.send(data);
		
		producer.close(); // finish with the producer
		
		// consume by index
		List<MessageList> listOfMessageList = simpleConsumer1.consume("test-topic", 0, 10000);
		assertTrue(listOfMessageList.size() == 1);
		MessageList messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 3);
		for(int i = 1; i <= 3; i++) {
			Message message = messageList.get(i - 1);
			assertEquals("test-message" + i, new String(message.getBytes()));
		}
		
		// consume by fanout id
		String fanoutId = "demo";
		listOfMessageList = simpleConsumer1.consume("test-topic", fanoutId, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 3);
		for(int i = 1; i <= 3; i++) {
			Message message = messageList.get(i - 1);
			assertEquals("test-message" + i, new String(message.getBytes()));
		}
	}

{% endcodeblock %}

####Send messages to different topics

{% codeblock SimpleDemo.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/SimpleDemo.java source %}

	@Test
	public void sendMessagesToDifferentTopics() throws Exception {
		Properties props = new Properties();
		props.put("serializer.class", StringEncoder.class.getName());
		props.put("broker.list", broker1);
		
		ProducerConfig config = new ProducerConfig(props);
		Producer<String, String> producer = new Producer<String, String>(config);
		
		ProducerData<String, String> data1 = new ProducerData<String, String>("test-topic1", "test-message1");
		producer.send(data1);
		
		ProducerData<String, String> data2 = new ProducerData<String, String>("test-topic2", "test-message2");
		producer.send(data2);
		
		producer.close(); // finish with the producer
		
		// consume by index
		List<MessageList> listOfMessageList = simpleConsumer1.consume("test-topic1", 0, 10000);
		assertTrue(listOfMessageList.size() == 1);
		MessageList messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		Message message = messageList.get(0);
		assertEquals("test-message1", new String(message.getBytes()));
		
		listOfMessageList = simpleConsumer1.consume("test-topic2", 0, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		message = messageList.get(0);
		assertEquals("test-message2", new String(message.getBytes()));
		
		// consume by fanoutId
		String fanoutId = "demo";
		listOfMessageList = simpleConsumer1.consume("test-topic1", fanoutId, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		message = messageList.get(0);
		assertEquals("test-message1", new String(message.getBytes()));
		
		listOfMessageList = simpleConsumer1.consume("test-topic2", fanoutId, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		message = messageList.get(0);
		assertEquals("test-message2", new String(message.getBytes()));
	}

{% endcodeblock %}


####Send messages with GZIP compression

{% codeblock SimpleDemo.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/SimpleDemo.java source %}

	@Test
	public void sendMessageWithGZIPCompression() throws Exception {
		Properties props = new Properties();
		props.put("serializer.class", StringEncoder.class.getName());
		props.put("broker.list", broker1);
		props.put("compression.codec", "1");
		
		ProducerConfig config = new ProducerConfig(props);
		Producer<String, String> producer = new Producer<String, String>(config);
		
		ProducerData<String, String> data = new ProducerData<String, String>("test-topic", "test-message");
		producer.send(data);
		
		producer.close(); // finish with the producer
		
		// consume by index
		List<MessageList> listOfMessageList = simpleConsumer1.consume("test-topic", 0, 10000);
		assertTrue(listOfMessageList.size() == 1);
		MessageList messageList = listOfMessageList.get(0);
		assertEquals(CompressionCodec.GZIP, messageList.getCompressionCodec());
		assertTrue(messageList.size() == 1);
		Message message = messageList.get(0);
		assertEquals("test-message", new String(message.getBytes()));
		
		// consume by fanout id
		String fanoutId = "demo";
		listOfMessageList = simpleConsumer1.consume("test-topic", fanoutId, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertEquals(CompressionCodec.GZIP, messageList.getCompressionCodec());
		assertTrue(messageList.size() == 1);
		message = messageList.get(0);
		assertEquals("test-message", new String(message.getBytes()));
	}

{% endcodeblock %}


####Send messages with async producer

{% codeblock SimpleDemo.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/SimpleDemo.java source %}

	@Test
	public void sendMessageWithAsyncProducer() throws Exception {
		Properties props = new Properties();
		props.put("serializer.class", StringEncoder.class.getName());
		props.put("producer.type", "async");
		props.put("broker.list", broker1);
		
		ProducerConfig config = new ProducerConfig(props);
		Producer<String, String> producer = new Producer<String, String>(config);
		
		ProducerData<String, String> data = new ProducerData<String, String>("test-topic", "test-message");
		producer.send(data);
		
		producer.close(); // finish with the producer
		
		// consume by index
		List<MessageList> listOfMessageList = simpleConsumer1.consume("test-topic", 0, 10000);
		assertTrue(listOfMessageList.size() == 1);
		MessageList messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		Message message = messageList.get(0);
		assertEquals("test-message", new String(message.getBytes()));
		
		// consume by fanout id
		String fanoutId = "demo";
		listOfMessageList = simpleConsumer1.consume("test-topic", fanoutId, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		message = messageList.get(0);
		assertEquals("test-message", new String(message.getBytes()));
	}

{% endcodeblock %}


####Send messages with custom partitioner

{% codeblock CustomPartitioner.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/CustomPartitioner.java source %}

package com.leansoft.luxun.quickstart;

import com.leansoft.luxun.producer.IPartitioner;

public class CustomPartitioner implements IPartitioner<String> {
	@Override
	public int partition(String key, int numBrokers) {
		return (key.length() % numBrokers);
	}
}

{% endcodeblock %}

{% codeblock SimpleDemo.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/SimpleDemo.java source %}

	@Test
	public void sendMessagesWithCustomPartitioner() throws Exception {
		Properties props = new Properties();
		props.put("serializer.class", StringEncoder.class.getName());
		props.put("broker.list", brokerList);
		props.put("partitioner.class", CustomPartitioner.class.getName());
		
		ProducerConfig config = new ProducerConfig(props);
		Producer<String, String> producer = new Producer<String, String>(config);
		
		// will be sent to broker 1 since (the length of key % num of brokers) = 0
		ProducerData<String, String> data1 = new ProducerData<String, String>("test-topic1", "key1", "test-message1");
		producer.send(data1);
		
		// will be went to broker 2 since (the length of key % num of brokers) = 1
		ProducerData<String, String> data2 = new ProducerData<String, String>("test-topic2", "key11", "test-message2");
		producer.send(data2);
		
		producer.close(); // finish with the producer
		
		// consume by index
		List<MessageList> listOfMessageList = simpleConsumer1.consume("test-topic1", 0, 10000);
		assertTrue(listOfMessageList.size() == 1);
		MessageList messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		Message message = messageList.get(0);
		assertEquals("test-message1", new String(message.getBytes()));
		
		listOfMessageList = simpleConsumer2.consume("test-topic2", 0, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		message = messageList.get(0);
		assertEquals("test-message2", new String(message.getBytes()));
		
		// consume by fanoutId
		String fanoutId = "demo";
		listOfMessageList = simpleConsumer1.consume("test-topic1", fanoutId, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		message = messageList.get(0);
		assertEquals("test-message1", new String(message.getBytes()));
		
		listOfMessageList = simpleConsumer2.consume("test-topic2", fanoutId, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		message = messageList.get(0);
		assertEquals("test-message2", new String(message.getBytes()));
	}

{% endcodeblock %}


####Send messages with custom encoder

{% codeblock LogEventEncoder.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/LogEventEncoder.java source %}

package com.leansoft.luxun.quickstart;

import com.leansoft.luxun.message.Message;
import com.leansoft.luxun.serializer.Encoder;
import com.leansoft.luxun.serializer.ThriftConverter;

public class LogEventEncoder implements Encoder<LogEvent> {

	@Override
	public Message toMessage(LogEvent event) {
		byte[] binary = ThriftConverter.toBytes(event);
		return new Message(binary);
 	}

}

{% endcodeblock %}

{% codeblock LogEventDecoder.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/LogEventDecoder.java source %}

package com.leansoft.luxun.quickstart;

import com.leansoft.luxun.message.Message;
import com.leansoft.luxun.serializer.Decoder;
import com.leansoft.luxun.serializer.ThriftConverter;

public class LogEventDecoder implements Decoder<LogEvent> {

	@Override
	public LogEvent toEvent(Message message) {
		byte[] binary = message.getBytes();
		return (LogEvent) ThriftConverter.toEvent(binary, LogEvent.class);
	}

}

{% endcodeblock %}

{% codeblock SimpleDemo.java lang:java https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/SimpleDemo.java source %}
	@Test
	public void sendMessageWithCustomEncoder() throws Exception {
		Properties props = new Properties();
		props.put("serializer.class", LogEventEncoder.class.getName());
		props.put("broker.list", broker1);
		
		
		ProducerConfig config = new ProducerConfig(props);
		Producer<String, LogEvent> producer = new Producer<String, LogEvent>(config);
		
		LogEvent logEvent = new LogEvent();
		logEvent.createdTime = System.currentTimeMillis();
		logEvent.hostId = "127.0.0.1";
		logEvent.logLevel = LogLevel.INFO;
		logEvent.message = "a test log message";
		
		ProducerData<String, LogEvent> data = new ProducerData<String, LogEvent>("log-topic", logEvent);
		producer.send(data);
		
		producer.close(); // finish with the producer
		
		// consume by index
		LogEventDecoder decoder = new LogEventDecoder();
		List<MessageList> listOfMessageList = simpleConsumer1.consume("log-topic", 0, 10000);
		assertTrue(listOfMessageList.size() == 1);
		MessageList messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		Message message = messageList.get(0);
		assertEquals(logEvent, decoder.toEvent(message));
		
		// consume by fanout id
		String fanoutId = "demo";
		listOfMessageList = simpleConsumer1.consume("log-topic", fanoutId, 10000);
		assertTrue(listOfMessageList.size() == 1);
		messageList = listOfMessageList.get(0);
		assertTrue(messageList.size() == 1);
		message = messageList.get(0);
		assertEquals(logEvent, decoder.toEvent(message));
	}

{% endcodeblock %}


## Step 6 : Study advanced demo
You can find the source of an advanced demo [here](https://github.com/bulldog2011/luxun/blob/master/src/test/java/com/leansoft/luxun/quickstart/AdvancedDemo.java), this demo shows:
>1. Multiple topics
2. Multiple threads concurrent producing and consuming
3. The use of `StreamFactory` to create advanced stream style consumers
4. Group consuming