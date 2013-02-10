---
layout: post
title: "nano on android tutorial 1"
date: 2013-02-10 16:09
comments: true
categories: [nano]
keywords: nano binding framework, xml binding on Android, jaxb, xml json data binding
description: a tutorial showing how to use Nano on Android to marshall objects to or unmarshall xml from sdcard.
---


This is my first post showing how to use Nano on Android, in this post, I will show you how to use Nano to export and import a set of domain object as XML to/from the SD card on Android. Nano is tailored for Android platform, you will see in this post how Nano can greatly simplify tedious and boring object and xml binding task in Android application development.

<!--more-->

###Prerequisite
Nano is quite light, On Android platform, it has no 3rd party jar dependency, all you need to do is to create a noraml Android project and add reference to the Nano jar, you can find latest Nano jar by following link [here](https://github.com/bulldog2011/nano), by the way, if necessary, you may also reference Nano source directly in your project.


###Define Domain Classes
As usual project, let's define domain classes first. The set of domain classes is small but illustrates how to serialize both aggregations and lists of other domain classes. At the root is an Organization class, that has a name, Address object and a list of Person objects. Each Person object has a name and an Address object. Below are the definitions of these domain classses, note, get/set methods are omitted to save space.

####Organization
``` java

package com.leansoft.nano.sample.domain;

import java.util.ArrayList;
import java.util.List;

import com.leansoft.nano.annotation.Attribute;
import com.leansoft.nano.annotation.Default;

@Default
public class Organization {
	
	private String name;
	
	private Address address = new Address();
	
	private List<Person> staff = new ArrayList<Person>();
	
	@Attribute
	private int count;

}

```

####Address

``` java

package com.leansoft.nano.sample.domain;

import com.leansoft.nano.annotation.Default;

@Default
public class Address {

	private String street;
	private String code;
	private String city;

}


```

####Person

``` java

package com.leansoft.nano.sample.domain;

import com.leansoft.nano.annotation.Default;

@Default
public class Person {
	
	private String name;
	private Address address = new Address();

}

```

Note, all domain classes are annotated with @Default Nano annotation, indicating that all fields of the class will be mapped to xml element during serialization, except that on the count filed of Organization, there is an explicit @Attribute annotation, indicating that the count field will be mapped to xml attribute during serialization.

Also, I've build a Generator class to auto-generate faked organization with faked person list, to save space, I don't  want to dump the Generator code here, you can find the source [here](https://github.com/bulldog2011/nano/blob/master/sample/nano-and-android/src/com/leansoft/nano/sample/domain/Generator.java).

###The Sample Flow
The sample has a quite simple flow :    
1. On applicaton startup, auto-generate a faked organization with a random number of persons.  
2. On click the 'Export Data' button, serialize the organization to xml on SD card using Nano Xml writer, then update status text with bytes written.  
3. On click the 'Import Data' button, deserialize the xml on SD card into organization object using Nano Xml reader, then update status text with person size got.  

{% img center /images/nano/nano_and_android.png 400 600 %}

The main UI of the sample cantains three components:  
1. A 'Export Data' button for serialization demo.   
2. A 'Import Data' button for deserialization demo.  
3. A TextView for status display.  

***Note***, faked data generation, export, import may be time consuming tasks depending the size of the data, executing these time consuming tasks directly on main UI thread may block it, leading to application crash, so it's a best practice to encapsulate these tasks as asyn tasks, and to use progress bar to let user know the progress of the task. I have encapsulated these tasks using Android AsyncTask structure, for details, please refer to the source of this demo project(link at the end of this post), for more background about Android AsyncTask, please refer to [its documentation](http://developer.android.com/reference/android/os/AsyncTask.html).

The buttons ***Export*** and ***Import*** have their click actions tied to the flowing methods:

``` java

Button exportButton = (Button)findViewById(R.id.exportButton);
exportButton.setOnClickListener( new OnClickListener() {

	@Override
	public void onClick(View arg0) {
		
	    ExportTask  task = new ExportTask(MainActivity.this, new OnCompletionListener() {
	        public void onCompletion(Object result) {
	            File file = (File) result;
	            appendStatus(String.format("Written %d bytes to %s", file.length(), file.getAbsolutePath()));
	        }
	    });
	    task.execute(organization, dirName, fileName);
		
	}
	
});

```

``` java

Button importButton = (Button)findViewById(R.id.importButton);
importButton.setOnClickListener(new OnClickListener() {

	@Override
	public void onClick(View v) {
		ImportTask  task = new ImportTask(MainActivity.this, new OnCompletionListener() {
	        public void onCompletion(Object result) {
	            organization = (Organization) result;
	            appendStatus(String.format("Imported organization '%s' having %d persons",
	                    organization.getName(), organization.getStaff().size()));
	        }
	    });
	    task.execute(dirName, fileName);
		
	}
	
});

```

The ***ExportTask*** and ***ImportTask*** classes have logic where real Nano marshalling and unmarshalling magic happens:

####Export Logic
``` java

@Override
protected File doInBackground(Object... args) {
    try {
        Organization organization = (Organization) args[0];
        String dirName = (String) args[1];
        String fileName = (String) args[2];
        File file = getFile(dirName,  fileName);
        Writer out = openForWrite(file);
        IWriter xmlWriter = NanoFactory.getXMLWriter();
        xmlWriter.write(organization, out);
        out.close();
        return file;
    } catch (Exception e) {
        Log.d(LOG, "Failed to export. ", e);
        return null;
    }
}

```

####Import Logic
``` java

protected Organization doInBackground(Object... args) {
    try {
        String dirName  = (String) args[0];
        String fileName = (String) args[1];
        Reader in = openForRead(getFile(dirName,  fileName));
        IReader xmlReader = NanoFactory.getXMLReader();
        Organization org = xmlReader.read(Organization.class, in);
        in.close();
        return org;
    } catch (Exception e) {
        Log.d(LOG, "Failed to import. ", e);
        return null;
    }
}

```

Now you see the power of Nano : there are only 2 lines of marshalling and unmarshalling code, amazing!


###The Resulting XML
It's easy to pull out the XML file from the SD card using the File Explorer of DDMS.

And here is an extract of its contents:


```xml

<?xml version='1.0' encoding='utf-8' ?>
<organization count="88">
  <address>
    <street>Street LY07</street>
    <code>VXUSB</code>
    <city>City SWT</city>
  </address>
  <staff>
    <address>
      <street>Street VRAW</street>
      <code>6NEA8</code>
      <city>City JLN</city>
    </address>
    <name>name ND4Q</name>
  </staff>
  <staff>
    <address>
      <street>Street XW8P</street>
      <code>DNSP7</code>
      <city>City PXP</city>
    </address>
    <name>name 5OMA</name>
  </staff>

...

  <name>Org IN3V9</name>
</organization>


```

###The Source
You can get the whole source of this demo project [here](https://github.com/bulldog2011/nano/tree/master/sample/nano-and-android), by the way, some source is adapted from [this post](http://blog.ribomation.com/2011/07/android-and-xml/), so part of the content of this post should be contributed to the author of that post.


###Conclusion
Without automatic xml binding tool, data serialization and deserialization on Android is a tedious and time-consuming development task, needless to say later maintenance. The Nano xml binding framework can greatly accelerate xml based Android application development by automating the tedious and error-prone xml binding tasks, letting developers put more focus on their application logic instead of xml parsing.

















