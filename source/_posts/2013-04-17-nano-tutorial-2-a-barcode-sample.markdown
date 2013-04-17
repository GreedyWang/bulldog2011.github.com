---
layout: post
title: "Nano Tutorial 2 - a BarCode Generator sample"
date: 2013-04-17 10:31
comments: true
categories: [nano]
keywords: android, java, soap, wsdl
description: Nano Tutorial 2 - a BarCode Generator sample
---

This is the second tutorial of Nano Tutorial series, in the [first tutorial](http://bulldog2011.github.io/blog/2013/04/15/nano-tutorial-1-a-number-conversion-sample/), I showed how to use Nano with a service called [Number Conversion](http://www.dataaccess.com/webservicesserver/numberconversion.wso) from data access worldwide. Number conversion is a simple service, its request/response structures are quite simple. Today I will show you how to use Pico with a more complex service called [Barcode Generator](http://www.webservicex.net/ws/WSDetails.aspx?CATID=8&WSID=76) from webserviceX.NET, barcode generator service has a more complex request/response structures, it will return encoded binary image data in the response.

<!--more-->

By the way, since the wsdl driven development process in both tutorials are quite similar, I won't repeat too much details in this tutorial, I suppose you have already read tutorial one and basically understand the wsdl driven development process supported by Nano.

The whole source of this tutorial is [here](https://github.com/bulldog2011/nano/tree/master/sample/webservice/BarCode).

Let's cut to the point:

##Step 1 - Generate Java Proxy from WSDL
Download [mwsc](https://github.com/bulldog2011/mwsc) and run following command in terminal to generate the proxy:

{% codeblock %}

bin\mwsc.bat -nano -d generated http://www.webservicex.net/genericbarcode.asmx?WSDL

{% endcodeblock %}

A few comments to the code generation:


>* During the code generation, you will get a few warnings say some ports in the wsdl will be ignored, just ignore these warnings, We won't use those ports in this tutorial, it's ok as long as the standard SOAP port is generated correctly. 
* By default, the proxy code will be generated in the sub-folder corresponding to the target namespace of the wsdl, you can use `-p` option to override the package name if needed.
* There is a generated sub-folder called `client`, the SOAP and XML proxy interface will be generated in this folder.

##Step 2 - Create New Android Project, Add Nano Library and Generated Proxy into Your Project
Create a simple Android project in eclipse(or other IDE you prefer), then:   
>1. Download [latest nano library jar](https://github.com/bulldog2011/bulldog-repo/blob/master/repo/releases/com/leansoft/nano/0.7.0/nano-0.7.0.jar) and add it to the `libs` folder of your Android project.
2. Drag the above generated proxy into the `src` folder of your Android project.
3. Add following user permissions in the `AndroidManifest.xml` file to enable network access.   
``` xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

Now build the project to ensure it can build successfully.

Below is an eclipse package explorer screenshot after finishing this step:

{% img center /images/nano/tutorial02/screen_shot1.png 400 600 %}

##Step 3 - Implement Appliction Logic and UI, Call Proxy to Invoke Web Service as Needed.

First, create a shared service client as below:

{% codeblock BarCodeServiceClient.java lang:java https://github.com/bulldog2011/nano/blob/master/sample/webservice/BarCode/src/net/webservicex/service/BarCodeServiceClient.java source %}

package net.webservicex.service;

import net.webservicex.client.BarCodeSoap_SOAPClient;

/**
 * http://www.webservicex.net/genericbarcode.asmx
 * 
 * @author bulldog
 *
 */
public class BarCodeServiceClient {
	
	// target endpoint
	public static String barCodeServiceUrl = "http://www.webservicex.net/genericbarcode.asmx";
	
	private static volatile BarCodeSoap_SOAPClient client = null;
	
	// get a shared client
	public static BarCodeSoap_SOAPClient getSharedClient() {
		if (client == null) {
			synchronized(BarCodeServiceClient.class) {
				if (client == null) {
					client = new BarCodeSoap_SOAPClient();
					client.setEndpointUrl(barCodeServiceUrl);
				}
			}
		}	
		return client;
	}
}

{% endcodeblock %}

Now open the main layout file and add three UI components:
>* A `EditText` for barcode number input,
* A `Button` for encoding trigger,
* A `ImageView` for encoded barcode display.



{% codeblock activity_main.xml lang:xml https://github.com/bulldog2011/nano/blob/master/sample/webservice/BarCode/res/layout/activity_main.xml source %}

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity" >

    <EditText
        android:id="@+id/barCodeText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentTop="true"
        android:layout_marginLeft="20dp"
        android:layout_marginTop="19dp"
        android:ems="10"
        android:singleLine="true"
        android:hint="Enter Barcode" >

        <requestFocus />
    </EditText>

    <Button
        android:id="@+id/encodeButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignBaseline="@+id/barCodeText"
        android:layout_alignBottom="@+id/barCodeText"
        android:layout_toRightOf="@+id/barCodeText"
        android:text="Encode" />

    <ImageView
        android:id="@+id/barCodeImage"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/encodeButton"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="98dp"
        android:src="@drawable/ic_launcher" />

</RelativeLayout>

{% endcodeblock %}

{% img center /images/nano/tutorial02/screen_shot2.png 300 500 %}

Now implement application logic in `MainActivity` class as below:

{% codeblock MainActivity.java lang:java https://github.com/bulldog2011/nano/blob/master/sample/webservice/BarCode/src/com/leansoft/nano/sample/MainActivity.java source  %}

package com.leansoft.nano.sample;

import com.leansoft.nano.soap11.Fault;
import com.leansoft.nano.ws.SOAPServiceCallback;

import net.webservicex.BarCodeData;
import net.webservicex.BarcodeOption;
import net.webservicex.BarcodeType;
import net.webservicex.CheckSumMethod;
import net.webservicex.GenerateBarCode;
import net.webservicex.GenerateBarCodeResponse;
import net.webservicex.ImageFormats;
import net.webservicex.ShowTextPosition;
import net.webservicex.client.BarCodeSoap_SOAPClient;
import net.webservicex.service.BarCodeServiceClient;
import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageView;
import android.widget.Toast;
import android.app.Activity;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;

public class MainActivity extends Activity {

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		
		Button encodeButton = (Button) this.findViewById(R.id.encodeButton);
		
		encodeButton.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View arg0) {
				// get shared client
				BarCodeSoap_SOAPClient client = BarCodeServiceClient.getSharedClient();
				client.setDebug(true); // enable soap message logging
				
				// build request
				BarCodeData barCodeData = new BarCodeData();
				barCodeData.height = 125;
				barCodeData.width = 225;
				barCodeData.angle = 0;
				barCodeData.ratio = 5;
				barCodeData.module = 0;
				barCodeData.left = 25;
				barCodeData.top = 0;
				barCodeData.checkSum = false;
				barCodeData.fontName = "Arial";
				barCodeData.barColor = "Black";
				barCodeData.bgColor = "White";
				barCodeData.fontSize = 10.0f;
				barCodeData.barcodeOption = BarcodeOption.BOTH;
				barCodeData.barcodeType = BarcodeType.CODE_2_5_INTERLEAVED;
				barCodeData.checkSumMethod = CheckSumMethod.NONE;
				barCodeData.showTextPosition = ShowTextPosition.BOTTOM_CENTER;
				barCodeData.barCodeImageFormat = ImageFormats.PNG;
				
				GenerateBarCode request = new GenerateBarCode();
				request.barCodeParam = barCodeData;
				request.barCodeText = ((EditText)findViewById(R.id.barCodeText)).getText().toString();
				
				// make API call with registered callbacks
				client.generateBarCode(request, new SOAPServiceCallback<GenerateBarCodeResponse>() {

					@Override
					public void onSuccess(GenerateBarCodeResponse responseObject) { // success
						ImageView barCodeImage = (ImageView)findViewById(R.id.barCodeImage);
						byte[] imageData = responseObject.generateBarCodeResult;
						Bitmap bitmap = BitmapFactory.decodeByteArray(imageData, 0, imageData.length);
						barCodeImage.setImageBitmap(bitmap);
					}
					
					@Override
					public void onFailure(Throwable error, String errorMessage) {// http or parsing error
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
	}

}


{% endcodeblock %}

The service invocation logic is quite self explanatory, you get shared SOAP barcode proxy client first, then build encoding request, make API call with the request and registered callbacks, if API call is successful, you just display the barcode image on UI, otherwise, you display error message accordingly using Android `Toast` UI component.

##Final Step - Run the Demo

See a screen shot below:

{% img center /images/nano/tutorial02/screen_shot3.png 600 800 %}

And the debug output:


{% codeblock %}

04-17 03:35:21.116: D/NanoSOAPClient(283): Sending request to : http://www.webservicex.net/genericbarcode.asmx
04-17 03:35:21.116: D/NanoSOAPClient(283): Request HTTP headers : 
04-17 03:35:21.116: D/NanoSOAPClient(283): {
04-17 03:35:21.116: D/NanoSOAPClient(283):     Accept="text/xml"
04-17 03:35:21.116: D/NanoSOAPClient(283): }
04-17 03:35:21.126: D/NanoSOAPClient(283): Request message : 
04-17 03:35:21.126: D/NanoSOAPClient(283): <?xml version='1.0' encoding='UTF-8' ?>
04-17 03:35:21.126: D/NanoSOAPClient(283): <soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://www.webservicex.net/">
04-17 03:35:21.126: D/NanoSOAPClient(283):   <soapenv:Body>
04-17 03:35:21.126: D/NanoSOAPClient(283):     <GenerateBarCode>
04-17 03:35:21.126: D/NanoSOAPClient(283):       <BarCodeParam>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <Height>125</Height>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <Width>225</Width>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <Angle>0</Angle>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <Ratio>5</Ratio>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <Module>0</Module>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <Left>25</Left>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <Top>0</Top>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <CheckSum>false</CheckSum>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <FontName>Arial</FontName>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <BarColor>Black</BarColor>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <BGColor>White</BGColor>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <FontSize>10.0</FontSize>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <barcodeOption>Both</barcodeOption>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <barcodeType>Code_2_5_interleaved</barcodeType>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <checkSumMethod>None</checkSumMethod>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <showTextPosition>BottomCenter</showTextPosition>
04-17 03:35:21.126: D/NanoSOAPClient(283):         <BarCodeImageFormat>PNG</BarCodeImageFormat>=
04-17 03:35:21.126: D/NanoSOAPClient(283):       </BarCodeParam>
04-17 03:35:21.126: D/NanoSOAPClient(283):       <BarCodeText>123456789</BarCodeText>
04-17 03:35:21.126: D/NanoSOAPClient(283):     </GenerateBarCode>
04-17 03:35:21.126: D/NanoSOAPClient(283):   </soapenv:Body>
04-17 03:35:21.126: D/NanoSOAPClient(283): </soapenv:Envelope>
04-17 03:35:21.176: D/dalvikvm(283): GC_FOR_MALLOC freed 12599 objects / 580360 bytes in 35ms
04-17 03:35:22.838: D/SOAPHttpResponseHandler(283): Response HTTP status : 200
04-17 03:35:22.838: D/SOAPHttpResponseHandler(283): Response HTTP headers : 
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283): {
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283):     X-AspNet-Version="4.0.30319"
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283):     Date="Wed, 17 Apr 2013 03:35:56 GMT"
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283):     Vary="Accept-Encoding"
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283):     Content-Length="1951"
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283):     Content-Encoding="gzip"
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283):     Content-Type="text/xml; charset=utf-8"
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283):     X-Powered-By="ASP.NET"
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283):     Server="Microsoft-IIS/7.0"
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283):     Cache-Control="private, max-age=0"
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283): }
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283): Response message : 
04-17 03:35:22.846: D/SOAPHttpResponseHandler(283): <?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"><soap:Body><GenerateBarCodeResponse xmlns="http://www.webservicex.net/"><GenerateBarCodeResult>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</GenerateBarCodeResult></GenerateBarCodeResponse></soap:Body></soap:Envelope>



{% endcodeblock %}
