来源于 Ry’s Objective-C Tutorial - RyPress

一个学习Objective-C基础知识的网站.
 
个人觉得很棒,所以决定抽时间把章节翻译一下.

本人的英语水平有限,有让大家误解或者迷惑的地方还请指正.

原文地址:http://rypress.com/tutorials/objective-c/memory-management

仅供学习,如有转摘,请注明出处.
***
如我们在[属性](http://www.jianshu.com/p/bccd94066442)章节讨论的, 任何一种内存管理系统的目的都是通过控制其所有对象的生命周期来减少内存占用. iOS和OS X应用程序通过对象拥有关系来完成(管理对象生命周期), (这种关系)确保对象应该存在的时间, 而不是或多或少.

这种对象拥有关系方案通过引用计数系统来实现的, 引用计数即是跟踪并记录每个对象的拥有者个数. 当你需要一个对象时, 便要增加该对象的引用数量, 并且当你不在使用该对象, 便减少它的引用计数. 只要一个对象的引用计数大于0, 便能保证对象存活, 一旦(引用的)数量达到0, 系统便会被允许销毁该对象.

![Destroying an object with zero references](http://upload-images.jianshu.io/upload_images/682074-b725ab3c194fbf14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在过去, 开发人员手动控制一个对象的引用数量, 通过调用在[NSObject protocol]中定义的特定内存管理方法. (这种管理方式)被称作手动保持与释放(MRR). 还好, Xcode4.2版本引入了自动引用计数(ARC), 意味着可以为你自动插入这些(自动内存管理的)方法. 现在的程序应该只使用ARC, 因为它更可靠而且让你专注于程序的功能, 而不是它的内存管理.

本模块解释了MRR中的引用计数核心概念, 然后讨论了ARC的一些实际中需要考虑的因素.

#### Manual Retain Release
在手动内存管理环境下, 程序中每个对象的拥有关系的得到和放弃都要你负责. 通过调用以下特定的内存相关方法来完成.

方法 | 行为 |
------------ | ------------- 
alloc | 新建一个对象并得到(对它的)拥有关系 
retain | 对已存在的对象得到(对它的)拥有关系
copy | 复制一个对象并得到(对它的)拥有关系
release | 放弃一个对象的拥有关系, 并立即销毁
autorelease | 放弃一个对象的拥有关系, 但是延迟销毁

手动控制对象拥有关系看起来是一件很可怕的任务, 但实际上很容易. 你所需要做的就是得到你需要的对象的拥有关系并记得用完以后放弃(拥有关系)就行了. 从实际情况来看, 上述意味着, 对于相同的对象, 你必须通过对应的release和autorelease来平衡该对象的每个alloc, retain以及copy操作.

一旦你忘记平衡这些操作, 两个事情中的一个就会发生. 如果你忘记释放一个对象, 那么它占用的内存永远不会被释放, 从而导致内存泄露. 稍小的泄露不会对程序造成明显的影响, 但是如果吃了足够的内存, 那程序最终会崩溃. 另一方面, 如果尝试多次释放一个对象, 会引起被称作指针悬挂的问题. 一旦你尝试访问这些悬挂指针, 便是在请求非法的内存地址, 那么程序也最有可能崩溃.

![Balancing ownership claims with reliquishes](http://upload-images.jianshu.io/upload_images/682074-9abd2793a743402d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

该部分的剩余内容将解释怎样适当使用上述方法来避免内存泄露和悬挂指针.
#####Enabling MRR - 激活MRR
在我们开始尝试手动内存管理之前, 我们需要关闭自动引用计数(系统). 在项目导航栏中点击项目图标, 确保选中了Build Setting选项, 然后在搜索栏中输入automatic reference counting. OC的自动引用计数编译选项就会出现. 将选项YES调成NO.
![Turning off Automatic Reference Counting](http://upload-images.jianshu.io/upload_images/682074-9ad8fc28d32cc229.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

记住, 我们仅为了学习的目的才这么干 - 在新项目中, 永远别使用MRR.
#####The alloc Method
在这个教程中, 我们已经用过alloc方法去创建对象了. 但是, 它不仅仅是为对象分配空间, 它同时也设置对象的引用数为1. 这完全可以理解, 因为, 如果我们不想让一个对象存在, 哪怕是一会, 我们就不会去创建该对象的.

```
// main.m
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSMutableArray *inventory = [[NSMutableArray alloc] init];
        [inventory addObject:@"Honda Civic"];
        NSLog(@"%@", inventory);
    }
    return 0;
}
```
上面的代码看着应该很熟悉, 我们做的就是实例化一个可变数组, 给它添加一个值, (随后)展示它的内容. 从内存管理的角度来看, 我们拥有了这个inventory对象, 就意味着什么时候释放它也成为了我们的责任.

但是, 由于我们并没有释放它, 所以程序目前有一处内存泄露. 可以通过静态内存分析工具运行项目来检查这个问题(内存泄露). 在菜单栏中, 选择导航 -> Product -> Analyze, 或者使用Shift + Cmd + B键盘快捷键(启动). (该功能)将会查找到代码中可预料的问题, 如下面main.m中展示的那样.

![A memory leak! Oh no!](http://upload-images.jianshu.io/upload_images/682074-11727dad8437062b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是一个小对象, 所以这个泄露并不是灾难性的. 然而, 如果它反复发生.(比如, 在一个长循环或者每次用户点击一个按钮(都会发生)), 那么这个程序最终内存溢出并崩溃.
#####The release Method
release方法通过减少一个对象的的引用数量来放弃相应的拥有关系. 因此, 我们可以通过在mian.m中的NSLog()调用之后添加下面这一样(代码)来避免内存泄露:

```
[inventory release];
```
现在, 我们的alloc已通过release(达到)平衡了, 静态分析器不再输出任何问题. 一般来说, 在方法中创建的对象, 都会在该方法的的结尾处放弃对该对象的拥有关系, 就像我们这里做的事情.  

太早的释放一个对象会产生悬挂指针(的问题). 比如, 试着将上面的代码移到调用NSLog()方法之前. 因为release会立即释放真实的内存, 所以在NSLog()中的inventory变量指向一个非法地址, 当你再尝试运行, 程序会报EXC_BAD_ACCESS错误并崩溃:

![Trying to access an invalid memory address](http://upload-images.jianshu.io/upload_images/682074-52a9d36120856ae5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

关键就在于, 在用完该对象之前不要放弃对它的拥有关系.
#####The retain Method
retain方法获取一个已存在对象的拥有关系. 像是在告诉操作系统, "嗨, 我也需要那个对象, 不要干掉它". 当其他对象需要保证(自身)属性引用的是一个合法实例(时), 这是一个必备的能力.

举个例子, 我们使用retain来得到对inventory数组的强引用. 新建一个CarStore类, 然后修改它的头文件如下:

```
// CarStore.h
#import <Foundation/Foundation.h>

@interface CarStore : NSObject

- (NSMutableArray *)inventory;
- (void)setInventory:(NSMutableArray *)newInventory;

@end
```
手动声明inventory属性的访问器. 第一次迭代中的CarStore.m中提供了getter, setter(使用一个实例变量来记录对象)的简易实现:

```
// CarStore.m
#import "CarStore.h"

@implementation CarStore {
    NSMutableArray *_inventory;
}

- (NSMutableArray *)inventory {
    return _inventory;
}

- (void)setInventory:(NSMutableArray *)newInventory {
    _inventory = newInventory;
}

@end
```
回到main.m, 将inventory变量分配给CarStore's的inventory属性:

```
// main.m
#import <Foundation/Foundation.h>
#import "CarStore.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSMutableArray *inventory = [[NSMutableArray alloc] init];
        [inventory addObject:@"Honda Civic"];
        
        CarStore *superstore = [[CarStore alloc] init];
        [superstore setInventory:inventory];
        [inventory release];

        // Do some other stuff...

        // Try to access the property later on (error!)
        NSLog(@"%@", [superstore inventory]);
    }
    return 0;
}
```
因为该对象(inventory)已经在main.m中提早被释放了, superStore对象只有一个对该数组的弱引用, 所以最后一行代码中的inventory属性已是一个悬挂指针. 为了将其调整成强引用, CarStore需要在它的setInventory: 访问器中得到一个对该数组的拥有关系:

```
// CarStore.m
- (void)setInventory:(NSMutableArray *)newInventory {
    _inventory = [newInventory retain];
}
```
这样就能保证inventory对象在superStore在使用它的过程中不被释放. 注意, retain方法返回的是对象本身, 所以允许我们将retain和赋值写成一行.

很遗憾, 这样的代码会产生其他的问题: 即, retain调用没有release来平衡(对应), 所以有内存泄露(的问题). 一旦我们给setInventory:传递其他值, 我们无法得到旧值, 意味着我们永远都不能释放它了. 为了改正(这个问题), setInventory:需要对旧值调用release:

```
// CarStore.m
- (void)setInventory:(NSMutableArray *)newInventory {
    if (_inventory == newInventory) {
        return;
    }
    NSMutableArray *oldValue = _inventory;
    _inventory = [newInventory retain];
    [oldValue release];
}
// 或者这么写
- (void)setInventory:(NSMultableArray*)newInventory {
	if (_inventory != newInventory) {
		[_inventory release];
		_inventory = [newInventory retain];
	}
}
```
这就是属性的retain以及strong特性本质做的事情. 很显然, 使用@property比我们自己创建这些访问器方便很多.

![Memory management calls on the inventory object](http://upload-images.jianshu.io/upload_images/682074-f9a5d092618548ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面的图解中, 各自的位置都形象的展示了我们在main.m中创建的inventory对象的内存管理调用. 如你所见, 所有的alloc和retain(调用)都有release来平衡, 从而保证真实的内存最终会被释放.
#####The copy Method
相对于retain, 另一个可选方案就是copy, (copy)会创建一个全新的对象实例, 并且增加引用计数, 并且能让原始对象不受影响[此处是相对而言]. 因此, 如果不想引用一个可变的inventory数组, 你可以copy它. 将setInventory: 调整如下:

```
// CarStore.m
- (void)setInventory:(NSMutableArray *)newInventory {
    if (_inventory == newInventory) {
        return;
    }
    NSMutableArray *oldValue = _inventory;
    _inventory = [newInventory copy];
    [oldValue release];
}
```
也许你会想起在[copy Attribute]()中, [this has the added perk of freezing mutable collections at the time of assignment][还没想好咋表达]. 一些类提供了多个copy方法(跟(一些类)提供多个init方法类似), 并且假设任何以copy开头方法都有相同的行为(这种想法)是安全的.

#####The autorelease method
与release类似, autorelease方法也是放弃对一个对象的拥有关系, 但不同的是, 它不会立即销毁该对象, 而是在程序中延缓真正的内存释放. 这样就允许你在应该释放该对象的时候释放它, 但却能仍旧被其他(对象)使用.

举例来说, 利用一个工厂方法来创建并返回一个CarStore对象:

```
// CarStore.h
+ (CarStore *)carStore;
```
技术上讲, 释放这个对象是carStore方法的责任, 因为没有一种方式告知调用者拥有这个返回的对象. 因此, 它的实现应该返回一个autorelease对象, 像这样:

```
// CarStore.m
+ (CarStore *)carStore {
    CarStore *newStore = [[CarStore alloc] init];
    return [newStore autorelease];
}
```
这种方式在创建carStore对象会会立即放弃对该对象的拥有关系, 但是将该对象保存在内存中足够长时间以便与调用者交互. 准确来讲, 该对象会等到在最近的一个@autorelease{}块结尾处, 调用常规的release方法[不显式调用]后再释放. 这就是为什么main()函数总是被@autorelease{}块包围着 - 因为这样能保证所有的autoreleased对象在程序执行完成后都能被释放. 

所有这些内置的工厂方法, 像NSString的stringWithFormt: 和 stringWithContentOfFile: 与我们的carStore方法用完全相同的方式工作. 在ARC之前, 这是一个便利的约定, 因为它能让你在不用担心何时释放对象的情况下使用对象.

如果现在将main()中的superStore构造器从alloc/init调整成下面的方式, 那么你就不用在函数结尾(显式)释放它了.

```
// main.m
CarStore *superstore = [CarStore carStore];
```
实际上, 你现在已不能释放superStore实例了, 因为你不再拥有它 - 而是carStore工厂方法拥有它. 避免显式地释放autoreleased对象很重要(否则, 将会产生悬挂指针, 导致程序崩溃).
#####The dealloc Method
一个对象的dealloc方法与它的init方法刚好相反. 当一个对象被释放之前, 该方法会被合适地调用, 以便给你清理内部对象的机会. 该方法是被runtime自动调用地 - 永远别尝试主动调用.

在MRR环境下, 在dealloc方法中最常做的事情就是释放存储在实例变量中的对象. 想象一下, 当一个(CarStore)实例被重新分配时, 当前的CareStore会发生什么.: 它的被setter方法retain的_inventory实例变量永远没机会被释放了. 这是另一种形式的内存泄露. 我们需要做的就是在CarStore.m中增加一个(准确说是重写)自定义的dealloc来修复这个问题:

```
// CarStore.m
- (void)dealloc {
    [_inventory release];
    [super dealloc];
}
```
注意, 每次都需要调用父类的dealloc来保证父类中的所有实例变量也都被正确地释放了. 作为一个普遍的规则, 为了尽可能的保持自定义的dealloc的简介, 不应该在dealloc中处理可以在其他地方处理的逻辑(问题).
#####MRR总结
概括来讲, 上述就是手动内存管理. 关键就是使用release或者autorelease来平衡每个alloc, retain和copy, 否则, 在你的程序中, 会遇到诸如悬挂指针, 或者某时刻的内存泄露的问题.

请记住, 该部分仅使用了MRR来理解iOS和OS X内存管理的的内部工作原理. 在现实中, 上述的大部分代码都是废弃的, 不过你可能会在比较老的文档中遇到. 像这种明确地得到和放弃(对某个对象的)拥有关系的方式已经完全被ARC取代了, 理解这点很重要.
####Automatic Reference Counting
现在可以把你头脑中的所有关于手动内存管理的内容都忘记了. 自动引用计数跟MRR的效果一样, 但是它能自动为你在(代码中)合适的位置插入(上述的)内存管理方法. 这对OC开发人员来说非常有益, 因为它能让开发人员将注意力放在自身程序的功能而不再操心它(内存管理)怎么实现.

相比较内存管理的人为错误, ARC几乎完美的解决了这个问题, 所以唯一不使用ARC的原因就是你还在与历史遗留代码打交道(但是, ARC, 在大部分情况下, 是向下兼容MRR程序的). 该章剩余部分讲解MRR与ARC之间的重大变化.
#####Enabling ARC
首先, 在项目中Building Settings 中恢复ARC. 将自动引用计数的选项改为YES. 再强调一下, 这是是所有Xcode模板的默认值, 并且这也是你开发所有程序应该使用的.

![Turning on Automatic Reference Counting](http://upload-images.jianshu.io/upload_images/682074-66948807db1f5b92.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#####No More Memory Methods
ARC通过分析你的代码以得知每个对象存在的理想生命周期来工作, 随后自动(在合适的位置)插入必要的retain以及release调用. 该算法需要对对象拥有关系的完全控制, 这意味着在你的程序中, 你不能手动调用retain, release或者autorelease.

在ARC程序中, 唯一涉及内存相关的方法就是alloc和copy. 你可以将这些理解为普通的构造器, 并忽略整个对象拥有关系这些事情.
#####New Property Attributes - 新的属性特性
ARC介绍了@property新的特性. 你应该用strong特性来代替retain, 用weak替代assign. 这些特性都已在[属性](http://www.jianshu.com/p/bccd94066442)章节讨论了.
#####The dealloc Method in ARC
ARC中的dealloc也有点不同. 我们不用再像在[dealloc Method](#The dealloc Method)中那样去释放实例变量 - ARC帮我们做了. 另外, 父类的dealloc方法也是自动调用的, 也不再需要我们做了.

大多数情况下, 我们无需自定义dealloc方法. 其中一个例外的情况就是, 你在使用较低级的(相对高级语言来说, 比如C相对于OC)内存分配函数, 比如[malloc](https://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man3/malloc.3.html). 这种情况下, 仍需要在dealloc中调用free()来避免内存泄露.
####总结
大多数情况下, ARC会让你完全忘记内存管理. (使用ARC)就是不再关注内存管理, 而专注于高级功能上. 唯一你需要操心的就是循环引用的问题, 这个已在[属性](http://www.jianshu.com/p/bccd94066442)章节论述了.

如果你想知道更多关于ARC的细节, 请参阅[Transitioning to ARC Release Notes](https://developer.apple.com/library/mac/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226).

好了, 截止到目前, 所有OC中你应该知道的基本上都学完了. 唯一我们还未涉及的是C和Foundation框架中提供的基本数据类型. 下一章节会介绍所有的标准类型, 从numbers(数值)到strings(字符串), arrays(数组), dictionaries(字典), 甚至dates(日期)等. 
***
写于15年11月13号, 完成于15年12月03号