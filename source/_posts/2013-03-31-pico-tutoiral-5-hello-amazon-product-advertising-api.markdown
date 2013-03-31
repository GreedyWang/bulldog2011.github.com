---
layout: post
title: "Pico Tutoiral 5 - Hello Amazon Product Advertising API"
date: 2013-03-31 10:09
comments: true
categories: [pico]
keywords: ios, iphone, wsdl, soap, Amazon Product Advertising API
description: Pico Tutorial 5 - Hello Amazon Product Advertising API
---

This is the fifth tutorial of Pico tutorial series, in this tutorial, I will show you how to integrate [Pico](https://github.com/bulldog2011/pico) with [Amazon Product Advertising API](https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html), if you are not familiar with this API, just have a quick review on its official site, basically, the Product Advertising API provides programmatic access to Amazon's product selection and discovery functionality so that developers like you can advertise Amazon products to monetize your website. In this tutorial, I will show you how to customize binding for [mwsc](https://github.com/bulldog2011/mwsc) code generation in case the wsdl does not follow convention, for example, the Amazon Product Advertising wsdl uses many anonymous inner types in its schema, without customized binding, the generated code will have many types with same names, leading to compile-time conflict. Also in this tutorial, I will show you show how to add custom SOAP header which is required by the authentication of Amazon Product Advertising API.

<!--more-->

***NOTE***

Per Amazon:

{% blockquote %}
You will not, without our express prior written approval, use any Product Advertising Content on or in connection with any site or application designed or intended for use with a mobile phone or other handheld device.
{% endblockquote %}

So please consult Amazon for permission before you can use its Product Advertising Content on any iOS devices.

The source of this tutorial is [here](https://github.com/bulldog2011/pico/tree/master/Examples/AWSECommerce).

<!--more-->

##Step 0 - Prerequisite
I suppose you have already read previous Pico tutorials, at least 1 and 2, and basically you should know:

>* The wsdl driven development process supported by Pico.
* How to reference Pico as a static library in your project.
* Or How to reference Pico Source directly in your project.


##Step 1 - Generate Objective-C Proxy from WSDL
The WSDL of Amazon Product Advertising API uses many `anonymous inner type` in its schema, this is regarded as an anti-pattern, [here](http://www.ibm.com/developerworks/webservices/library/ws-avoid-anonymous-types/?ca=drs-) is an article on IBM developerWorks which explains what is `anonymous inner type` and why it shoud be avoided. 

By default, the [mwsc](https://github.com/bulldog2011/mwsc) code generator will have no problem to generate proxy from the WSDL of Amazon Product Advertising API, but there will be many types with same names, leading to compile-time conflict. JAX-WS/JAXB(upon which mwsc is built) can resolve the `anonymous inner type` issue, by using custom binding, I have provided the custom binding file for you, see its definition below:

{% codeblock jaxws-bindings.xml lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/AWSECommerce/CustomBinding/jaxws-bindings.xml source %}

<jaxws:bindings xmlns:xs="http://www.w3.org/2001/XMLSchema"
                                xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"
                                xmlns:jaxws="http://java.sun.com/xml/ns/jaxws"
                                xmlns:jaxb="http://java.sun.com/xml/ns/jaxb">
		<jaxws:bindings
			    node="wsdl:definitions/wsdl:types/xs:schema[@targetNamespace='http://webservices.amazon.com/AWSECommerceService/2011-08-01']">
			<jaxb:bindings node="//xs:complexType[@name='CartAddRequest']/xs:sequence/xs:element[@name='Items']/xs:complexType">
			    <jaxb:class name="CartAddItems"/>
			</jaxb:bindings>
			<jaxb:bindings node="//xs:complexType[@name='CartAddRequest']/xs:sequence/xs:element[@name='Items']/xs:complexType/xs:sequence/xs:element[@name='Item']/xs:complexType">
			    <jaxb:class name="CartAddItem"/>
			</jaxb:bindings>
			<jaxb:bindings node="//xs:complexType[@name='CartCreateRequest']/xs:sequence/xs:element[@name='Items']/xs:complexType">
			    <jaxb:class name="CartCreateItems"/>
			</jaxb:bindings>
			<jaxb:bindings node="//xs:complexType[@name='CartCreateRequest']/xs:sequence/xs:element[@name='Items']/xs:complexType/xs:sequence/xs:element[@name='Item']/xs:complexType">
			    <jaxb:class name="CartCreateItem"/>
			</jaxb:bindings>
			<jaxb:bindings node="//xs:complexType[@name='CartCreateRequest']/xs:sequence/xs:element[@name='Items']/xs:complexType/xs:sequence/xs:element[@name='Item']/xs:complexType/xs:sequence/xs:element[@name='MetaData']/xs:complexType">
			    <jaxb:class name="ItemMetaData"/>
			</jaxb:bindings>
			<jaxb:bindings node="//xs:complexType[@name='CartModifyRequest']/xs:sequence/xs:element[@name='Items']/xs:complexType">
			    <jaxb:class name="CartModifyItems"/>
			</jaxb:bindings>
			<jaxb:bindings node="//xs:complexType[@name='CartModifyRequest']/xs:sequence/xs:element[@name='Items']/xs:complexType/xs:sequence/xs:element[@name='Item']/xs:complexType">
			    <jaxb:class name="CartModifyItem"/>
			</jaxb:bindings>
		</jaxws:bindings>
</jaxws:bindings>

{% endcodeblock %}

I won't elaborate on the details of custom binding here, if you are interested, please refer to doc [here](http://jax-ws.java.net/nonav/customizations/http.java.sun.com.xml.n/element/bindings.html) and [here](http://docs.oracle.com/cd/E17802_01/webservices/webservices/docs/2.0/tutorial/doc/JAXBUsing4.html). Basically, we need to change the naming of some `anonymous inner type` to avoid conflict, for example, to change `Item` in `CartAddRequest` to `CartAddItem`, in order to avoid conflict with other `Item` defined in other places.

Now download [mwsc](https://github.com/bulldog2011/mwsc), etract it into a local folder, copy the custom binding file into the same folder, create a folder called `generated` in the same folder, then run following command in terminal to generate the proxy:

{% codeblock %}

bin/mwsc -pico -d generated -b jaxws-bindings.xml  http://webservices.amazon.com/AWSECommerceService/AWSECommerceService.wsdl

{% endcodeblock %}


Depends on the network speed, you may need to wait a few moments to let the code generator download the wsdl and generate code, you may also download the wsdl to local and run the code generator with a local wsdl.

The target proxy will be generated in the `generated` folder you created.

##Step 2 - Create New iOS Project, Add Pico Library and Generated Proxy into Your Project

Create a new simple iOS single view application named "HelloAWSECommerce", don't choose ARC since Pico does not support ARC yet.

In this tutorial, we will reference Pico as a static library, suppose you have downloaded Pico source project from github site, then:

>1.  Drag the Pico xcodeproj into your project, 
2.  In the Build Phases of the target,  add `libPico.a` and `libxml2.dylib` to "Link Binary With Libraries" section.
3.  In the Build Setting of the target, add [your path to Pico source] to "User Header Search Paths" setting, choose "recursive" seach path.

Build the project to ensure that it can build successfully.

Now drag the proxy generated in step 1 into the project,  choose "Copy items to destination group's folder" and "add to targets" when prompted.

Build the project again to ensure that it can build successfully. ***Note***, you may get a few compiler warnings, say some property setter does not follow cocoa naming convertion, that's ok, just ignore them.


The code generation will generate both SOAP and XML based interfaces from the WSDL of Amazon Product Advertising for us,
since we will use SOAP based interface in this tutorial, you may now review the generated Amazon ECommerce service(aka Amazon Product Advertising API) SOAP interface(in the generated `client` folder) to learn what kinds of functions are provided by Amazon ECommerce service, and what kinds of parameters are needed to call the serivce, the interface is posted below:

{% codeblock AWSECommerceServicePortType_SOAPClient.h lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/AWSECommerce/AWSECommerce/awsecommerceservice/_2011_08_01/client/AWSECommerceServicePortType_SOAPClient.h source %}

// Generated by wsdl compiler for ios/objective-c
// DO NOT CHANGE!


#import <Foundation/Foundation.h>
#import "PicoSOAPClient.h"
#import "CartAdd.h"
#import "ItemLookupResponse.h"
#import "CartClearResponse.h"
#import "CartClear.h"
#import "CartModify.h"
#import "CartCreateResponse.h"
#import "CartModifyResponse.h"
#import "ItemLookup.h"
#import "ItemSearchResponse.h"
#import "BrowseNodeLookup.h"
#import "CartGet.h"
#import "CartAddResponse.h"
#import "ItemSearch.h"
#import "SimilarityLookup.h"
#import "CartGetResponse.h"
#import "CartCreate.h"
#import "SimilarityLookupResponse.h"
#import "BrowseNodeLookupResponse.h"


/**
 This class is the SOAP client to the AWSECommerceServicePortType Web Service.
*/ 
@interface AWSECommerceServicePortType_SOAPClient : PicoSOAPClient {

}

/**
 public method
*/
-(void)itemSearch:(ItemSearch *) requestObject 
      success:(void (^)(ItemSearchResponse *responseObject))success
      failure:(void (^)(NSError *error, id<PicoBindable> soapFault))failure;

/**
 public method
*/
-(void)itemLookup:(ItemLookup *) requestObject 
      success:(void (^)(ItemLookupResponse *responseObject))success
      failure:(void (^)(NSError *error, id<PicoBindable> soapFault))failure;

/**
 public method
*/
-(void)browseNodeLookup:(BrowseNodeLookup *) requestObject 
      success:(void (^)(BrowseNodeLookupResponse *responseObject))success
      failure:(void (^)(NSError *error, id<PicoBindable> soapFault))failure;

/**
 public method
*/
-(void)similarityLookup:(SimilarityLookup *) requestObject 
      success:(void (^)(SimilarityLookupResponse *responseObject))success
      failure:(void (^)(NSError *error, id<PicoBindable> soapFault))failure;

/**
 public method
*/
-(void)cartGet:(CartGet *) requestObject 
      success:(void (^)(CartGetResponse *responseObject))success
      failure:(void (^)(NSError *error, id<PicoBindable> soapFault))failure;

/**
 public method
*/
-(void)cartCreate:(CartCreate *) requestObject 
      success:(void (^)(CartCreateResponse *responseObject))success
      failure:(void (^)(NSError *error, id<PicoBindable> soapFault))failure;

/**
 public method
*/
-(void)cartAdd:(CartAdd *) requestObject 
      success:(void (^)(CartAddResponse *responseObject))success
      failure:(void (^)(NSError *error, id<PicoBindable> soapFault))failure;

/**
 public method
*/
-(void)cartModify:(CartModify *) requestObject 
      success:(void (^)(CartModifyResponse *responseObject))success
      failure:(void (^)(NSError *error, id<PicoBindable> soapFault))failure;

/**
 public method
*/
-(void)cartClear:(CartClear *) requestObject 
      success:(void (^)(CartClearResponse *responseObject))success
      failure:(void (^)(NSError *error, id<PicoBindable> soapFault))failure;


@end

{% endcodeblock %}

All the methods in the interface follow same calling paradigm - you call the service with required request object and register success callback(for success handling logic) and failure callback(for error handling logic) using Objective-C block.

##Step 3 - Implement Appliction Logic and UI, Call Proxy to Invoke Web Service as Needed.

First, create a shared service client as below:

{% codeblock AWSECommerceServiceClient.h lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/AWSECommerce/AWSECommerce/AWSECommerceServiceClient.h source %}


#import <UIKit/UIKit.h>
#import "AWSECommerceServicePortType_SOAPClient.h"

extern NSString *const AWSAccessKeyId;

@interface AWSECommerceServiceClient : AWSECommerceServicePortType_SOAPClient

+ (AWSECommerceServiceClient *)sharedClient;

- (void)authenticateRequest:(NSString *)action;

@end

{% endcodeblock %}

{% codeblock AWSECommerceServiceClient.m lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/AWSECommerce/AWSECommerce/AWSECommerceServiceClient.m source  %}

#import "AWSECommerceServiceClient.h"
#import "AmazonAuthUtils.h"
#import "PicoXMLElement.h"


@implementation AWSECommerceServiceClient

/**
 update url according to your local location, see a list of supported location at the end of the wsdl:
 http://webservices.amazon.com/AWSECommerceService/AWSECommerceService.wsdl
 */
//static NSString *const AWSECServiceURLString = @"https://webservices.amazon.cn/onca/soap?Service=AWSECommerceService";
static NSString *const AWSECServiceURLString = @"https://webservices.amazon.com/onca/soap?Service=AWSECommerceService";

NSString *const AWSAccessKeyId = @"YOUR AWS ACCESS KEY";
NSString *const AWSSecureKeyId = @"YOUR AWS SECURE KEY";

static NSString *const AuthHeaderNS = @"http://security.amazonaws.com/doc/2007-01-01/";


+ (AWSECommerceServiceClient *)sharedClient {
    static AWSECommerceServiceClient *_sharedClient = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _sharedClient = [[AWSECommerceServiceClient alloc] initWithEndpointURL:[NSURL URLWithString:AWSECServiceURLString]];
    });
    
    return _sharedClient;
}

/**
 Authentication of SOAP request
 see details here: http://docs.aws.amazon.com/AWSECommerceService/latest/DG/NotUsingWSSecurity.html
*/
- (void)authenticateRequest:(NSString *)action {
    
    // build timestamp
    NSDateFormatter *dataFormatter = [[[NSDateFormatter alloc] init] autorelease];
    [dataFormatter setDateFormat:@"yyyy-MM-dd'T'HH:mm:ss'Z'"];
    [dataFormatter setTimeZone:[NSTimeZone timeZoneWithName:@"UTC"]];
    NSString *timestamp = [dataFormatter stringFromDate:[NSDate date]];
    
    // build signature
    NSString *signatureInput = [action stringByAppendingString:timestamp];
    NSString *signature = [AmazonAuthUtils sha256HMac:[signatureInput dataUsingEncoding:NSUTF8StringEncoding] withKey:AWSSecureKeyId];
    
    // add SOAP headers
    self.customSoapHeaders = [NSMutableArray array];
    PicoXMLElement *accessKeyElement = [[[PicoXMLElement alloc] init] autorelease];
    accessKeyElement.nsUri = AuthHeaderNS;
    accessKeyElement.name = @"AWSAccessKeyId";
    accessKeyElement.value = AWSAccessKeyId;
    [self.customSoapHeaders addObject:accessKeyElement];
    PicoXMLElement *timestampElement = [[[PicoXMLElement alloc] init] autorelease];
    timestampElement.nsUri = AuthHeaderNS;
    timestampElement.name = @"Timestamp";
    timestampElement.value = timestamp;
    [self.customSoapHeaders addObject:timestampElement];
    PicoXMLElement *signatureElement = [[[PicoXMLElement alloc] init] autorelease];
    signatureElement.nsUri = AuthHeaderNS;
    signatureElement.name = @"Signature";
    signatureElement.value = signature;
    [self.customSoapHeaders addObject:signatureElement];
}

@end


{% endcodeblock %} 

Let me give more comments:
>1. Amazon Product Advertising API requires per-call request authentication, see details [here](http://docs.aws.amazon.com/AWSECommerceService/latest/DG/NotUsingWSSecurity.html), the authentication logic is implemented in the `(void)authenticateRequest:(NSString *)action` method.
2. To make the authentication work, you need to fill in your `AWSAccessKeyId` and `AWSSecureKeyId` in the shared client, if you are a registed Amazon Product Advertising API developer, you can get these keys from your account on Amazon developer site.
3. The authentication info is added in the SOAP request as SOAP header, to add SOAP header, we use `PicoXMLElement`(XML helper class provided in Pico) to build header element, then add the header element one by one into the `customSoapHeader` property(which is of type `NSMuatableArray`) of the client, for example: `[self.customSoapHeaders addObject:signatureElement]`, at runtime, Pico will add these elements into the SOAP request header.
4. The authentication needs to be done on per-call basis, means everytime you call an Amazon ECommerce service, you need to authenticate the request before the service call, see sample later.
4. The authentication relies on an utility class called `AmazonAuthUtils`, copy them from [here](https://github.com/bulldog2011/pico/blob/master/Examples/AWSECommerce/AWSECommerce/AmazonAuthUtils.h) and [here](https://github.com/bulldog2011/pico/blob/master/Examples/AWSECommerce/AWSECommerce/AmazonAuthUtils.m).

Now the UI part, we will search one book from Amazon by keywords, since this is a hello world like sample, we just need a UITextField for keyword input and UIButton to trigger Amazon search by invoking method `searchButtonPressed` which will indirectly call Amazon ECommerce `itemSearch` API through the proxy, fairly simple, see definition in header file [ViewController.h](https://github.com/bulldog2011/pico/blob/master/Examples/AWSECommerce/AWSECommerce/ViewController.h) and instantiation in implementation file [ViewController.m](https://github.com/bulldog2011/pico/blob/master/Examples/AWSECommerce/AWSECommerce/ViewController.m).

Now implement the `searchButtonPressed` method by invoking service as below:

{% codeblock ViewController.m lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/AWSECommerce/AWSECommerce/ViewController.m source  %}

#import "ViewController.h"
#import "AWSECommerceServiceClient.h"
#import "CommonTypes.h"
#import "Toast+UIView.h"
#import "SOAP11Fault.h"

- (void)searchButtonPressed:(id)sender
{
    // Hide the keyboard.
    [_searchText resignFirstResponder];
    
    if (_searchText.text.length > 0) {
        
        // start progress activity
        [self.view makeToastActivity];
        
        // get shared client
        AWSECommerceServiceClient *client = [AWSECommerceServiceClient sharedClient];
        client.debug = YES;
        
        // build request, see details here:
        ItemSearch *request = [[[ItemSearch alloc] init] autorelease];
        request.associateTag = @"tag"; // seems any tag is ok
        request.shared = [[[ItemSearchRequest alloc] init] autorelease];
        request.shared.searchIndex = @"Books";
        request.shared.responseGroup = [NSMutableArray arrayWithObjects:@"Images", @"Small", nil];
        ItemSearchRequest *itemSearchRequest = [[[ItemSearchRequest alloc] init] autorelease];
        itemSearchRequest.title = _searchText.text;
        request.request = [NSMutableArray arrayWithObject:itemSearchRequest];
        
        // authenticate the request
        // http://docs.aws.amazon.com/AWSECommerceService/latest/DG/NotUsingWSSecurity.html
        [client authenticateRequest:@"ItemSearch"];
        [client itemSearch:request success:^(ItemSearchResponse *responseObject) {
            // stop progress activity
            [self.view hideToastActivity];
            
            // success handling logic
            if (responseObject.items.count > 0) {
                Items *items = [responseObject.items objectAtIndex:0];
                if (items.item.count > 0) {
                    Item *item = [items.item objectAtIndex:0];
                    
                    // start image downloading progress activity
                    [self.view makeToastActivity];
                    // get gallery image
                    NSURL *imageURL = [NSURL URLWithString:item.smallImage.url];
                    NSData *imageData = [NSData dataWithContentsOfURL:imageURL];
                    // stop progress activity
                    [self.view hideToastActivity];
                    
                    UIImage *image = [UIImage imageWithData:imageData];
                    [self.view makeToast:item.itemAttributes.title duration:3.0 position:@"center" title:@"Success" image:image];
                } else {
                    // no result
                    [self.view makeToast:@"No result" duration:3.0 position:@"center"];
                }
                
            } else {
                // no result
                [self.view makeToast:@"No result" duration:3.0 position:@"center"];
            }
            // TODO
        } failure:^(NSError *error, id<PicoBindable> soapFault) {
            // stop progress activity
            [self.view hideToastActivity];
            
            // error handling logic
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


More comments to the serivce call code:

>1. I've added comments in the code so the whole service call flow should be easy to understand.
2. Before service call, the request must be authenticated by the calling `authenticateRequest` method, such as `[client authenticateRequest:@"ItemSearch"];`, `ItemSearch` is the target action or API name.
3. In the success handling logic, we show the title and the image of the returned item, for demo, the image is downloaded synchronously, but in practice, you should download image asynchronously in order not to block main UI.
4. In failure handling logic, we need to check both a `NSError`(indicating http or parsing error) and a `SOAPFault`(indicating server side returned error), then handle them accordingly.
5. Amazon ECommerce service supports response resident error(RRE), so even we get success response, we still need to check response for resident error, in the demo code above, I omitted response resident error checking for abbreviation, but in practice, you should check response resident error to make your app robust.
6. The generated code provides type hint as code comments, if you are confused with the type of a property, just consult the corresponding header file, for example, the `responseGroup` property of `ItemSearchRequest` type is of type `NSMuatableArray`, what's the actual entry type? By checking code comments in the `ItemSearchRequest.h` file, you will find the entry type is `NSString`, like below:

{% codeblock ItemSearchRequest.h lang:objc https://github.com/bulldog2011/pico/blob/master/Examples/AWSECommerce/AWSECommerce/awsecommerceservice/_2011_08_01/ItemSearchRequest.h source  %}

/**
(public property)
entry type : NSString, wrapper for primitive string
*/

@property (nonatomic, retain) NSMutableArray *responseGroup;

{% endcodeblock %}

At last, please don't forget to include the shared client and SOAP related header files, it's a best practice to include the generated CommonTypes.h file which can free you from writing many import statements required by request building and response handling.

##Final Step - Run the Demo

Let's run the demo in iPhone simulator, see a sceen shot below:

{% img center /images/pico/tutorial05/screen_shot1.png 300 500 %}

and the debug output:

{% codeblock %}

2013-03-31 12:33:14.912 HelloAWSECommerce[2604:c07] Sending request to : https://webservices.amazon.com/onca/soap?Service=AWSECommerceService
2013-03-31 12:33:14.913 HelloAWSECommerce[2604:c07] Request message:
2013-03-31 12:33:14.913 HelloAWSECommerce[2604:c07] <?xml version="1.0" encoding="UTF-8" ?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://webservices.amazon.com/AWSECommerceService/2011-08-01">
	<soapenv:Header xmlns:ns0="http://security.amazonaws.com/doc/2007-01-01/">
		<ns0:AWSAccessKeyId>***************</ns0:AWSAccessKeyId>
		<ns0:Timestamp>2013-03-31T04:33:14Z</ns0:Timestamp>
		<ns0:Signature>***************</ns0:Signature>
	</soapenv:Header>
	<soapenv:Body>
		<ItemSearch>
			<Request>
				<Title>Ios programming</Title>
			</Request>
			<Shared>
				<ResponseGroup>Images</ResponseGroup>
				<ResponseGroup>Small</ResponseGroup>
				<SearchIndex>Books</SearchIndex>
			</Shared>
			<AssociateTag>tag</AssociateTag>
		</ItemSearch>
	</soapenv:Body>
</soapenv:Envelope>
2013-03-31 12:33:14.913 HelloAWSECommerce[2604:c07] Request HTTP Headers : 
{
    Accept = "text/xml";
    "Accept-Language" = "en, fr, de, ja, nl, it, es, pt, pt-PT, da, fi, nb, sv, ko, zh-Hans, zh-Hant, ru, pl, tr, uk, ar, hr, cs, el, he, ro, sk, th, id, ms, en-GB, ca, hu, vi, en-us;q=0.8";
    "Content-Type" = "text/xml";
    SOAPAction = "http://soap.amazon.com/ItemSearch";
    "User-Agent" = "HelloAWSECommerce/1.0 (iPhone Simulator; iOS 6.0; Scale/1.00)";
}
2013-03-31 12:33:16.963 HelloAWSECommerce[2604:4533] Response HTTP status : 
200
2013-03-31 12:33:16.963 HelloAWSECommerce[2604:4533] Response HTTP headers : 
{
    "Content-Encoding" = gzip;
    "Content-Type" = "text/xml;charset=UTF-8";
    Date = "Sun, 31 Mar 2013 04:33:29 GMT";
    Server = Server;
    "Transfer-Encoding" = Identity;
    Vary = "Accept-Encoding,User-Agent";
    nnCoection = close;
}
2013-03-31 12:33:16.964 HelloAWSECommerce[2604:3e67] Response message : 
2013-03-31 12:33:16.964 HelloAWSECommerce[2604:3e67] <?xml version="1.0" ?><soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"><soapenv:Body><ItemSearchResponse xmlns="http://webservices.amazon.com/AWSECommerceService/2011-08-01"><OperationRequest><HTTPHeaders><Header Name="UserAgent" Value="HelloAWSECommerce/1.0 (iPhone Simulator; iOS 6.0; Scale/1.00)"></Header></HTTPHeaders><RequestId>6d6556d8-ec6f-4d8b-8461-bc25398d1244</RequestId><Arguments><Argument Name="Service" Value="AWSECommerceService"></Argument></Arguments><RequestProcessingTime>0.0923510000000000</RequestProcessingTime></OperationRequest><Items><Request><IsValid>True</IsValid><ItemSearchRequest><ResponseGroup>Images</ResponseGroup><ResponseGroup>Small</ResponseGroup><SearchIndex>Books</SearchIndex><Title>Ios programming</Title></ItemSearchRequest></Request><TotalResults>113</TotalResults><TotalPages>12</TotalPages><MoreSearchResultsUrl>http://www.amazon.com/gp/redirect.html?camp=2025&amp;creative=386001&amp;location=http%3A%2F%2Fwww.amazon.com%2Fgp%2Fsearch%3Fkeywords%3DIos%2Bprogramming%26url%3Dsearch-alias%253Dstripbooks&amp;linkCode=sp1&amp;tag=tag&amp;SubscriptionId=AKIAIEKQPISXI6IPHDFQ</MoreSearchResultsUrl><Item><ASIN>0321821521</ASIN><DetailPageURL>http://www.amazon.com/iOS-Programming-Ranch-Edition-Guides/dp/0321821521%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D165953%26creativeASIN%3D0321821521</DetailPageURL><ItemLinks><ItemLink><Description>Technical Details</Description><URL>http://www.amazon.com/iOS-Programming-Ranch-Edition-Guides/dp/tech-data/0321821521%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321821521</URL></ItemLink><ItemLink><Description>Add To Baby Registry</Description><URL>http://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3D0321821521%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321821521</URL></ItemLink><ItemLink><Description>Add To Wedding Registry</Description><URL>http://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3D0321821521%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321821521</URL></ItemLink><ItemLink><Description>Add To Wishlist</Description><URL>http://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3D0321821521%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321821521</URL></ItemLink><ItemLink><Description>Tell A Friend</Description><URL>http://www.amazon.com/gp/pdp/taf/0321821521%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321821521</URL></ItemLink><ItemLink><Description>All Customer Reviews</Description><URL>http://www.amazon.com/review/product/0321821521%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321821521</URL></ItemLink><ItemLink><Description>All Offers</Description><URL>http://www.amazon.com/gp/offer-listing/0321821521%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321821521</URL></ItemLink></ItemLinks><SmallImage><URL>http://ecx.images-amazon.com/images/I/41ybikkCyLL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">52</Width></SmallImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/41ybikkCyLL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">112</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/41ybikkCyLL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">349</Width></LargeImage><ImageSets><ImageSet Category="primary"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/41ybikkCyLL._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">21</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/41ybikkCyLL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">52</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/41ybikkCyLL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">52</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/41ybikkCyLL._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">77</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/41ybikkCyLL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">112</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/41ybikkCyLL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">349</Width></LargeImage></ImageSet></ImageSets><ItemAttributes><Author>Joe Conway</Author><Author>Aaron Hillegass</Author><Manufacturer>Big Nerd Ranch Guides</Manufacturer><ProductGroup>Book</ProductGroup><Title>iOS Programming: The Big Nerd Ranch Guide (3rd Edition) (Big Nerd Ranch Guides)</Title></ItemAttributes></Item><Item><ASIN>1449342752</ASIN><DetailPageURL>http://www.amazon.com/iOS-Programming-Cookbook-Vandad-Nahavandipoor/dp/1449342752%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D165953%26creativeASIN%3D1449342752</DetailPageURL><ItemLinks><ItemLink><Description>Technical Details</Description><URL>http://www.amazon.com/iOS-Programming-Cookbook-Vandad-Nahavandipoor/dp/tech-data/1449342752%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449342752</URL></ItemLink><ItemLink><Description>Add To Baby Registry</Description><URL>http://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3D1449342752%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449342752</URL></ItemLink><ItemLink><Description>Add To Wedding Registry</Description><URL>http://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3D1449342752%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449342752</URL></ItemLink><ItemLink><Description>Add To Wishlist</Description><URL>http://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3D1449342752%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449342752</URL></ItemLink><ItemLink><Description>Tell A Friend</Description><URL>http://www.amazon.com/gp/pdp/taf/1449342752%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449342752</URL></ItemLink><ItemLink><Description>All Customer Reviews</Description><URL>http://www.amazon.com/review/product/1449342752%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449342752</URL></ItemLink><ItemLink><Description>All Offers</Description><URL>http://www.amazon.com/gp/offer-listing/1449342752%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449342752</URL></ItemLink></ItemLinks><SmallImage><URL>http://ecx.images-amazon.com/images/I/51esysk15lL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></SmallImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51esysk15lL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">122</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51esysk15lL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">381</Width></LargeImage><ImageSets><ImageSet Category="primary"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/51esysk15lL._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">23</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/51esysk15lL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/51esysk15lL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/51esysk15lL._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">84</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51esysk15lL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">122</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51esysk15lL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">381</Width></LargeImage></ImageSet><ImageSet Category="variant"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/51m-eUUQZkL._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">23</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/51m-eUUQZkL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/51m-eUUQZkL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/51m-eUUQZkL._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">84</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51m-eUUQZkL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">122</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51m-eUUQZkL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">381</Width></LargeImage></ImageSet></ImageSets><ItemAttributes><Author>Vandad Nahavandipoor</Author><Manufacturer>O'Reilly Media</Manufacturer><ProductGroup>Book</ProductGroup><Title>iOS 6 Programming Cookbook</Title></ItemAttributes></Item><Item><ASIN>1449365760</ASIN><DetailPageURL>http://www.amazon.com/Programming-iOS-6-Matt-Neuburg/dp/1449365760%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D165953%26creativeASIN%3D1449365760</DetailPageURL><ItemLinks><ItemLink><Description>Technical Details</Description><URL>http://www.amazon.com/Programming-iOS-6-Matt-Neuburg/dp/tech-data/1449365760%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449365760</URL></ItemLink><ItemLink><Description>Add To Baby Registry</Description><URL>http://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3D1449365760%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449365760</URL></ItemLink><ItemLink><Description>Add To Wedding Registry</Description><URL>http://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3D1449365760%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449365760</URL></ItemLink><ItemLink><Description>Add To Wishlist</Description><URL>http://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3D1449365760%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449365760</URL></ItemLink><ItemLink><Description>Tell A Friend</Description><URL>http://www.amazon.com/gp/pdp/taf/1449365760%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449365760</URL></ItemLink><ItemLink><Description>All Customer Reviews</Description><URL>http://www.amazon.com/review/product/1449365760%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449365760</URL></ItemLink><ItemLink><Description>All Offers</Description><URL>http://www.amazon.com/gp/offer-listing/1449365760%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449365760</URL></ItemLink></ItemLinks><SmallImage><URL>http://ecx.images-amazon.com/images/I/51ZCvz9cG8L._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></SmallImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51ZCvz9cG8L._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">122</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51ZCvz9cG8L.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">381</Width></LargeImage><ImageSets><ImageSet Category="primary"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/51ZCvz9cG8L._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">23</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/51ZCvz9cG8L._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/51ZCvz9cG8L._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/51ZCvz9cG8L._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">84</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51ZCvz9cG8L._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">122</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51ZCvz9cG8L.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">381</Width></LargeImage></ImageSet></ImageSets><ItemAttributes><Author>Matt Neuburg</Author><Manufacturer>O'Reilly Media</Manufacturer><ProductGroup>Book</ProductGroup><Title>Programming iOS 6</Title></ItemAttributes></Item><Item><ASIN>1118449959</ASIN><DetailPageURL>http://www.amazon.com/iOS-Programming-Pushing-Limits-Application/dp/1118449959%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D165953%26creativeASIN%3D1118449959</DetailPageURL><ItemLinks><ItemLink><Description>Technical Details</Description><URL>http://www.amazon.com/iOS-Programming-Pushing-Limits-Application/dp/tech-data/1118449959%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118449959</URL></ItemLink><ItemLink><Description>Add To Baby Registry</Description><URL>http://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3D1118449959%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118449959</URL></ItemLink><ItemLink><Description>Add To Wedding Registry</Description><URL>http://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3D1118449959%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118449959</URL></ItemLink><ItemLink><Description>Add To Wishlist</Description><URL>http://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3D1118449959%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118449959</URL></ItemLink><ItemLink><Description>Tell A Friend</Description><URL>http://www.amazon.com/gp/pdp/taf/1118449959%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118449959</URL></ItemLink><ItemLink><Description>All Customer Reviews</Description><URL>http://www.amazon.com/review/product/1118449959%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118449959</URL></ItemLink><ItemLink><Description>All Offers</Description><URL>http://www.amazon.com/gp/offer-listing/1118449959%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118449959</URL></ItemLink></ItemLinks><SmallImage><URL>http://ecx.images-amazon.com/images/I/51gEXI6QMZL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">58</Width></SmallImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51gEXI6QMZL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">124</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51gEXI6QMZL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">388</Width></LargeImage><ImageSets><ImageSet Category="primary"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/51gEXI6QMZL._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">23</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/51gEXI6QMZL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">58</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/51gEXI6QMZL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">58</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/51gEXI6QMZL._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">85</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51gEXI6QMZL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">124</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51gEXI6QMZL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">388</Width></LargeImage></ImageSet></ImageSets><ItemAttributes><Author>Rob Napier</Author><Author>Mugunth Kumar</Author><Manufacturer>Wiley</Manufacturer><ProductGroup>Book</ProductGroup><Title>iOS 6 Programming Pushing the Limits: Advanced Application Development for Apple iPhone, iPad and iPod Touch</Title></ItemAttributes></Item><Item><ASIN>0321741838</ASIN><DetailPageURL>http://www.amazon.com/Learning-OpenGL-iOS-Hands-Programming/dp/0321741838%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D165953%26creativeASIN%3D0321741838</DetailPageURL><ItemLinks><ItemLink><Description>Technical Details</Description><URL>http://www.amazon.com/Learning-OpenGL-iOS-Hands-Programming/dp/tech-data/0321741838%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321741838</URL></ItemLink><ItemLink><Description>Add To Baby Registry</Description><URL>http://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3D0321741838%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321741838</URL></ItemLink><ItemLink><Description>Add To Wedding Registry</Description><URL>http://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3D0321741838%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321741838</URL></ItemLink><ItemLink><Description>Add To Wishlist</Description><URL>http://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3D0321741838%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321741838</URL></ItemLink><ItemLink><Description>Tell A Friend</Description><URL>http://www.amazon.com/gp/pdp/taf/0321741838%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321741838</URL></ItemLink><ItemLink><Description>All Customer Reviews</Description><URL>http://www.amazon.com/review/product/0321741838%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321741838</URL></ItemLink><ItemLink><Description>All Offers</Description><URL>http://www.amazon.com/gp/offer-listing/0321741838%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321741838</URL></ItemLink></ItemLinks><SmallImage><URL>http://ecx.images-amazon.com/images/I/51wqjU0VTSL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">58</Width></SmallImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51wqjU0VTSL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">124</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51wqjU0VTSL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">388</Width></LargeImage><ImageSets><ImageSet Category="primary"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/51wqjU0VTSL._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">23</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/51wqjU0VTSL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">58</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/51wqjU0VTSL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">58</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/51wqjU0VTSL._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">85</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51wqjU0VTSL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">124</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51wqjU0VTSL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">388</Width></LargeImage></ImageSet></ImageSets><ItemAttributes><Author>Erik M. Buck</Author><Manufacturer>Addison-Wesley Professional</Manufacturer><ProductGroup>Book</ProductGroup><Title>Learning OpenGL ES for iOS: A Hands-on Guide to Modern 3D Graphics Programming</Title></ItemAttributes></Item><Item><ASIN>0321636848</ASIN><DetailPageURL>http://www.amazon.com/Learning-Core-Audio-Hands-On-Programming/dp/0321636848%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D165953%26creativeASIN%3D0321636848</DetailPageURL><ItemLinks><ItemLink><Description>Technical Details</Description><URL>http://www.amazon.com/Learning-Core-Audio-Hands-On-Programming/dp/tech-data/0321636848%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321636848</URL></ItemLink><ItemLink><Description>Add To Baby Registry</Description><URL>http://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3D0321636848%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321636848</URL></ItemLink><ItemLink><Description>Add To Wedding Registry</Description><URL>http://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3D0321636848%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321636848</URL></ItemLink><ItemLink><Description>Add To Wishlist</Description><URL>http://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3D0321636848%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321636848</URL></ItemLink><ItemLink><Description>Tell A Friend</Description><URL>http://www.amazon.com/gp/pdp/taf/0321636848%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321636848</URL></ItemLink><ItemLink><Description>All Customer Reviews</Description><URL>http://www.amazon.com/review/product/0321636848%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321636848</URL></ItemLink><ItemLink><Description>All Offers</Description><URL>http://www.amazon.com/gp/offer-listing/0321636848%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321636848</URL></ItemLink></ItemLinks><SmallImage><URL>http://ecx.images-amazon.com/images/I/51AzGxb1nNL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">58</Width></SmallImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51AzGxb1nNL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">124</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51AzGxb1nNL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">388</Width></LargeImage><ImageSets><ImageSet Category="primary"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/51AzGxb1nNL._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">23</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/51AzGxb1nNL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">58</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/51AzGxb1nNL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">58</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/51AzGxb1nNL._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">85</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51AzGxb1nNL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">124</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51AzGxb1nNL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">388</Width></LargeImage></ImageSet></ImageSets><ItemAttributes><Author>Chris Adamson</Author><Author>Kevin Avila</Author><Manufacturer>Addison-Wesley Professional</Manufacturer><ProductGroup>Book</ProductGroup><Title>Learning Core Audio: A Hands-On Guide to Audio Programming for Mac and iOS</Title></ItemAttributes></Item><Item><ASIN>1118362403</ASIN><DetailPageURL>http://www.amazon.com/Professional-iOS-Network-Programming-Connecting/dp/1118362403%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D165953%26creativeASIN%3D1118362403</DetailPageURL><ItemLinks><ItemLink><Description>Technical Details</Description><URL>http://www.amazon.com/Professional-iOS-Network-Programming-Connecting/dp/tech-data/1118362403%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118362403</URL></ItemLink><ItemLink><Description>Add To Baby Registry</Description><URL>http://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3D1118362403%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118362403</URL></ItemLink><ItemLink><Description>Add To Wedding Registry</Description><URL>http://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3D1118362403%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118362403</URL></ItemLink><ItemLink><Description>Add To Wishlist</Description><URL>http://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3D1118362403%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118362403</URL></ItemLink><ItemLink><Description>Tell A Friend</Description><URL>http://www.amazon.com/gp/pdp/taf/1118362403%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118362403</URL></ItemLink><ItemLink><Description>All Customer Reviews</Description><URL>http://www.amazon.com/review/product/1118362403%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118362403</URL></ItemLink><ItemLink><Description>All Offers</Description><URL>http://www.amazon.com/gp/offer-listing/1118362403%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1118362403</URL></ItemLink></ItemLinks><SmallImage><URL>http://ecx.images-amazon.com/images/I/519rZ2Pp9aL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">59</Width></SmallImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/519rZ2Pp9aL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">126</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/519rZ2Pp9aL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">394</Width></LargeImage><ImageSets><ImageSet Category="primary"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/519rZ2Pp9aL._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">24</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/519rZ2Pp9aL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">59</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/519rZ2Pp9aL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">59</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/519rZ2Pp9aL._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">87</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/519rZ2Pp9aL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">126</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/519rZ2Pp9aL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">394</Width></LargeImage></ImageSet></ImageSets><ItemAttributes><Author>Jack Cox</Author><Author>Nathan Jones</Author><Author>John Szumski</Author><Manufacturer>Wrox</Manufacturer><ProductGroup>Book</ProductGroup><Title>Professional iOS Network Programming: Connecting the Enterprise to the iPhone and iPad</Title></ItemAttributes></Item><Item><ASIN>1449359345</ASIN><DetailPageURL>http://www.amazon.com/Learning-iOS-Programming-Xcode-Store/dp/1449359345%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D165953%26creativeASIN%3D1449359345</DetailPageURL><ItemLinks><ItemLink><Description>Technical Details</Description><URL>http://www.amazon.com/Learning-iOS-Programming-Xcode-Store/dp/tech-data/1449359345%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449359345</URL></ItemLink><ItemLink><Description>Add To Baby Registry</Description><URL>http://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3D1449359345%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449359345</URL></ItemLink><ItemLink><Description>Add To Wedding Registry</Description><URL>http://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3D1449359345%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449359345</URL></ItemLink><ItemLink><Description>Add To Wishlist</Description><URL>http://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3D1449359345%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449359345</URL></ItemLink><ItemLink><Description>Tell A Friend</Description><URL>http://www.amazon.com/gp/pdp/taf/1449359345%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449359345</URL></ItemLink><ItemLink><Description>All Customer Reviews</Description><URL>http://www.amazon.com/review/product/1449359345%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449359345</URL></ItemLink><ItemLink><Description>All Offers</Description><URL>http://www.amazon.com/gp/offer-listing/1449359345%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D1449359345</URL></ItemLink></ItemLinks><SmallImage><URL>http://ecx.images-amazon.com/images/I/41N%2BYIc3wTL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></SmallImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/41N%2BYIc3wTL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">122</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/41N%2BYIc3wTL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">381</Width></LargeImage><ImageSets><ImageSet Category="primary"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/41N%2BYIc3wTL._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">23</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/41N%2BYIc3wTL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/41N%2BYIc3wTL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/41N%2BYIc3wTL._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">84</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/41N%2BYIc3wTL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">122</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/41N%2BYIc3wTL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">381</Width></LargeImage></ImageSet></ImageSets><ItemAttributes><Author>Alasdair Allan</Author><Manufacturer>O'Reilly Media</Manufacturer><ProductGroup>Book</ProductGroup><Title>Learning iOS Programming: From Xcode to App Store</Title></ItemAttributes></Item><Item><ASIN>0321699424</ASIN><DetailPageURL>http://www.amazon.com/Learning-iOS-Game-Programming-Hands-On/dp/0321699424%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D165953%26creativeASIN%3D0321699424</DetailPageURL><ItemLinks><ItemLink><Description>Technical Details</Description><URL>http://www.amazon.com/Learning-iOS-Game-Programming-Hands-On/dp/tech-data/0321699424%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321699424</URL></ItemLink><ItemLink><Description>Add To Baby Registry</Description><URL>http://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3D0321699424%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321699424</URL></ItemLink><ItemLink><Description>Add To Wedding Registry</Description><URL>http://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3D0321699424%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321699424</URL></ItemLink><ItemLink><Description>Add To Wishlist</Description><URL>http://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3D0321699424%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321699424</URL></ItemLink><ItemLink><Description>Tell A Friend</Description><URL>http://www.amazon.com/gp/pdp/taf/0321699424%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321699424</URL></ItemLink><ItemLink><Description>All Customer Reviews</Description><URL>http://www.amazon.com/review/product/0321699424%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321699424</URL></ItemLink><ItemLink><Description>All Offers</Description><URL>http://www.amazon.com/gp/offer-listing/0321699424%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321699424</URL></ItemLink></ItemLinks><SmallImage><URL>http://ecx.images-amazon.com/images/I/51cbd0Dp-7L._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></SmallImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51cbd0Dp-7L._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">122</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51cbd0Dp-7L.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">381</Width></LargeImage><ImageSets><ImageSet Category="primary"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/51cbd0Dp-7L._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">23</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/51cbd0Dp-7L._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/51cbd0Dp-7L._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">57</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/51cbd0Dp-7L._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">84</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/51cbd0Dp-7L._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">122</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/51cbd0Dp-7L.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">381</Width></LargeImage></ImageSet></ImageSets><ItemAttributes><Author>Michael Daley</Author><Manufacturer>Addison-Wesley Professional</Manufacturer><ProductGroup>Book</ProductGroup><Title>Learning iOS Game Programming: A Hands-On Guide to Building Your First iPhone Game</Title></ItemAttributes></Item><Item><ASIN>0321773772</ASIN><DetailPageURL>http://www.amazon.com/iOS-Programming-Ranch-Edition-Guides/dp/0321773772%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D165953%26creativeASIN%3D0321773772</DetailPageURL><ItemLinks><ItemLink><Description>Technical Details</Description><URL>http://www.amazon.com/iOS-Programming-Ranch-Edition-Guides/dp/tech-data/0321773772%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321773772</URL></ItemLink><ItemLink><Description>Add To Baby Registry</Description><URL>http://www.amazon.com/gp/registry/baby/add-item.html%3Fasin.0%3D0321773772%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321773772</URL></ItemLink><ItemLink><Description>Add To Wedding Registry</Description><URL>http://www.amazon.com/gp/registry/wedding/add-item.html%3Fasin.0%3D0321773772%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321773772</URL></ItemLink><ItemLink><Description>Add To Wishlist</Description><URL>http://www.amazon.com/gp/registry/wishlist/add-item.html%3Fasin.0%3D0321773772%26SubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321773772</URL></ItemLink><ItemLink><Description>Tell A Friend</Description><URL>http://www.amazon.com/gp/pdp/taf/0321773772%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321773772</URL></ItemLink><ItemLink><Description>All Customer Reviews</Description><URL>http://www.amazon.com/review/product/0321773772%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321773772</URL></ItemLink><ItemLink><Description>All Offers</Description><URL>http://www.amazon.com/gp/offer-listing/0321773772%3FSubscriptionId%3DAKIAIEKQPISXI6IPHDFQ%26tag%3Dtag%26linkCode%3Dsp1%26camp%3D2025%26creative%3D386001%26creativeASIN%3D0321773772</URL></ItemLink></ItemLinks><SmallImage><URL>http://ecx.images-amazon.com/images/I/412zhvqyIXL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">52</Width></SmallImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/412zhvqyIXL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">110</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/412zhvqyIXL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">345</Width></LargeImage><ImageSets><ImageSet Category="primary"><SwatchImage><URL>http://ecx.images-amazon.com/images/I/412zhvqyIXL._SL30_.jpg</URL><Height Units="pixels">30</Height><Width Units="pixels">21</Width></SwatchImage><SmallImage><URL>http://ecx.images-amazon.com/images/I/412zhvqyIXL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">52</Width></SmallImage><ThumbnailImage><URL>http://ecx.images-amazon.com/images/I/412zhvqyIXL._SL75_.jpg</URL><Height Units="pixels">75</Height><Width Units="pixels">52</Width></ThumbnailImage><TinyImage><URL>http://ecx.images-amazon.com/images/I/412zhvqyIXL._SL110_.jpg</URL><Height Units="pixels">110</Height><Width Units="pixels">76</Width></TinyImage><MediumImage><URL>http://ecx.images-amazon.com/images/I/412zhvqyIXL._SL160_.jpg</URL><Height Units="pixels">160</Height><Width Units="pixels">110</Width></MediumImage><LargeImage><URL>http://ecx.images-amazon.com/images/I/412zhvqyIXL.jpg</URL><Height Units="pixels">500</Height><Width Units="pixels">345</Width></LargeImage></ImageSet></ImageSets><ItemAttributes><Author>Joe Conway</Author><Author>Aaron Hillegass</Author><Manufacturer>Big Nerd Ranch Guides</Manufacturer><ProductGroup>Book</ProductGroup><Title>iOS Programming: The Big Nerd Ranch Guide (2nd Edition) (Big Nerd Ranch Guides)</Title></ItemAttributes></Item></Items></ItemSearchResponse></soapenv:Body></soapenv:Envelope>

{% endcodeblock %}


If you forget to fill in your AWS Access Key and Secure Key in the shared client, then you will get error like below:

{% img center /images/pico/tutorial05/screen_shot2.png 300 500 %}


This is just a bare minimum Amazon ECommerce service based application, for a demo with more functions, please see the AWSECDemoApp sample in the `Examples` folder of Pico source, AWSECDemoApp calls following APIs in one app:
>1. ***itemSearch*** for book search
2. ***cartCreate*** to add chosen book into shopping cart

Below is a few screen shots:

Search:
{% img center /images/pico/tutorial05/screen_shot3.png 300 500 %}

Details:
{% img center /images/pico/tutorial05/screen_shot4.png 300 500 %}

Add To Cart:
{% img center /images/pico/tutorial05/screen_shot5.png 300 500 %}

Now it's your turn to create iOS applications based on Amazon ECommerce web serivces, see your next great serivce based app.


### Update 1
The Amazon Product Advertising API Proxy has been extracted as a standalone project, hosted [here](https://github.com/bulldog2011/PicoAWSECommerceServiceClient), and the corresponding appledoc is hosted [here](http://bulldog2011.github.com/PicoAWSECommerceServiceClient/), the appledoc is a useful programming reference. By the way, the doc annotations in wsdl are not only generted into the proxy code, but into the appledoc, assisting your development.










