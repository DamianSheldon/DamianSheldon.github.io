---
layout: post
title: "创建 Cordova plugin 及其 Ionic Native"
date: 2019-09-30 09:59:14 +0800
comments: true
categories: [Archives, Web Development, Android Development]
keywords: Cordova, Plugin, Ionic Native
description: How cordova plugin works? How to create a cordova plugin and wrapper as Ionic Native?
---

本文介绍如何创建 Cordova plugin 及其 Ionic Native，主要内容如下：  

* Cordova Plugin 的工作原理
* 如何创建 Cordova Plugin
* 如何创建 Cordova Plugin 对应的 Ionic Native

###Cordova Plugin 的工作原理

我们简要介绍下 Cordova Plugin 的工作原理，这样我们才能解决在开发中遇到的问题。  

Plugin js 端的入口方法签名形式如下:

```
exec(<successFunction>, <failFunction>, <service>, <action>, [<args>]);
```

successFunction, failFunction 是成功和失败的回调函数，args 则是传递给原生端的参数。service, action 则是用来映射到原生端的对象和方法。这个映射是通过 plugin.xml 建立起来的。在 plugin.xml 中我们有如下元数据信息:

```
<feature name="<service_name>">
    <param name="android-package" value="<full_name_including_namespace>" />
</feature>
```

service 就是对应 service_name 指定的对象，而 action 则是该对象能处理方法。

具体是怎么映射的呢？

以 android 为例，我们安装的 plugin 信息会保存在 config.xml 中，`cordova prepare android` 命令会将 config.xml 在 android 的资源文件目录中生成一个同名文件，两个文件内容大致相同，plugin 的信息会改成如下形式表示：

```
<feature name="<service_name>">
    <param name="android-package" value="<full_name_including_namespace>" />
</feature>
```

这个形式就是我们上面见过的形式。  

当 js 端调用 exec 方法时，它会通过 webview 建立的通信通道(通常是用 WebView.addJavascriptInterface)调用 PluginManager 的 exec 方法，PluginManager 则根据 service_name 查找或创建 plugin ，然后调用 plugin 的 exec 方法，并将 action 作为参数传入，于是我们便可按需响应 action 请求。

###如何创建 Cordova Plugin

####安装 plugman

```
$ npm install -g plugman
```

####创建 cordova plugin

```
$ plugman create --name <pluginName> --plugin_id <pluginID> --plugin_version <version> [--path <directory>] [--variable NAME=VALUE]

eg.
$plugman create --name cordova-plugin-onsite-signature --plugin_id cordova-plugin-onsite-signature --plugin_version 0.0.1
```

####添加 platform

```
$plugman platform add --platform_name android
```

####创建 package.json 文件

```
$ plugman createpackagejson <directory>

eg.
$plugman createpackagejson .
```

####安装 cordova plugin

```
// 方法一：这种方式的命令和添加官方的插件类似，个人推荐此方法，可以少记一个命令
$cordova plugin add git+ssh://git@192.168.8.91/git/cordova-plugin-onsite-signature.git

// 方法二：
$ plugman install --platform android --project platforms/android --plugin ../LogicLinkPlugin/
```

####卸载 cordova plugin

```
// 与安装的方法对应有两种卸载方法
// 方法一：$cordova plugin remove cordova-plugin-onsite-signature

// 方法二：
$ plugman uninstall --platform android --project platforms/android --plugin ../LogicLinkPlugin/
```

####发布

```
// Create a tag
$git tag <tagname>

// Push to repository
$git push origin master
```

####升级 cordova plugin

现在暂时没有直接升级的命令，采用的是先卸载后安装新版本的方法。

###如何创建 Cordova Plugin 对应的 Ionic Native

####Creating Plugin Wrappers

```
// Navigate to ionic-native root path
$cd  ionic-native

// Install dependencies first time
$npm install

// Create plugin wrapper
$gulp plugin:create -n PluginName
```

####安装

```
// You need to run npm run build in the ionic-native project, this will create a dist directory. The dist directory will contain a sub directory @ionic-native with all the packages compiled in there.
$npm run build

//Copy the package(s) you created/modified to your app's node_modules under the @ionic-native directory.
$cp -r dist/@ionic-native/plugin-name ../my-app/node_modules/@ionic-native/
```

###使用

```
// Import the plugin in a @NgModule and add it to the list of Providers. 
// app.module.ts
import { APIClient } from '@ionic-native/api-client/ngx';
...

@NgModule({
  ...

  providers: [
    ...
    APIClient
    ...
  ]
  ...
})
export class AppModule { }

// After the plugin has been declared, it can be imported and injected like any other service:

// login.service.ts
import { APIClient } from '@ionic-native/api-client/ngx';
import { ServiceName } from '@ionic-native/api-client/ngx';

constructor(private apiClient: APIClient) { }

this.apiClient.get(ServiceName.Login, JSON.stringify(user))
      .then((result: string) => {
        console.log('api client login:', result);
        //TODO: Parse server return json to UserExt object
        const routePath = this.simulateLogin(username);        
        resolve(routePath);
      })
      .catch((error: string) => {
        console.log('api client login error:', error);

        reject(error);
      });
```

#Reference:

* [Create-a-custom-Cordova-plugin](https://github.com/RootSoft/Create-a-custom-Cordova-plugin)  
* [Ionic Native Developer Guide](https://github.com/ionic-team/ionic-native/blob/master/DEVELOPER.md)  



