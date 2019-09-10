---
title: Block深度软文
urlPath: Block
date: 2016-12-29 
updated: 2016-12-29
---

## 前言
深究block可以说会涉及不少东西，笔者欲通过循序渐进的方式来谈及block相关，略陈固陋。

阅读本文前，希望我们还是先一起来过一下几个概念：

* 指针和对象，都是内存块。一个大，一个小。一个在栈中，一个在堆中。
* iOS中，我们可以生命一个指针，也可以通过alloc获取一块内存。
* 我们可以直接消灭一个指针，将其置为nil，但是我们没办法直接消灭一块对象内存。对于对象内存，我们永远只能依靠系统去回收。即当这个对象不被任何指针所拥有时，系统就会收回该对象内存。
* 函数在栈区，函数调用完毕后其stack frame将被弹出结束其生命周期。
* Objective-C的对象在内存中是以堆的方式分配空间。

<!-- more -->

## 正文
###1、ARC strong和weak指针
在讲block之前呢先讲一下strong和weak指针的问题，以便于更好的理解下面block中的循环引用以及变量截获等问题。我们知道ARC消除了手动管理内存的烦琐，编译器会自动在适当的地方插入适当的retain、release、autorelease语句。规则很简单，只要还有一个变量指向对象，对象就会保持在内存中。当指针指向新值，或者指针不再存在时，相关联的对象就会自动释放。如下图动画模拟引用计数回收器，红色闪烁表示引用计数行为，引用计数的优势在于垃圾会被很快检测到，你可以看到红色闪烁过后紧接着该区域变黑（[图片来自](https://github.com/kenfox/gc-viz)）。

![REF_COUNT](http://i1.piimg.com/582490/0e069c17c9f92dee.gif)

* strong指针

比如在控制器上有个nameField属性，我在文本框中输入henvy，那么就可以说，nameField的text属性是NSString对象的指针，也就是拥有者，该对象保存了文本输入框的内容。

![](http://p1.bpimg.com/582490/6a231599ff8c1e3b.png)

如果执行了`NSString *name = self.nameField.text;`后，@“henvy”对象就有了多个拥有者，也就是有两个指针指向同一个对象。

![](http://p1.bpimg.com/582490/e393fb17d17140dd.png)

接下来我又在文本框中输入了新的内容比如@"Leslie"，此时nameFeild的text属性就指向了新的NSString对象。但原来的NSString对象仍然还有一个所有者(name变量)，因此会继续保留在内存中。

![](http://p1.bpimg.com/582490/e52fc130e059e8f5.png)

当name变量获得新值,或者不再存在时(如局部变量方法返回时、实例变量对象释放时),原先的NSString对象就不再拥有任何所有者,retain计数降为0,这时对象会被释放
如，给name变量赋予一个新值`name = @"Eason"`时。

![](http://i1.piimg.com/582490/f851f317dea65dcf.png)

我们称name和nameField.text指针为"Strong指针"，因为它们能够保持对象的生命。默认所有成员变量和局部变量都是Strong指针。

* weak指针

weak型的指针变量依然可以指向一个对象，但不属于对象的拥有者，就像我是很喜欢你，但是却得不到你一样。依然是我们上面的例子在输入框输入henvy后执行`__weak NSString *name = self.nameField.text;`后，虽然同时指向但name并不真正拥有henvy。

![](http://p1.bpimg.com/582490/95bd32d9b0749d01.png)

此时如果文本框内容重新输入@“Leslie”，则原先的henvy对象就没有拥有者，就会被释放，此时name变量会自动变成nil，称为空指针。weak型的指针变量自动变为nil避免了野指针的产生。

![](http://i1.piimg.com/582490/e1b0134308dc1e59.png)

举一个典型的weak指针的例子，即我们的代理模式，控制器ViewController强引用一个myTableView，myTableView的dataSource和delegate都是weak指针,指向你的ViewController。这也是cocoa设定的一个规则，即父对象建立子对象的强引用，而子对象只对父对象建立弱引用。

![](http://p1.bpimg.com/582490/1c82867a795ee3d8.png)

###2、Block的类型
好吧原谅我前面ARC讲了那么多，当然还是希望读者能够体会笔者的良苦用心。

* NSGlobalBlock

该类型的block存储在程序的数据区域(text段)，不引用外部变量，只对自己的参数做操作，自给自足的状态，可以当做函数使用，例如：


    typedef int (^GlobalBlock)(int);
    GlobalBlock block = ^(int count){
        return count;
    };  //nslog:<__NSGlobalBlock__: 0x10d090200>
* NSStackBlock

该类型的block在非ARC模式存储在栈区，内部引用外部变量，当栈block结束运行的时候会被请出栈，生命周期结束，再次调用当然crash掉，避免这一点可以通过手动copy将其拷贝到安全的堆上来，脱离栈的危险地带，因为本身栈区就是过河拆桥、兔死狗烹的状态。不像堆区讲究循环利用，生死由天定（无指针拥有被系统回收）。当然在ARC模式完全不用担心，ARC模式改写了天规杜绝NSStackBlock状况的发生，他会自动将block拷贝到堆上去（block作为方法或函数的参数传递时，编译器不会自动调用copy方法），从而演变成了第三种NSMallocBlock，此时的堆上的block就会像一个ObjC对象一样被放入autoreleasepool里面，从而保证了返回后的block仍然可以正确执行。因此在本该是NSStackBlock的情况下打印结果就会变成NSMallocBlock。

    typedef void (^StackBlock)();
    NSString *str = @"henvy";
    StackBlock block = ^{
        NSLog(@"%@",str);
    };  //nslog :<__NSMallocBlock__: 0x7fe412d18790>
* NSMallocBlock

该类型的block存储在堆区，引用外部变量，由NSStackBlock Block_copy()生成。在ARC模式下可以理解为只存在NSGlobalBlock和NSMallocBlock两种类型。
###3、Block对外部变量的存储管理
我们都知道内存有堆和栈两个部分，堆在高地址向下走，栈在低地址向上走。在每个函数调用的时候，系统都会为其生成一个栈的stack frame，该函数结束后这个frame被弹出去；然而堆对象的生存不从属于某个函数，即便是创建这个堆对象的函数结束了，堆对象也可以继续存在，因此内存泄漏都是堆对象惹的祸，ObjC里的引用计数就是用来管理堆对象这个东西，由于arc中没有引用计数的概念，只有强引用和弱引用的概念。当一个变量没有指针指向它时，就会被系统释放。因此我们通过下面的代码分别来测试。

* 静态变量、全局变量、全局静态变量

        - (void)testStaticObj
        {
        static NSString *staticString = nil;
        staticString = @"henvy";
    
        printf("%p\n", &staticString);//0x10b0d6138
        printf("%p\n", staticString);//0x10b0d5290

        void (^testBlock)() = ^{
        
        printf("%p\n", &staticString);//0x10b0d6138
        printf("%p\n", staticString);//0x0

        NSLog(@"%@", staticString);//null
        };
        staticString = nil;
    
        testBlock();
        }  
我这里只放上静态变量的测试代码，同全局变量、全局静态变量。我们发现staticString对象在block的外部和内部对象地址、指针地址都不变，且都在堆区。全局变量和全局静态变量由于作用域在全局，所以在block内访问和读写这两类变量和普通函数没什么区别，而静态变量作用域在block之外，静态变量通过指针传递，将变量传递到block内，进而来修改变量的值，即所谓的地址传递。

* 局部变量
        
        - (void)testLocalObj
        {
        NSString *localString = nil;
        localString = @"henvy";
    
        printf("%p\n", &localString); //0x7fff569cca48
        printf("%p\n", localString); //0x109234290

        void (^testBlock)(void) = ^{
        
        printf("%p\n", &localString); //0x7fcd20511100
        printf("%p\n", localString); //0x109234290

        NSLog(@"%@", localString); //henvy
        };
        localString = nil;

        testBlock();
        printf("%p\n", &localString); //0x7fff569cca48
        printf("%p\n", localString); //0x0
        }
我们发现局部变量在block定义前在栈上开辟指针空间，在堆上开辟对象空间，当然遵循ObjC对象的规则，在block内部指针位置发生了变化，对象位置不变，在block定义后同定义前。因而我们发现block对于局部变量只对其对象的值进行了拷贝，并不关心局部变量在外面的死活，跟block内部没有半点关系，正所谓的值传递。

* block变量
    
        - (void)testBlockObj
        {
        __block NSString *blockString = @"henvy";
        printf("%p\n", &blockString); //0x7fff54507a38
        printf("%p\n", blockString); //0x10b6f9290

        void (^testBlock)(void) = ^{
        printf("%p\n", &blockString); //0x7feb79c1e4b8
        printf("%p\n", blockString); //0x10b6f9290
        
        NSLog(@"%@", blockString);//henvy
        };
    
        testBlock();
        printf("%p\n", &blockString); //0x7feb79c1e4b8
        printf("%p\n", blockString); //0x10b6f9290
        }
我们发现`__block`修饰符的变量在block内部指针地址发生了变化，在block定义后地址彻底改为了新的地址，也就是说值彻底发生了变化，此时的blockString已经不是当年的那个blockString了。

总结一下：静态变量、全局变量和全局静态变量是通过指针传递，将变量传递到block内，进而来修改变量值。而外部变量是通过值传递，自然没法对获取到的外部变量进行修改，当我们需要修改外部变量时，可以用`__block`标记变量，也就是说没有`__block`标记的变量，其值会被复制一份到block私有内存区，而有`__block`标记的变量，其地址会被记录一份在block私有内存区。
### 4、Block循环引用
了解了强弱引用之后循环引用的问题就很好理解了，在ARC下，copy到堆上的block会强引用进入到该block中的外部变量，这因而导致循环引用的问题，一旦出现循环引用那么对象就会常驻内存，这显然是谁都不想看到的结果。此时需要用到`__weak`来打破这个闭合的环。

* __weak

ViewController控制器内有两个属性：

    @property (nonatomic, copy)NSString *string;
    @property (nonatomic, copy)void(^myBlock)();
在先分析下面的代码：

    self.string = @"henvy";
    self.myBlock = ^{
        NSLog(@"%@",self.string);
    };
    self.myBlock();
首先self强引用myBlock，当myBlock被copy到堆上时，myBlock开始强引用self.string，myBlock的拥有者self在Block作用域内部又引用了自己，因此导致了Block的拥有者永远无法释放内存，就出现了循环引用的内存泄漏。解决办法是`__weak`：

    #define HLWeakSelf(type)  __weak typeof(type) weak##type = type
    self.string = @"henvy";
    HLWeakSelf(self);
    self.myBlock = ^{
        NSLog(@"%@",weakself.string);
    };
    self.myBlock();
`__weak`就在Block内部对拥有者使用弱引用，通过这种方式告诉block，不要在block内部对self进行强制strong引用了。

* weak-strong dance

在有些特殊情况下，我们在block中又使用`__strong`来修饰这个在block外刚刚用`__weak`修饰的变量。这么做其实是为了避免在block的执行过程中，突然出现self被释放的尴尬情况而导致crash，官方说法weak-strong dance。列举经典到发光的AFNetworking中`AFNetworkReachabilityManager.m`的一段代码：

    __weak __typeof(self)weakSelf = self;
    AFNetworkReachabilityStatusBlock callback = ^(AFNetworkReachabilityStatus status) {
    __strong __typeof(weakSelf)strongSelf = weakSelf;

    strongSelf.networkReachabilityStatus = status;
    if (strongSelf.networkReachabilityStatusBlock) {
        strongSelf.networkReachabilityStatusBlock(status);
    }
    };

为了验证weak-strong dance下面我在一个HLBlockVC类中做如下实验，实验目的在于观察block中的weakSelf到底有没有释放，在该类中会并发两个线程，一个for循环到50后将weakSelf指针置空，另一个线程继续for循环到100，实验可能存在两种结果，一个是for循环到50block结束运行即失败，另一种情况block仍然继续输出到100即实验成功，下面代码说话：

在HLBlockVC类的viewDidLoad方法中加载一个线程：

    __block HLBlockVC *block = [[HLBlockVC alloc]init];
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        [block toPrintNum];
    });
    for (int i = 0; i < 51; i ++) {
        if (i == 50) {
            block = nil;
            NSLog(@"BLOCK WAS NIL");
        }
    }
添加toPrintNum方法，此时单单用weakSelf：

    - (void) toPrintNum{
    typedef void (^testBlock)();
    __weak __typeof(self)weakSelf = self;
    testBlock block = ^{        
        for (int i = 0; i < 100; i ++) {
            [weakSelf go:i];
        }
    };
    block();
    }
    
    -(void)go:(int)number{
    NSLog(@"%d",number);
    }
 
    2016-12-30 17:02:23.791 blockDemo[8520:343178] 48
    2016-12-30 17:02:23.791 blockDemo[8520:343098] BLOCK WILL NIL
    2016-12-30 17:02:23.792 blockDemo[8520:343178] 49
    2016-12-30 17:02:23.792 blockDemo[8520:343098] BLOCK WILL NIL
    2016-12-30 17:02:23.792 blockDemo[8520:343178] 50
    2016-12-30 17:02:23.792 blockDemo[8520:343098] BLOCK WAS NIL
代码很清晰，看上面的打印在循环到50的时候block被干掉了，执行结束，weakSelf下没问题。接下来换上weak-strong dance：

    - (void) toPrintNum{
    typedef void (^testBlock)();
    __weak __typeof(self) weakSelf = self;
    testBlock block = ^{
        __strong __typeof(weakSelf) strongSelf = weakSelf;
        
        for (int i = 0; i < 100; i ++) {
            [strongSelf go:i];
        }
    };
    block();
    }

    2016-12-30 18:03:42.934 blockDemo[8752:353486] 97
    2016-12-30 18:03:42.935 blockDemo[8752:353486] 98
    2016-12-30 18:03:42.935 blockDemo[8752:353486] 99
通过打印的数据可以看出__strong已经安全的保护了block中的weakSelf使之运行至block结束。可以说weak-strong dance是一种强引用 --> 弱引用 --> 强引用的变换过程，可能会被误解为绕了一圈什么都没做，其实不然，前者的强变弱是为了打破闭环的僵局，后者弱变强是为了block能够一直持有弱引用的对象生命，而strongSelf是一个自动变量会在函数执行完释放。

### 5、写在最后
回想一下或许很多时候我们遇到的问题很小，确实，就像文中的weak-strong dance，小到我们连遇到他犯错的机会都甚少，但坚持把小事做透、以小见大方能防微杜渐，步步为营！

夜深人静，除了键盘声，就是耳机里传来的歌声[【旅行团--生命是场马拉松】](http://music.163.com/#/song?id=29750167)夹杂着深夜琐碎的思绪，希望每一次发文都是对自己的一次洗礼。最后，晚安。