---
layout:     post   				    # 使用的布局（不需要改）
title:      WKWebView 初体验				# 标题 
subtitle:    #副标题
date:       2020-06-13 				# 时间
author:     jianjingbao 						# 作者
header-img: img/post-bg-2015.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - WKWebView
---

# WKWebView 初体验

[toc]

## WKWebView介绍
WKWebView是苹果在iOS 8之后推出的框架WebKit中的浏览器控件, 其加载速度比UIWebView快了许多, 但内存占用率却下降很多, 也解决了加载网页时的内存泄露问题. 

## WKWebView使用
### WKWebView的使用主要涉及下面几个类:
> WKWebView <br>
WKWebViewConfiguration <br>
WKUserScript <br>
WKUserContentController <br>
WKWebsiteDataStore <br>

### WKWebView的使用两个代理

>WKNavigationDelegate <br>
WKUIDelegate<br>

## WKWebView 属性介绍
```c
// 导航代理
@property (nullable, nonatomic, weak) id <WKNavigationDelegate> navigationDelegate;
// UI代理
@property (nullable, nonatomic, weak) id <WKUIDelegate> UIDelegate;

// 页面标题, 一般使用KVO动态获取
@property (nullable, nonatomic, readonly, copy) NSString *title;
// 页面加载进度, 一般使用KVO动态获取
@property (nonatomic, readonly) double estimatedProgress;

// 可返回的页面列表, 已打开过的网页, 有点类似于navigationController的viewControllers属性
@property (nonatomic, readonly, strong) WKBackForwardList *backForwardList;

// 页面url
@property (nullable, nonatomic, readonly, copy) NSURL *URL;
// 页面是否在加载中
@property (nonatomic, readonly, getter=isLoading) BOOL loading;
// 是否可返回
@property (nonatomic, readonly) BOOL canGoBack;
// 是否可向前
@property (nonatomic, readonly) BOOL canGoForward;
// WKWebView继承自UIView, 所以如果想设置scrollView的一些属性, 需要对此属性进行配置
@property (nonatomic, readonly, strong) UIScrollView *scrollView;
// 是否允许手势左滑返回上一级, 类似导航控制的左滑返回
@property (nonatomic) BOOL allowsBackForwardNavigationGestures;

//自定义UserAgent, 会覆盖默认的值 ,iOS 9之后有效
@property (nullable, nonatomic, copy) NSString *customUserAgent
```
## WKWebView 方法介绍

```c
// 带配置信息的初始化方法
// configuration 配置信息
- (instancetype)initWithFrame:(CGRect)frame configuration:(WKWebViewConfiguration *)configuration
// 加载请求
- (nullable WKNavigation *)loadRequest:(NSURLRequest *)request;
// 加载HTML
- (nullable WKNavigation *)loadHTMLString:(NSString *)string baseURL:(nullable NSURL *)baseURL;
// 返回上一级
- (nullable WKNavigation *)goBack;
// 前进下一级, 需要曾经打开过, 才能前进
- (nullable WKNavigation *)goForward;
// 刷新页面
- (nullable WKNavigation *)reload;
// 根据缓存有效期来刷新页面
- (nullable WKNavigation *)reloadFromOrigin;
// 停止加载页面
- (void)stopLoading;
// 执行JavaScript代码
- (void)evaluateJavaScript:(NSString *)javaScriptString completionHandler:(void (^ _Nullable)(_Nullable id, NSError * _Nullable error))completionHandler;
```

## WKWebViewConfiguration
```c
// 通过此属性来执行JavaScript代码来修改页面的行为
@property (nonatomic, strong) WKUserContentController *userContentController;

//***********下面属性一般不需要设置
// 首选项设置,  
//可设置最小字号, 是否允许执行js
//是否通过js自动打开新的窗口
@property (nonatomic, strong) WKPreferences *preferences;
// 是否允许播放媒体文件
@property (nonatomic) BOOL allowsAirPlayForMediaPlayback
// 需要用户来操作才能播放的多媒体类型
@property (nonatomic) WKAudiovisualMediaTypes mediaTypesRequiringUserActionForPlayback
// 是使用h5的视频播放器在线播放, 还是使用原生播放器全屏播放
@property (nonatomic) BOOL allowsInlineMediaPlayback;
```

