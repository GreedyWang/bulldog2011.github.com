---
layout: post
title: "nano benchmark on android"
date: 2013-02-08 10:04
comments: true
categories: [nano]
keywords: xml benchmark on android, xml data binding, nano binding framework, dom, sax, xml pull, jaxb
description: xml parser and Nano benchmark on android
---

Since I have built a light-weight XML/JSON binding framework tailored for Android platform, I want to test its performance on Android when compared with typical xml parsers on Android like SAX, DOM and Xml Pull, I will show you detailed test results in this post.

<!--more-->

I did the benchmark on my own mobile phone(it was quite cheap when I bought it:)), below is the spec of my mobile phone:
>1. Brand : Samsung SCH-i559(Galaxy Mini)
2. CPU : ARM6 600MHz
3. RAM : 128MB
4. OS : Android 2.2.1

The payload I used is a faked person list, you can find the scheam [here](https://github.com/bulldog2011/nano/blob/master/performance/nano-vs-jaxb/src/main/resources/person.xsd) and all payloads [here](https://github.com/bulldog2011/nano/tree/master/performance/nano-on-android/assets), I created these payloads with Nano and an auto-fake data generator called [podam](http://www.jemos.eu/projects/podam/). I tested 3 kinds of payload size :
>1. 10 records - 4KB xml, 2KB json
2. 50 records - 17KB xml, 9KB json
3. 300 records - 100KB xml, 50KB json

Basically, the size of json payload is almost half of the size of xml, this is because json has a more compact messsage format, the 300 records case is used for testing performance when dealing with big payload size.

The benchmark program itself is a typcial Android application, I adaped the test project from [this](http://www.developer.com/ws/android/development-tools/Android-XML-Parser-Performance-3824221.htm) post, I added Nano cases and SAX, DOM, XML pull parsing cases in the benchmark program, you can download the whole benchmark program [here](https://github.com/bulldog2011/nano/tree/master/performance/nano-on-android).

Below is the UI of the benchmark application on PC emulator, 

{% img /images/nano/nano-benchmark-app1.png 300 600 %}  {% img /images/nano/nano-benchmark-app2.png 300 600 %}

following choices are avaliable for combined benchmark:
>1. Thread Number - 1, 3, 5 threads
2. Payload Size - 10, 50, 300 records
3. Test Type - Nano Xml Read, Nano Json Read, SAX Read, DOM Read, XML Pull Read, Nano Xml Write, Nano Json Write



###Test Result

***Note:***
>1. the unit of test result is milliseconds
2. all test results are average of 20 runs.
3. for read test, the time includes file reading time, for write test, serialized content is only written in memory, not real file. 

#### 1 Thread Read(Unmarshall) Test


<table>
   <tr>
      <td>[Records]</td>
      <td>[Nano XML]</td>
      <td>[Nano JSON]</td>
      <td>[RAW SAX]</td>
      <td>[RAW DOM]</td>
      <td>[RAW Pull]</td>
   </tr>
   <tr>
      <td>10</td>
      <td>34</td>
      <td>31</td>
      <td>18</td>
      <td>53</td>
      <td>24</td>
   </tr>
   <tr>
      <td>50</td>
      <td>133</td>
      <td>67</td>
      <td>70</td>
      <td>217</td>
      <td>90</td>
   </tr>
   <tr>
      <td>300</td>
      <td>724</td>
      <td>292</td>
      <td>388</td>
      <td>1318</td>
      <td>497</td>
   </tr>
</table>  


#### 3 Threads Read(Unmarshall) Test


<table>
   <tr>
      <td>[Records]</td>
      <td>[Nano XML]</td>
      <td>[Nano JSON]</td>
      <td>[RAW SAX]</td>
      <td>[RAW DOM]</td>
      <td>[RAW Pull]</td>
   </tr>
   <tr>
      <td>10</td>
      <td>102</td>
      <td>99</td>
      <td>53</td>
      <td>163</td>
      <td>76</td>
   </tr>
   <tr>
      <td>50</td>
      <td>405</td>
      <td>215</td>
      <td>206</td>
      <td>713</td>
      <td>272</td>
   </tr>
   <tr>
      <td>300</td>
      <td>2318</td>
      <td>951</td>
      <td>1186</td>
      <td>3812</td>
      <td>1599</td>
   </tr>
</table>  


#### 1 Thread Write(Marshall) Test


<table>
   <tr>
      <td>[Records] </td>
      <td>[Nano XML]</td>
      <td>[Nano JSON]</td>
   </tr>
   <tr>
      <td>10</td>
      <td>15</td>
      <td>23</td>
   </tr>
   <tr>
      <td>50</td>
      <td>61</td>
      <td>87</td>
   </tr>
   <tr>
      <td>300</td>
      <td>403</td>
      <td>508</td>
   </tr>
</table>  


#### 3 Threads Write(Marshall) Test


<table>
   <tr>
      <td>[Records] </td>
      <td>[Nano XML]</td>
      <td>[Nano JSON]</td>
   </tr>
   <tr>
      <td>10</td>
      <td>39</td>
      <td>60</td>
   </tr>
   <tr>
      <td>50</td>
      <td>118</td>
      <td>276</td>
   </tr>
   <tr>
      <td>300</td>
      <td>1290</td>
      <td>1664</td>
   </tr>
</table>  


###Conclusion
####For Read Test:
1. Of all the tests, the best performers are Nano JSON and RAW SAX, this is not to say that the parsing speed of JSON is identical to SAX, but JSON message format is more compact, and its payload size is almost half of XML, leading to better performance.  
2. Of all the tests, the worst performer is RAW DOM, this is obvious since DOM needs to put whole doc tree in memory and this will triger much GC on memory limited mobile device, leading to much slower speed.  
3. The performance of Nano XML is in the middle, internally, Nano XML also use SAX paring technology, but it also needs to do automatic binding by reflection, this causes Nano XML to lose almost half of its performance, however, one the other hand, the maintainability and development effciency of Nano XML are much better than SAX.    

####For Write Test:
Both Nano XML and Nano JSON perform quite good, in singe thread case they both can serialize 300 records(100KB xml, 50KB json) with less than half second.

Another finding is, there is almost a linear relationship between thread number and performance, when more threads are addded to benchmark, performance will degrade in proportion, considering the computation resource limitation of mobile device, it's not advisable to do multi-threading xml parsing on mobile device.

###My Recommendation:
If your mobile application is performance critical, Nano JSON or RAW SAX is the way to go, I perfer Nano JSON since its automatic binding feature will have better maintainability and can impove development effciency a lot.  
In other cases, Nano XML is also a good choice since it's a good balance among maintainability, development effciency, readability and performance. This is especially the case when you have a complex business domain, in such case, parsing a large amount of domain class will be a big development headache, never to say later maintenance, instead, the automatic code generation and binding features of Nano binding framework will be a big help in such case.

 










 




