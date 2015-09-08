来源于 Ry’s Objective-C Tutorial - RyPress

一个学习Objective-C基础知识的网站.

个人觉得很棒,所以决定抽时间把章节翻译一下.

本人的英语水平有限,有让大家误解或者迷惑的地方还请指正.
***
协议是一组可以让任何类实现的相关属性和方法.它们比一个普通的类接口更灵活,因为它们可以让你在不相关的类之间重用一个API.这就让*表示现有类之间的横向关系*成为可能.
![Unrelated classes adopting the StreetLegal protocol](http://rypress.com/tutorials/objective-c/media/protocols/protocol-overview.png)

这只是涵盖协议基础知识中的一部分(内容).我们将会学习协议是如何与OC的动态机制相匹配的.
####创建协议
步骤略.

创建名为StreetLegal的协议.
该协议约束了合法交通工具的必须行为.在协议中定义这些特性(属性或者方法)让你可以将它们应用到任意类,而不是必须从同样地抽象父类继承(才能实现这个目标).
StreetLegal协议的一个简单版本内容像下面这样:

```
// StreetLegal.h
#import <Foundation/Foundation.h>

@protocol StreetLegal <NSObject>

- (void)signalStop;
- (void)signalLeftTurn;
- (void)signalRightTurn;

@end
```
任何遵守该协议的类都必须得保证实现它的所有方法.在协议名之后的<NSObject>是表明StreetLegal Protocol包含[NSObject 协议](http://developer.apple.com/library/ios/#documentation/cocoa/reference/foundation/Protocols/NSObject_Protocol/Reference/NSObject.html)(不要与[Object类](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/nsobject_Class/Reference/Reference.html)混淆了).实际上,任何一个遵守StreetLegal协议的类都被要求遵守NSObject协议.
####遵守协议
一个类可以通过在其类名/超类名之后添加尖括号(在尖括号内写上协议名)来遵守上述(协议中)的API.创建一个Bicycle类,并修改如下.请注意,在你使用协议之前也需要先将其导入.

```
// Bicycle.h
#import <Foundation/Foundation.h>
#import "StreetLegal.h"

@interface Bicycle : NSObject <StreetLegal>

- (void)startPedaling;
- (void)removeFrontWheel;
- (void)lockToStructure:(id)theStructure;

@end
```
遵守协议就像是把StreetLegal.h中的所有方法加到Bicycle.h中.这与Bicycle继承自一个超类的效果完全一样.可以通过用逗号分隔来遵守多个协议(比如,<StreetLegal, SomeOtherProtocol>).

Bicycle的实现没什么特别-仅需保证Bicycle.h与StreetLegal.h中的方法全部被实现了就行.

```
// Bicycle.m
#import "Bicycle.h"

@implementation Bicycle

- (void)signalStop {
    NSLog(@"Bending left arm downwards");
}
- (void)signalLeftTurn {
    NSLog(@"Extending left arm outwards");
}
- (void)signalRightTurn {
    NSLog(@"Bending left arm upwards");
}
- (void)startPedaling {
    NSLog(@"Here we go!");
}
- (void)removeFrontWheel {
    NSLog(@"Front wheel is off."
          "Should probably replace that before pedaling...");
}
- (void)lockToStructure:(id)theStructure {
    NSLog(@"Locked to structure. Don't forget the combination!");
}

@end
```
现在,当你使用Bicycle类时,你就能假定它会对协议中定义的API做出响应了.感觉就像signalStop,signalLeftTurn, and signalRightTurn (这些方法)都是在Bicycle.h中声明地.

```
// main.m
#import <Foundation/Foundation.h>
#import "Bicycle.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Bicycle *bike = [[Bicycle alloc] init];
        [bike startPedaling];
        [bike signalLeftTurn];
        [bike signalStop];
        [bike lockToStructure:nil];
    }
    return 0;
}
```
####使用协议进行类型检查
跟类一样,协议也能用于变量类型检查.为了保证一个对象遵守某个协议,可以在变量声明时将协议名写在数据类型后面,如下所示.下面的代码段假设你已经创建好了一个遵守StreetLegal协议的Car类:

```
// main.m
#import <Foundation/Foundation.h>
#import "Bicycle.h"
#import "Car.h"
#import "StreetLegal.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        id <StreetLegal> mysteryVehicle = [[Car alloc] init];
        [mysteryVehicle signalLeftTurn];
        
        mysteryVehicle = [[Bicycle alloc] init];
        [mysteryVehicle signalLeftTurn];
    }
    return 0;
}
```
即使Car与Bicycle都集成自相同的超类也没关系-实际上,它们都遵守StreetLegal协议.所以我们可以通过声明id<StreetLegal>变量来存储它们.这是一个展示无关类之间如何使用公有功能的例子.

也可以使用NSObject协议中定义的conformToProtocol:方法来检查对象是否遵守某个协议.它将一个协议对象当做一个参数,该参数可以通过@protocol()指令获得.这与@selector()指令很类似,但是你需要传递的是协议名而不是方法名,如下所示:

```
if ([mysteryVehicle conformsToProtocol:@protocol(StreetLegal)]) {
    [mysteryVehicle signalStop];
    [mysteryVehicle signalLeftTurn];
    [mysteryVehicle signalRightTurn];
}
```
这种使用协议的方式就像是在说,"保证这个对象有这组特殊的功能."对动态类型(机制)来说,这是一个强大的工具,因为它能让你在不用操心跟什么类型的对象打交道的情况下,使用一个定义良好的API.
####现实世界(OC)中的协议
在你日常的iOS与OS X开发中,你会遇到更实际使用(协议)的情况.任何一个app的入口都是"application delegate(应用程序代理)",一个能处理程序生命周期中重要事件的对象.[UIKit Framework]()仅仅让你遵守一个协议,而不是强迫(要求)这个delegate继承某个特殊的超类.

```
@interface YourAppDelegate : UIResponder <UIApplicationDelegate>
```
任何一个能响应[UIApplicationDelegate](https://developer.apple.com/library/ios/documentation/uikit/reference/uiapplicationdelegate_protocol/Reference/Reference.html)中定义的方法的对象都能作为应用程序代理.当重新组织应用程序(代码)时,通过协议替代子类(继承)来实现代理设计模式给了开发人员更多回旋余地(更容易解耦).

你可以在[Ry's Cocoa Tutorial](http://rypress.com/tutorials/cocoa)的[Interface Builder](http://rypress.com/tutorials/cocoa/interface-builder)章节查看具体的例子.它使用项目中默认的app代理去响应用户输入.
####总结
这个模块中,我们(在我们已收集的东西)添加了另一个组织(代码)工具.协议是一个让专有文件能抽象共享属性和方法的途径.这有助于减少冗余代码,并允许你动态地检查一个对象是否支持任意一组功能.

在Cocoa 框架中你会发现很多协议.一个常用的情况是,让你不用子类化某些类(的情况下)就可修改它们的行为.比如,[Table View],[Outline View]以及[Collection View]这些UI组件都使用数据源(data source)以及代理对象去设定它们内部的行为.数据源和代理都被定义成协议,因此,可以在任何对象中实现必要的方法.

下个模块将会介绍分类.
(which are a flexible option for modularizing classes and providing opt-in support for an API.)**大致意思:是实现类模块化以及给API提供可选支持的弹性(灵活)选择**
***
写于15年09月08号