## WKUserContentController
WKUserContentController 是JavaScript与原生进行交互的桥梁, 主要使用的方法有:

``` c
// 注入JavaScript与原生交互协议
// JS 端可通过 window.webkit.messageHandlers.<name>.postMessage(<messageBody>) 发送消息
- (void)addScriptMessageHandler:(id <WKScriptMessageHandler>)scriptMessageHandler name:(NSString *)name;
// 移除注入的协议, 在deinit方法中调用
- (void)removeScriptMessageHandlerForName:(NSString *)name;

// 通过WKUserScript注入需要执行的JavaScript代码
- (void)addUserScript:(WKUserScript *)userScript;
// 移除所有注入的JavaScript代码
- (void)removeAllUserScripts;
```
使用WKUserContentController注入的交互协议, 需要遵循WKScriptMessageHandler协议, 在其协议方法中获取JavaScript端传递的事件和参数:

``` c
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message;
```
WKScriptMessage包含了传递的协议名称及参数, 主要从下面的属性中获取:

``` c
// 协议名称, 即上面的add方法传递的name
@property (nonatomic, readonly, copy) NSString *name;
// 传递的参数
@property (nonatomic, readonly, copy) id body;
```

## WKUserScript

WKUserScript用于往加载的页面中添加额外需要执行的JavaScript代码, 主要是一个初始化方法:

``` c
/*
source: 需要执行的JavaScript代码
injectionTime: 加入的位置, 是一个枚举
typedef NS_ENUM(NSInteger, WKUserScriptInjectionTime) {
    WKUserScriptInjectionTimeAtDocumentStart,
    WKUserScriptInjectionTimeAtDocumentEnd
} API_AVAILABLE(macosx(10.10), ios(8.0));

forMainFrameOnly: 是加入所有框架, 还是只加入主框架
*/
- (instancetype)initWithSource:(NSString *)source injectionTime:(WKUserScriptInjectionTime)injectionTime forMainFrameOnly:(BOOL)forMainFrameOnly;

```

## WKUIDelegate
这个代理方法, 主要是用来处理使用系统的弹框来替换JS中的一些弹框的,比如: 警告框, 选择框, 输入框, 主要使用的是下面三个代理方法:
``` c
/**
 webView中弹出警告框时调用, 只能有一个按钮

 @param webView webView
 @param message 提示信息
 @param frame 可用于区分哪个窗口调用的
 @param completionHandler 警告框消失的时候调用, 回调给JS
 */
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler {
    
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"警告" message:message preferredStyle:(UIAlertControllerStyleAlert)];
    UIAlertAction *ok = [UIAlertAction actionWithTitle:@"我知道了" style:(UIAlertActionStyleDefault) handler:^(UIAlertAction * _Nonnull action) {
        completionHandler();
    }];
    
    [alert addAction:ok];
    [self presentViewController:alert animated:YES completion:nil];
}

/** 对应js的confirm方法
 webView中弹出选择框时调用, 两个按钮

 @param webView webView description
 @param message 提示信息
 @param frame 可用于区分哪个窗口调用的
 @param completionHandler 确认框消失的时候调用, 回调给JS, 参数为选择结果: YES or NO
 */
- (void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL result))completionHandler {
    
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"请选择" message:message preferredStyle:(UIAlertControllerStyleAlert)];
    UIAlertAction *ok = [UIAlertAction actionWithTitle:@"同意" style:(UIAlertActionStyleDefault) handler:^(UIAlertAction * _Nonnull action) {
        completionHandler(YES);
    }];
    
    UIAlertAction *cancel = [UIAlertAction actionWithTitle:@"不同意" style:(UIAlertActionStyleCancel) handler:^(UIAlertAction * _Nonnull action) {
        completionHandler(NO);
    }];
    
    [alert addAction:ok];
    [alert addAction:cancel];
    [self presentViewController:alert animated:YES completion:nil];
}

/** 对应js的prompt方法
 webView中弹出输入框时调用, 两个按钮 和 一个输入框

 @param webView webView description
 @param prompt 提示信息
 @param defaultText 默认提示文本
 @param frame 可用于区分哪个窗口调用的
 @param completionHandler 输入框消失的时候调用, 回调给JS, 参数为输入的内容
 */
- (void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(nullable NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * _Nullable result))completionHandler {
    
    
    UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"请输入" message:prompt preferredStyle:(UIAlertControllerStyleAlert)];

    
    [alert addTextFieldWithConfigurationHandler:^(UITextField * _Nonnull textField) {
        textField.placeholder = @"请输入";
    }];
    
    UIAlertAction *ok = [UIAlertAction actionWithTitle:@"确定" style:(UIAlertActionStyleDefault) handler:^(UIAlertAction * _Nonnull action) {
        
        UITextField *tf = [alert.textFields firstObject];
        
                completionHandler(tf.text);
    }];
    
    UIAlertAction *cancel = [UIAlertAction actionWithTitle:@"取消" style:(UIAlertActionStyleCancel) handler:^(UIAlertAction * _Nonnull action) {
                completionHandler(defaultText);
    }];
    
    [alert addAction:ok];
    [alert addAction:cancel];
    [self presentViewController:alert animated:YES completion:nil];
}

```

