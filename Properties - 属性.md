来源于 Ry’s Objective-C Tutorial - RyPress

一个学习Objective-C基础知识的网站.

个人觉得很棒,所以决定抽时间把章节翻译一下.

本人的英语水平有限,有让大家误解或者迷惑的地方还请指正.
***
一个对象的属性允许其他对象查看或者修改它的状态.但是,在一个良好的面向对象设计中,是不可能(允许)直接获取一个对象的内部状态.而是通过存取器(getters与setters)与对象的内部数据进行交互.

![Interacting with a property via accessor methods](http://rypress.com/tutorials/objective-c/media/properties/accessor-methods.png) 

@property指令的目的是通过自动生成存取器方法来方便的创建,配置属性.它允许你在语义上指定公有属性的行为,并且不用你操心实现细节.

这个模块概览这些不同的允许你变更getters与setters行为的特性.其中的一些特性决定了属性如何控制(管理)自己内存,所以该模块也是对OC内存管理的一个实用介绍.对于(这方面)更详细的讨论,请参阅[内存管理](http://rypress.com/tutorials/objective-c/memory-management.html).

####@property指令
首先,让我们先看一下当使用@property指令时到底发生了什么事情.请看下面的一个简单Car类的接口和对应实现.

``` 
// Car.h
#import <Foundation/Foundation.h>

@interface Car : NSObject

@property BOOL running;

@end
```

``` 
// Car.m
#import "Car.h"

@implementation Car

@synthesize running = _running;    // Optional for Xcode 4.4+

@end
```
编译器为running属性(分别)生成了getter和setter.默认的命名规约是使用其属性名称作为getter,在属性名之前加上set作为setter,其对应的实例变量则是在属性名前加上下划线,就像这样:

``` 
- (BOOL)running {
    return _running;
}
- (void)setRunning:(BOOL)newValue {
    _running = newValue;
}
```
在你通过@property指令声明了属性之后,你就能调用这些方法,好像(这些方法)已经被包含在你的类接口和实现文件中.你也可以在Car.m中重写它们以提供自定义的getter/setter,这样的话就得必须使用@synthesize指令.然而,你应该没有必要要自定义存取器,尽管@property指令允许在抽象的层级上这样做.

通过点语法访问属性实际上是转换成通过上述的存取器方法,所以下面(代码块)的honda.running代码在给其赋值时实际上是调用setRunning:在读取它的值时是调用running方法:

``` 
// main.m
#import <Foundation/Foundation.h>
#import "Car.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Car *honda = [[Car alloc] init];
        honda.running = YES;                // [honda setRunning:YES]
        NSLog(@"%d", honda.running);        // [honda running]
    }
    return 0;
}
```
为了更改已生成的存取器的行为,你可以在@property指令后的小括号内指定其特性.下面的模块将介绍这些可用的特性.
#### getter= 与 setter= 特性
如果你不喜欢@property这种默认的命名规约,你可以使用getter=与setter=特性来更改getter/setter的方法名.对于Boolean属性,常用的getter方法名是(在其属性名)添加is前缀.将Car.h的属性声明调整成下面这样:

``` 
@property (getter=isRunning) BOOL running;
```
此时生成的存取器就是isRunning以及setRunning了.注意,公有属性(名称)仍然是running,这也是你在点语法中该使用的名称:

``` 
Car *honda = [[Car alloc] init];
honda.running = YES;                // [honda setRunning:YES]
NSLog(@"%d", honda.running);        // [honda isRunning]
NSLog(@"%d", [honda running]);      // Error: method no longer exists
```
这是唯一一个需要参数的属性特性(用于标识存取器方法) - 其他的都是boolean标识.
####只读特性
只读特性让你方便地将一个属性变成只读的.它会忽略setter方法,并会阻止通过点语法(给其)赋值,但是对getter没有影响.作为例子,将Car接口修改如下,注意我们怎么使用逗号分隔来给属性指定多个特性.

``` 
#import <Foundation/Foundation.h>

@interface Car : NSObject

@property (getter=isRunning, readonly) BOOL running;

- (void)startEngine;
- (void)stopEngine;

@end
```
现在,我们将通过startEngine和stopEngine方法在内部设置running属性,而不是让其他对象修改.对应的实现如下.

``` 
// Car.m
#import "Car.h"

@implementation Car

- (void)startEngine {
    _running = YES;
}
- (void)stopEngine {
    _running = NO;
}

@end
```
请记住,@property也为我们生成了一个实例变量,这是我们为什么在不声明的情况下,也可以使用_running的原因(在此处,我们不能使用self.running因为它是只读的).我们在main.m中添加如下代码来测试新的Car类.

``` 
Car *honda = [[Car alloc] init];
[honda startEngine];
NSLog(@"Running: %d", honda.running);
honda.running = NO;                      // Error: read-only property
```
到此(来看),属性是一个避免我们自己(根据样板)写getter/setter方法的快捷方式.(与readonly相比),这种情况不适用于其余的那些能显著变更属性行为的特性.它们只适用于存储OC对象的属性(并非C语言的基本类型).

####非原子特性
原子性必须处理线程环境下的属性行为.当有多个线程时,有可能会同时调用setter与getter方法.这也就意味着getter/setter会被其他操作打断,也就可能会产生坏数据.

原子属性通过锁定一个基础的(普通的)对象来阻止这种事情的发生,并保证get/set操作的完整性(数据完整性).然而,使用原子属性只是线程安全的一个方面,并不意味着你的代码是线程安全的,理解这一点很重要.

@property声明的属性默认是原子特性,这会产生一些管理成本(开销).所以,如果处在一个非多线程的环境(或者你自己实现了线程安全),你肯定会用nonatomic特性来重写(变更)这个行为,像这样:

``` 
@property (nonatomic) NSString *model;
```
这有一个关于atomic特性小的,实用的警告.对于具有原子特性的属性,它的存取器要么自己生成,要么自定义.只有non-atomic特性的属性才允许混合-既可使用synthesized,又可以自定义.可以通过移除上面代码中的nonatomic,并在Car.m增加一个自定义的getter来验证这一点.
####内存管理
在任何一种OOP(objected-oriented Pragram)语言中,对象都驻留在电脑内存中,在移动设备上- 这些(内存)是稀缺资源.内存管理的目标通过一种高效的创建,销毁对象的方式来保证程序不会占用超出它们所需(内存)空间的额外空间.

很多编程语言通过垃圾回收机制来实现(内存管理),而OC使用另一种更高效的,被称作object ownership的机制.当你与一个对象开始进行交互时,你便拥有了那个对象,也就意味着这种关系(拥有该对象)能保证在你使用期间,该对象一直存在.当你不在使用,你便打破(放弃)这种拥有关系,此时,如果该对象没有其他拥有者,系统便会销毁该对象,并释放其内存.

![Destroying an object with no owners](http://rypress.com/tutorials/objective-c/media/properties/object-ownership.png)

随着自动引用计数的出现,编译器会帮自动给帮你管理这些拥有关系.意味着大多数情况下,你都不用操心内存管理如何工作.但是,你还是都明白属性的strong,weak与copy特性,因为它们告诉编译器这种对象关系属于哪一种.
#####strong特性
strong特性下,无论给属性分配什么样的对象,都会创建一个拥有关系.这对所有的对象属性来说都是明确地行为,这样就保证了只要属性被赋值,那值就会存储(存在).

我们创建另一个Person类来看一下上诉的工作原理.该类的接口仅声明了一个name属性:

``` 
// Person.h
#import <Foundation/Foundation.h>

@interface Person : NSObject

@property (nonatomic) NSString *name;

@end
```
下面是实现.它使用@property默认生成的存取器.还重写了NSObject的description方法,该方法是用来返回对象的字符串描述.

``` 
// Person.m
#import "Person.h"

@implementation Person

- (NSString *)description {
    return self.name;
}

@end
```
接下来,我们在Car类中添加一个Person属性,修改Car.h如下.

``` 
// Car.h
#import <Foundation/Foundation.h>
#import "Person.h"

@interface Car : NSObject

@property (nonatomic) NSString *model;
@property (nonatomic, strong) Person *driver;

@end
```
然后,调整(又一次出现的)main.m的代码:

``` 
// main.m
#import <Foundation/Foundation.h>
#import "Car.h"
#import "Person.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *john = [[Person alloc] init];
        john.name = @"John";
        
        Car *honda = [[Car alloc] init];
        honda.model = @"Honda Civic";
        honda.driver = john;
        
        NSLog(@"%@ is driving the %@", honda.driver, honda.model);
    }
    return 0;
}
```
因为driver是一个strong关系(强引用),所以handa对象拥有john.这样就保证了john(对象)只要在honda对象需要,它就一直有效(存在).

#####weak特性
大多数情况,在设置对象属性时,第一直觉都是给它strong特性.然而,strong引用(在一些情况下)会引起一个问题.举个例子,我们需要一个对Car对象的引用来标识人正在开车.首先,我们在Person.h中增加一个car属性.

``` 
// Person.h
#import <Foundation/Foundation.h>

@class Car;

@interface Person : NSObject

@property (nonatomic) NSString *name;
@property (nonatomic, strong) Car *car;

@end
```
@class Car这一行是对Car类的向前声明.就像是在告诉编译器,"相信我,Car类肯定存在,不用现在去加载它."我们必须用这种方式来替代我们常用的#import语句,因为Car也可以导入Person.h,这样我们就会陷入一个无尽的导入循环.(编译器可不喜欢无尽的循环.)

接下来,在main.m中,给honda.driver分配值之后加入下面一行代码:

``` 
honda.driver = john;
john.car = honda;       // Add this line
```
现在,我们有一个honda拥有john的关系,也有一个jhon拥有honda的关系.这意味着两个对象相互拥有(引用),导致内存管理系统无法销毁它们,即使这两个对象不再需要了.

![A retain cycle between the Car and Person classes](http://rypress.com/tutorials/objective-c/media/properties/retain-cycle.png)

这被称作是**retain cycle**,一种内存泄露的形式,内存泄露**are bad**.幸运的是,很容易修复这个问题-只要告诉其中的一个属性对另外一个对象保持一个**弱引用**即可.在Person.h中,将car的声明调整如下:

``` 
@property (nonatomic, weak) Car *car;
```
weak特性创建一个对car的非拥有关系.这样就允许john在避免retain cycle的情况下,仍存在一个对honda的引用.这意味,很可能存在(下述的)这种情况,honda已经被销毁了,而john仍有一个对honda的引用.**这不应该发生**,weak特性会设置car为nil以避免指针悬挂.

![A weak reference from the Person class to Car](http://rypress.com/tutorials/objective-c/media/properties/weak-reference.png)

一个常用weak特性的情况是在父子数据结构中.按照规约,父对象应该保持对子对象的强引用,子对象则保持对父对象的弱引用.弱引用也是代理设计模式中与生俱来部分.

(上述的)关键点在于,两个对象永远不要互相强引用.weak特性让在不产生retain cycle的情况下保持一个循环引用的关系成为可能.
#####copy特性
相对于strong特性,copy特性可以作为替代(品).它不形成一个拥有关系,而是对任何分配给属性的对象创建一个副本,然后对副本有拥有关系.只有遵从[NSCopying 协议](https://developer.apple.com/library/ios/documentation/cocoa/Reference/Foundation/Protocols/NSCopying_Protocol/Reference/Reference.html#//apple_ref/occ/intf/NSCopying)的对象才可以使用这个特性.

Properties that represent values (opposed to connections or relationships) are good candidates for copying. 比如,开发人员一般都会复制NSString属性,而不是强引用它们:

``` 
// Car.h
@property (nonatomic, copy) NSString *model;
```
现在,无论你给model赋什么值,Car都将存储一个崭新的实例.如果你使用的是可变值,那么当(属性)第一次被赋值时,它便是不变的了.* If you’re working with mutable values, this has the added perk of freezing the object at whatever value it had when it was assigned.*.可以通过下面的代码证明:

``` 
// main.m
#import <Foundation/Foundation.h>
#import "Car.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Car *honda = [[Car alloc] init];
        NSMutableString *model = [NSMutableString stringWithString:@"Honda Civic"];
        honda.model = model;
        
        NSLog(@"%@", honda.model);
        [model setString:@"Nissa Versa"];
        NSLog(@"%@", honda.model);            // Still "Honda Civic"
    }
    return 0;
}
```
NSMutableString是NSString的一个子类,可以随时编辑.如果model没有对原始的实例创建一个副本,我们将会看到第二行NSLog()输入的是改变后的字符串(Nissan Versa) .
####其他特性
对于最新的OC程序(iOS5+),上述的@property特性是你应该需要(使用)的,但是这仍然有一些其他的(特性),你可能会在比较旧的库或者文档中遇到.
#####retain特性
retain特性是strong特性在手动引用计数中的版本,它有着同样地效果:分配值时要求一个拥有关系.在自动引用计数环境下,你不应该再使用它.
#####unsafe_unretained 特性
有着unsafe_unretained特性的属性在行为上与(有着)weak特性的属性类似,但是如果被引用的对象被销毁后,它们的值不会自动设为nil.你应该使用unsafe_unretained特性的唯一原因就是为了让你的类与不支持weak属性的代码兼容.
#####assign特性
assign特性在给属性赋值时不会执行任何一种内存管理要求.它是基本数据类型的默认行为,并且它曾是在iOS5之前实现弱引用的一种方式.与retain一样,在最新的(当代)应用程序你不需要明确地使用它.
####总结
这个模块展示了@property所有的供选择的特性,希望你对修改已生成的存取器方法感觉相对适应.请记住,这些特性的目的是帮助你关注什么样的数据应该被记录.(这些)特性被总结如下.

特性 | 描述 
------------ | -------------
Content Cell | Content Cell
getter= | Use a custom name for the getter method.
setter= | Use a custom name for the setter method.
readonly |	Don’t synthesize a setter method.
nonatomic | Don’t guarantee the integrity of accessors in a multi-threaded environment. This is more efficient than the default atomic behavior.
strong | Create an owning relationship between the property and the assigned value. This is the default for object properties.
weak | Create a non-owning relationship between the property and the assigned value. Use this to prevent retain cycles.
copy | Create a copy of the assigned value instead of referencing the existing instance.

现在,我们已经搞定了属性,我们将深入了解OC类中的另一半:方法.我们将探讨(与方法相关的)一切,从命名规约背后的怪癖(原因)到动态方法调用.
***
写于15年09月06号

