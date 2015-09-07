来源于 Ry’s Objective-C Tutorial - RyPress

一个学习Objective-C基础知识的网站.
 
个人觉得很棒,所以决定抽时间把章节翻译一下.

本人的英语水平有限,有让大家误解或者迷惑的地方还请指正.
***
方法代表着一个对象如何执行操作.它们与代表着对象数据的属性在逻辑上等同.你可以把方法当做附加给对象的函数,但他们有着不同的语法.

在这个模块,我们将探讨OC方法的命名规约,(这些规约)对有着C++,Java,Python以及其他类似语言的开发经验人员来说,会比较frustrating(难受).我们也会简要的讨论OC的访问修饰符(用来设置访问权限),并学习怎样使用selectors(选择器)关联方法.

####命名规约
OC的方法旨在(被设计成)移除API中任何含糊之处,所以导致了方法名很冗长(啰嗦的吓人),但是不可否认的是,描述性很强.遵守以下三条规则以实现OC方法的命名.
1.不要使用缩写
2.明确规定方法自身的参数名
3.明确描述方法的返回值
在你阅读下面的Car类接口时把这些规则记在心里.

``` 
// Car.h
#import <Foundation/Foundation.h>

@interface Car : NSObject

// Accessors
- (BOOL)isRunning;
- (void)setRunning:(BOOL)running;
- (NSString *)model;
- (void)setModel:(NSString *)model;

// Calculated values
- (double)maximumSpeed;
- (double)maximumSpeedUsingLocale:(NSLocale *)locale;

// Action methods
- (void)startEngine;
- (void)driveForDistance:(double)theDistance;
- (void)driveFromOrigin:(id)theOrigin toDestination:(id)theDestination;
- (void)turnByAngle:(double)theAngle;
- (void)turnToAngle:(double)theAngle;

// Error handling methods
- (BOOL)loadPassenger:(id)aPassenger error:(NSError **)error;

// Constructor methods
- (id)initWithModel:(NSString *)aModel;
- (id)initWithModel:(NSString *)aModel mileage:(double)theMileage;

// Comparison methods
- (BOOL)isEqualToCar:(Car *)anotherCar;
- (Car *)fasterCar:(Car *)anotherCar;
- (Car *)slowerCar:(Car *)anotherCar;

// Factory methods
+ (Car *)car;
+ (Car *)carWithModel:(NSString *)aModel;
+ (Car *)carWithModel:(NSString *)aModel mileage:(double)theMileage;

// Singleton methods
+ (Car *)sharedCar;

@end
```
#####缩写
让方法能够被理解,被预料(可以被猜测)的最简单方式就是避免缩写.大多数OC编程人员都希望方法能被完全写出,这是所有标准框架里的规约,从[Foundation](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/ _classic/_index.html)到[UIKit](http://developer.apple.com/library/ios/#documentation/uikit/reference/UIKit_Framework/_index.html)都是这样.这就是上述接口中选择使用maximumSpeed取代更简写的maxSpeed的原因.

#####参数
另一个清楚(反映)OC冗长设计哲学的例子就是方法参数的命名规约.C++,Java以及其他类似风格的语言中,将参数与方法分离成单独的实体,而OC的方法名实则包括它所有的参数名.

举个例子,在C++中为了让Car(方向)转90°,你也许会使用这样turn(90)的方法.但是,OC觉着这太含糊了.搞不清楚turn()使用什么样的参数-可能是一个新的方向角,也可能是在调整(增加)你当前的方向角.OC方法通过一个介词来描述让它明确.这种方式实现的API保证了方法永远不会被误用:它(方法)要么是turnByAngle:90.要么是turnToAngle:90.

当一个方法可接受多个参数时,那么每个参数名都被包含在方法名中.上面的例子中,initWithModel:mileage:方法把Model和mileage都带上了.正如等会我们所见,这使得方法调用非常明确.

#####返回值
你肯定也注意到任何方法都返回一个明确状态的值.有时,会简单到只要说明返回的类型(class)就行,但其他时候,你需要增加一个形容词做前缀.

比如,(上述的)工厂方法以car开头,它明确说明该方法返回一个Car类型的实例.(再比如)那两个比较方法,fasterCar:与slowerCar:返回接受者以及参数中的faster/slower,API也会被描述的很清楚.但对于单类方法,遵守这种模式没有任何意义(比如,sharedCar),因为这种规约的实例方法名本身就是含糊的.

更多关于命名的规约,请方法官方的[Cocoa Coding Guidlines](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html)文档
####方法调用
正如在实例化与使用章节讨论过地,你通过在方括号中放置用空格分开的对象以及所需方法来执行方法.参数与方法名用冒号分隔.

``` 
[porsche initWithModel:@"Porsche"];
```
当有多个参数时,则按照下面的方式,跟在初始参数之后.每个参数都有对应的标签,并且需要用空格分隔,然后再跟上冒号:

``` 
[porsche initWithModel:@"Porsche" mileage:42000.0];
```
当你站在(方法)调用角度去看,就应该更容易理解上述命名规约的目的了.它们(命名规约)是的方法调用读起来像是人类语言,而非机器语言.比如,比较下面OC与其他语言的类似的方法调用:

```
// Python/Java/C++
porsche.drive("Home", "Airport");

// Objective-C
[porsche driveFromOrigin:@"Home" toDestination:@"Airport"];
```
这可能会有更多的种类(冗长的方法名),所以Xcode 带有如此好的自动补全功能.当你离开(遗忘)你的代码,但是几个月后再回来修复 bug 时,你就会对这种冗长(的命名)感激万分.这种清晰(冗长的方法名)也使得使用第三方库以及维护大型代码库变得更容易.
#####方法嵌套调用
嵌套调用方法是OC程序中常见的方式.把一个(方法)调用结果传递给另一个是很自然的.概念上来说,它们跟方法链(调用)完全一样,只是方括号语法让它们看起来有些不同罢了.

```
// JavaScript
Car.alloc().init()

// Objective-C
[[Car alloc] init];
```
[Car alloc]先被执行,然后init方法被其返回值调用.

####受保护以及私有方法
对于OC方法来说,没有受保护或者私有的访问修饰符-它们都是公有的.然而,OC提供了一种可替代的组织范例来让你实现等同的效果.

"私有"方法可以通过在实现文件中定义,但是在接口文件中省略来创建.因为其他对象(包括子类)永远不准导入实现文件,所以,除了(定义这些方法的)类本身,这些方法有效地对其他对象隐藏了一切.

作为受保护方法的替代,OC提供了分类这种对隔离API更普遍的(大众的)解决方案.我们会在Protected Methods中看到一个(关于分类的)完整的例子.

####选择器

选择器是方法名在OC内部的表示.它们允许你将一个方法当做独立的实体,从而你可以将(方法的)行为与需要执行(该行为)的对象分离.这是目标-行为设计模式的基础,这在[Ry's Cocoa Tutorial](http://rypress.com/tutorials/cocoa)的[Interface Builder](http://rypress.com/tutorials/cocoa/interface-builder)章节中有介绍.它也是OC动态类型机制的组成部分.

可以通过两种方式来得到方法名对应的选择器.@selector()指令,可以将源代码中的方法名转成选择器,而NSSelectorFromString()函数则可以将一个字符转成选择器(这个没有前者高效).这两种方式都返回一个称作SEL的特殊数据类型.SEL的使用与BOOL,int或者其他数据类型完全一样.

``` 
// main.m
#import <Foundation/Foundation.h>
#import "Car.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Car *porsche = [[Car alloc] init];
        porsche.model = @"Porsche 911 Carrera";
        
        SEL stepOne = NSSelectorFromString(@"startEngine");
        SEL stepTwo = @selector(driveForDistance:);
        SEL stepThree = @selector(turnByAngle:quickly:);
        
        // This is the same as:
        // [porsche startEngine];
        [porsche performSelector:stepOne];

        // This is the same as:
        // [porsche driveForDistance:[NSNumber numberWithDouble:5.7]];
        [porsche performSelector:stepTwo
                      withObject:[NSNumber numberWithDouble:5.7]];
        
        if ([porsche respondsToSelector:stepThree]) {
            // This is the same as:
            // [porsche turnByAngle:[NSNumber numberWithDouble:90.0]
            //              quickly:[NSNumber numberWithBool:YES]];
            [porsche performSelector:stepThree
                          withObject:[NSNumber numberWithDouble:90.0]
                          withObject:[NSNumber numberWithBool:YES]];
        }
        NSLog(@"Step one: %@", NSStringFromSelector(stepOne));
    }
    return 0;
}
```
任何对象都可以通过performSelector:以及(其他)相关的方法来执行选择器.withObject:版本(方法)允许你给方法传参数,但是这些参数必须是对象.如果这对你的需求有太多限制,你可以查看[NSInvocation](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSInvocation_Class/Reference/Reference.html)的高级用法.当你不确定目标对象是否已定义了这个方法,那你应该在尝试执行选择器之前使用respondsToSelector:进行检查.

方法的术语名是串联所有用冒号分隔参数标签得到的基本方法名(The technical name for a method is the primary method name concatenated with all of its parameter labels, separated by colons.**这句翻译的太难受了,自己理解吧**).让冒号成为方法名的一部分可能会让刚学OC的人感到迷惑.它们的用法可以归纳如下:无参方法永远不包括冒号,而有参的方法永远都是以冒号结尾.

下面是一个关于上述Car类的接口和实现的例子.请注意我们必须给performSelector:withObject:方法的参数传递[NSNumber](http://rypress.com/tutorials/objective-c/data-types/nsnumber.html),而不是传递double(类型),因为它不允许你传递C语言的基本数据类型.

``` 
// Car.h
#import <Foundation/Foundation.h>

@interface Car : NSObject

@property (copy) NSString *model;

- (void)startEngine;
- (void)driveForDistance:(NSNumber *)theDistance;
- (void)turnByAngle:(NSNumber *)theAngle
            quickly:(NSNumber *)useParkingBrake;

@end
```

``` 
// Car.m
#import "Car.h"

@implementation Car

@synthesize model = _model;

- (void)startEngine {
    NSLog(@"Starting the %@'s engine", _model);
}

- (void)driveForDistance:(NSNumber *)theDistance {
    NSLog(@"The %@ just drove %0.1f miles",
          _model, [theDistance doubleValue]);
}

- (void)turnByAngle:(NSNumber *)theAngle
            quickly:(NSNumber *)useParkingBrake {
    if ([useParkingBrake boolValue]) {
        NSLog(@"The %@ is drifting around the corner!", _model);
    } else {
        NSLog(@"The %@ is making a gentle %0.1f degree turn",
              _model, [theAngle doubleValue]);
    }
}

@end
```
####总结
这个模块解释了OC方法命名规约背后的原因,我们也知晓了OC方法中没有访问修饰符,知晓怎样利用@selector去动态地执行方法.

适应一种新的规约会是一个痛苦的(难受的)过程,并且这种OC与其他OOP语言的戏剧性语法差异不会让你觉得轻松( and the dramatic syntactic differences between Objective-C and other OOP languages won’t make your life any easier.**又是忧伤的一句**).不要强迫自己把OC语言加到你头脑中已存在的编程模型世界中,而是要去理解内在的本质(Instead of forcing Objective-C into your existing mental model of the programming universe, it helps to approach it in its own right. **更忧伤了**).尝试设计一些简单的程序之后,再给OC这种冗长的(设计)哲学下判断下判断.

那些(前些章节)涵盖了OC面向对象编程的基础(知识).这部教程的剩余内容会探讨更牛逼的方式来组织代码.首先列出的是协议,它允许在多个类之间共享一个API.
***
写于15年09月07号