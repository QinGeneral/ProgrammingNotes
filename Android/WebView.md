# WebView

[TOC]

## WebView 核心点

- addJavascriptInterface JSBridge 调用
- WebViewSetting
  - setUserAgentString
- WebChromeClient
  - onProgressChanged
  - GPS 权限处理 onGeolocationPermissionsShowPrompt
  - onShowFileChooser
  - onReceivedTitle
  - onPermissionRequest
- WebViewClient
  - doUpdateVisitedHistory
  - onLoadResource
  - onPageCommitVisible
  - shouldOverrideUrlLoading
  - shouldInterceptRequest
  - onReceivedSslError
  - onReceivedHttpError
  - onReceivedError
  - onPageStarted
  - onPageFinished

## 缓存

```java
webView.getSettings().setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);

// 清理缓存

// 清除网页访问留下的缓存
// 由于内核缓存是全局的因此这个方法不仅仅针对webview而是针对整个应用程序.
webView.clearCache(true);

// 清除当前webview访问的历史记录
// 只会webview访问历史记录里的所有记录除了当前访问记录
webView.clearHistory()；

// 这个api仅仅清除自动完成填充的表单数据，并不会清除WebView存储到本地的数据
webView.clearFormData()；
```

## Cookie

```java
// 设置 Cookie
CookieSyncManager.createInstance(this);
CookieManager cookieManager = CookieManager.getInstance();
cookieManager.setAcceptCookie(true);

String cookie = "name=xxx;age=1";
cookieManager.setCookie(URL, cookie);
CookieSyncManager.getInstance().sync();

// 获取 Cookie
CookieManager cookieManager = CookieManager.getInstance();
String cookie = cookieManager.getCookie(URL);

// 清除 Cookie
CookieSyncManager.createInstance(context);
CookieManager cookieManager = CookieManager.getInstance();
cookieManager.removeAllCookie();
CookieSyncManager.getInstance().sync();
```

## Native 和 JS 调用

### JavaScript 调用 Native

前两种 JS 到 Native 的调用都是无返回值的，需要 Native 再异步调用 JS。

第三种方式的 onJsPrompt 方法劫持可以返回任意值。

1. 通过 WebView 的 addJavascriptInterface 方法，将 Java 对象映射到 JavaScript 中：

```java
WebSettings settings = mWebView.getSettings();
// 开启JavaScript
settings.setJavaScriptEnabled(true);

mWebView.loadUrl("");
// 此行代码可以保证JavaScript的Alert弹窗正常弹出
mWebView.setWebChromeClient(new WebChromeClient());

// 核心方法, 用于处理 JavaScript 被执行后的回调
// 参数 1 是回调接口的实现，参数 2 是 JavaScript 回调对象的名称
mWebView.addJavascriptInterface(new JsCallback() {
    // 注意:此处一定要加该注解,否则在4.1+系统上运行失败
    @JavascriptInterface
    @Override
    public void onJsCallback() {
        System.out.println("JavaScript调用Android");
    }
}, "NativeBridge");

//定义回调接口
public interface JsCallback {
    public void onJsCallback();
}
```

2. Url 劫持：通过 WebViewClient 的 shouldOverrideUrlLoading 方法，拦截 url，根据协议进行处理：

```java
WebView webview = (WebView) findViewById(R.id.webview);
webview.loadUrl('');
webview.setWebViewClient(new WebViewClient() {
  @Override
  public boolean shouldOverrideUrlLoading(WebView view, String url) {
    if(url.equals('sdk:hello')) {
      System.out.println('hello world');
      return true;
    }

    return super.shouldOverrideUrlLoading(view, url);
  }
});
```

3. 通过 WebChromeClient 中的 `onJsAlert()、onJsConfirm()、onJsPrompt()` 拦截 JS 中的 `alert()、confirm()、prompt()` 消息。而 alert、confirm、prompt 代表 JS 中三种常用提示框，第一种没有返回值，第二种返回布尔值，第三种可返回任意值。为了灵活性，我们可以劫持 onJsPrompt 方法：

```java
// 定义好劫持回调类
private class HijackWebChromeClient extends WebChromeClient {  
  public boolean hijack(String text) {
    if(text.equals('sdk:hello')) {
      System.out.println('hello world');
      return true;
    }

    return false;
  }

  @Override
  public boolean onJsPrompt(WebView view, String message, String defaultValue, JSPromptResult result) {
    if(this.hijack(message)) {
      return true;
    }

    return super.onJsPrompt(view, url, message, defaultValue, result);
  }

  @Override
  public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
    if(this.hijack(message)) {
      return true;
    }
    
    return super.onJsAlert(view, url, message, result);
  }

  @Override
  public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
    if(this.hijack(message)) {
      return true;
    }
    
    return super.onJsConfirm(view, url, message, result);
  }

  @Override  
  public boolean onConsoleMessage(ConsoleMessage consoleMessage) {  
    String message = consoleMessage.message();
    if(this.hijack(message)) {
      return true;
    }
    
    return super.onConsoleMessage(consoleMessage);  
  }  

  @Override  
  public void onConsoleMessage(String message, int lineNumber, String sourceID) {
    if(this.hijack(message)) {
      return true;
    }
    super.onConsoleMessage(message, lineNumber, sourceID);  
  }  
}

// 注入劫持回调类
WebView webview = (WebView) findViewById(R.id.webview);
webview.loadUrl('');
webview.setWebChromeClient(new HijackChromeClient);
```

### Native 调用 JavaScript

这两种方式拿到返回值，都是异步调用的方式。

1. loadUrl 无返回值和回调，如果需要拿到回调，需要 JS 再异步调用 Native；
2. evaluateJavascript 有回调拿到返回值。

```java
// 方法 1
mWebView.loadUrl("javascript:wave()");

// 方法 2
webView.evaluateJavascript("javascript:wave()", valueCallback);
```
