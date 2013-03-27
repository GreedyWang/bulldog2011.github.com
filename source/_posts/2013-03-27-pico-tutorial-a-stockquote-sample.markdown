---
layout: post
title: "Pico Tutorial - a StockQuote sample"
date: 2013-03-27 20:07
comments: true
categories: [pico]
keywords: wsdl, soap, iOS, iPhone
description: Pico Tutoiral Series 1 - a StockQuote sample
---

This is the first post of Pico Tutorial serices, in this post, I will show you how easy to use [Pico framework](https://github.com/bulldog2011/pico) to carry out WSDL driven application development on iOS platform. If this is the first time you get to know Pico, then after this tutorial, you will bascially understand what Pico can do for you, and the basic development flow when using Pico to carry out WSDL driven development on iOS. If you want to see the big picture, plese read [this post](http://bulldog2011.github.com/blog/2013/03/25/wsdl-driven-development-on-ios-the-big-picture/) first.

WSDL driven development using Pico framework has following steps:

>1. Genearte Objective-C proxy from WSDL
2. Create new iOS project, add Pico runtime and generated proxy into your project
3. Implement appliction logic and UI, call proxy to invoke web service as needed.

Let me cut to the point and show you each step using a simple while popular [StockQueue](http://www.webservicex.net/ws/WSDetails.aspx?WSID=9&CATID=2) web serivce from WebserviceX.NET.

<!--more-->

##Step 1 - Generate Objective-c Proxy from WSDL
Pico has an accompanying code generator which can generate Objective-C proxy from wsdl, the tool is called [mwsc](https://github.com/bulldog2011/mwsc), please download the latest zip package [here](https://github.com/bulldog2011/bulldog-repo/blob/master/repo/releases/com/leansoft/mwsc/0.5.0/mwsc-0.5.0-bin.zip) then extract it into you local folder. 
***Note*** : mwsc code generator needs Java 1.6 or above to run, so please ensure Java is installed on your MacOS, if not, please install it first.

The command line script `mwsc` is in the bin folder, please add executable right to it before you run it, you may optionally created a folder as your code generation target(for example, `generated`), otherwise, mwsc will generate code in current folder, now lets generte code from wsdl by running flowing command in the terminal:

{% codeblock %}

bin/mwsc -pico -d generated http://www.webservicex.net/stockquote.asmx?WSDL

{% endcodeblock %}

If everything works fine, you will see the code generator output `generating code…` and `done` in the end, the target proxy will be generated in the `generated` folder, the code generator may throw a few warnings, asking us to add extension options for some ports, since we won't use those ports, so just ignore them right now, it's all right as long as the SOAP port is generated correctly.

##Step 2 - Create New iOS Project, Add Pico Library and Generated Proxy into Your Project
For this demo, we just create a simple iOS single view application, please don't choose ARC when you create the project, since Pico does not support ARC yet.

Now we need to import Pico library into the project first, there are two options, you can either include all Pico source into your project, or add Pico as static library into your project, in this tutorial, I just show you the second option, if you want to include the whole Pico source, please see Pico github site for instructions.

Suppose you have downloaded Pico source project from github site, then:

>1.  Drag the Pico xcodeproj into your project, 
2.  In the Build Phases of the target,  add `libPico.a` and `libxml2.dylib` to "Link Binary With Libraries" section.
3.  In the Build Setting of the target, add [path to Pico source] to your "User Header Search Paths", choose "recursive" seach path.

Now create a group in your project called `Proxy`(or other meaningful name you choose), then drap the step 1 generated proxy into this group, choose "Copy items to destination group's folder" and "add to targets" when prompted.

Now since both Pico library and the StockQuote web service proxy have been added in the project, you can try to build the project, if no build error, job well done, you can continue to the next step, otherwise, please do some toubleshooting, or check the source of this tutourial(in the Examples folder of Pico).

Below is a screenshot after finish this step:

{% img center /images/pico/tutorial01/screen_shot1.png 600 800 %}

Note, in the demo project, I also added a third part library called Toast which can generate toast style message, this is just for the convenience of demo, it is not necessary to do so in your real project.


##Step 3 - Implement Appliction Logic and UI, Call Proxy to Invoke Web Service as Needed.
Pico use [AFNetworking](https://github.com/AFNetworking/AFNetworking) Http client for low level communication, as an AFNetworking best practice, it's not necessary for you to initiate a new client everytime you call service, a singleton client instance is enough for the whole application, so before writing any applicaiton logic, lets create a a shared StockQuote service client for later use, see code below:

{% codeblock StockQuoteServiceClient.h lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/StockQuote/StockQuote/StockQuoteServiceClient.h source %}

#import "StockQuoteSoap_SOAPClient.h"

@interface StockQuoteServiceClient : StockQuoteSoap_SOAPClient

+ (StockQuoteServiceClient *)sharedClient;

@end

{% endcodeblock %}


{% codeblock StockQuoteServiceClient.m lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/StockQuote/StockQuote/StockQuoteServiceClient.m source  %}

#import "StockQuoteServiceClient.h"

static NSString *const stockQuoteServiceURLString = @"http://www.webservicex.net/stockquote.asmx";

@implementation StockQuoteServiceClient

+ (StockQuoteServiceClient *)sharedClient {
    static StockQuoteServiceClient *_sharedClient = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _sharedClient = [[StockQuoteServiceClient alloc] initWithEndpointURL:[NSURL URLWithString:stockQuoteServiceURLString]];
    });
    
    return _sharedClient;
}

@end

{% endcodeblock %}

The code is quite self-explanatory, the `shareClient` static factory method just returns a StockQuoteServiceClient(SOAP proxy generated from wsdl, you can find it in the `Proxy` group) with specified target service endpoint address, and there will be only one client instance within the applicaiton.


//to be continue






 




