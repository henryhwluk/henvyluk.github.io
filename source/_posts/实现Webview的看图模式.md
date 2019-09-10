---
title: 实现Webview的看图模式
urlPath: WebviewPictureMode
date: 2017-02-09 
updated: 2017-02-09
---

## 前言
由于项目需求，看了看关于js和webview交互的一些东西，由于以前没接触过这块，算是自己的一个盲区，因此记录下来，代码实现的过程很简短。

<!-- more -->

## 正文
###1、获取 `<img>` 元素
在webview的代理方法`webViewDidFinishLoad`中用js实现对页面图片URL数组的获取：
  
    static  NSString *const jsGetImages =
    @"function getImages(){\
    var objs = document.getElementsByTagName(\"img\");\
    var imgScr = '';\
    for(var i=0;i<objs.length;i++){\
    imgScr = imgScr + objs[i].src + '+';\
    };\
    return imgScr;\
    };";
    //注入js方法
    [webView stringByEvaluatingJavaScriptFromString:jsGetImages];
    NSString *urlResurlt = [webView stringByEvaluatingJavaScriptFromString:@"getImages()"];
urlResurlt获取到的是URL字符串，将其拆分到数组：

    _mUrlArray = [NSMutableArray arrayWithArray:[urlResurlt componentsSeparatedByString:@"+"]];
    //去除最后一个空元素
    if (_mUrlArray.count >= 2) {
        [_mUrlArray removeLastObject];
    }
###2、添加图片可点击js
    
    [_webView stringByEvaluatingJavaScriptFromString:@"function registerImageClickAction(){\
     var imgs=document.getElementsByTagName('img');\
     var length=imgs.length;\
     for(var i=0;i<length;i++){\
     img=imgs[i];\
     img.onclick=function(){\
     window.location.href='image-preview:'+this.src}\
     }\
     }"];
    [_webView stringByEvaluatingJavaScriptFromString:@"registerImageClickAction();"];
    
 
###3、捕获到图片的点击事件
在webView的`shouldStartLoadWithRequest`代理方法中获取被点击图片的url：

    - (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    
    if ([request.URL.scheme isEqualToString:@"image-preview"]) {
        NSString* path = [request.URL.absoluteString substringFromIndex:[@"image-preview:" length]];
        path = [path stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
        //path 就是被点击图片的url
        [self loadPhotoWith:path];
        return NO;
    }
    return YES;
    }
###4、显示图片
这里我用的是IDMPhotoBrowser：
   
  
     - (void)loadPhotoWith: (NSString *)path {
    // URLs array
    __block NSInteger index;//图片点击的序号
    NSMutableArray *photosURL = [NSMutableArray array];
    [_mUrlArray enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        NSURL *url = [NSURL URLWithString:obj];
        [photosURL addObject:url];
        if ([path isEqualToString:obj]) {
             index = idx;
        }
    }];

    // Create an array to store IDMPhoto objects
    NSMutableArray *photos = [NSMutableArray new];
    
    for (NSURL *url in photosURL) {
        IDMPhoto *photo = [IDMPhoto photoWithURL:url];
        [photos addObject:photo];
    }
    IDMPhotoBrowser *browser = [[IDMPhotoBrowser alloc] initWithPhotos:photos];
    browser.displayCounterLabel = YES;
    browser.displayActionButton = NO;
    browser.displayArrowButton = YES;
    [browser setInitialPageIndex:index];
    [self presentViewController:browser animated:YES completion:nil];
    }
