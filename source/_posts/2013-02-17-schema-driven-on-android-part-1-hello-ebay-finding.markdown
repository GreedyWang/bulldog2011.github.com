---
layout: post
title: "Schema driven web service client development on Android, Part 1: Hello eBay Finding"
date: 2013-02-17 12:48
comments: true
categories: [nano, ebay api]
keywords: schema driven web API client development, xml binding on Andorid, ebay Finding API, eBay Finding API proxy for Android
description: Showing show to generate eBay Finding API proxy using mxjc binding compiler then call eBay Finding API using nano-rest framework for Android.
---

Scheam driven web service development is quite popular in Java world, JAXB and JAX-WS are both mature standard, and frameworks like CXF, Axis are famous amount many developers for fast service development. It's common to do scheam driven developent on server side, can scheam driven client development be done on Android? Yes, it can. Today I will show you how do do scheam driven development on Android by leveraging following light-weight frameworks:

<!--more-->

>1. [Nano-rest](https://github.com/bulldog2011/nano-rest) restful client framework for Android, with Nano xml binding support.
2. [mxjc](https://github.com/bulldog2011/mxjc) scheam to Android java binding compiler.

The serivce I choose for demo is [eBay Finding service](https://www.x.com/developers/ebay/products/finding-api), here are why I choose this service:

>1. I have some direct experience with this service.
2. eBay Finding serivce supports schema driven development since it provides a wsdl, at the same time, it also supports RESTful style service call.

If you are not familar with this serivce, please visit its [official site](https://www.x.com/developers/ebay/products/finding-api).

###The Big Picture

{% img center /images/nano-rest/big_picture.png 600 800 %}

The picture above is the blueprint of scheam driven development on Android. The left part of the blueprint is a build time view, here we leverage mxjc binding compiler to automatically generate service or domain classes from scheam or wsdl; The right part of the blueprint is a runtime view, a typical flow starts from your Android app, it issues request object on proxy component, the proxy passes the request to the Nano restful framework which will delegate the object to xml marshalling work to Nano binding framework and send the xml request to external service through HTTP transportation component, when an xml resposne is received by the HTTP transportation component, the Nano restful framework will also delegate the xml to object unmarshalling work to Nano binding framework and passes the response object to the proxy which will return the response object back to the calling app.

Let's follow the blueprint and build a simple Android app step by step:

###Step 1 : Generate service classes from schema
The first step of scheam driven development is to generate service classes from schema or wsdl, let's download eBay Finding serivce wsdl [here](http://developer.ebay.com/webservices/finding/latest/FindingService.wsdl), also, let's download [mxjc binding compiler](https://github.com/bulldog2011/mxjc) zip package by following links on the github site, extract the zip package, suppose we put the wsdl in the same folder as the extracted zip package, then we execute command(suppose we are in Windows environment, Unix environment will be similar).

```
bin\mxjc.bat -wsdl FindingService.wsdl
```

When we execute this command, the schema inside the wsdl will be parsed by the binding compiler and Nano bindable classes will be generated, by default, the binding compiler will derive package name from wsdl, but you can override this by providing custom package name directly as command line options. 
'-wsdl' option is needed since we are generating from wsdl, not default xsd.   
By default, the service classes will be generated in the current directory.

###Step 2: Create a New Android Project
Now let's create a new Android project in IDE like Eclipse with ADT installed, to use Nano-rest client framework, the Android API version must be equal to or newer than 7(Androdi 2.1 or above), this is required since Nano-rest leverages Android Service mechanism for asynchronous service invocation.

With new Android project created, let's add Nano-rest jar in the ***libs*** folder of the project, the Nano-rest jar can be downloaded by following link on the [Nano-rest github site](https://github.com/bulldog2011/nano-rest), the shaded jar is preferred since it already includes Nano xml binding framework dependency, otherwise, you have to add Nano xml binding framework jar separately.
Now, let's copy the above generated service classes into the ***src*** folder the project, if we refresh the project, there should be no compiling error since all imports in the generated classes can be resolved by the Nano-rest shaded jar reference.

To let Nano-rest framework work correctly at runtime, we ***must*** added following declarations in the manifest file of the project:

```xml

  <uses-permission android:name="android.permission.INTERNET" />

  <uses-sdk android:minSdkVersion="7" />

  <application>
    <service android:name="com.leansoft.nanorest.service.HTTPRequestExecutorService" >
    </service>
  </application>

```

The uses-permission declaration is needed for internet access, the minSdkVersion declaration has be explained above, and the HTTPRequestExecutorService class is required for asynchronous service invocation, synchronous service invocation may block Android main UI, leading to process crash, this is not expected, so as a best practice, we should always invoke service asynchronously on Android.

###Step 3 : Create eBay Finding API Proxy
Web service client proxy can simplify service invocation code, usually, web serivce framework like CXF or JAX-WS has tool to auto-generate service proxy from wsdl, however, current mxjc binding compiler does not support proxy generation yet(it can only generate service classes from schema), but it's not hard for us to write proxy class manually since Nano-rest has already encapsulated generic service invocation logic for us, we only need to extend it and add a few service specific logic, let's just do it:

First, let's definite some constants which will be used later:

``` java

package com.ebay.finding;

public interface FindingConstants {
	
	public static String PRODUCTION_ENDPOINT = "http://svcs.ebay.com/services/search/FindingService/v1";
	
	public static String SANDBOX_ENDPOINT = "http://svcs.sandbox.ebay.com/services/search/FindingService/v1";
	
	public static String X_EBAY_SOA_OPERATION_NAME = "X-EBAY-SOA-OPERATION-NAME";
	
	public static String X_EBAY_SOA_SECURITY_APPNAME = "X-EBAY-SOA-SECURITY-APPNAME";

}

```
These constants are quite self explanatory, so I won't give more comments.

Then we build a simple authentication class, eBay Finding serivce need a ***APP NAME*** as one of http request headers, so we just add it, 

``` java

package com.ebay.finding.auth;

import com.ebay.finding.FindingConstants;
import com.leansoft.nanorest.auth.AuthenticationProvider;
import com.leansoft.nanorest.client.BaseRestClient;

public class AppNameAuthenticationProvider implements AuthenticationProvider {

	@Override
	public void authenticateRequest(BaseRestClient client) {
		client.addHeader(FindingConstants.X_EBAY_SOA_SECURITY_APPNAME, "YOUR_APPNAME_HERE");
	}

}

```

***Note***, before you can run the final finished application, you mush replace the APP NAME placeholder with your own eBay developer APP NAME which can be applied on eBay developer site.

Next, let's build a generic request processor for eBay Finding serivce, with this base request processor, all specific request processors(supported by eBay Finding service) can be easily built later, let's see the full definition of this base request processor:

``` java

package com.ebay.finding.request;

import com.ebay.finding.FindingConstants;
import com.ebay.finding.auth.AppNameAuthenticationProvider;
import com.ebay.marketplace.search.v1.services.AckValue;
import com.ebay.marketplace.search.v1.services.BaseServiceResponse;
import com.ebay.marketplace.search.v1.services.ErrorMessage;
import com.leansoft.nanorest.callback.HttpCallback;
import com.leansoft.nanorest.client.RestClient;
import com.leansoft.nanorest.domain.ResponseStatus;
import com.leansoft.nanorest.logger.ALog;
import com.leansoft.nanorest.parser.NanoXmlResponseParser;
import com.leansoft.nanorest.request.NanoXmlRequestProcessor;

public class BaseFindingRequestProcessor<T> extends NanoXmlRequestProcessor<T> {
	
	private final Class<T> responseType;

	public BaseFindingRequestProcessor(Object requestObject, String opName, Class<T> responseType, 
			HttpCallback<T> callback) {
		
		super(requestObject,
			  responseType,
		      callback);
		
		this.responseType = responseType;
		
		
		RestClient client = getRestClient();
		client.setUrl(FindingConstants.PRODUCTION_ENDPOINT);
		client.setAuthentication(new AppNameAuthenticationProvider());
		
		client.addHeader(FindingConstants.X_EBAY_SOA_OPERATION_NAME,  opName);
	}
	
	private NanoXmlResponseParser<ErrorMessage> errorMessageParser = 
			new NanoXmlResponseParser<ErrorMessage>(ErrorMessage.class);
	
	@Override
    protected void handleResponse() {
    	
        final RestClient client = getRestClient();
        final ResponseStatus status = client.getResponseStatus();
        String response = client.getResponse();
        ALog.d(TAG, status.toString());
        if (status.getStatusCode() < 200 || status.getStatusCode() >= 300) {
        	if (isXmlResponse(response)) {
        		parseErrorMessage(response);
        	} else {
        		getResponseHandler().handleError(status);
        	}
        } else {
        	parseHttpResponse(response);
        }
    	
    }
	
	private void parseErrorMessage(String response) {
        try {
            final ErrorMessage errorMessage = errorMessageParser.parse(response);
            
            T responeData = responseType.newInstance();
            BaseServiceResponse baseServiceResponse = (BaseServiceResponse)responeData;
            baseServiceResponse.setAck(AckValue.FAILURE);
            baseServiceResponse.setErrorMessage(errorMessage);
            
            getResponseHandler().handleSuccess(responeData);
        } catch (final Exception e) {
            ResponseStatus responseStatus = ResponseStatus.getParseErrorStatus();
            ALog.d(TAG, responseStatus.toString(), e);
            getResponseHandler().handleError(responseStatus);
        }
	}
	
	private boolean isXmlResponse(String response) {
		if (response == null) return false;
		return response.startsWith("<?xml");
	}
}



```  

Base request processor is the core of the proxy, let me give more comments about the base processor:  
1. we extend NanoXmlRequestProcessor class, since we want to leverage Nano xml framework for request marshalling and response unmarshalling.  
2. In the constuctor, we provides request object, operaion name and 
a HttpCallback instance as parameters and delegate these parameters to the super class, the requst object is an eBay Finding request object like FindItemsByKeywordsRequest object, the operation name is required by eBay Finding service as request header, and I will give more explanation about the HttpCallback instance later.   
3. Also in the constructor, we set eBay Finding service production endpoint url on the rest client(replace here if you need to access sandbox instead), plug in authentication provider defined above, and add operation name as request header. Note, authentication provider plugin is not necessary, you can always add APP NAME directly on the rest client as a request header, I use authentication provider here just to show the formal way to do authentication when Nano-rest framework is used.  
4. The response handling logic of the eBay Finding serivce needs a special fix, usually, eBay service supports RRE(resposne resident error), means error message are wrapped in a normal response message, but in some cases, eBay Finding service may return a single error xml message without a wrapping response. In order to fix this, I overrided the Nano-rest response handling logic, please see handleResponse and 
parseErrorMessage methods for details. 

The responseHandler(defined in super class and returned by getResponseHandler method) is assocated with the HttpCallback instance passed in the constructor, it only has two methods: handleSuccess and handleError, usually, if the service invocation is successful, then handleSuccess method should be called, otherwise, handleError method should be called, responseHandler is only responsilbe for the callback, concrete success or error handling logic is defined by application logic.

With base processor defined, it's quite easy to define a specific request processor, let's define a FindItemsByKeywordsReqeustProcessor since I will use this function of eBay Finding service as demo:

``` java
package com.ebay.finding;

import com.ebay.finding.request.BaseFindingRequestProcessor;
import com.ebay.marketplace.search.v1.services.FindItemsByKeywordsRequest;
import com.ebay.marketplace.search.v1.services.FindItemsByKeywordsResponse;
import com.leansoft.nanorest.RequestProcessor;
import com.leansoft.nanorest.callback.HttpCallback;

public class FindingService {
	
	public static final String TAG = FindingService.class.getSimpleName();
	
	public static RequestProcessor getFindItemsByKeywordsRequestProcessor( 
			FindItemsByKeywordsRequest requestObject, 
			HttpCallback<FindItemsByKeywordsResponse> callback) {
		return new BaseFindingRequestProcessor<FindItemsByKeywordsResponse>(requestObject, "findItemsByKeywords", FindItemsByKeywordsResponse.class, callback);
	}

}


```

Not much code, and no magic here if you are familar with Java generic. We just define a factory method, provide specific parameters requried by FindItemsByKeywords API call. Use this method as template, it's not hard for you to create other request processors, like   
getFindItemsAdvandedRequestProcessor, getFindItemsByCategoryRequestProcessor, etc, all functions of eBay Finding service can be defined in this way, actually, you can even build a complete eBay Finding SDK in this way for later reuse.

###Step 4: Write real application logic and UI
Now it's time for us to write real application logic and UI, in this demo, I just want to show minimum application logic and UI, the UI part only has an EditText as keywords input and a Button to trigger search by calling FindingItemsByKeywords API, and a Toast for response display.
The application logic is even simpler, let's see the whole definition of the Main Activity:

``` java
package com.leansoft.nanorest.sample;

import com.ebay.finding.FindingService;
import com.ebay.marketplace.search.v1.services.AckValue;
import com.ebay.marketplace.search.v1.services.FindItemsByKeywordsRequest;
import com.ebay.marketplace.search.v1.services.FindItemsByKeywordsResponse;
import com.ebay.marketplace.search.v1.services.PaginationInput;
import com.leansoft.nanorest.RequestProcessor;
import com.leansoft.nanorest.callback.HttpCallback;
import com.leansoft.nanorest.domain.ResponseStatus;

import android.os.Bundle;
import android.view.View;
import android.view.View.OnClickListener;
import android.widget.Button;
import android.widget.Toast;
import android.widget.EditText;
import android.app.Activity;


public class MainActivity extends Activity {
	
	private Button btn;

	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
			
        btn = (Button) this.findViewById(R.id.btn);
        btn.setOnClickListener(new OnClickListener() {

			@Override
			public void onClick(View arg0) {
				String keywords = ((EditText) findViewById(R.id.edit_input)).getText().toString();	
				FindItemsByKeywordsRequest request = new FindItemsByKeywordsRequest();
				request.setKeywords(keywords);
				PaginationInput pi = new PaginationInput();
		        pi.setPageNumber(1);
		        pi.setEntriesPerPage(1);
		        request.setPaginationInput(pi);
		        
				RequestProcessor requestProcessor = FindingService.getFindItemsByKeywordsHttpRequest(
						request, 
						new FindItemsByKeywordsCallback()
						);
				requestProcessor.invokeAsync(MainActivity.this.getApplicationContext());
			}
        	
        });
	}

	private final class FindItemsByKeywordsCallback implements HttpCallback<FindItemsByKeywordsResponse> {

		@Override
		public void onSuccess(FindItemsByKeywordsResponse responseData) {
			
			if (responseData.getAck() == AckValue.SUCCESS) {
	            Toast.makeText(getApplicationContext(),
	            		responseData.getSearchResult().getItem().get(0).getTitle(),
	                    Toast.LENGTH_LONG).show();
			} else {
	            Toast.makeText(getApplicationContext(),
	            		responseData.getErrorMessage().getError().get(0).getMessage(),
	                    Toast.LENGTH_LONG).show();
			}
		}

		@Override
		public void onHttpError(ResponseStatus responseCode) {
            Toast.makeText(getApplicationContext(),
                    responseCode.getStatusCode() + " " + responseCode.getStatusMessage(),
                    Toast.LENGTH_LONG).show();
			
		}
		
	}

}


```

The code is easy to understand if you know Java and Android, let me give a few comments:  
1. We register a click listener to the button, inside the listener is the main application logic: 
get keywords input and build a FindItemByKeywordsRequest object, get a FindItemByKeywordsRequestProcessor from the factory we defined above, and invoke the request processor asynchronously at last.  
2. When we build the FindItemByKeywordsRequestProcessor instance, we passed in a FindItemsByKeywordsCallback instance as parameter, FindItemsByKeywordsCallback implements HttpCallback interface, it contains the main response logic, this is the place when final asynchronous response handling happen. Usually, inside a HttpCallback, we update UI according to the success or failure of the response, in this case, if the response is successful, we show the title of the result item, if there is http error, we show the status code and message. Since eBay Finding service support Response Resident Error(RRE), even in the onSuccess callback, we still need to check the Ack value to find out if there is application error and handle accordingly.

Here I just show a typical asynchronous service invocation paradigm using Nano-rest framework, you can follow this paradigm for other service invocations according to your real needs.

###Step 5 : Run the application
Now lets run the application, note again, please fill in your APP NAME before you run the application, below is the UI of a success case:

{% img center /images/nano-rest/hello_ebay_finding_success.png 400 600 %}

Let's provide an invalid APP NAME and try again, below is the UI of the failed case since the APP NAME is invalid:

{% img center /images/nano-rest/hello_ebay_finding_failure.png 400 600 %}

To facilitate debug and trouble shooting, you can always check the request or response xml by looking at the log in the LogCat provided by ADT.


###Conclusion:
The demo shown in this tutorial is just a bare minimum or starter kit, there is still much work to do before you can release a fully functional service backed Android application, but anyway, with a reuseable service proxy, you are free from low level and error-prone serivce message parsing and http handling anymore, instead, you can put your effor on main application logic and UI, this can definitly accelerate the development of the application.

Scheam driven development is a popular and mature development methodology, when used appropriately, it can improve development efficiency and enhance the reliability and maintainability of the application. Now with the support of Nano-rest framework and the mxjc binding compiler, we can also do schema driven client development on Android.

You can find the whole source of the sample application and all the generated eBay Finding service classes [here](https://github.com/bulldog2011/nano-rest/tree/master/sample/HelloEBayFinding).









 












 