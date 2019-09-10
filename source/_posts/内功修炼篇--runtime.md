---
title: 内功修炼篇--runtime
urlPath: runtime
date: 2016-12-21
updated: 2016-12-21
---

## 前言
因为前些日子写了个关于导航栏控制器的Demo[地址在这](https://github.com/henvyluk/HLNavigationController)，开篇我想先稍微讲一下这个，我是觉得原生的导航栏在UI如此丰富以及多层VC的情形下，导航条的颜色、按钮、标题、隐藏等定性的修改显得不够圆滑，因此就想采用一种透明的方式，将VC用NC包装一层再push出去，这里就用到了AssociatedObjects，为所推的VC添加了属性。关联对象只是运行时中的一点，本篇文章想就关联对象和运行时的一些其他常见用法姑且谈谈吾之愚见，望抛砖引玉。

<!-- more -->

## 正文

其实网上关于运行时的东西多如牛毛，但感觉都像在一遍一遍的嚼舌根又不好理解，我就坦诚相见，拒绝抽象。

    #import <objc/runtime.h>
运行时其实就是用C编写的我们oc的基石，我们通过运行时所提供的方法等可以跨越oc层直接与C交互，当然对性能也会有所提升。运行时会对一个类进行完全的分解，将类或者对象的每一个部分抽象成一种类型，如果把oc的类比作一个组装机器人，那他就会被运行时拆分为手臂、腿、身体等，我们可以通过运行时直接获取到机器人的手臂一样，这对于操作一个类的属性或者方法是非常方便的。

我们在开发中切实可以用到的一些场景我做了归纳，下面一一讲解：   
### 1、关联对象
关联对象相关的函数主要有三个，命名相当友好到一看就知道其实就是get/set方法，我们可以在category中使用它们实现动态向类中添加属性和方法。

* objc_setAssociatedObject
* objc_getAssociatedObject
* objc_removeAssociatedObjects

看一个添加属性的例子，我们创建一个NSObject的分类CategoryProperty：
    
    @interface NSObject (CategoryProperty)
    
    @property (nonatomic, strong) NSObject *property;
    
    @end
    
    @implementation NSObject (CategoryProperty)

    - (NSObject *)property {
    return objc_getAssociatedObject(self, @selector(property));
    }

    - (void)setProperty:(NSObject *)value {
    objc_setAssociatedObject(self, @selector(property), value, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    }

    @end
##### key值
这三个函数的参数key值推荐三种命名方式：

* 声明 `static char kAssociatedObjectKey`，使用 `&kAssociatedObjectKey` 作为 `key` 值;
* 声明 `static void *kAssociatedObjectKey` = `&kAssociatedObjectKey` ，使用 `kAssociatedObjectKey` 作为 `key` 值；
* 用 `selector` ，使用 `getter` 方法的名称作为 `key` 值。

上面的例子用的是第三种方法，省的命名了也算简单。
##### 关联策略
至于关联策略有五种可供选择，有强弱引用和原子非原子的区分，在绝大多数情况下，我们都会使用`OBJC_ASSOCIATION_RETAIN_NONATOMIC` 的关联策略，这可以保证我们持有关联对象不会被过早的释放。

在看一个添加方法的例子，我们创建一个UIButton的分类block：
    
    typedef void (^btnBlock)();

    @interface UIButton (block)

    - (void)handelWithBlock:(btnBlock)block;

    @end

    static const char btnKey;

    @implementation UIButton (block)
    
    - (void)handelWithBlock:(btnBlock)block{
    if (block){
        objc_setAssociatedObject(self, &btnKey, block, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
        }
    [self addTarget:self action:@selector(btnAction) forControlEvents:UIControlEventTouchUpInside];
    }
    
    - (void)btnAction{
    btnBlock block = objc_getAssociatedObject(self, &btnKey);
    block();
    }
    
    @end
这样我们就为button添加了一个block的方法，在调用button的时候就可以直接用handelWithBlock来回调了。
### 2、方法交换
顾名思义，就是两个方法执行交换，我们建一个UIViewController的分类VCCategory：

    @implementation UIViewController (VCCategory)

    + (void)load
    {
    //方法交换应该被保证在程序中只会执行一次
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        SEL systemSel = @selector(viewWillAppear:);
        SEL henvySel = @selector(hl_viewWillAppear:);
        Method systemMethod = class_getInstanceMethod([self class], systemSel);
        Method henvyMethod = class_getInstanceMethod([self class], henvySel);
        BOOL isAdd = class_addMethod(self, systemSel, method_getImplementation(henvyMethod), method_getTypeEncoding(henvyMethod));
        if (isAdd) {
            //如果成功，说明类中不存在这个方法的实现
            //将被交换方法的实现替换到这个并不存在的实现
            class_replaceMethod(self, henvySel, method_getImplementation(systemMethod), method_getTypeEncoding(systemMethod));
        }else{
            //否则，交换两个方法的实现
            method_exchangeImplementations(systemMethod, henvyMethod);
        }
    });
    }
    - (void)hl_viewWillAppear:(BOOL)animated{
    //这里自己调用自己，表面上循环引用其实已经被viewWillAppear替换掉了
    [self hl_viewWillAppear:animated];
    NSLog(@"henvy");
    }
    @end
这个时候在一个自己定义的viewController中viewWillAppear方法中就可以看到输出henvy。
### 3、发送消息
发送消息即objc_msgSend方法很简单，这里就举个很简单的例子,比如你要调用形如一下的一个方法，
    
    //类、方法、参数
    [someObject messageName:parameter];
还可以用objc_msgSend写作为：

    objc_msgSend（someObject,@selector（messageName),parameter)；
