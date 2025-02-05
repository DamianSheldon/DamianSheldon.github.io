---
layout: post
title: "iOS UIWebView与JavaScript交互"
date: 2014-05-14 11:05:53 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS, UIWebView, JavaScript, 交互
description: iOS UIWebView与JavaScript交互
---

##1.UIWebView调用JavaScript
###1.1调用html中已有的JavaScript function
假设html中的JavaScript有名为JSFunction(arg)的函数，可用如下方法调用：
``` objective-c 
NSString *js = [NSString stringWithFormat:@"JSFunction('OC---Call-->JS')"];
    
NSString *result = [self.webView stringByEvaluatingJavaScriptFromString:js];
```

###1.2注入JavaScript function,然后调用
我们还可以先向html中注入JavaScript function,然后调用。
通常可以在UIWebViewDelegate中注入。示例如下：
``` objective-c 
-(void)webViewDidFinishLoad:(UIWebView *)webView {
    
    [self.webView stringByEvaluatingJavaScriptFromString:@"function injectJSFunction (parameter) { return parameter + 1;}"];
}

// Call injectJSFunction from somewhere else
    NSString *result = [self.webView stringByEvaluatingJavaScriptFromString:@"injectJSFunction(1)"];

```
<!-- more -->

##2.JavaScript调用Objective-C Method
JavaScript调用Objective-C方法的原理是利用UIWebView的重定向请求，传一些命令到我们的UIWebView,在UIWebView的delegate的方法中接收这些命令，并根据命令执行相应的Objc方法。示例如下：
``` javascript
function sendCommand(cmd,param){  
    var url="objc:"+cmd+":"+param;  
    document.location = url;  
}  
function clickLink(){  
    sendCommand("alert","hello objective-c method");  
}  
```
``` objective-c
#pragma mark --  
#pragma mark UIWebViewDelegate  
  
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {  
      
    NSString *requestString = [[request URL] absoluteString];  
    NSArray *components = [requestString componentsSeparatedByString:@":"];  
    if ([components count] > 1 && [(NSString *)[components objectAtIndex:0] isEqualToString:@"objc"]) {  
        if([(NSString *)[components objectAtIndex:1] isEqualToString:@"alert"])   
        {  
            UIAlertView *alert = [[UIAlertView alloc]   
                                  initWithTitle:@"Alert from Cocoa Touch" message:[components objectAtIndex:2]  
                                  delegate:self cancelButtonTitle:nil  
                                  otherButtonTitles:@"OK", nil];  
            [alert show];  
        }  
        return NO;  
    }  
    return YES;  
} 
```
##3.相互传值
##3.1UIWebView传值给JavaScript
UIWebView调用JavaScript接口方法的返回值就是JavaScript传给UIWebView的值。示例如下：  

``` objective-c
NSString *result = [self.webView stringByEvaluatingJavaScriptFromString:@"injectJSFunction(1)"];
```

##3.2JavaScript传值给UIWebView

最简单的方法是将参数作为URL的一部分，然后在delegate方法里截取出来。这种方法只能传简单的参数，如果是一个很复杂的对象，那么URL的编解码会很复杂。示例如下：

```
	dataString = JSON.stringify(data);
    identifier = (methodName+dataString).hashCode().toString();
    window.CTCallBackList[identifier] = callback;

    url = "casa://nativeapi?callbackIdentifier="+identifier+"&data="+dataString+"&methodName="+methodName;
    window.location = url;
```

## Hybird App
在某些项目的开发中，我们可能有不更新应用版本而又添加新的功能的需求，在应用中嵌入 Web 页面是一个不错的方法，这样的应用我们称为 Hybird App。开发这样的应用时，我们可以引入第三方的JSBridge库来简化开发工作。


