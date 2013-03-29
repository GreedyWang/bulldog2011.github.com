---
layout: post
title: "Pico Tutorial 2 - a Currency Converter Sample"
date: 2013-03-28 20:38
comments: true
categories: [pico]
keywords: ios, iphone, soap, wsdl
description: Pico Tutorial 2 - a Currency Converter Sample
---

This is the second tutorial of Pico Tutorial series, in the [first tutorial](http://bulldog2011.github.com/blog/2013/03/27/pico-tutorial-a-stockquote-sample/), I showed how to use Pico with a service called [StockQuote](http://www.webservicex.net/ws/WSDetails.aspx?CATID=2&WSID=9) from webserivceX.NET. Today I will show you how to use Pico with another service called [CurrencyConverter](http://www.webservicex.net/ws/WSDetails.aspx?CATID=2&WSID=10), also from webserviceX.NET, in first tutorial, I showed you how to reference Pico as a static library, in this tutorial, I will show you how to reference the Pico source in your project. By the way, since the wsdl driven development process in both tutorials are quite similar, I won't repeat too much details in this tutorial, I suppose you have already read tutorial one and basically understand the wsdl driven development process supported by Pico.

The whole source of this tutorial is [here](https://github.com/bulldog2011/pico/tree/master/Examples/CurrencyConverter).

Let's cut to to the point:

<!--more-->

##Step 1 - Generate Objective-C Proxy from WSDL

Download [mwsc](https://github.com/bulldog2011/mwsc) and run following command in terminal to generate the proxy:

{% codeblock %}

bin/mwsc -pico -d generated http://www.webservicex.net/CurrencyConvertor.asmx?WSDL

{% endcodeblock %}

A few comments to the generated code:


>* By default, the proxy code will be generated in the sub-folder corresponding to the target namespace of the wsdl.
* There is a geneated folder called `client`, the SOAP and XML proxy interface will be generated in this folder.
* There is a generted folder called `common`, a common header file will be generated in this folder, the common header includes headers of all types generated from wsdl/schema, use this header file can free you from writing many import statements in your project when you build request or handle response needed by service call.

##Step 2 - Create New iOS Project, Add Pico Library and Generated Proxy into Your Project

Create a new simple iOS single view application named "CurrencyConverter", don't choose ARC, download Pico source and drag the whole `PicoSource` folder into the project, choose "Copy items to destination group's folder" and "add to targets" when prompted. Then do following settings to the project:


>1. In Target Build Setting, add the `-ObjC` flag to your "Other Linker flags".
2. In Target Build Setting, add `/usr/include/libxml2` to your "Header Search Paths"
3. In Target Build Phases, link binary with library `libxml2.dylib`

Build the the project to ensure that it can build successfully.

Now drag the proxy generated in step 1 into the project,  choose "Copy items to destination group's folder" and "add to targets" when prompted.

Build the the project again to ensure that it can build successfully.

The finished project should look like the screen shot below:

{% img center /images/pico/tutorial02/screen_shot1.png 600 800 %}

##Step 3 - Implement Appliction Logic and UI, Call Proxy to Invoke Web Service as Needed.

First, create a shared serivice client as below:

{% codeblock CurrencyConverterSerivceClient.h lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/CurrencyConverter/CurrencyConverter/CurrencyConverterSerivceClient.h source %}

#import "CurrencyConvertorSoap_SOAPClient.h"

@interface CurrencyConverterSerivceClient : CurrencyConvertorSoap_SOAPClient

+ (CurrencyConverterSerivceClient *)sharedClient;

@end

{% endcodeblock %}

{% codeblock CurrencyConverterSerivceClient.m lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/CurrencyConverter/CurrencyConverter/CurrencyConverterSerivceClient.m source  %}

#import "CurrencyConverterSerivceClient.h"
static NSString *const currencyConverterServiceURLString = @"http://www.webservicex.net/CurrencyConvertor.asmx";

@implementation CurrencyConverterSerivceClient

+ (CurrencyConverterSerivceClient *)sharedClient {
    static CurrencyConverterSerivceClient *_sharedClient = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _sharedClient = [[CurrencyConverterSerivceClient alloc] initWithEndpointURL:[NSURL URLWithString:currencyConverterServiceURLString]];
    });
    
    return _sharedClient;
}

@end

{% endcodeblock %}

Now open ViewController_iPhone.xib in interface builder, then add a few UI components like following screen shot:

{% img center /images/pico/tutorial02/ui.png 300 500 %}

Add `IBOutlet` properties and `IBAction` method in [ViewController.h](https://github.com/bulldog2011/pico/blob/master/Examples/CurrencyConverter/CurrencyConverter/ViewController.h), then wire the properties and method with UI components accordingly, bascially, the application will convert the `From` currency to `To` currency and show conversion rate, when the `Convert` button is clicked(which will trigger `onConvertClicked` method internally).

Now implement the `onConvertClicked` method by invoking service as below:

{% codeblock ViewController.m lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/CurrencyConverter/CurrencyConverter/ViewController.m source  %}

#import "ViewController.h"
#import "CurrencyConverterSerivceClient.h"
#import "SOAP11Fault.h"



#pragma mark - UI Event Handlers
- (IBAction)onConvertClicked:(id)sender {
    
    if (!self.fromCurrencyTextField.text.length || !self.toCurrencyTextField.text.length)
    {
        UIAlertView* alert = [[UIAlertView alloc]initWithTitle:@"Invalid Parameters" message:@"Please enter valid from and to currency types and try again" delegate:self cancelButtonTitle:@"OK" otherButtonTitles: nil];
        [alert show];
        return;
    }
    
    [self.activityIndicator setHidden:NO];
    [self.activityIndicator startAnimating];
    
    // Get shared client
    CurrencyConverterSerivceClient *client = [CurrencyConverterSerivceClient sharedClient];
    client.debug = YES; // enable message logging
    client.timeoutInverval = 10; // http timeout in seconds
    
    // build request
    ConversionRate *request = [[[ConversionRate alloc] init] autorelease];
    request.fromCurrency = self.fromCurrencyTextField.text;
    request.toCurrency = self.toCurrencyTextField.text;
    
    // make API call
    [client conversionRate:request success:^(ConversionRateResponse *responseObject) {
        
        // success handling logic
        [self.activityIndicator stopAnimating];
        
        UIAlertView* alert = [[UIAlertView alloc]initWithTitle:@"Success!" message:[NSString stringWithFormat:@"Currency Conversion Rate is %@",responseObject.conversionRateResult] delegate:self cancelButtonTitle:@"OK" otherButtonTitles: nil];
        [alert show];
        
    } failure:^(NSError *error, id<PicoBindable> soapFault) {
        
        [self.activityIndicator stopAnimating];
        
        // error handling logic
        if (error) { // http or parsing error
            UIAlertView* alert = [[UIAlertView alloc]initWithTitle:@"Error" message:error.localizedDescription delegate:self cancelButtonTitle:@"OK" otherButtonTitles: nil];
            [alert show];
        } else if (soapFault) { // soap fault
            SOAP11Fault *soap11Fault = (SOAP11Fault *)soapFault;
            UIAlertView* alert = [[UIAlertView alloc]initWithTitle:@"SOAP Fault" message:soap11Fault.faultstring delegate:self cancelButtonTitle:@"OK" otherButtonTitles: nil];
            [alert show];
        }
    }];
    
}

{% endcodeblock %}

Please don't forget to include the shared CurrencyConverterSerivceClient.

##Final Step - Run the Demo

See a sceen shot below:

{% img center /images/pico/tutorial02/screen_shot2.png 300 500 %}

And the debug output:

{% codeblock %}

2013-03-28 21:45:27.641 CurrencyConverter[2153:c07] Sending request to : http://www.webservicex.net/CurrencyConvertor.asmx
2013-03-28 21:45:27.642 CurrencyConverter[2153:c07] Request message:
2013-03-28 21:45:27.643 CurrencyConverter[2153:c07] <?xml version="1.0" encoding="UTF-8" ?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://www.webserviceX.NET/">
	<soapenv:Body>
		<ConversionRate>
			<FromCurrency>USD</FromCurrency>
			<ToCurrency>CAD</ToCurrency>
		</ConversionRate>
	</soapenv:Body>
</soapenv:Envelope>
2013-03-28 21:45:27.643 CurrencyConverter[2153:c07] Request HTTP Headers : 
{
    Accept = "text/xml";
    "Accept-Language" = "en, fr, de, ja, nl, it, es, pt, pt-PT, da, fi, nb, sv, ko, zh-Hans, zh-Hant, ru, pl, tr, uk, ar, hr, cs, el, he, ro, sk, th, id, ms, en-GB, ca, hu, vi, en-us;q=0.8";
    "Content-Type" = "text/xml";
    SOAPAction = "http://www.webserviceX.NET/ConversionRate";
    "User-Agent" = "CurrencyConverter/1.0 (iPhone Simulator; iOS 6.0; Scale/1.00)";
}
2013-03-28 21:45:28.495 CurrencyConverter[2153:1b03] Response HTTP status : 
200
2013-03-28 21:45:28.495 CurrencyConverter[2153:1b03] Response HTTP headers : 
{
    "Cache-Control" = "private, max-age=0";
    "Content-Encoding" = gzip;
    "Content-Length" = 316;
    "Content-Type" = "text/xml; charset=utf-8";
    Date = "Thu, 28 Mar 2013 13:45:47 GMT";
    Server = "Microsoft-IIS/7.0";
    Vary = "Accept-Encoding";
    "X-AspNet-Version" = "4.0.30319";
    "X-Powered-By" = "ASP.NET";
}
2013-03-28 21:45:28.495 CurrencyConverter[2153:5503] Response message : 
2013-03-28 21:45:28.495 CurrencyConverter[2153:5503] <?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"><soap:Body><ConversionRateResponse xmlns="http://www.webserviceX.NET/"><ConversionRateResult>1.0167</ConversionRateResult></ConversionRateResponse></soap:Body></soap:Envelope>

{% endcodeblock %}

There are other similar demos in the [Examples](https://github.com/bulldog2011/pico/tree/master/Examples) folder of Pico source, like the [BarCode](https://github.com/bulldog2011/pico/tree/master/Examples/BarCode) demo which calls web service that will return base64 encoded barcode data and the [Weather](https://github.com/bulldog2011/pico/tree/master/Examples/Weather) demo which shows the weather given a zip code, see screen shots below, I won't create tuturials for all these simple demos, since they are quite similar. Next time, I plan to show you how to use Pico with industrial level web serivces, like Amazon and eBay web serivces, just stay tuned.

The screen shot of barcode demo:
{% img center /images/pico/tutorial02/screen_shot3.png 300 500 %}


The screen shot of weather demo:
{% img center /images/pico/tutorial02/screen_shot4.png 300 500 %}