### 4、字典转模型
KVC是把字典中所有值给模型的属性赋值，这个是要求字典中的Key必须要在模型里能找到相应的值，如果找不到就会报错，因此我们可以通过重写KVC中的forUndefinedKey这个方法。当然我们可以通过runtime的方式去实现，把KVC的原理倒过来，通过遍历模型的值，从字典中取值，这里新建一个模型ModelClass：

    + (instancetype)modelWithDict:(NSDictionary *)dict{
    
    id objc = [[self alloc] init];
    // count:成员变量个数
    unsigned int count = 0;
    // 获取成员变量数组
    Ivar *ivarList = class_copyIvarList(self, &count);
    
    // 遍历所有成员变量
    for (int i = 0; i < count; i++) {
        // 获取成员变量
        Ivar ivar = ivarList[i];
        
        // 获取成员变量名字
        NSString *ivarName = [NSString stringWithUTF8String:ivar_getName(ivar)];
        // 获取成员变量类型
        NSString *ivarType = [NSString stringWithUTF8String:ivar_getTypeEncoding(ivar)];
        // 格式化
        ivarType = [ivarType stringByReplacingOccurrencesOfString:@"\"" withString:@""];
        ivarType = [ivarType stringByReplacingOccurrencesOfString:@"@" withString:@""];
        // 获取key
        NSString *key = [ivarName substringFromIndex:1];
        
        // 去字典中查找对应value        
        id value = dict[key];
        
        // 二级转换:判断下value是否是字典
        if ([value isKindOfClass:[NSDictionary class]] && ![ivarType hasPrefix:@"NS"]) {
            // 获取类
            Class modelClass = NSClassFromString(ivarType);
            value = [modelClass modelWithDict:value];
        }
        // 给模型中属性赋值
        if (value) {
            [objc setValue:value forKey:key];
        }
    }
    
    return objc;
    }

### 写在最后
总体上来说运行时在开发中比较常用的到的场景我就先总结这么多，当然也欢迎大神能够来补充是最好不过，文章中的测试代码我都写在前言的[Demo](https://github.com/henvyluk/HLNavigationController)里了，同时也欢迎到我的[Github](https://github.com/henvyluk)讨论，如果本文有什么不太对的地方，也请一定要给我指正，感激不尽！

祝大家晚安！另外送上一首歌[【李志—墙上的向日葵】](http://music.163.com/#/song?id=30212877)
