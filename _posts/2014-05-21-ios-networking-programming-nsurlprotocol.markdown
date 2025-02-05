---
layout: post
title: "iOS Networking Programming--NSURLProtocol"
date: 2014-05-21 20:15:52 +0800
published: false
comments: true
categories: [Archives, iOS Development]
keywords: iOS, Networking, Programming, NSURLProtocol
description: iOS Networking Programming--NSURLProtocol
---
##Networking Programming

##Unit Test

##Concurrency Programming

##Core Graphics

##Core Animation

##Design Pattern

##URL Loading
NSURLSession
NSURLConnection
NSURLDownload
NSURLResponse (NSHTTPURLResponse)
NSURLRequest (NSMutableURLRequest)

##Authentication and Credentials
NSURLProtectionSpace
NSURLCredentialStorage
NSURLCredential
NSURLAuthenticationChanllenge

##Configuration Management
NSURLSessionConfiguration

##Protocol Support
NSURLProtocol

##Cookie Storage
NSHTTPCookieStorage
NSHTTPCookie

##Cache Management
NSURLCache
NSCacheURLRequest

##NSURLProtocol
The URL loading system design allows a client app to extend the protocols that are supported for transferring data. The URL loading system natively supports the http, https, file, ftp, and data protocols.

You can implement a custom protocol by subclassing NSURLProtocol and then registering the new class with the URL loading system using the NSURLProtocol class method registerClass:. When an NSURLSession, NSURLConnection, or NSURLDownload object initiates a connection for an NSURLRequest object, the URL loading system consults each of the registered classes in the reverse order of their registration. The first class that returns YES for a canInitWithRequest: message is used to handle the request.

If your custom protocol requires additional properties for its requests or responses, you support them by creating categories on the NSURLRequest, NSMutableURLRequest, and NSURLResponse classes that provide accessors for those properties. The NSURLProtocol class provides methods for setting and getting property values in those accessors.

The URL loading system is responsible for creating and releasing NSURLProtocol instances when connections start and complete. Your app should never create an instance of NSURLProtocol directly.

When an NSURLProtocol subclass is initialized by the URL loading system, it is provided a client object that conforms to the NSURLProtocolClient protocol. The NSURLProtocol subclass sends messages from the NSURLProtocolClient protocol to the client object to inform the URL loading system of its actions as it creates a response, receives data, redirects to a new URL, requires authentication, and completes the load. If the custom protocol supports authentication, then it must conform to the NSURLAuthenticationChallengeSender protocol.

##Subclass Note
Follow Methods must implement:
``` objective-c
+ (BOOL)canInitWithRequest:(NSURLRequest *)request;

+ (NSURLRequest *)canonicalRequestForRequest:(NSURLRequest *)request;

- (void)startLoading;

- (void)stopLoading;
```
