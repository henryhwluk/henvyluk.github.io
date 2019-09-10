---
title: iOS IJKPlayer RTMP播放器的集成
urlPath: iOSIJKPlayer
date: 2016-12-05 21:33:06
updated: 2016-12-05
---


# iOS IJKPlayer RTMP播放器的集成
### 前言
前些日子公司要做视频直播，一直也是项目的原因没来得及整理内容。周末闲暇时间特来写篇文章，对IJKPlayer播放器的集成做一下归纳，希望对要做iOS视频直播方向的开发者有所帮助。

<!-- more -->

### 1、环境搭建
总结来说就是[HomeBrew](http://brew.sh/index_zh-cn.html) or [MacPorts](https://www.macports.org/install.php)、git、yasm的安装(Homebrew是Mac OSX上的软件包管理工具，当时macOS Sierra刚刚出来我就手贱更新了，导致HomeBrew安装出现了问题，所以采用了MacPorts，官方推荐HomeBrew)。

这里我只做一下版本检查：
![version Screenshot](http://ww1.sinaimg.cn/large/bd65c956gw1fa6x3vv5utj20fr0a276y.jpg)
首先HomeBrew：
    
    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
待HomeBrew安装完毕即git、yasm：
     
     brew install git
     brew install yasm

### 2.下载ijkplayer编译
1、 首先新建要下载的文件夹ijkplayer并cd到该目录下。

2、 紧接着将ijkplayer文件克隆到新建的文件夹内，在终端输入：
    
    //git克隆
    git clone https://github.com/Bilibili/ijkplayer.git ijkplayer-ios
    
    //进入ijkplayer-ios
    cd ijkplayer-ios
    
    //切换分支
    git checkout -B latest k0.7.5
如图所示：
![download Screenshot](http://ww3.sinaimg.cn/large/bd65c956gw1fa80uhh7udj20fq0a2q79.jpg)
3、下载ffmpeg并编译

这一步比较纠结国外网络访问的问题，如果失败就多试几次。

    //依然在ijkplayer-ios下载ffmpeg
    ./init-ios.sh
    
    //进入ios目录
    cd ios
    
 这一步如果前面没有问题，此时的terminal就像打了鸡血一样狂奔......
 (注意中途会有n多个警告，但不要出错就没问题)
    
    //clean
    ./compile-ffmpeg.sh clean
    
    //编译
    ./compile-ffmpeg.sh all
    
 成功走到这一步就离成功不远了，按步骤走是不会出现问题的，即便有大部分也是网络的原因毕竟大天朝对国外的网络都懂的。

### 3、demo的处理
1、打开官方demo并运行
![file Screenshot](http://ww3.sinaimg.cn/large/bd65c956gw1fa81grkbobj20la0bzdj3.jpg)

2、只要前面的流程没报错，这里编译运行都不会出现问题：
可以在Online Samples中选择一个m3u8测试ijkplayer是否运行正常如图：
![Samples Screenshot](http://ww4.sinaimg.cn/large/bd65c956gw1fa81xdraz9j20f8091t9i.jpg)

### 4、制作framework
1、打开ijkplayer如图：
![file Screenshot](http://ww3.sinaimg.cn/large/bd65c956gw1fa82aawbmvj20l80by77d.jpg
)
2、选择edit scheme，下图：
![edit Screenshot](http://ww1.sinaimg.cn/large/bd65c956gw1fa82kh3xkqj20cd05hgme.jpg)

3、将build configuration改为Release后点Close，如图：
![Release Screenshot](http://ww2.sinaimg.cn/large/bd65c956gw1fa82mft3vej20ot0dw75x.jpg)

4、分别在模拟器和真机(Generic iOS Device也可以)上编译：
![](http://ww4.sinaimg.cn/large/bd65c956gw1fa82nzpo4lj209m010glj.jpg)
![](http://ww2.sinaimg.cn/large/bd65c956gw1fa82oevlx3j209a0103ye.jpg)

5、打开framework所在的目录:
![](http://ww4.sinaimg.cn/large/bd65c956gw1fa82pkzyy4j20az07vdh1.jpg)

6、看名字就知道一个是模拟器一个是真机，此时cd到Products目录：

    //合并
    lipo -create Release-iphoneos/IJKMediaFramework.framework/IJKMediaFramework Release-iphonesimulator/IJKMediaFramework.framework/IJKMediaFramework -output IJKMediaFramework
    //将合并后的framework拷贝到iphoneos/IJKMediaFramework.framework中
    cp IJKMediaFramework Release-iphoneos/IJKMediaFramework.framework/
7、此时framework就制作好了，将制作好的iphoneos/IJKMediaFramework.framework复制到要集成的项目中Add Files...

8、在所在的项目中添加动态库
![lib Screenshot](http://ww4.sinaimg.cn/large/bd65c956gw1fa84ty8rbgj20ky0b140n.jpg)

9、测试集成，将本段代码复制到ViewController.m中，可直接使用：

    #import "ViewController.h"
    #import <IJKMediaFramework/IJKFFMoviePlayerController.h>
    @interface ViewController ()
    @property(nonatomic,strong)IJKFFMoviePlayerController * player;
    @end

    @implementation ViewController

    - (void)viewDidLoad {
    [super viewDidLoad];

    IJKFFOptions *options = [IJKFFOptions optionsByDefault]; //使用默认配置
    NSURL * url = [NSURL URLWithString:@"rtmp://live.hkstv.hk.lxdns.com/live/hks"]; 
    self.player = [[IJKFFMoviePlayerController alloc] initWithContentURL:url withOptions:options]; //初始化播放器，播放在线视频或直播(RTMP)
    self.player.view.autoresizingMask = UIViewAutoresizingFlexibleWidth|UIViewAutoresizingFlexibleHeight;
    self.player.view.frame = self.view.bounds;
    self.player.scalingMode = IJKMPMovieScalingModeAspectFit; //缩放模式
    self.player.shouldAutoplay = YES; //开启自动播放

    self.view.autoresizesSubviews = YES;
    [self.view addSubview:self.player.view];
    }

    - (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    [self.player prepareToPlay];
    }

    -(void)viewDidDisappear:(BOOL)animated {
    [super viewDidDisappear:animated];
    [self.player shutdown];
    }
    - (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
    }

    @end
    
OK完美收工：

![](http://ww3.sinaimg.cn/large/bd65c956gw1fa835tggdlj208j0fsgmm.jpg)

### 3、github
地址在这[https://github.com/henvyluk/IJKMediaPlayer](https://github.com/henvyluk/IJKMediaPlayer),另附demo一份，望大神不吝赐教再会！