---
layout: post
title: "pico and ebay trading api integration how to"
date: 2013-04-01 20:18
comments: true
categories: [pico]
keywords: ios, iphone, wsdl, soap, ebay trading api
description: pico and ebay trading api integration how to
---

I've written a series of tutorials showing to put wsdl driven development into practice on iOS platform using [Pico](https://github.com/bulldog2011/pico) framework. In [tutorial 3](http://bulldog2011.github.com/blog/2013/03/29/pico-tutorial-3-hello-ebay-finding/) and [tutorial 4](http://bulldog2011.github.com/blog/2013/03/30/pico-tutorial-4-hello-ebay-shopping/) I showed how to integrate Pico with eBay Finding and Shopping API. This post is about Pico integration with [eBay Trading API](https://www.x.com/developers/ebay/products/trading-api) - the most heavy-weight and feature rich API from eBay, I am not going to write another tutorial for eBay Trading API integration, since the flow is quite smiliar to that already elaborated in tutorial 3 and 4, instead, I will dump a few comments here to give you some guide in case you want to integrate eBay Trading API on iOS device using Pico framework.

<!--more-->

### 1. eBay Trading WSDL Fix

The [eBay Trading WSDL](http://developer.ebay.com/webservices/latest/ebaySvc.wsdl) has issue to work with [mwsc](https://github.com/bulldog2011/mwsc) code generator directly, following fix is needed before mwsc can correctly generate code from the wsdl:

>1. Remove an any element declaration in ReviseInventoryStatusRequestType, since any has already been declarated in the AbstractRequestType(from which ReviseInventoryStatusRequestType extends).
2. Add following annotation in the schema definition(at the beginning):

{% codeblock lang:xml %}
   <xs:annotation>
	 <xs:appinfo>
	   <jaxb:globalBindings typesafeEnumMaxMembers="1000" />
	 </xs:appinfo>
   </xs:annotation>
{% endcodeblock %}
This is because the numbe of members in some enum types in wsdl exceeds the maximum allowed by defult xjc/jaxb processor. Of couse, you also need to declare jaxb namespace and version in wsdl/schema definition. [Here](https://github.com/bulldog2011/PicoEBayTradingClient/tree/master/wsdl) is a fixed eBay Trading wsdl(version 815), you can search `mwsc` to find out what I have fixed.

### 2. Code Generation Command
The command to generate eBay Trading proxy from wsdl is:

{% codeblock %}

bin/mwsc -pico -prefix Trading_ -ebaytrading ebaySvc_815_fix.wsdl

{% endcodeblock %}

the `-ebaytrading` is a special option I added for eBay Trading API code generation only, for auto-generating per-call HTTP header setting, this is not a generic code generator option.

### 3. eBay Trading WSDL pruning
The eBay Trading WSDL is a super big wsdl, more than 4M in size, the generated code is also very big in size, leading to slow compilation in Xcode, while in most cases, your application only needs a few calls in eBay Trading API, fortunately, eBay provides a [pruner](http://developer.ebay.com/DevZone/codebase/wsdlpruner/pruner.zip) tool which let you prune the bulky wsdl down to the operations that you want to use. [here](https://github.com/bulldog2011/PicoEBayTradingClient/tree/master/Examples/HelloeBayTrading/wsdl) and [here](https://github.com/bulldog2011/PicoEBayTradingClient/tree/master/Examples/eBayDemoApp/wsdl) are two wsdl I pruned for demo, the first one only supports `geteBayOfficialTime` call, and the second only supports `getWatchList` and `getMyeBayBuying` calls. You may prune the eBay Trading wsdl according to your real needs before code generation.

### 4. Samples
I've created two samples to demonstrate Pico integration with eBay Trading API, the [first one](https://github.com/bulldog2011/PicoEBayTradingClient/tree/master/Examples/HelloeBayTrading) is a hello world like sample, just call eBay Trading `geteBayOfficalTime` API to show official time on eBay server; the [second one](https://github.com/bulldog2011/PicoEBayTradingClient/tree/master/Examples/eBayDemoApp) is a composite sample which calls eBay Finding, Shopping and Trading APIs behind:

>1. Search items on eBay by keywords by invoking eBay Finding `findItemsByKeywords` API.
2. Show detail of an item by invoking eBay Shopping `getSingleItem` API. 
3. Add item to watch list by invoking eBay Trading `addToWatchList` API.
4. View watch list by invoking eBay Trading `getMyeBayBuying` API.

You may review and study these samples before you create your own Pico and eBay API based app.

To run first sample, you must fill in your `eBay AppId` and `eBay Auth Token` in the shared client class `EBayTradingServiceClient`

To run second sample, you mst fill in your `eBay AppId` in three shared client classes: `EBayFindingServiceClient`, `EBayShoppingServiceClient` and `EBayTradingServiceClient`, then fill in your `eBay Auth Token` in `EBayTradingServiceClient`, this just looks tedious and silly, I did so since this is just for demo, in real world app, I suggest you to centralize the credential setting in one place.

***Note***, for demo, credentials like eBay AppId and AuthToken are hardcoded in the sample, in real-world application, for security consideration, you need to integrate with eBay [Authentication & Authorization](http://developer.ebay.com/DevZone/XML/docs/WebHelp/wwhelp/wwhimpl/common/html/wwhelp.htm?context=eBay_XML_API&file=GettingTokens-Getting_Tokens_for_Applications_with_Multiple_Users.html) flow on your iOS device, and ensure the seurity of user credentials on device.

### 5. Regrading Pico Integration
Basically, the Pico integration with eBay Trading API is similar to Pico integartion with eBay Finding and Shopping API, one difference is eBay Trading API call needs to append a few request parameters(see details [here](http://developer.ebay.com/DevZone/XML/docs/WebHelp/wwhelp/wwhimpl/common/html/wwhelp.htm?context=eBay_XML_API&file=InvokingWebServices-.html)) as query string, plese refer to code [here](https://github.com/bulldog2011/PicoEBayTradingClient/blob/master/Examples/HelloeBayTrading/HelloeBayTrading/EBayTradingServiceClient.m) to learn how to add request parameters as query sting on Pico client. 

### 6. Standalone Site for eBay Trading Proxy
The eBay Trading Proxy has been extracted as a standalone project, hosted [here](https://github.com/bulldog2011/PicoEBayTradingClient), and the corresponding appledoc is hosted [here](http://bulldog2011.github.com/PicoEBayTradingClient/), the appledoc is a useful programming reference. By the way, the doc annotations in wsdl are not only generted into the proxy code, but into the appledoc, assisting your development.


###Final Word
eBay Trading API is a powerful API, you can use it to build applications such as listing, selling and post-sales management applications, manage user information, and initiate the item purchase flow on eBay. See your next greap iOS app based on Pico and eBay Trading API.