## WKNavigationDelegate
``` c
// 决定导航的动作，通常用于处理跨域的链接能否导航。
// WebKit对跨域进行了安全检查限制，不允许跨域，因此我们要对不能跨域的链接单独处理。
// 但是，对于Safari是允许跨域的，不用这么处理。
// 这个是决定是否Request
- (void)webView:(WKWebView *)webView decidePolicyForNavigationAction:(WKNavigationAction *)navigationAction decisionHandler:(void (^)(WKNavigationActionPolicy))decisionHandler{
    //  在发送请求之前，决定是否跳转
    decisionHandler(WKNavigationActionPolicyAllow);  
}

// 是否接收响应
- (void)webView:(WKWebView *)webView decidePolicyForNavigationResponse:(WKNavigationResponse *)navigationResponse decisionHandler:(void (^)(WKNavigationResponsePolicy))decisionHandler{
    // 在收到响应后，决定是否跳转和发送请求之前那个允许配套使用
    decisionHandler(WKNavigationResponsePolicyAllow);
}

//用于授权验证的API，与AFN、UIWebView的授权验证API是一样的
- (void)webView:(WKWebView *)webView didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential *__nullable credential))completionHandler{
    
    completionHandler(NSURLSessionAuthChallengePerformDefaultHandling ,nil);
}

// main frame的导航开始请求时调用
- (void)webView:(WKWebView *)webView didStartProvisionalNavigation:(null_unspecified WKNavigation *)navigation{
   
}

// 当main frame接收到服务重定向时调用
- (void)webView:(WKWebView *)webView didReceiveServerRedirectForProvisionalNavigation:(null_unspecified WKNavigation *)navigation{
    // 接收到服务器跳转请求之后调用
}

// 当main frame开始加载数据失败时，会回调
- (void)webView:(WKWebView *)webView didFailProvisionalNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error {

}

// 当内容开始返回时调用
- (void)webView:(WKWebView *)webView didCommitNavigation:(null_unspecified WKNavigation *)navigation{  
}

//当main frame导航完成时，会回调
- (void)webView:(WKWebView *)webView didFinishNavigation:(null_unspecified WKNavigation *)navigation{
    // 页面加载完成之后调用
}

// 当main frame最后下载数据失败时，会回调
- (void)webView:(WKWebView *)webView didFailNavigation:(null_unspecified WKNavigation *)navigation withError:(NSError *)error {
}

// 当web content处理完成时，会回调
- (void)webViewWebContentProcessDidTerminate:(WKWebView *)webView {
}
```


