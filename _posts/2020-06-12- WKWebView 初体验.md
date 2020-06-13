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
&emsp;&emsp;WKWebView是苹果在iOS 8之后推出的框架WebKit中的浏览器控件, 其加载速度比UIWebView快了许多, 但内存占用率却下降很多, 也解决了加载网页时的内存泄露问题. 

## WKWebView使用
### &emsp;&emsp;WKWebView的使用主要涉及下面几个类:
> WKWebView <br>
WKWebViewConfiguration <br>
WKUserScript <br>
WKUserContentController <br>
WKWebsiteDataStore <br>

### &emsp;&emsp;WKWebView的使用两个代理

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


