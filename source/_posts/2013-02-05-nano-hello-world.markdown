---
layout: post
title: "nano hello world"
date: 2013-02-05 18:35
comments: true
categories: nano
keywords: nano framework, xml data binding, android, jaxb
description: hello world like tutorial for nano framework
---

***Nano*** is my light-weight xml/json binding framework, it is a light-weight alternative to JAXB. Both Nano and JAXB are annotation driven, you annotate your domain classes, then use Nano or JAXB to convert Java POJO to/from XML.Two highlights of Nano are :   
>1. Tailored for Android platform.  
2. Support both xml and json binding.  

In this tutorial, I will show you simple usage of Nano in normal Java environment, in later posts, I will show you how to use Nano on Androd platform.

<!--more-->

####Basic concepts:  
>1. Marshalling or Serialization - Convert a Java object into a Xml or Json content.  
2. Unmarshalling or Deserialization - Convert Xml or Json content to a Java object.

####Prerequisite:  
>1. JDK 1.6 or above  
2. Nano 0.6.1 or above

Working with Nano is easy, just annotate your domain class with Nano annotations, later use nanoWriter.write() or nanoReader.read() to do the object / Xml(or Json) conversion.

###1. Nano Dependency
Nano 0.6.1 can be downloaded [here](https://github.com/bulldog2011/bulldog-repo/raw/master/repo/releases/com/leansoft/nano/0.6.1/nano-0.6.1-all.zip), extract the zip file, and put all 3 jars in lib folder on your project classpath.  
***Note***:  
On normal Java platform, Nano depends on Kxml and org.json library, however, on Android, Nano has no such dependency, since Kxml and org.json are built in Android Platform.

###2. Nano Annotation
For object that needs to be converted to / from XML file, it has to be annotated with Nano annotations. The annotations are pretty self-explanatory, see sample below:

{% codeblock lang:java %}

package com.leansoft.nano.sample;

import com.leansoft.nano.annotation.Attribute;
import com.leansoft.nano.annotation.Element;
import com.leansoft.nano.annotation.RootElement;

@RootElement
public class Customer {
	
	@Element
	private String name;
	
	@Element
	private int age;
	
	@Attribute
	private int id;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	
	public String toString() {
		return "[name=" + name +", age=" + age + ", id=" + id + "]";
	}
}

{% endcodeblock %}


On class level, RootElement(optional) annotation indicates that this class is Nano bindable, on field level, Element annotation indicates that this field maps to an Xml element, Attribute annotation indicates that this filed maps to an Xml attribute.  
***Note***  
field level annotations can only be used on fields(private is ok), not on getter or setter methods.

###3. Convert Object to Xml or Json
Instantiate an object, get an Xml or Json writer instance from NanoFactory class, then write the object to output(file, console or plain string). 

{% codeblock lang:java %}

Customer customer = new Customer();
customer.setId(100);
customer.setName("bulldog");
customer.setAge(30);

// Marshalling
try {
	
	File xmlFile = new File("D:\\custom_file.xml");
	
	// for pretty output
	Format format = new Format(true);
	IWriter xmlWriter = NanoFactory.getXMLWriter(format);
	
	xmlWriter.write(customer, new FileOutputStream(xmlFile));
	System.out.println("xml output : ");
	xmlWriter.write(customer, System.out);
	System.out.println();
	
	File jsonFile = new File("D:\\custom_file.json");
	
	IWriter jsonWriter = NanoFactory.getJSONWriter();
	
	jsonWriter.write(customer, new FileOutputStream(jsonFile));
	System.out.println("json output : ");
	jsonWriter.write(customer, System.out);
	System.out.println();
} catch (Exception e) {
	e.printStackTrace();
}
		

{% endcodeblock %}


Output

{% codeblock %}

xml output : 
<?xml version='1.0' encoding='utf-8' ?>
<customer id="100">
  <age>30</age>
  <name>bulldog</name>
</customer>
json output : 
{"customer":{"@id":100,"age":30,"name":"bulldog"}}

{% endcodeblock %}

###4. Convert XML or JSON back to Object
Get an Xml or Json reader instance from NanoFactory class, then read content(has just been written above) back into object instance.  
Note that when you read back, you need to explictly tell Nano the target class name.

{% codeblock lang:java %}

// Unmarshalling
try {
	
	File xmlFile = new File("D:\\custom_file.xml");
	
	IReader xmlReader = NanoFactory.getXMLReader();
	
	customer = xmlReader.read(Customer.class, new FileInputStream(xmlFile));
	System.out.println("customer read from xml : ");
	System.out.println(customer);
	
	File jsonFile = new File("D:\\custom_file.json");
	
	IReader jsonReader = NanoFactory.getJSONReader();
	
	customer = jsonReader.read(Customer.class, new FileInputStream(jsonFile));
	System.out.println("customer read from json : ");
	System.out.println(customer);
} catch (Exception e) {
	e.printStackTrace();
}
		

{% endcodeblock %}

Output
{% codeblock %}

customer read from xml : 
[name=bulldog, age=30, id=100]
customer read from json : 
[name=bulldog, age=30, id=100]

{% endcodeblock %}


###5. Now Your Turn
The usage of Nano can't be simpler, now it's your turn to try Nano framework, if necessary, you can find the whole source of this tutorial [here](https://github.com/bulldog2011/nano/tree/master/sample/helloworld).



