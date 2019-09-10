---
title: 为你的库添加cocoapods支持
urlPath: cocoapods
date: 2016-12-13 21:57:06
updated: 2016-12-13
---

# 为你的库添加cocoapods支持
### 前言
本文意在教大家一步一步将自己的pods发布到CocoaPods中，将自己写的组件或库开源出去，让别人轻轻pod install一下即可安装。

<!-- more -->

自己在上传pods过程中也遇到过一些小坑，也在此做了说明。测试文件为一个很简单的DynamicLabel类，旨在上传pods的过程，写的不好的地方望砖下留情。
### 1、环境
cocoapods的安装这里就不再说了，另外要说明的是你首先要把项目push到github，并release一个版本打上tag标签（目的在于让cocoapods能够根据你提供的tag来锁定版本），如果没有push可cd到你的项目根目录如下：


    //添加
    git add -A
    //commit
    git commit -m"version description"
    //push
    git push origin master

    //打上标签
    git tag'0.0.3'
    //推送
    git push --tags
    
### 2、创建podspec文件
cd到你的项目根目录如下：

    //创建podspec文件
    pod spec create DynamicLabel
  
之后会生成一个.podspec文件，我这里用sublime打开，可以看到里面有很多待编辑项，顾名思义，我这里编辑项如下：

    s.name         = "DynamicLabel"
    s.version      = "0.0.3"
    s.summary      = "limited label Scroll display"
    s.description  = <<-DESC
    limited label Scroll display.
                   DESC

    s.homepage     = "https://github.com/henvyluk/DynamicLabel"
    s.license      = "MIT"
    s.author             = { "henvyluk" => "henvyluk@163.com" }
    s.platform     = :ios, "7.0"
    s.source       = { :git => "https://github.com/henvyluk/DynamicLabel.git", :tag => "0.0.3" }

    s.source_files  = "Classes", "DynamicLabel/Classes/**/*.{h,m}"

    s.exclude_files = "Classes/Exclude"

    s.framework  = "UIKit"
    s.requires_arc = true
  
  值得注意的是s.source_files需要根据podspec文件的相对位置来写，表示DynamicLabel 下的Classes文件夹下的所有文件下的所有.h/.m文件，s.framework是你的项目所用到的库，我这里只用到了UIKit，如果你的项目中依赖多个库，可以使用：
  
    s.frameworks = "SomeFramework", "AnotherFramework"
当我们开发的库中也可能还依赖第三方库，例如JSONKit，那么可以使用:

    s.dependency "JSONKit", "~> 1.4"
  另外如果要添加xib文件，在pod中,xib不能当成源文件(即s.source_files),虽然可能会通过检测，但是pod install之后会报错，所以必须要将xib放入资源文件中(即s.resources)，我就遇到过这种情况，只好更新了一个版本，

    "Unable to run command 'StripNIB xxx.nib' - this target might include its own product".
再一个添加图片资源的话，类似于xib,不需其它操作，我是将xib和图片都放在s.Resource中形如：

    s.resources = "xxxx/xxx/*.{png,xib}"
  这里看一下我的文件目录：
  ![finder Screenshot](http://p1.bqimg.com/4851/bcbaadb24d2d15bf.png)

确认完毕后可通过如下做文件校验：

    pod lib lint
    
 此时如果有红色错误The spec did not pass validation, due to 1 error可通过在上述指令后加--verbose来看出错误出在哪里，根据提示的信息在做修改，这里提醒s.source_files处容易出错，注意文件的位置，以及s.framework不要出错，否则会项目内的代码不识别。
 
当出现如下的提示时就代表验证通过了，可以进行下一步了：
 
 ![terminal Screenshot](http://p1.bqimg.com/4851/132b0f8ba0964752.png)
 
### 3、注册Trunk
    //分别是你的邮箱和描述
    pod trunk register henvyluk@163.com --description='henvy'
    
之后你的邮箱会收到确认邮件，点击邮件中的链接后验证后：

    pod trunk me
    
如图则表示注册成功，可以进行接下来的push了
  ![terminal Screenshot](http://p1.bqimg.com/4851/5ee9c81e4b1a20ab.png)
  
### 4、Trunk push

执行：

    pod trunk push
    
如果push过程中出错，再检查一下podspec文件，我之前因为版本匹配问题出了错。如果看到如下图即代表上传成功，我的pod版本比较新，好像旧的版本跟这有点区别，会给dataURL和日至打印，这个新版本的比较人性化一点，但为啥我觉得很幼稚有木有。
![terminal Screenshot](http://i1.piimg.com/4851/9afe61b7af212484.png)

### 5、验证
说是push成功了怎么说也要验证一下吧，来search一下：

    pod search DynamicLabel
 
 一看握草！！

![terminal Screenshot](http://i1.piimg.com/4851/65566d8ca8c24f2f.png)

push出错了？其实不然，cocoapods官网已经有了我们的代码，不信可以搜搜看，See Podspec还可以看到我们的项目在Specs仓库中的具体位置。
![coocapods Screenshot](http://i1.piimg.com/4851/a084bbacd8500f82.png)
问题是我们的电脑~/.cocoapods/repos/master/Specs目录并未更新，执行：
    
    //更新pod库
    pod setup
这一步具体做了什么东西呢？将官方的Specs仓库文件目录下载下来，然后和我们本地的Specs目录进行比对，增加的增加，删除的删除。

第一次会有点慢，之后再setup的话基本上是秒更，最后setup completed,好了现在是最新的了，再来search一下，
![terminal Screenshot](http://i1.piimg.com/4851/65566d8ca8c24f2f.png)

要命了，仍然搜不到，我当初就是卡在了这一步，卡的莫名其妙的，因为实在想不通还有什么会影响search，后来在[stackoverflow](http://stackoverflow.com)上有提到search_index.json，这是搜索的缓存目录，

    //清除索引缓存
    rm ~/Library/Caches/CocoaPods/search_index.json
之后pod search DynamicLabel，等待重建索引后：
![terminal Screenshot](http://i1.piimg.com/4851/82256f629a06c02b.png)

### 6、写在最后
好了至此制作自己的整个开源库的过程就完成了，如果以后要更新版本，同样修改podspec文件重新push就好了，要注意的是如果你之前提交过pod，那么你需要去[Claim your Pod](https://trunk.cocoapods.org/claims/new)认领:
![Claim your Pod Screenshot](http://i1.piimg.com/4851/8741e76fd78a0318.png)

至此结束，也望大神不吝指教，邮件henvyluk@163.com,同时欢迎跳转我的[GitHub主页](https://github.com/henvyluk)讨论，再会！