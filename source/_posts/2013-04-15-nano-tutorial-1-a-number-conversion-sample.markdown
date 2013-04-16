---
layout: post
title: "Nano Tutorial 1 - a Number Conversion Sample"
date: 2013-04-15 20:14
comments: true
categories: [nano]
keywords: wsdl, soap, android, java
description: Nano Tutorial Series 1 - a Number Conversion Sample
---

This is the first post of Nano Tutorial Series, in this post, I will show you how easy to use [Nano framework](https://github.com/bulldog2011/nano) to put WSDL driven application development on Android platform into practice. If this is the first time you get to know Nano, then after this tutorial, you will basically understand what Nano can do for you, and the basic development process when using Nano to carry out WSDL driven development on Android. If you want to see the big picture, please read [this post](http://bulldog2011.github.io/blog/2013/04/15/wsdl-driven-development-on-android-the-big-picture/) first. 

The whole source of this demo is [here](https://github.com/bulldog2011/nano/tree/master/sample/webservice/NumberConverter).

WSDL driven development using Nano framework has following steps:

>1. Generate Java proxy from WSDL
2. Create new Android project, add Nano runtime and generated proxy into your project
3. Implement appliction logic and UI, call proxy to invoke web service as needed.

Let me cut to the point and show you each step using a simple [Number Conversion](http://www.dataaccess.com/webservicesserver/numberconversion.wso) web serivce from [Data Access Worldwide](http://www.dataaccess.com/).

<!--more-->

##Step 1 - Generate Java Proxy from WSDL
Nano has an accompanying code generator which can generate Java proxy for Android from wsdl, the tool is called [mwsc](https://github.com/bulldog2011/mwsc), please download the latest zip package [here](https://github.com/bulldog2011/bulldog-repo/blob/master/repo/releases/com/leansoft/mwsc/0.6.0/mwsc-0.6.0-bin.zip) then extract it into your local folder. 
***Note*** : mwsc code generator needs Java 1.6 or above to run, so please ensure Java is installed on your OS, if not, please install it first.

The command line script `mwsc.bat` is in the bin folder, note, `mwsc.bat` is for windows, for linux, please use `mwsc` in the same folder. Before code generation, you may optionally create a folder as your code generation target(for example, `generated`), otherwise, mwsc will generate code in current folder. Now let's generate code from wsdl by running flowing command in the terminal:

{% codeblock %}

bin\mwsc.bat -nano -d generated http://www.dataaccess.com/webservicesserver/numberconversion.wso?WSDL

{% endcodeblock %}

If everything works fine, you will see the code generator output `generating code…` and `done` at the end, the target proxy will be generated in the `generated` folder.

##Step 2 - Create New Android Project, Add Nano Library and Generated Proxy into Your Project
Now let's create a simple Android project in eclipse(or other IDE you prefer), then you need to:   
>1. Download [latest nano library jar](https://github.com/bulldog2011/bulldog-repo/blob/master/repo/releases/com/leansoft/nano/0.7.0/nano-0.7.0.jar) and add it to the `libs` folder of your Android project.
2. Drag the above generated proxy into the `src` folder of your Android project.
3. Add following user permissions in the `AndroidManifest.xml` file to enable network access.   
``` xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

Now since both Nano runtime library and the NumberConversion web service proxy have been added in the project, you can try to refresh(or build) the project, if no build error, job well done, you can continue to the next step, otherwise, please do some troubleshooting, or check the source of this tutorial(in the sample/webservice folder).

Below is an eclipse package explorer screenshot after finishing this step:

{% img center /images/nano/tutorial01/screen_shot1.png 600 800 %}

##Step 3 - Implement Appliction Logic and UI, Call Proxy to Invoke Web Service as Needed.
A SOAP web service client called `NumberConversionSoapType_SOAPClient` has been generated in the `com.dataaccess.webservicesserver.client` package, it's the main proxy to the external number conversion service, you can invoke number conversion service by calling methods on a `NumberConversionSoapType_SOAPClient` instance. As a Nano best practice, it's not necessary for you to initiate a new client everytime you need to call service, a singleton client instance is enough for the whole application, so before writing any application logic, let's create a shared NumberConversion service client for later use first, see code below:

{% codeblock NumberConversionServiceClient.java lang:java https://github.com/bulldog2011/nano/blob/master/sample/webservice/NumberConverter/src/com/dataaccess/service/NumberConversionServiceClient.java source %}

package com.dataaccess.service;

import com.dataaccess.webservicesserver.client.NumberConversionSoapType_SOAPClient;

/**
 * The Number Conversion Web Service, implemented with Visual DataFlex, 
 * provides functions that convert numbers into words or dollar amounts.
 * 
 * http://www.dataaccess.com/webservicesserver/numberconversion.wso
 * 
 * @author bulldog
 *
 */
public class NumberConversionServiceClient {
	
	// target endpoint
	public static String numberConversionServiceUrl = "http://www.dataaccess.com/webservicesserver/numberconversion.wso";
	
	private static volatile NumberConversionSoapType_SOAPClient client = null;
	
	// get a shared client
	public static NumberConversionSoapType_SOAPClient getSharedClient() {
		if (client == null) {
			synchronized(NumberConversionServiceClient.class) {
				if (client == null) {
					client = new NumberConversionSoapType_SOAPClient();
					client.setEndpointUrl(numberConversionServiceUrl);
				}
			}
		}
		
		return client;
	}

}


{% endcodeblock %}

The code is quite self-explanatory, the `getSharedClient` static factory method just returns a NumberConversionSoapType_SOAPClient instance with specified target service endpoint address, and there will be only one such client instance within the application.

With the help of the NumberConversion service proxy generated from WSDL, web service call through Nano is extremely simple, let's review and understand the generated numberToWords proxy interface first before writing calling logic:


{% codeblock NumberConversionSoapType_SOAPClient.java lang:java https://github.com/bulldog2011/nano/blob/master/sample/webservice/NumberConverter/src/com/dataaccess/webservicesserver/client/NumberConversionSoapType_SOAPClient.java source  %}

// Generated by wsdl compiler for android/java
// DO NOT CHANGE!
package com.dataaccess.webservicesserver.client;


import com.leansoft.nano.ws.SOAPServiceCallback;
import com.leansoft.nano.ws.NanoSOAPClient;
import com.dataaccess.webservicesserver.NumberToDollars;
import com.dataaccess.webservicesserver.NumberToWords;
import com.dataaccess.webservicesserver.NumberToWordsResponse;
import com.dataaccess.webservicesserver.NumberToDollarsResponse;


/**
 This class is the SOAP client to the NumberConversionSoapType Web Service.
*/ 
public class NumberConversionSoapType_SOAPClient extends NanoSOAPClient {


    /**
     Returns the word corresponding to the positive number passed as parameter. Limited to quadrillions.
    */
    public void numberToWords(NumberToWords requestObject, SOAPServiceCallback<NumberToWordsResponse> serviceCallback) {
       
        
        super.invoke(requestObject, serviceCallback, NumberToWordsResponse.class);
    }

    /**
     Returns the non-zero dollar amount of the passed number.
    */
    public void numberToDollars(NumberToDollars requestObject, SOAPServiceCallback<NumberToDollarsResponse> serviceCallback) {
       
        
        super.invoke(requestObject, serviceCallback, NumberToDollarsResponse.class);
    }


}

{% endcodeblock %}


This is just a plain old interface generated from wsdl, unlike normal interface, this interface is asynchronous, when you call `numberToWords` service, you need to provide a request of type `NumberToWords` as first parameter, also you need to provide(or register) a callback implementing interface `SOAPServiceCallback`, see definition of this interface below:
{% codeblock XMLServiceCallback.java lang:java https://github.com/bulldog2011/nano/blob/master/src/main/java/com/leansoft/nano/ws/XMLServiceCallback.java source  %}

package com.leansoft.nano.ws;

public interface XMLServiceCallback<R> {
	
	public void onSuccess(R responseObject);
	
	public void onFailure(Throwable error, String errorMessage);

}

{% endcodeblock %}

{% codeblock SOAPServiceCallback.java lang:java https://github.com/bulldog2011/nano/blob/master/src/main/java/com/leansoft/nano/ws/SOAPServiceCallback.java source  %}

package com.leansoft.nano.ws;

public interface SOAPServiceCallback<R> extends XMLServiceCallback<R> {
	
	public void onSOAPFault(Object soapFault);

}

{% endcodeblock %}


`SOAPServiceCallback` interface supports three callback methods, `onSuccess`, `onFailure` and `onSOAPFault` callbacks, success callback will be called if the service invocation succeed, and you will get a response object of type `NumberToWordsResponse`, usually, in success callback, you implement response handling logic and update UI according to the response; failure callback will be called if there is any HTTP or Nano parsing error, you may get an exception and an error message in such case, usually, you implement error handling logic in failure callback and update UI accordingly; soap fault callback will be called if Nano get a SOAP fault from service response, usually, you implement error handling logic in the soap fault callback and update UI accordingly.

By the way, although not necessary, I suggest you to review all proxy code generated by mwsc, this will help you better understand the inner working of Nano.

Now time to the UI part, for this simple demo, we only need an `EditText` for number input, and two `Button`, one for number to dollars conversion, the other for number to words conversion, see graphical layout screenshot below:

{% img center /images/nano/tutorial01/screen_shot2.png 400 600 %}

Now let's implement application logic by calling the `numberToWords` or `numberToDollars` services,
the application logic is implemented in the `MainActivity` class, a registered `onClick` method will be triggered when you click the `Number To Words` button on the UI:

{% codeblock MainActivity.m lang:java https://github.com/bulldog2011/nano/blob/master/sample/webservice/NumberConverter/src/com/leansoft/nano/sample/MainActivity.java source  %}

	Button numberToWordsButton = (Button) this.findViewById(R.id.numToWordsButton);
	
	numberToWordsButton.setOnClickListener(new OnClickListener() {

		@Override
		public void onClick(View arg0) {
			// get shared client
			NumberConversionSoapType_SOAPClient client = NumberConversionServiceClient.getSharedClient();
			client.setDebug(true); // enable soap message logging
			
			// build request
			NumberToWords request = new NumberToWords();
			try {
				String number = ((EditText)findViewById(R.id.numerInputText)).getText().toString();
				request.ubiNum = new BigInteger(number);
			} catch (NumberFormatException ex) {
				Toast.makeText(MainActivity.this, "Invalid integer number", Toast.LENGTH_LONG).show();
				return;
			}
			
			// make API call and register callbacks
			client.numberToWords(request, new SOAPServiceCallback<NumberToWordsResponse>() {

				@Override
				public void onSuccess(NumberToWordsResponse responseObject) { // success
					Toast.makeText(MainActivity.this, responseObject.numberToWordsResult, Toast.LENGTH_LONG).show();
				}

				@Override
				public void onFailure(Throwable error, String errorMessage) { // http or parsing error
					Toast.makeText(MainActivity.this, errorMessage, Toast.LENGTH_LONG).show();
				}

				@Override
				public void onSOAPFault(Object soapFault) { // soap fault
					Fault fault = (Fault)soapFault;
					Toast.makeText(MainActivity.this, fault.faultstring, Toast.LENGTH_LONG).show();
				}
				
			});
			
		}
		
	});


{% endcodeblock %}

The call logic can't be simpler, I've added comments in the service call code for you to better understand the call flow. Also, I've enabled the debug mode of the proxy, so you can see the detailed request/response messages when you run the demo, this feature is extremely useful for troubleshooting.


##Final Step - Run the Demo

Now it's time to run the demo, let's try it in the simulator(you may use a real device of course),  below is a screenshot of the simulator after converting a number to dollars:

{% img center /images/nano/tutorial01/screen_shot3.png 400 600 %}
 


You can also find the debug output in the Android LogCat, like below:

{% codeblock  %}
04-15 13:54:26.146: D/NanoSOAPClient(281): Sending request to : http://www.dataaccess.com/webservicesserver/numberconversion.wso
04-15 13:54:26.146: D/NanoSOAPClient(281): Request HTTP headers : 
04-15 13:54:26.146: D/NanoSOAPClient(281): {
04-15 13:54:26.146: D/NanoSOAPClient(281):     Accept="text/xml"
04-15 13:54:26.146: D/NanoSOAPClient(281): }
04-15 13:54:26.146: D/NanoSOAPClient(281): Request message : 
04-15 13:54:26.146: D/NanoSOAPClient(281): <?xml version='1.0' encoding='UTF-8' ?>
04-15 13:54:26.146: D/NanoSOAPClient(281): <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://www.dataaccess.com/webservicesserver/">
04-15 13:54:26.146: D/NanoSOAPClient(281):   <soapenv:Body>
04-15 13:54:26.146: D/NanoSOAPClient(281):     <NumberToDollars>
04-15 13:54:26.146: D/NanoSOAPClient(281):       <dNum>128.32</dNum>
04-15 13:54:26.146: D/NanoSOAPClient(281):     </NumberToDollars>
04-15 13:54:26.146: D/NanoSOAPClient(281):   </soapenv:Body>
04-15 13:54:26.146: D/NanoSOAPClient(281): </soapenv:Envelope>
04-15 13:54:27.356: D/SOAPHttpResponseHandler(281): Response HTTP status : 200
04-15 13:54:27.366: D/SOAPHttpResponseHandler(281): Response HTTP headers : 
04-15 13:54:27.366: D/SOAPHttpResponseHandler(281): {
04-15 13:54:27.366: D/SOAPHttpResponseHandler(281):     Web-Service="Visual DataFlex 17.0"
04-15 13:54:27.366: D/SOAPHttpResponseHandler(281):     Content-Type="text/xml; charset=utf-8"
04-15 13:54:27.366: D/SOAPHttpResponseHandler(281):     Date="Mon, 15 Apr 2013 13:54:52 GMT"
04-15 13:54:27.366: D/SOAPHttpResponseHandler(281):     Content-Length="397"
04-15 13:54:27.366: D/SOAPHttpResponseHandler(281):     X-Powered-By="ASP.NET"
04-15 13:54:27.366: D/SOAPHttpResponseHandler(281):     Server="Microsoft-IIS/6.0"
04-15 13:54:27.366: D/SOAPHttpResponseHandler(281):     Cache-Control="private, max-age=0"
04-15 13:54:27.366: D/SOAPHttpResponseHandler(281): }
04-15 13:54:27.376: D/SOAPHttpResponseHandler(281): Response message : 
04-15 13:54:27.376: D/SOAPHttpResponseHandler(281): <?xml version="1.0" encoding="utf-8"?>
04-15 13:54:27.376: D/SOAPHttpResponseHandler(281): <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
04-15 13:54:27.376: D/SOAPHttpResponseHandler(281):   <soap:Body>
04-15 13:54:27.376: D/SOAPHttpResponseHandler(281):     <m:NumberToDollarsResponse xmlns:m="http://www.dataaccess.com/webservicesserver/">
04-15 13:54:27.376: D/SOAPHttpResponseHandler(281):       <m:NumberToDollarsResult>one hundred and twenty eight dollars and thirty two cents</m:NumberToDollarsResult>
04-15 13:54:27.376: D/SOAPHttpResponseHandler(281):     </m:NumberToDollarsResponse>
04-15 13:54:27.376: D/SOAPHttpResponseHandler(281):   </soap:Body>
04-15 13:54:27.376: D/SOAPHttpResponseHandler(281): </soap:Envelope>
{% endcodeblock %}


If you have any problem to run the demo, please check the debug output first.

Now you see the power of Nano framework, you don't get troubled with error prone and tedious SOAP/XML parsing or http handling, the generic Nano framework will do these stuff for you, you only need to use a plain old asynchronous interface for service invocation, this can not only accelerate application development, but reduce the long term maintenance cost.

NumberConversion is just a bare minimum demo, there are other more featured demos(like Amazon, eBay search demo) in the sample folder of Nano project, please try them to better understand how Nano works, then you can begin to develop your own service based applications, using wsdl driven methodology supported by Nano.

See your next great Android application!