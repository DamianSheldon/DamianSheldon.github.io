---
layout: post
title: "Ionic App 使用 cordova hot code push 实现热更新"
date: 2020-01-07 17:02:35 +0800
comments: true
categories: [Archives, Web Development]
keywords: ionic, cordova hot code push, live update, angular
description: Ionic App use cordova hot code push achieve live update.
---

本文主要记录 Ionic App 使用 cordova hot code push 实现热更新时遇到问题的解决方法，另外也简单记录下使用方法，方便日后查阅。

Cordova hot code push 插件的原作者已经不维护了，我们可以选择一个可能最好的 fork 来使用。 [gitpop2](http://gitpop2.herokuapp.com/) 可以帮助我们选择，我从中选择了当前 star 最多的一个 fork。

Ionic App 使用 cordova hot code push 实现热更新的基本步骤如下：

1. 在 ionic 工程中添加 cordova hot code push plugin
	
	```
	$ ionic cordova plugin add https://github.com/snipking/cordova-hot-code-push.git
	```
	
2. 安装 Cordova Hot Code Push CLI client
	
	```
	$ npm install -g cordova-hot-code-push-cli
	```

3. 为指定平台编译工程
	
	```
	$ ionic cordova prepare android
	```
	
4. 执行插件初始化

	```
	$ cd /path/to/project/root
	$ cordova-hcp init
	```
5. 生成插件配置文件
	
	```
	$ cordova-hcp build
	```
	
6. 运行到设备上
	
7. 开发和发布应用新版本的 web
	
	```
	// 1. 开发
	// 2. 为指定平台编译工程生成 web 
	$ ionic build --engine=cordova --platform=android
	// 3. 生成新插件配置文件
	$ cordova-hcp build
	// 4. 部署到服务器
	```


在使用的过程中遇到的第一个问题是更新之后白屏。使用 Chrome 的 remote devices 调试 android webview 找到了问题的原因，ionic 应用中 `<base href="/" />`， cordova hot code push 会将 web 代码拷贝到外部存储上，webview 使用形如 `file:///data/user/0/com.tenneshop.liveupdatedemo/files/cordova-hot-code-push-plugin/2020.01.07-16.16.39/www/index.html` 的路径来加载应用，此时 `document.baseURI = /`，加载其他相对路径的 js 文件时，是相对这个路径，例如 `<script src="cordova.js"></script>`，就是以 `/cordova.js` 去加载，于是就会提示找不到文件。从上面的分析我们也知道，解决问题的一个办法是修正 base href 的值，我们可以在 index.html 的 head 元素加入下面的代码：

```
<script>
    document.write('<base href="' + document.location.href + '" />');
</script>

```

这样我们就修正文件路径的问题，很不巧，虽然文件的路径是对了，但是 ionic 默认不响应 file schema 的请求，我们需要做些工作，先让 WebViewLocalServer.java 支持响应 file schema，将 createHostingDetails 改成如下实现：

{% codeblock %}
private void createHostingDetails() {
  final String assetPath = this.basePath;

  if (assetPath.indexOf('*') != -1) {
    throw new IllegalArgumentException("assetPath cannot contain the '*' character.");
  }

  PathHandler handler = new PathHandler() {
    @Override
    public InputStream handle(Uri url) {
      InputStream stream = null;
      String path = url.getPath();
      try {
        if (url.getScheme().equals("file")) {
          stream = protocolHandler.openFile(path);
        } else if (path.startsWith(contentStart)) {
          stream = protocolHandler.openContentUrl(url);
        } else if (path.startsWith(fileStart) || !isAsset) {
          if (!path.startsWith(fileStart)) {
            path = basePath + url.getPath();
          }
          stream = protocolHandler.openFile(path);
        } else {
          stream = protocolHandler.openAsset(assetPath + path);
        }
      } catch (IOException e) {
        Log.e(TAG, "Unable to open asset URL: " + url);
        Log.e(TAG, e.getLocalizedMessage());
        return null;
      }

      return stream;
    }
  };

  registerUriForScheme(httpScheme, handler, authority);
  registerUriForScheme(httpsScheme, handler, authority);
  if (!customScheme.equals(httpScheme) && !customScheme.equals(httpsScheme)) {
    registerUriForScheme(customScheme, handler, authority);
  }

  registerUriForScheme("file", handler, "");

}

{% endcodeblock %}

然后是 isLocalFile 方法：

{% codeblock %}
private boolean isLocalFile(Uri uri) {
  String path = uri.getPath();
  if (path.startsWith(contentStart) || path.startsWith(fileStart) || uri.getScheme().equals("file")) {
    return true;
  }
  return false;
}
{% endcodeblock %}

做完这些工作后 ionic 就可以响应 file schema 请求了。

继续测试，我发现更新后第二次打开还是显示 App bundle asset 中的 web，这有点奇怪。仔细查看日志，确实有加载外部存储的 web , 但却被 `http://localhost/` 的请求覆盖了，这是什么原因呢？经过对代码逻辑的一番梳理，我发现是 IonicWebViewEngine 中 onPageStarted 方法的原因：

{% codeblock %}
public void onPageStarted(WebView view, String url, Bitmap favicon) {
  super.onPageStarted(view, url, favicon);
  String launchUrl = parser.getLaunchUrl();
  if (!launchUrl.contains(WebViewLocalServer.httpsScheme) && !launchUrl.contains(WebViewLocalServer.httpScheme) && url.equals(launchUrl)) {
    view.stopLoading();
    // When using a custom scheme the app won't load if server start url doesn't end in /
    String startUrl = CDV_LOCAL_SERVER;
    if (!scheme.equalsIgnoreCase(WebViewLocalServer.httpsScheme) && !scheme.equalsIgnoreCase(WebViewLocalServer.httpScheme)) {
      startUrl += "/";
    }
    view.loadUrl(startUrl);
  }
}
{% endcodeblock %}

MainActivity 触发 webview 加载 `file:///android_asset/www/index.html`，然后 cordova hot code push plugin 启动工作，它会让 webview 加载外部存储的 web，之后 IonicWebViewEngine 的 onPageStarted 收到 `file:///android_asset/www/index.html` 的请求的回调，它先停止了 webview 的加载工作，即 cordova hot code push plugin 启动加载外部存储的 web 的请求，再开始 `http://localhost/` 的请求，也就是打印出来日志的记录。正是这个方法时序的问题导致成功更新之后再重启应用仍然加载 app bundle asset 的 web。一种解决办法是我们直接让 MainActivity 直接加载 `http://localhost/`，就像下面这样:

{% codeblock %}
 public void onCreate(Bundle savedInstanceState)
{
   super.onCreate(savedInstanceState);

   // enable Cordova apps to be started in the background
   Bundle extras = getIntent().getExtras();
   if (extras != null && extras.getBoolean("cdvStartInBackground", false)) {
       moveTaskToBack(true);
   }
   launchUrl = "http://localhost/";
   // Set by <content src="index.html" /> in config.xml
   loadUrl(launchUrl);
}

{% endcodeblock %}

这样热更新就可以正常工作了。  

我继续做了点测试，又发现一个和 ionic icon 相关的问题，ionic 4 使用了 Fetch API 来请求 ionic icon 的 svg 资源，由于现在是使用 file schema 来指定资源路径，由于 Fetch API 不支持 file schema 所以就报错 `Fetch API cannot load file:///xxx/www/svg/md-star.svg. URL scheme "file" is not supported. ` 我们得想办法来解决这个问题，一个办法替换 fetch 方法的实现，如:  

{% codeblock %}
<script>
   document.write('<base href="' + document.location.href + '" />');

   var originalFetch = window.fetch;

   window.fetch = function() {
       var args = [];
       for (var _i = 0; _i < arguments.length; _i++) {
           args[_i] = arguments[_i];
       }
       var url = args[0];
       if (typeof url === 'string' && url.match(/\.svg/)) {
           return new Promise(function(resolve, reject) {
               var req = new XMLHttpRequest();
               req.open('GET', url, true);
               req.addEventListener('load', function() {
                   resolve({
                       ok: true,
                       status: 200,
                       text: function() {
                           return Promise.resolve(req.responseText);
                       }
                   });
               });
               req.addEventListener('error', reject);
               req.send();
           });
       } else {
           return originalFetch.apply(void 0, args);
       }
   };
</script>
{% endcodeblock %}

在这些测试过程中，我还发现 cordova hot code push 更新时只做了版本字符是否相等的判断，这在服务器端的版本低于本地版本时，插件仍然会做更新，这是有问题的，我们需要严格这里的判断，让它只有在服务端的版本高于本地版本时才做更新。相关代码位于 UpdateLoaderWorker 的 run 方法中。  

最后一个要考虑的问题是如何将我们修改的代码和 ionic 的代码很好的整合起来？我现在的想法是创建一个私有的扩展 IonicWebViewEngine 和 WebViewLocalServer，然后借鉴 ionic 通过 config.xml 的 web 偏好设置的方法，像下面的代码: 

```
<preference name="webView" value="com.ionicframework.cordova.webview.IonicWebViewEngine" />
```

回头测试下这个想法，好了有时间也许可以整理好代码提个 Pull Request。

Reference:

* [Cannot run angular 2+ from file:/// - looks like 'base href="/"' is the issue](https://github.com/angular/angular/issues/13948)  
* [IONIC v4: Icon won't work in android](https://github.com/ionic-team/ionicons/issues/572)  

