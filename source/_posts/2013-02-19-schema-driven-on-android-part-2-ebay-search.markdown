---
layout: post
title: "Schema driven web service client development on Android, Part 2: eBay Search App"
date: 2013-02-19 20:47
comments: true
categories: [nano, ebay api]
keywords: ebay Finding API, eBay Finding API proxy for Android, eBay search app on Android
description: build a full eBay search Android app by leveraging code base created in part 1
---

This is the second part of my schema driven web service client development on Android series, in [part one](http://bulldog2011.github.com/blog/2013/02/17/schema-driven-on-android-part-1-hello-ebay-finding/), I introduced the blueprint of scheam driven development on Android, then I created a web service client proxy for eBay Finding API and built a minimum App as demo. In this second part, I will continue to create a functional eBay search app on Android by leveraging the proxy created in part one.

<!--more-->

There is a [good post](http://huguesjohnson.com/programming/java/android-ebay/) showing how to integrate Android Application with the eBay API, the author did a very good job, the steps are shown in great detail, this is definitely the recommended reading if you want to develop eBay API based application on Android, however, the author created the application with much low-level passing and plumbing code, there are at least three problems with such approach:  
1. The effort to create such simple application is nontrival, this is enough to scare away average developers.  
2. The application is hard to maintain, whenever eBay changes some API interface, much effort is needed to rewrite and debug the json parsing code.  
3. The application is hard to scale, if more functions and more API calls are needed, much effort is needed to write additional parsing and calling code.  

I've created a similar application without much effort, I just leveraged the proxy code base created in part one of this series and focused my effort on writing some application logic and UI code. In fact, since I don't get troubled in backend parsing and plumbing code, I can write far less code than the above mentioned author, this is the power of scheam driven or proxy driven development, below is the main UI of the finished application.

{% img center /images/nano-rest/ebay_search.png 400 600 %}

I won't post much code here since this is a typical andorid application and the code of the whole application is quite self explanatory, you can download the whole application [here](https://github.com/bulldog2011/nano-rest/tree/master/sample/EBaySearch)

Let me drop a few notes about this application here:

1. The application uses the FindItemsByKeywords request processor built in part 1, and the API call follows the same paradigm shown in part 1.
2. This is a typical eBay search application, there is only one [main activity](https://github.com/bulldog2011/nano-rest/blob/master/sample/EBaySearch/src/com/leansoft/nanorest/sample/FindingActivity.java) which extends ListActivity, user input a search keyword and click search, the application calls the FindItemsByKeywords request processor with asynchronous callback specified, inside the callback, UI is updated by populating the list view with search result items, not much code in the main activity.
3. [Droidfu WebImageView](https://github.com/mttkay/droid-fu/blob/master/src/main/java/com/github/droidfu/widgets/WebImageView.java) component is used to shown ebay item gallary thumbnail image, note, I removed the cache part of the component to minimize the code base, in real application, image cache is required for better performance.
4. I've added a dynamic sliding with pagination feature which makes the applciation look cool and friendly.
5. Before you run the application, please don't forget to fill your ***APP NAME*** in the [AppNameAuthenticationProvider class](https://github.com/bulldog2011/nano-rest/blob/master/sample/EBaySearch/src/com/ebay/finding/auth/AppNameAuthenticationProvider.java).


With the eBay Finding API proxy as SDK and the sample app as template, it's not hard for you to create a more functional eBay Finding App on Android, I am looking forward that someone will build one, and let me know when you build one.

Let me reinterate three main ***highhights*** of scheam driven development on Android:

1. Development effort is minimized, focus on application and UI logic instead of low level plumbling code.
2. Easy to maintain and scale, whenever the schema changes, resynchronize the proxy with the new schema and update application and UI logic accordingly, no manual xml parsing and debugging anymore.
3. Better quality and reliability, code generation and component resue foster better software quality and reliability.

####Update 1 (2.23.2013)
The eBay Search App has been enhanced to show how to mix API calls in one App:

1. Search eBay using eBay Finding [FindItemsByKeywords](http://developer.ebay.com/DevZone/finding/CallRef/findItemsByKeywords.html) API.
2. Show item details using eBay Shopping [GetSingleItem](http://developer.ebay.com/DevZone/shopping/docs/CallRef/GetSingleItem.html) API.
3. Add item to watch list using eBay Trading [AddToWatchList](http://developer.ebay.com/DevZone/XML/docs/Reference/eBay/AddToWatchList.html) API

below is the item details UI:

{% img center /images/nano-rest/ebay_demo.png 400 600 %}

Now with eBay Finding/Shopping/Trading API proxy as SDK and the demo app as template, you may create whatever eBay application you can think of, just let loose your imagination!

You can get the whole source of the enhanced App [here](https://github.com/bulldog2011/nano-rest/tree/master/sample/EBayDemo), Note, before you run the App, please fill in your eBay AppId and auth token in the [ConfigFactory](https://github.com/bulldog2011/nano-rest/blob/master/sample/EBayDemo/src/com/leansoft/nanorest/sample/ConfigFactory.java) class.











