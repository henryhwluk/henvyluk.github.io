---
title: PaintCode让Logo飞出屏幕
urlPath: PaintCode
date: 2016-12-25 
updated: 2016-12-25
---

## 前言
相信大家都见过Twitter APP的加载界面，logo飞出屏幕的感觉很是生动，PaintCode这款软件可以自动生成你的图案代码，如图是模仿Twitter的加载界面。

<!-- more -->

## 正文
### 1、Sketch
在运用PaintCode之前呢，需要先用到Sketch这款软件，目的是制作SVG格式的文件导入到PaintCode，不多说直接上工具。

>SVG （可缩放矢量图形）
>
>SVG可缩放矢量图形（Scalable Vector Graphics）是基于可扩展标记语言（XML），用于描述二维矢量图形的一种图形格式。SVG是W3C("World Wide Web ConSortium" 即 " 国际互联网标准组织")在2000年8月制定的一种新的二维矢量图形格式，也是规范中的网络矢量图形标准。SVG严格遵从XML语法，并用文本格式的描述性语言来描述图像内容，因此是一种和图像分辨率无关的矢量图形格式。

首先将原图导入Sketch，如图：
![Sketch](http://i1.piimg.com/1949/07e8631cffc64b8f.png)
接着在我们左上角选用insert->vector，我们的钢笔工具，在图片的各个顶点处链接起来，链接完成后就是我们如图的样子：
![vector](http://i1.piimg.com/1949/4c6bc2c8b63af936.png)
然后双击我们任意一个顶点的时候，右边一栏会出现如图的工具，我们选用Mirrored将四边做调节以贴合logo的轨迹。

![Mirrored](http://i1.piimg.com/1949/75e43f85f7519211.png)

最后调节后的效果：
![Sketch](http://i1.piimg.com/1949/f0894929f7418ddf.png)
最后点击我们的路径，右下角会有Export板，选择SVG格式后保存即可，到此为止Sketch已经完成他的使命了，下面该我们的PaintCode上场了。
![Export](http://i1.piimg.com/1949/6e45baf127e893af.png)

### 2、PaintCode
PaintCode的用法很简单，直接将上面保存的SVG图片扔给他就好了，他会自动生成代码，包括swift和oc，直接将生成的代码复制即可。
![PaintCode](http://i1.piimg.com/1949/4b6efa91bbf790d9.png)
### 3、xcode
这时候回到我们的xcode项目中来，我新建了一个继续于UIView的LaunchViewTwitter类作为整个启动界面的版面，又添加了launchView属性，作为显示logo的view。

我先将PaintCode生成的代码放进bezierPath方法中：

    -(UIBezierPath *)bezierPath
    {
       //PaintCode生成的代码
    }


接着添加launchView和layer：

    -(void)addLayerToLaunchView
    {
    //添加launchView
    self.launchView = [[UIView alloc]initWithFrame:CGRectMake(0, 0, 150, 150)];
    self.launchView.backgroundColor = [UIColor clearColor];
    self.launchView.center = self.center;
    [self addSubview:self.launchView];
    
    添加layer
    CAShapeLayer *layer = [[CAShapeLayer alloc]init];
    layer.path = [self bezierPath].CGPath;
    layer.bounds = CGPathGetBoundingBox(layer.path);
    self.backgroundColor = [UIColor colorWithRed:0.18 green:0.70 blue:0.90 alpha:1.0];
    layer.position = CGPointMake(self.launchView.bounds.size.width / 2, self.launchView.bounds.size.height/ 2);
    layer.fillColor = [UIColor whiteColor].CGColor;
    [self.launchView.layer addSublayer:layer];
    
    //执行动画 
    [self performSelector:@selector(startLaunch) withObject:nil afterDelay:1.0];
    }
为launchView添加动画：

    - (void)startLaunch
    {
    [UIView animateWithDuration:1 animations:^{
    //先缩小launchView
        self.launchView.transform = CGAffineTransformMakeScale(0.5, 0.5);
        
    } completion:^(BOOL finished) {
        [UIView animateWithDuration:1 animations:^{
        
        //在无线放大launchView
            self.launchView.transform = CGAffineTransformMakeScale(50, 50);
            self.launchView.alpha = 0;
        } completion:^(BOOL finished) {
        
        //最后移除
            [self.launchView removeFromSuperview];
            [self removeFromSuperview];
        }];;
    }];
    }
此时加载LaunchViewTwitter的类就写完了，回到我们主控制器，添加继承与LaunchViewTwitter的launchView属性，在viewDidAppear方法中，加载动画类：

    - (void)launchAnimation
    {
    //获取到LaunchScreen控制器(不要忘记id)
    UIViewController *viewController = [[UIStoryboard storyboardWithName:@"LaunchScreen" bundle:nil] instantiateViewControllerWithIdentifier:@"LaunchScreen"];
    
    //获取LaunchScreen的view
    UIView *launchView = viewController.view;
    UIWindow *mainWindow = [UIApplication sharedApplication].keyWindow;
    launchView.frame = [UIApplication sharedApplication].keyWindow.frame;
    [mainWindow addSubview:launchView];
    
    //添加launchView类
    self.launchView = [[LaunchViewTwitter alloc]initWithFrame:[UIScreen mainScreen].bounds];
    [self.launchView addLayerToLaunchView];
    [launchView addSubview:self.launchView];
    
    //最后移除
    [UIView animateWithDuration:0.5f delay:2.5f options:UIViewAnimationOptionBeginFromCurrentState animations:^{
        launchView.alpha = 0.0f;
    } completion:^(BOOL finished) {
        [launchView removeFromSuperview];
    }];
    }
到这里加载界面就做完了，就是我们前言的demo动画。
### 4、写在最后
确实在UI/UE如此丰富的今天，很多时候一些人性化或者较有设计感的界面或动画会大幅度的提升用户的好感度，毕竟人人看脸的时代，我们的APP也绝不例外，快去让自己的APP更加充满活力吧。另外文中的测试demo[在这里](https://github.com/henvyluk/PaintCode)同时如果有什么不清楚的欢迎留言或者到我的[github](https://github.com/henvyluk)讨论。


最后祝大家晚安，老样子送上一首歌曲[【张国荣--心跳呼吸正常】](http://music.163.com/#/song?id=186490)