---
layout: post
title: "nano compare to jaxb"
date: 2013-02-06 20:44
comments: true
categories: [nano]
keywords: nano binding framework, xml json data binding, jaxb, android
description: a simple comparison between nano and jaxb
---

I just read an interesting post [How Does JAXB Compare to Simple](http://blog.bdoughan.com/2010/10/how-does-jaxb-compare-to-simple.html), since I have just built a leight-weight xml and json binding framework called [Nano](http://github.com/bulldog2011/nano), in this post I'll run a similar comparison between Nano and JAXB. By the way, since I am a lazy developer, I shamelessly copied much content from that post:), anyway, part of the content of this post should be contributed to the original author of that post.

<!--more-->

###Java Model
We will use the following model for this example. The classes represent customer data. The get/set methods have been omitted to save space.

``` java

package com.leansoft.domain.nano;

import java.util.ArrayList;
import java.util.List;

public class Customer {

	private long id;
	
	private String name;
	
	private Address address;
	
	private List<PhoneNumber> phoneNumbers;
	
	public Customer() {
		phoneNumbers = new ArrayList<PhoneNumber>();
	}
}

```

``` java

package com.leansoft.domain.nano;

public class Address {
	
	private String city;

	private String street;

}

```

``` java

package com.leansoft.domain.nano;

public class PhoneNumber {
	
	private String type;

	private String number;

}

```

###Customer Data
The following instance of Customer will be marshalled to XML using both Nano and JAXB.

``` java

package com.leansoft.domain.nano;

public class Data {
	
	public static Customer CUSTOMER;
	
    static {
    	CUSTOMER = new Customer();
    	CUSTOMER.setId(123);
    	CUSTOMER.setName("Jane Doe");
 
        Address address = new Address();
        address.setStreet("1 A Street");
        address.setCity("Any Town");
        CUSTOMER.setAddress(address);
 
        PhoneNumber workPhoneNumber = new PhoneNumber();
        workPhoneNumber.setType("work");
        workPhoneNumber.setNumber("555-WORK");
        CUSTOMER.getPhoneNumbers().add(workPhoneNumber);
 
        PhoneNumber cellPhoneNumber = new PhoneNumber();
        cellPhoneNumber.setType("cell");
        cellPhoneNumber.setNumber("555-CELL");
        CUSTOMER.getPhoneNumbers().add(cellPhoneNumber);
    }

}

```


###Marshall Code
This is the code we will use to convert the objects to XML.

####Nano
The following code will be used to marshall the instance of Customer to an OutputStream.
The Nano code is quite compact. A little technical details here, the xmlWriter instance got from NanoFactory is thread safe, and unlike JAXB, Nano internally uses an on-demand strategy to scan mapping metadata before real marshalling (and unmarshalling), metadata scan happens once per class, and then the mapping metadata will be cached.

``` java

package com.leansoft.nano.sample;

import com.leansoft.domain.nano.NanoData;
import com.leansoft.nano.IWriter;
import com.leansoft.nano.NanoFactory;

public class NanoDemo {

	public static void main(String[] args) throws Exception {
		IWriter xmlWriter = NanoFactory.getXMLWriter();
		xmlWriter.write(NanoData.CUSTOMER, System.out);
	}

}

```

####JAXB
The following code will be used to marshall the instance of Customer to an OutputStream. A couple of differences are already apparent:

>1. A JAXBContext needs to be initialized on the binding metadata before the marshal operation can occur. This initialization enables JAXB to optimize how the convertion will be done. The JAXB Context is thread safe and only needs to be created once.
2. Unlike Nano, JAXB does not format the XML by default, so we will enable this feature.
3. With no metadata specified we need to supply JAXB with a root element name (and namespace).

``` java

package com.leansoft.nano.sample;

import javax.xml.bind.JAXBContext;
import javax.xml.bind.JAXBElement;
import javax.xml.bind.Marshaller;
import javax.xml.namespace.QName;

import com.leansoft.domain.jaxb.Customer;
import com.leansoft.domain.jaxb.JaxbData;

public class JaxbDemo {

	public static void main(String[] args) throws Exception {
		JAXBContext jc = JAXBContext.newInstance(Customer.class);
		 
        Marshaller marshaller = jc.createMarshaller();
        marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);
        JAXBElement<Customer> jaxbElement = new JAXBElement<Customer>(new QName("customer"), Customer.class, JaxbData.CUSTOMER);
        marshaller.marshal(jaxbElement, System.out);
	}

}

```


###Default XML Output
First we will examine the XML output produced by both Nano and JAXB if no metadata is used to customize the output.

####Nano
Nano will only output a root tag if no metadata has been defined

``` xml

<?xml version='1.0' encoding='utf-8' ?>
<customer />

```

We will instract Nano to marshall all fields to default Xml Elements by adding @Default annotation on all classes.

``` java

package com.leansoft.domain.nano;

import java.util.ArrayList;
import java.util.List;

import com.leansoft.nano.annotation.Default;

@Default
public class Customer {

	private long id;
	
	private String name;
	
	private Address address;
	
	private List<PhoneNumber> phoneNumbers;
	
	public Customer() {
		phoneNumbers = new ArrayList<PhoneNumber>();
	}
}

```



``` java

package com.leansoft.domain.nano;

import com.leansoft.nano.annotation.Default;

@Default
public class Address {
	
	private String city;

	private String street;
	
}


```

``` java

package com.leansoft.domain.nano;

import com.leansoft.nano.annotation.Default;

@Default
public class PhoneNumber {
	
	private String type;

	private String number;

}

```

Now Nano will produce following XML:

``` xml

<customer>
  <id>123</id>
  <address>
    <street>1 A Street</street>
    <city>Any Town</city>
  </address>
  <name>Jane Doe</name>
  <phoneNumbers>
    <number>555-WORK</number>
    <type>work</type>
  </phoneNumbers>
  <phoneNumbers>
    <number>555-CELL</number>
    <type>cell</type>
  </phoneNumbers>
</customer>

```


####JAXB
JAXB will produces the followinig XML.


``` xml

<customer>
    <address>
        <city>Any Town</city>
        <street>1 A Street</street>
    </address>
    <id>123</id>
    <name>Jane Doe</name>
    <phoneNumbers>
        <number>555-WORK</number>
        <type>work</type>
    </phoneNumbers>
    <phoneNumbers>
        <number>555-CELL</number>
        <type>cell</type>
    </phoneNumbers>
</customer>

```


###Field Access
For this example we will configure our XML binding tools to interact directly with the fields(instance variables).

####Nano

Nano uses field access by default and only supports field access.

####JAXB
By default JAXB will access public fields and properties. We can configure JAXB to use field access with the following package level annotation:


``` java

@XmlAccessorType(XmlAccessType.FIELD)
package com.leansoft.domain.jaxb;
 
import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;

```


###Renaming Elements
Next we will look at how to tweak the XML output using the appropriate mapping metadata. First we will rename some elements. As you will see the amount of configuration requried is almost identical.

####Nano

For Nano, we will use @Element with a name parameter to configure the phoneNumbers property.

``` java

package com.leansoft.domain.nano;

import java.util.ArrayList;
import java.util.List;

import com.leansoft.nano.annotation.Default;
import com.leansoft.nano.annotation.Element;

@Default
public class Customer {

	private long id;
	
	private String name;

	private Address address;
	
	@Element(name="phone-number")
	private List<PhoneNumber> phoneNumbers;
	
	public Customer() {
		phoneNumbers = new ArrayList<PhoneNumber>();
	}
}

```


####JAXB

For JAXB we will use @XmlRootElement to configure the root element, and @XmlElement to configure the phoneNumbers property.

``` java

package com.leansoft.domain.jaxb;

import java.util.ArrayList;
import java.util.List;

import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement
public class Customer {

	private long id;
	
	private String name;
	
	private Address address;

	@XmlElement(name="phone-number")
	private List<PhoneNumber> phoneNumbers;
	
	public Customer() {
		phoneNumbers = new ArrayList<PhoneNumber>();
	}
}

```

####XML Output
At this point the same XML is being produced by Nano and JAXB.

``` xml

<customer>
    <id>123</id>
    <name>Jane Doe</name>
    <address>
        <city>Any Town</city>
        <street>1 A Street</street>
    </address>
    <phone-number>
        <type>work</type>
        <number>555-WORK</number>
    </phone-number>
    <phone-number>
        <type>cell</type>
        <number>555-CELL</number>
    </phone-number>
</customer>

```

###Change the Order of Elements
We will tweak the document again to make sure that when marshalling an Address object the "street" element will always appear before the "city" element.

####Nano
Current Nano framework does not support this feature.


####JAXB

For JAXB we will use @XmlType to configure the ordering of elements.

``` java

package com.leansoft.domain.jaxb;

import javax.xml.bind.annotation.XmlType;

@XmlType(propOrder={"street", "city"})
public class Address {
	
	private String city;
	
	private String street;

}

```

####XML Output

The XML output by JAXB.


``` xml

<customer>
    <id>123</id>
    <name>Jane Doe</name>
    <address>
        <street>1 A Street</street>
        <city>Any Town</city>
    </address>
    <phone-number>
        <type>work</type>
        <number>555-WORK</number>
    </phone-number>
    <phone-number>
        <type>cell</type>
        <number>555-CELL</number>
    </phone-number>
</customer>

```


###Mapping to an Attribute

Now we will look at how to tweak the XML output using the appropriate mapping metadata to produce XML attributes. As you will see the amount of configuration required is almost identical.

####Nano

For Nano we will use @Attribute to configure the id property to be represented as an XML attribute.


``` java

package com.leansoft.domain.nano;

import java.util.ArrayList;
import java.util.List;

import com.leansoft.nano.annotation.Attribute;
import com.leansoft.nano.annotation.Default;
import com.leansoft.nano.annotation.Element;

@Default
public class Customer {

	@Attribute
	private long id;
	
	private String name;
	
	private Address address;
	
	@Element(name="phone-number")
	private List<PhoneNumber> phoneNumbers;
	
	public Customer() {
		phoneNumbers = new ArrayList<PhoneNumber>();
	}
}

```

####JAXB

For JAXB we will use @XmlAttribute to configure the id property to be represented as an XML attribute.

``` java

package com.leansoft.domain.jaxb;

import java.util.ArrayList;
import java.util.List;

import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlElement;
import javax.xml.bind.annotation.XmlRootElement;

@XmlRootElement
public class Customer {

	@XmlAttribute
	private long id;
	
	private String name;
	
	private Address address;

	@XmlElement(name="phone-number")
	private List<PhoneNumber> phoneNumbers;
	
	public Customer() {
		phoneNumbers = new ArrayList<PhoneNumber>();
	}

}

```

####XML Output

The XML output is the same for both JAXB and Nano.

``` xml

<customer id="123">
    <name>Jane Doe</name>
    <address>
        <street>1 A Street</street>
        <city>Any Town</city>
    </address>
    <phone-number>
        <type>work</type>
        <number>555-WORK</number>
    </phone-number>
    <phone-number>
        <type>cell</type>
        <number>555-CELL</number>
    </phone-number>
</customer>

```

###Mapping Objects to Simple Content

To compact our document even further we will map the PhoneNumber class to a complex type with simple content.

####Nano

With Nano we will use the @Attribute and @Value annotations on the PhoneNumber class.

``` java

package com.leansoft.domain.nano;

import com.leansoft.nano.annotation.Attribute;
import com.leansoft.nano.annotation.Value;

public class PhoneNumber {
	
	@Attribute
	private String type;

	@Value
	private String number;

}

```


####JAXB

For JAXB we will use the @XmlAttribute and @XmlValue annotations on the PhoneNumber class.

``` java

package com.leansoft.domain.jaxb;

import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlValue;

public class PhoneNumber {
	
	@XmlAttribute
	private String type;
	
	@XmlValue
	private String number;

}

```

####XML Output

The XML output is the same for both JAXB and Nano.

``` xml

<customer id="123">
    <name>Jane Doe</name>
    <address>
        <street>1 A Street</street>
        <city>Any Town</city>
    </address>
    <phone-number type="work">555-WORK</phone-number>
    <phone-number type="cell">555-CELL</phone-number>
</customer>

```


###Applying Namespaces

We will now namespace qualify the XML document.

####Nano
We will use the @RootElement with a namespace parameter to specify a namespace for the Customer class.

``` java

package com.leansoft.domain.nano;

import java.util.ArrayList;
import java.util.List;

import com.leansoft.nano.annotation.Attribute;
import com.leansoft.nano.annotation.Default;
import com.leansoft.nano.annotation.Element;
import com.leansoft.nano.annotation.RootElement;

@Default
@RootElement(namespace="http://www.example.com")
public class Customer {

	@Attribute
	private long id;
	
	private String name;
	
	private Address address;
	
	@Element(name="phone-number")
	private List<PhoneNumber> phoneNumbers;

}

```

####JAXB

We can configure the namespace information using the @XmlScheam package level annotation:

``` java

@XmlAccessorType(XmlAccessType.FIELD)
@XmlSchema(namespace="http://www.example.com",
elementFormDefault=XmlNsForm.QUALIFIED)
package com.leansoft.domain.jaxb;
 
import javax.xml.bind.annotation.XmlAccessType;
import javax.xml.bind.annotation.XmlAccessorType;
import javax.xml.bind.annotation.XmlNsForm;
import javax.xml.bind.annotation.XmlSchema;

```

####XML Output

####Nano

``` xml

<customer id="123" xmlns="http://www.example.com">
  <address>
    <street>1 A Street</street>
    <city>Any Town</city>
  </address>
  <name>Jane Doe</name>
  <phone-number type="work">555-WORK</phone-number>
  <phone-number type="cell">555-CELL</phone-number>

```

####JAXB

``` xml

<customer xmlns:ns2="http://www.example.com" id="123">
    <ns2:name>Jane Doe</ns2:name>
    <ns2:address>
        <ns2:street>1 A Street</ns2:street>
        <ns2:city>Any Town</ns2:city>
    </ns2:address>
    <ns2:phone-number type="work">555-WORK</ns2:phone-number>
    <ns2:phone-number type="cell">555-CELL</ns2:phone-number>
</customer>

```


###JSON Support

It is perferred that a binding framework can produce not only XML but also JSON.

####Nano
Nano supports this feature, we only need to get a jsonWriter from Nanofactory then do marshalling:

``` java

package com.leansoft.nano.sample;

import com.leansoft.domain.nano.NanoData;
import com.leansoft.nano.IWriter;
import com.leansoft.nano.NanoFactory;

public class NanoDemo {

	public static void main(String[] args) throws Exception {
		IWriter xmlWriter = NanoFactory.getXMLWriter();
		xmlWriter.write(NanoData.CUSTOMER, System.out);
		
		IWriter jsonWriter = NanoFactory.getJSONWriter();
		jsonWriter.write(NanoData.CUSTOMER, System.out);
	}

}

```


####JAXB
As I know, without external library support, JAXB does not support json binding directly.

####JSON Output

Below is the json produced by Nano:

``` json

{"customer": {
    "@id": 123,
    "address": {
        "city": "Any Town",
        "street": "1 A Street"
    },
    "name": "Jane Doe",
    "phone-number": [
        {
            "@type": "work",
            "__value__": "555-WORK"
        },
        {
            "@type": "cell",
            "__value__": "555-CELL"
        }
    ]
}}

```



###Android Support

Android mobile platfrom is quite popular these days, it would be nice if a binding framework can support Android platform.

####Nano
Nano is tailored for Android platform, I will show you how to use Nano on Android platform in my later posts.

####JAXB
JAXB does not support Android platform, even if some people made it run on Android, the performance will be very bad since JAXB a heavy weight enterprise library targeting desktop and server side development, not mobile development.

###Summary
Both Nano and JAXB are quite easy to do simple binding stuff. If you need a mature binding framework for enterprise development, JAXB is the way to go; If you just need a light-weight alternative, or you need to do binding work on Android platform, Nano is definitely the way to go.

 



 