## WKWebsiteDataStore
WKWebsiteDataStore 提供了网站所能使用的数据类型，包括 cookies，硬盘缓存，内存缓存活在一些WebSQL的数据持久化和本地持久化。可通过 WKWebViewConfiguration 类的属性 websiteDataStore 进行相关的设置。WKWebsiteDataStore 相关的API也比较简单：
``` c
// 默认的data store
+ (WKWebsiteDataStore *)defaultDataStore;

// 如果为webView设置了这个data Store，则不会有数据缓存被写入文件
// 当需要实现隐私浏览的时候，可使用这个
+ (WKWebsiteDataStore *)nonPersistentDataStore;

// 是否是可缓存数据的，只读
@property (nonatomic, readonly, getter=isPersistent) BOOL persistent;

// 获取所有可使用的数据类型
+ (NSSet<NSString *> *)allWebsiteDataTypes;

// 查找指定类型的缓存数据
// 回调的值是WKWebsiteDataRecord的集合
- (void)fetchDataRecordsOfTypes:(NSSet<NSString *> *)dataTypes completionHandler:(void (^)(NSArray<WKWebsiteDataRecord *> *))completionHandler;

// 删除指定的纪录
// 这里的参数是通过上面的方法查找到的WKWebsiteDataRecord实例获取的
- (void)removeDataOfTypes:(NSSet<NSString *> *)dataTypes forDataRecords:(NSArray<WKWebsiteDataRecord *> *)dataRecords completionHandler:(void (^)(void))completionHandler;

// 删除某时间后修改的某类型的数据
- (void)removeDataOfTypes:(NSSet<NSString *> *)websiteDataTypes modifiedSince:(NSDate *)date completionHandler:(void (^)(void))completionHandler;

// 保存的HTTP cookies
@property (nonatomic, readonly) WKHTTPCookieStore *httpCookieStore
```

> 获取所有可使用的数据类型
``` c
// 硬盘缓存
WKWebsiteDataTypeDiskCache,

// HTML离线web应用程序缓存
WKWebsiteDataTypeOfflineWebApplicationCache,

// 内存缓存
WKWebsiteDataTypeMemoryCache,

// 本地缓存
WKWebsiteDataTypeLocalStorage,

// cookies
WKWebsiteDataTypeCookies,

// HTML会话存储
WKWebsiteDataTypeSessionStorage,

//  IndexedDB 数据库
WKWebsiteDataTypeIndexedDBDatabases,

// WebSQL 数据库
WKWebsiteDataTypeWebSQLDatabases
```
> WKWebsiteDataRecord
``` c
// 展示名称, 通常是域名
@property (nonatomic, readonly, copy) NSString *displayName;

// 包含的数据类型
@property (nonatomic, readonly, copy) NSSet<NSString *> *dataTypes;
```

### WKHTTPCookieStore
关于cookie, 从WKWebsiteDataStore 的实例对象的数属性httpCookieStore 可获取一个 WKHTTPCookieStore 的实例对象, 通过此对象, 我们可以对cookie进行相关的操作, 官方提供的API也不难理解:
``` c
/*!  查找所有已存储的cookie
 */
- (void)getAllCookies:(void (^)(NSArray<NSHTTPCookie *> *))completionHandler;

/*! 保存一个cookie, 保存成功后, 会走一次回调方法
 */
- (void)setCookie:(NSHTTPCookie *)cookie completionHandler:(nullable void (^)(void))completionHandler;

/*! 删除一个cookie, 待删除的cookie对象可通过 'getAllCookies' 方法获取
 */
- (void)deleteCookie:(NSHTTPCookie *)cookie completionHandler:(nullable void (^)(void))completionHandler;

/*! 添加一个观察者, 需要遵循协议 WKHTTPCookieStoreObserver 
当cookie发送变化时, 会通过 WKHTTPCookieStoreObserver 的协议方法通知该观察者, 在使用完后需要移除观察者
 */
- (void)addObserver:(id<WKHTTPCookieStoreObserver>)observer;

/*! 移除观察者
 */
- (void)removeObserver:(id<WKHTTPCookieStoreObserver>)observer;

```
### WKHTTPCookieStoreObserver协议方法
``` c
@protocol WKHTTPCookieStoreObserver <NSObject>
@optional
- (void)cookiesDidChangeInCookieStore:(WKHTTPCookieStore *)cookieStore;
@end
```

