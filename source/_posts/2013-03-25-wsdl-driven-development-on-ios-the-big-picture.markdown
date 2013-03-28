---
layout: post
title: "wsdl driven development on iOS - the big picture"
date: 2013-03-25 20:03
comments: true
categories: [pico]
keywords: wsdl, soap, ios, iphone
description: show the big picture of wsdl driven development on iOS platform
---

WSDL driven development is is a popular and mature development methodology on platforms like Java and .Net, tools like Axis, CXF, JAX-WS, WCF are used by many developers for rapid web service development. With WSDL as interface contract, both server side and client side proxy can be automatically generated from WSDL, developer can work with plain old interfaces/objects directly, no need to worry about low level SOAP/XML serialization/deserialization and invocation details(which are very tedious and error-prone), this kind of model driven development(or meta-data driven development) can not only significantly reduce initial development cost, but reduce long term maintenance cost. 

<!--more-->

Although nowadays there is a trend toward RESTful service(which has no formal interface definition like WSDL), many industries(such as ecommerce industry) have a complex business domain, it's very hard to expose complex business logic as RESTful service(a typical maintenance nightmare), so we will keep seeing that many enterprises will keep exposing their services as traditional SOAP or XML based web services, some examples are [Amazon Product Advertising Web Service](https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html), eBay [Finding](https://www.x.com/developers/ebay/products/finding-api), [Shopping](https://www.x.com/developers/ebay/products/shopping-api) and [Trading](https://www.x.com/developers/ebay/products/trading-api) Web Services, etc.

Can WSDL driven development be put into practice on iOS platform? Yes, now with [Pico Web Service Client Framework](https://github.com/bulldog2011/pico) and [WSDL compiler for iOS](https://github.com/bulldog2011/mwsc), you can also leverage WSDL driven development technology on iOS platform, tremendously improving your application development speed.

Let's see the big picture:

{% img center /images/pico/big_picture.png 600 800 %}

The picture above is the blueprint of WSDL driven development on iOS. The left part of the blueprint is a build-time phase view, in this phase, we will leverage [mwsc](https://github.com/bulldog2011/mwsc) wsdl compiler to automatically generate service proxy from wsdl, the service proxy alone can't be used directly, it must be integrated with the generic [Pico Web Service client framework](https://github.com/bulldog2011/pico) to take effect; The right part of the blueprint is a runtime view, a typical flow starts from your iOS app, your app issues request on service proxy, the proxy passes the request to the Pico runtime framework which will delegate the object to XML/SOAP marshalling work to Pico binding framework then send XML/SOAP request to external service through AFNetworking HTTP client component, when an XML/SOAP response is received by the HTTP client component, the Pico runtime framework will also delegate the XML/SOAP to object unmarshalling work to Pico binding framework and passes the response object to the proxy which will eventually return the response object back to the calling app. 

We used the synchronous flow in the picture just for convenient demonstration, the real call flow in the runtime is asynchronous, by leveraging NSOperation, NSOperationQueue and GCD, in order not to block main UI thread.

The Pico binding framework is the core of the Pico runtime, the real object<->xml binding magic happens here, below is the architecture of the binding runtime:

{% img center /images/pico/binding.png 500 600 %}

There are four main components:

>1. ***Marshaller*** - responsible for object to xml marshalling
2. ***Unmarshaller*** - responsible for xml to object unmarshalling
3. ***BindingSchema*** - store object<->xml mapping information, used by both Marshaller and Unmarshaller to guide the marshalling/unmarshalling process at runtime, schema is extracted from classes and is cached for better performance.
4. ***Converter*** - type converter for primitive types or frequently used types.

The marshalling/unmarshalling algorithm is recursive in nature:   

>1. for fields of primitive or frequently used types, use corresponding converter to convert the field directly.
2. for fields of object type, convert the fields of the object one by one and recursively.

The code generator component is based on [JAX-WS Wsimport](http://docs.oracle.com/javase/6/docs/technotes/tools/share/wsimport.html) and [Freemarker](http://freemarker.org/) template engine, see the whole architecture below:

{% img center /images/pico/codegen-arch.png 600 750 %}

The whole architecture can be summarized as ***Model + Template = Code***, code generation flow as following:   

>1. WSDL doc is first fed into Wsimport and a JAXB/JAX-WS model is generated.
2. The JAXB/JAX-WS model is then transformed into a language independent intermediate codegen model.
3. The codegen model and corresponding target language templates(Objective-C or Android) are then fed into the Freemarker template engine.
4. The Freemarker template engine will eventually transform the in-memory model into target code, guided by the templates.

There are other WSDL to Objective-C code generation tools, like [wsdl2objc](http://code.google.com/p/wsdl2objc/) and [SudzC](http://sudzc.com/), it's ok to use these tools to generate code from simple wsdl, but when fed with an industry level wsdl like eBay or Amazon wsdl, seems these tools will always break. Unlike these simple tools, the mwsc code generator provided by Pico framework is backed by JAX-WS Wsimport, which is mature and stable, and can recognize most standard WSDL/Schema components, by delegating the most tricky and complex wsdl model transformation task to Wsimport, the mwsc code generator solved the WSDL to Objective-C(or Android Java) problem in a simple while elegant way.

The mwsc code generator not only generates simple bean classes from WSDL/Schema, but also generates the XML<->object mapping information(schema) and record these information in the class as meta-data, the meta-data will later be leveraged by Pico binding framework to guide the XML<->Object transformation at runtime.

With a generic code generation tool and a generic web service client runtime, web service based app development on iOS platform becomes easy, there is no low level xml parsing and http handling(which are tedious, error-prone and hard to maintain) any more, developers only need to work with a plain old service interface for service invocation, now they can put their real effort on application logic and UI, leading to agile iOS app development.

In later posts, I will show how to put WSDL driven iOS app development into practice in a series of tutorials, just stay tuned.
 







