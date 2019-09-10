---
title: github+hexo提交到百度谷歌搜索引擎
urlPath: github&hexo
date: 2016-12-17 
updated: 2016-12-17
---
## 前言
前些日子用GitHub Pages+hexo搭建了一个网站，后来发现百度谷歌都是无法搜索到我的网站上的内容。

## 确认收录情况
在百度或者谷歌上面输入下面格式来判断，如果能搜索到就说明被收录，否则就没有.

<!-- more -->

    
    site:henvyluk.com
![](http://p1.bqimg.com/4851/d8f8483a9ed400da.png)
## 1、验证网站
验证网站的目的说白了就是证明这个网站是你的，这是你使用站长平台更多功能的前提条件，另外没有梯子的童鞋可以看这里[shadowsocks](http://www.ishadowsocks.info)当然如果你有自己的或者不需要谷歌搜索可以忽略。下面是两个搜索引擎入口：

* [Google搜索引擎提交入口](https://www.google.com/webmasters/tools/home?hl=zh-CN)
* [百度搜索引擎提交入口](http://zhanzhang.baidu.com/linksubmit/url)


百度站长平台为未使用百度统计的站点提供三种验证方式：文件验证、html标签验证、CNAME验证，应该说文件和CNAME验证较为简单，我这里统一使用CNAME验证。

* 文件验证：您需要下载验证文件，将文件上传至您的服务器，放置于域名根目录下。
* html标签验证：将html标签添加至网站首页html代码的<head>标签与</head>标签之间。
* CNAME验证：您需要登录域名提供商或托管服务提供商的网站，添加新的DNS记录。

CNAME验证需要到你的域名商站点添加DNS解析即可，主机和指向百度已经给你了的，我的域名是在[godaddy](https://sg.godaddy.com/zh/)上，分别添加主机和指向如图下：
![添加CNAME](http://i1.piimg.com/4851/603534cdc827a654.png)
然后点完成验证即可：
![通过验证](http://i1.piimg.com/4851/3504a089e48f703a.png)

谷歌的就更简单了，直接点验证，等待自动验证结束即可。

![谷歌验证](http://i1.piimg.com/4851/add226355d0ea30f.png)

## 2、站点地图
站点地图是一种文件，您可以通过该文件列出您网站上的网页，从而将您网站内容的组织架构告知Google和其他搜索引擎。Googlebot等搜索引擎网页抓取工具会读取此文件，以便更加智能地抓取您的网站。

首先在hexo根目录分别安装百度谷歌的站点地图文件：

    npm install hexo-generator-sitemap --save
    npm install hexo-generator-baidu-sitemap --save
在博客目录的_config.yml中添加路径：
   
    //自动生成sitemap
    sitemap:
    path: sitemap.xml
    baidusitemap:
    path: baidusitemap.xml
接下来编译：

    hexo g
    
此时会发现在public下面发现生成了sitemap.xml以及baidusitemap.xml文件，可以看一下这两个文件，大致就是你的域名下的几篇文章的链接，如果不是以你的域名（github），github禁止了百度爬虫，提交了百度也是不会访问的，所以需要改为你自己的域名，我这里自动生成的是我自己的，接下来部署：

    hexo d
部署成功后分别访问：

    http://henvyluk.com/sitemap.xml
    http://henvyluk.com/baidusitemap.xml

## 3、谷歌收录
让谷歌收录我们的博客直接向[Google站长工具](https://www.google.com/webmasters/tools)提交sitemap文件即可，登录谷歌账号，选择当前站点，左边一栏抓取-站点地图-添加站点地图即可，相信谷歌的效率，明天你的站点谷歌就搜索得到了。

![添加站点地图](http://i1.piimg.com/4851/23774a2e7364ec72.png
)
## 4、百度收录
谷歌的不要太简单，百度就稍微麻烦一点，分为四种方式来提交你的链接，

* 主动推送：最为快速的提交方式，推荐您将站点当天新产出链接立即通过此方式推送给百度，以保证新链接可以及时被百度收录。
* 自动推送：最为便捷的提交方式，请将自动推送的JS代码部署在站点的每一个页面源代码中，部署代码的页面在每次被浏览时，链接会被自动推送给百度。可以与主动推送配合使用。
* sitemap：您可以定期将网站链接放到sitemap中，然后将sitemap提交给百度。百度会周期性的抓取检查您提交的sitemap，对其中的链接进行处理，但收录速度慢于主动推送。
* 手动提交：一次性提交链接给百度，可以使用此种方式。
比较推荐前三种提交方式，从效率上来说，接下来依次讲解：


        主动推送>自动推送>sitemap
### 主动推送
主动推送官网上也有说明，不过需要一定的代码功底，我这里通过第三方写好的一个插件来做说明，首先，在Hexo根目录下，安装插件：

        npm install hexo-baidu-url-submit --save
然后，同样在根目录下，把以下内容配置到_config.yml文件中:

        baidu_url_submit:
        count: 3 ## 比如3，代表提交最新的三个链接
        host: www.henvyluk.com ## 在百度站长平台中注册的域名
        token: your_token ## 请注意这是您的秘钥， 请不要发布在公众仓库里!
        path: baidu_urls.txt ## 文本文档的地址， 新链接会保存在此文本文档里
其次，记得查看_config.ym文件中url的值， 必须包含是百度站长平台注册的域名（一般有www）， 比如:

        # URL
        url: http://www.henvyluk.com
        root: /
        permalink: :year/:month/:day/:title/
接下来添加一个新的deploy 的类型，用减号区分：

        deploy:
        - type: git
        repository: https://github.com/henvyluk/henvyluk.github.io
        branch: master
        - type: baidu_url_submitter
执行hexo deploy的时候，新的连接就会被推送了。这里讲一下原理：
* 新链接的产生， hexo generate 会产生一个文本文件，里面包含最新的链接
* 新链接的提交， hexo deploy 会从上述文件中读取链接，提交至百度搜索引擎 

### 自动推送
如果是next主题，next主题配置文件中的baidu_push设置为true，就可以了。我的并不是next主题，我用的是icarus，这就要想办法塞进以下的全站点js代码：

    <script>
    (function(){
    var bp = document.createElement('script');
    var curProtocol = window.location.protocol.split(':')[0];
    if (curProtocol === 'https') {
        bp.src = 'https://zz.bdstatic.com/linksubmit/push.js';        
    }
    else {
        bp.src = 'http://push.zhanzhang.baidu.com/push.js';
    }
    var s = document.getElementsByTagName("script")[0];
    s.parentNode.insertBefore(bp, s);
    })();
    </script>
    
一般在以下目录加入中加入即可，
    
    blog\themes\xxxx\layout\_partial\head.ejs
我的是在

    blog\themes\icarus\layout\common\head.ejs

### sitemap
sitemap很简单直接提交[http://henvyluk.com/baidusitemap.xml](http://henvyluk.com/baidusitemap.xml)就可以了
![sitemap](http://p1.bqimg.com/4851/e66a07fdbb0fb1ff.png)

## 5、写在最后
相比较百度还是要比谷歌慢，总体谷歌提交简单的太多了，谷歌第二天就可以搜索到网站内容，百度不太给力，也使得这篇文章托到了现在，大致的提交流程就是这样，再会！

