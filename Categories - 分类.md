来源于 Ry’s Objective-C Tutorial - RyPress

一个学习Objective-C基础知识的网站.

个人觉得很棒,所以决定抽时间把章节翻译一下.

本人的英语水平有限,有让大家误解或者迷惑的地方还请指正.
***
分类是一个能让类的定义分到多个文件中.这样做的目的是为了减轻(将类模块化时需要)维护大量代码的压力.从而避免你的源代码变成一个有10000+代码行的文件,因为这样的文件肯定很难操控(导航),而且对独立的开发人员来说,如果想定义一个类中的一些特殊的,良好定义的部分也是很困难的.

![Using multiple files to implement a class](http://rypress.com/tutorials/objective-c/media/categories/category-overview.png)

在这个模块,我们将会在不碰源文件的情况下,使用分类来对已有的类进行扩展.同样地,我们也将看到这样的功能是怎样被用于实现(模拟)受保护方法的.Extensions跟分类关系很大(很类似),我们也会简单的学一下.

####Setting Up
在我们开始实践分类之前,我们需要一个有效的基础类.创建一个或者修改已存在Car类接口,如下:

```
// Car.h
#import <Foundation/Foundation.h>

@interface Car : NSObject

@property (copy) NSString *model;
@property (readonly) double odometer;

- (void)startEngine;
- (void)drive;
- (void)turnLeft;
- (void)turnRight;

@end
```
对应的实现文件仅仅输出一些描述信息,以便我们知道是在调用不同的方法.

```
// Car.m
#import "Car.h"

@implementation Car

@synthesize model = _model;

- (void)startEngine {
    NSLog(@"Starting the %@'s engine", _model);
}
- (void)drive {
    NSLog(@"The %@ is now driving", _model);
}
- (void)turnLeft {
    NSLog(@"The %@ is turning left", _model);
}
- (void)turnRight {
    NSLog(@"The %@ is turning right", _model);
}

@end
```
现在,假设你想增加另一组有关Car维修的方法.那么你可以在一个专用的分类中添加这些新方法,而不是继续往原有的Car.h与Car.m里面塞.
####创建分类
略.
分类命名的唯一要求就是,当基于同一个类创建分类时,不能与其他分类名冲突(相同).公认的文件命名规约是使用加号隔开类名与分类名,所以你在保存上述创建的分类后,你会在Xcode's 的项目导航看到一个Car+Maintenance.h和一个Car+Maintenance.m文件.

正如你在Car+Maintenance.h中所见,一个分类的接口看起来完全像一个普通类接口,除了在类名之后跟了一个带括号的分类名.(现在),让我们继续给分类添加一些方法.

```
// Car+Maintenance.m
#import "Car+Maintenance.h"

@implementation Car (Maintenance)

- (BOOL)needsOilChange {
    return YES;
}
- (void)changeOil {
    NSLog(@"Changing oil for the %@", [self model]);
}
- (void)rotateTires {
    NSLog(@"Rotating tires for the %@", [self model]);
}
- (void)jumpBatteryUsingCar:(Car *)anotherCar {
    NSLog(@"Jumped the %@ with a %@", [self model], [anotherCar model]);
}

@end
```
在运行时,这些方法则会成为Car类的一部分.尽管它们被声明在一个不同的文件中,但你仍可以访问它们,好像它们(这些方法)就是在Car.h原始文件中声明地.

当然,你必对给这些分类接口中方法进行实现,以便它们能处理事情.分类的实现跟一个标准的类实现基本上一样,除了分类的名称是放在类名之后的括号内的.

```
// Car+Maintenance.m
#import "Car+Maintenance.h"

@implementation Car (Maintenance)

- (BOOL)needsOilChange {
    return YES;
}
- (void)changeOil {
    NSLog(@"Changing oil for the %@", [self model]);
}
- (void)rotateTires {
    NSLog(@"Rotating tires for the %@", [self model]);
}
- (void)jumpBatteryUsingCar:(Car *)anotherCar {
    NSLog(@"Jumped the %@ with a %@", [self model], [anotherCar model]);
}

@end
```
请注意很重要的一点,一个分类也可以被用来重写在基类中的方法(比如,Car类中的drive方法),但你永远都不应该这么做.问题在于分类是一个平行的结构.如果你在Car+Maintenance.m重写了已存在的方法,随后你却要在另一个分类中变更它(已存在的这个方法)的行为,这对OC来说,无法识别该使用哪个实现.这种情况下,使用子类往往是更好的选择.
####使用分类

任何需要使用一个分类中定义的API的文件都需要导入分类的头文件,这与一个普通类的使用相同.在你导入Car+Maintenance.h后,它所有的方法都能直接通过Car类使用.

```
// main.m
#import <Foundation/Foundation.h>
#import "Car.h"
#import "Car+Maintenance.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Car *porsche = [[Car alloc] init];
        porsche.model = @"Porsche 911 Turbo";
        Car *ford = [[Car alloc] init];
        ford.model = @"Ford F-150";
        
        // "Standard" functionality from Car.h
        [porsche startEngine];
        [porsche drive];
        [porsche turnLeft];
        [porsche turnRight];
        
        // Additional methods from Car+Maintenance.h
        if ([porsche needsOilChange]) {
            [porsche changeOil];
        }
        [porsche rotateTires];
        [porsche jumpBatteryUsingCar:ford];
    }
    return 0;
}
```
如果你移除了Car+Maintenance.h这个导入语句,那么Car类将恢复到它的原始状态,编译器也会抱怨说,来自Maintenance分类的needsOilChange,changeOil以及其他方法都不存在.

####"受保护"方法
分类不仅仅用来将类的定义分在几个文件而已.它们也是一个强大的组织(代码的)工具.(这个工具)通过导入分类来将一个API的指定的一部分(方法)成为任意类中的方法,而对其他的类文件来说(除了当前的这个这个导入分类的文件),这些API仍保持隐藏(状态).

回忆一下Methods模块中描述的,OC中是不存在受保护的方法的.然而,可以通过分类的导入来实现与受保护访问修饰符等同的效果.就是在一个专用的分类中定义一个受保护的API,然后只把它导入它的子类实现中.这就使得这个受保护的方法对子类有效(可用),而保持对程序中其他部分的隐藏.比如:

```
// Car+Protected.h
#import "Car.h"

@interface Car (Protected)

- (void)prepareToDrive;

@end
```

```
// Car+Protected.m
#import "Car+Protected.h"

@implementation Car (Protected)

- (void)prepareToDrive {
    NSLog(@"Doing some internal work to get the %@ ready to drive",
          [self model]);
}

@end
```

上述这个Protected分类定义了一个让Car以及它子类内部使用的方法.为了看到这种情景,我们修改Car.m的drive方法,让它使用这个受保护的prepareToDrive方法:

```
// Car.m
#import "Car.h"
#import "Car+Protected.h"

@implementation Car
...
- (void)drive {
    [self prepareToDrive];
    NSLog(@"The %@ is now driving", _model);
}
...
```
接下来,创建一个Car的子类 - Coupe,来看一下受保护方法是怎样工作的(实现的).在(Coupe)接口中没任何特别,但注意类实现是怎样通过导入Car+Protected.h来指定(使用)这个受保护API的.如果你想,你也可以在Coupe.m中重新定义这个方法来重写这个受保护的方法.

```
// Coupe.h
#import "Car.h"

@interface Coupe : Car
// Extra methods defined by the Coupe subclass
@end
```

```
// Coupe.m
#import "Coupe.h"
#import "Car+Protected.h"

@implementation Coupe

- (void)startEngine {
    [super startEngine];
    // Call the protected method here instead of in `drive`
    [self prepareToDrive];
}

- (void)drive {
    NSLog(@"VROOOOOOM!");
}

@end
```
为了迫使Car+Protected.h中方法的受保护状态,即只对其子类有效,那么千万别把这个分类导入到其他文件中.在下面的main.m中,你会看到受保护的prepareToDrive方法通过[ford drive]以及[porsche startEngine]调用,如果你尝试直接调用,那编译就不愿意了(会抱怨).

```
// main.m
#import <Foundation/Foundation.h>
#import "Car.h"
#import "Coupe.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Car *ford = [[Car alloc] init];
        ford.model = @"Ford F-150";
        [ford startEngine];
        [ford drive]; // Calls the protected method
        
        Car *porsche = [[Coupe alloc] init];
        porsche.model = @"Porsche 911 Turbo";
        [porsche startEngine]; // Calls the protected method
        [porsche drive];
        
        // "Protected" methods have not been imported,
        // so this will *not* work
        // [porsche prepareToDrive];
        
        SEL protectedMethod = @selector(prepareToDrive);
        if ([porsche respondsToSelector:protectedMethod]) {
            // This *will* work
            [porsche performSelector:protectedMethod];
        }
        
        
    }
    return 0;
}
```
当然,你可以通过performSelector:来动态的访问到prepareToDrive:.再强调一次,OC中的所有方法都是公有的,也没有方式真正实现将这些方法对客户端代码(client code,直译的)隐藏.分类仅仅是一个基于规约来控制API中哪些部分应该对其他文件可用的方式.

####扩展
***
待续...
