---
layout: post
title: "Pico Tutorial - a StockQuote sample"
date: 2013-03-27 20:07
comments: true
categories: [pico]
keywords: wsdl, soap, iOS, iPhone
description: Pico Tutoiral Series 1 - a StockQuote sample
---

This is the first post of Pico Tutorial series, in this post, I will show you how easy to use [Pico framework](https://github.com/bulldog2011/pico) to put WSDL driven application development on iOS platform into practice. If this is the first time you get to know Pico, then after this tutorial, you will bascially understand what Pico can do for you, and the basic development process when using Pico to run WSDL driven development on iOS. If you want to see the big picture, plese read [this post](http://bulldog2011.github.com/blog/2013/03/25/wsdl-driven-development-on-ios-the-big-picture/) first.

The whole source of this demo is [here](https://github.com/bulldog2011/pico/tree/master/Examples/StockQuote).

WSDL driven development using Pico framework has following steps:

>1. Genearte Objective-C proxy from WSDL
2. Create new iOS project, add Pico runtime and generated proxy into your project
3. Implement appliction logic and UI, call proxy to invoke web service as needed.

Let me cut to the point and show you each step using a simple while popular [StockQueue](http://www.webservicex.net/ws/WSDetails.aspx?WSID=9&CATID=2) web serivce from WebserviceX.NET.

<!--more-->

##Step 1 - Generate Objective-C Proxy from WSDL
Pico has an accompanying code generator which can generate Objective-C proxy from wsdl, the tool is called [mwsc](https://github.com/bulldog2011/mwsc), please download the latest zip package [here](https://github.com/bulldog2011/bulldog-repo/blob/master/repo/releases/com/leansoft/mwsc/0.5.0/mwsc-0.5.0-bin.zip) then extract it into your local folder. 
***Note*** : mwsc code generator needs Java 1.6 or above to run, so please ensure Java is installed on your MacOS, if not, please install it first.

The command line script `mwsc` is in the bin folder, please add executable right to it before you run it, you may optionally create a folder as your code generation target(for example, `generated`), otherwise, mwsc will generate code in current folder, now lets generte code from wsdl by running flowing command in the terminal:

{% codeblock %}

bin/mwsc -pico -d generated http://www.webservicex.net/stockquote.asmx?WSDL

{% endcodeblock %}

If everything works fine, you will see the code generator output `generating code…` and `done` at the end, the target proxy will be generated in the `generated` folder, the code generator may throw a few warnings, asking us to add extension options for some ports, since we won't use those ports, just ignore them right now, it's all right as long as the SOAP port is generated correctly.

##Step 2 - Create New iOS Project, Add Pico Library and Generated Proxy into Your Project
For this demo, we just create a simple iOS single view application, please don't choose ARC when you create the project, since Pico does not support ARC yet.

Now we need to import Pico library into the project first, there are two options, you can either include all Pico source into your project, or add Pico as a static library into your project, in this tutorial, I just show you the second option, if you want to include the whole Pico source, please see Pico github site for instructions.

Suppose you have downloaded Pico source project from github site, then:

>1.  Drag the Pico xcodeproj into your project, 
2.  In the Build Phases of the target,  add `libPico.a` and `libxml2.dylib` to "Link Binary With Libraries" section.
3.  In the Build Setting of the target, add [your path to Pico source] to "User Header Search Paths" setting, choose "recursive" seach path.

Now create a group in your project called `Proxy`(or other meaningful name you choose), then drag the proxy generated in step 1 into this group, choose "Copy items to destination group's folder" and "add to targets" when prompted.

Now since both Pico library and the StockQuote web service proxy have been added in the project, you can try to build the project, if no build error, job well done, you can continue to the next step, otherwise, please do some toubleshooting, or check the source of this tutourial(in the Examples folder of Pico).

Below is an Xcode screenshot after finish this step(please save the screenshot to local then view if the web version is not large enough to view clearly):

{% img center /images/pico/tutorial01/screen_shot1.png 600 800 %}

Note, in the demo project, I also added a third party library called Toast which can produce toast style message, this is just for the convenience of demo, it is not necessary for you to do so in your real project.


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

The code is quite self-explanatory, the `shareClient` static factory method just returns a StockQuoteServiceClient(which extends StockQuoteSoap_SOAPClient generated from wsdl, you can find StockQuoteSoap_SOAPClient in the `Proxy` group) instance with specified target service endpoint address, and there will be only one such client instance within the applicaiton.


Now time to the UI part, for this simple Demo, we only need an UITextField for company symbol input, an UIButton to triget getQuote service call and an UITextView for result dislay, quite simple, see definition in header file [ViewController.h](https://github.com/bulldog2011/pico/blob/master/Examples/StockQuote/StockQuote/ViewController.h) and instantiation in implementation file [ViewController.m](https://github.com/bulldog2011/pico/blob/master/Examples/StockQuote/StockQuote/ViewController.m), in method `viewDidLoad`.

Now let's implement application logic by calling the getQuote service, with the help of the StockQuote service proxy generated from WSDL, web service call through Pico is extremely simple, let's review and understand the generated getQuote proxy interface first before writing calling logic:

{% codeblock StockQuoteSoap_SOAPClient.m lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/StockQuote/StockQuote/Proxy/client/StockQuoteSoap_SOAPClient.h source  %}

// Generated by wsdl compiler for ios/objective-c
// DO NOT CHANGE!


#import <Foundation/Foundation.h>
#import "PicoSOAPClient.h"
#import "GetQuoteResponse.h"
#import "GetQuote.h"


/**
 This class is the SOAP client to the StockQuoteSoap Web Service.
*/ 
@interface StockQuoteSoap_SOAPClient : PicoSOAPClient {

}

/**
 Get Stock quote for a company Symbol
*/
-(void)getQuote:(GetQuote *) requestObject 
      success:(void (^)(GetQuoteResponse *responseObject))success
      failure:(void (^)(NSError *error, id<PicoBindable> soapFault))failure;

@end

{% endcodeblock %}

This is just a plain old interface generated from wsdl, unlike normal interface, this interface is asynchronous, when you call `getQuote` service, you need to provide a request of type `GetQuote` as first parameter, also you need to provide(or register) one `success` and one `failure` callbacks using Objective-C block, success callback will be called if the service invocation succeed, and you will get a response object of type `GetQuoteResponse`, usually, in success callback, you implement response handling logic and update UI according to the reasponse; failure callback will be called if the service invocation fail, or there is any HTTP or Pico parsing error, you may either get a `NSError`(indicationg HTTP or Pico parsing error), or get a `SOAPFault`(indicating service call fail), usually, you implement error handling logic in failure callback and update UI accordingly.

By the way, although not necessary, I suggest you to review the proxy code generated by mwsc, this will help you better understand the inner working of Pico.

Let's see the service call implementation of the demo, it's in the `getQuotePressed` method in the ViewController implementation, `getQuotePressed` will be triggered when you click the GetQuote button on the UI:

{% codeblock ViewController.m lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/StockQuote/StockQuote/ViewController.m source  %}

- (void)getQuotePressed:(id)sender
{
    // Hide the keyboard.
    [_symbolText resignFirstResponder];
    
    if (_symbolText.text.length > 0) {
        
        // start progress activity
        [self.view makeToastActivity];
        
        // Get shared service client
        StockQuoteServiceClient *client = [StockQuoteServiceClient sharedClient];
        client.debug = YES; // enable request/response message logging
        
        // Build request object
        GetQuote *request = [[[GetQuote alloc] init] autorelease];
        request.symbol = _symbolText.text;
        
        // make API call and register callbacks
        [client getQuote:request success:^(GetQuoteResponse *responseObject) {
            
            // stop progress activity
            [self.view hideToastActivity];
            
            // show result
            _resultText.text = responseObject.getQuoteResult;
        } failure:^(NSError *error, id<PicoBindable> soapFault) {
            
            // stop progress activity
            [self.view hideToastActivity];
            
            if (error) { // http or parsing error
                [self.view makeToast:[error localizedDescription] duration:3.0 position:@"center" title:@"Error"];
            } else if (soapFault) {
                SOAP11Fault *soap11Fault = (SOAP11Fault *)soapFault;
                [self.view makeToast:soap11Fault.faultstring duration:3.0 position:@"center" title:@"SOAP Fault"];
            }
        }];
        
    }
    
}

{% endcodeblock %}


The call logic can't simpler, I've added comments in the service call code for you to better understand the call flow. Also, I've enabled the debug mode of the proxy, so you can see the detailed request/response messages when you run the demo, this feature is extremely useful for troubleshooting.



##Final Step - Run the Demo

Now it's time to run the demo, let's try it in the simulator(you may use a real device of cause ), I just want to see the stock quote of Apple(company symbol AAPL), belew is a screenshot of the simulator.

{% img center /images/pico/tutorial01/screen_shot2.png 300 500 %}

The StockQuote web service just return stock quote information in XML format, so you see the result in xml.

You can also find the debug output in the XCode, like below:

{% codeblock  %}

2013-03-28 11:33:35.793 StockQuoteDemo[761:c07] Sending request to : http://www.webservicex.net/stockquote.asmx
2013-03-28 11:33:35.794 StockQuoteDemo[761:c07] Request message:
2013-03-28 11:33:35.794 StockQuoteDemo[761:c07] <?xml version="1.0" encoding="UTF-8" ?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://www.webserviceX.NET/">
	<soapenv:Body>
		<GetQuote>
			<symbol>AAPL</symbol>
		</GetQuote>
	</soapenv:Body>
</soapenv:Envelope>
2013-03-28 11:33:35.795 StockQuoteDemo[761:c07] Request HTTP Headers : 
{
    Accept = "text/xml";
    "Accept-Language" = "en, fr, de, ja, nl, it, es, pt, pt-PT, da, fi, nb, sv, ko, zh-Hans, zh-Hant, ru, pl, tr, uk, ar, hr, cs, el, he, ro, sk, th, id, ms, en-GB, ca, hu, vi, en-us;q=0.8";
    "Content-Type" = "text/xml";
    SOAPAction = "http://www.webserviceX.NET/GetQuote";
    "User-Agent" = "StockQuoteDemo/1.0 (iPhone Simulator; iOS 6.0; Scale/1.00)";
}
2013-03-28 11:33:37.367 StockQuoteDemo[761:4a03] Response HTTP status : 
200
2013-03-28 11:33:37.367 StockQuoteDemo[761:4a03] Response HTTP headers : 
{
    "Cache-Control" = "private, max-age=0";
    "Content-Encoding" = gzip;
    "Content-Length" = 634;
    "Content-Type" = "text/xml; charset=utf-8";
    Date = "Thu, 28 Mar 2013 03:33:54 GMT";
    Server = "Microsoft-IIS/7.0";
    Vary = "Accept-Encoding";
    "X-AspNet-Version" = "4.0.30319";
    "X-Powered-By" = "ASP.NET";
}
2013-03-28 11:33:37.367 StockQuoteDemo[761:1c03] Response message : 
2013-03-28 11:33:37.367 StockQuoteDemo[761:1c03] <?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"><soap:Body><GetQuoteResponse xmlns="http://www.webserviceX.NET/"><GetQuoteResult>&lt;StockQuotes&gt;&lt;Stock&gt;&lt;Symbol&gt;AAPL&lt;/Symbol&gt;&lt;Last&gt;452.08&lt;/Last&gt;&lt;Date&gt;3/27/2013&lt;/Date&gt;&lt;Time&gt;4:00pm&lt;/Time&gt;&lt;Change&gt;-9.056&lt;/Change&gt;&lt;Open&gt;456.80&lt;/Open&gt;&lt;High&gt;456.80&lt;/High&gt;&lt;Low&gt;450.7301&lt;/Low&gt;&lt;Volume&gt;11836042&lt;/Volume&gt;&lt;MktCap&gt;424.5B&lt;/MktCap&gt;&lt;PreviousClose&gt;461.136&lt;/PreviousClose&gt;&lt;PercentageChange&gt;-1.96%&lt;/PercentageChange&gt;&lt;AnnRange&gt;419.00 - 705.07&lt;/AnnRange&gt;&lt;Earns&gt;44.107&lt;/Earns&gt;&lt;P-E&gt;10.45&lt;/P-E&gt;&lt;Name&gt;Apple Inc.&lt;/Name&gt;&lt;/Stock&gt;&lt;/StockQuotes&gt;</GetQuoteResult></GetQuoteResponse></soap:Body></soap:Envelope>

{% endcodeblock %}

If you have any problem to run the demo, please check the debug output first.

Now you see the Power of pico framework, you don't get troubled with error prone and tedious SOAP/XML parsing or http handling, the generic Pico framework will do these stuff for you, you only need to use a plain old asynchronous interface for service invocation, this can not only accelerate application development, but reduce the long term maintenance cost.

StockQuote is just a bare minimum demo, there are other more featured demos(like Amazon, eBay search demo) in the Examples folder of Pico source project, please try them to better understand how Pico works, then you can begin to develop your own service based applications, using wsdl driven methodology supported by Pico.

See your next great application!









 




