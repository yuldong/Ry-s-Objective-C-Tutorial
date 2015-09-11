来源于 Ry’s Objective-C Tutorial - RyPress

一个学习Objective-C基础知识的网站.

个人觉得很棒,所以决定抽时间把章节翻译一下.

本人的英语水平有限,有让大家误解或者迷惑的地方还请指正.
***
块(下文直接用Block)是OC中的匿名函数.允许你在对象之间传递任意的语句,这种方式往往比查阅(引用)某处定义的函数更直观.此外,Block被实现成闭包,使它很容易捕获其周围(上下文)(其他变量)的状态.

####创建Block
Block的使用跟普通函数完全一样.你可以声明一个block变量,(这个过程)跟你声明一个函数一样,然后定义这个Block,也跟实现函数一样,随后调用这个Block,还是跟调用函数一样:

```
// main.m
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // Declare the block variable
        double (^distanceFromRateAndTime)(double rate, double time);
        
        // Create and assign the block
        distanceFromRateAndTime = ^double(double rate, double time) {
            return rate * time;
        };
        // Call the block
        double dx = distanceFromRateAndTime(35, 1.5);
        
        NSLog(@"A car driving 35 mph will travel "
              @"%.2f miles in 1.5 hours.", dx);
    }
    return 0;
}
```

(^)这个符号来标识distanceFromRateAndTime变量是一个block.跟函数声明类似,你也需要包含返回值类型,参数类型以便编译器能保证(强制)类型安全.^与指针类型变量(比如,int *aPOinter)前的星号表现行为相似.^只在声明block时使用,随后你就可以跟使用其他普通变量一样使用block.

block本身其实是一个没有函数名的函数定义.上述的^double(double rate, double time)签名解释为:返回值为double类型,有两个double类型的参数(返回类型可以被忽略,如果你想的话).在花括号{}内可以写任意语句,这跟普通函数一样.

在把block赋值给distanceFromRateAndTime变量后,我们就可以把这个变量当做函数来调用.

#####无参数的Blocks
如果一个block不带任何参数,那么你可以忽略这个block中的参数列表.同样的,block中的返回值始终是可选的,所以你可以把(block)定义(采用的符号)缩短成^ {}:

```
double (^randomPercent)(void) = ^ {
    return (double)arc4random() / 4294967295;
};
NSLog(@"Gas tank is %.1f%% full",
      randomPercent() * 100);
```
内置的arc4random()函数返回一个随机的32位整型值.通过除以能返回的可能最大值(4294967295)来得到一个0到1之间的浮点值.

截止目前,block看起来就像一个更复杂的定义函数方式.而实际上是,它以闭包来实现,从而开启了令人兴奋的新编程机会的大门( But, the fact that they’re implemented as closures opens the door to exciting new programming opportunities).

####闭包
在block内部,你有权访问数据,包括局部变量,传递给bloclk的参数,以及全局变量/函数.但由于blocks以闭包实现,也就意味着你也有权访问非局部变量.非局部变量是定义在block上下文中,而并不在block内部.比如,getFullCarName可以引用在block之前定义的make变量.

```
NSString *make = @"Honda";
NSString *(^getFullCarName)(NSString *) = ^(NSString *model) {
    return [make stringByAppendingFormat:@" %@", model];
};
NSLog(@"%@", getFullCarName(@"Accord"));    // Honda Accord
```
非局部变量实际上是被拷贝后与block一起当做常量存储的,这就意味着它们是只读的.如果尝试着在block内给make变量赋新值,就会抛出一个编译错误.

![Accessing non-local variables as const copies](http://rypress.com/tutorials/objective-c/media/blocks/const-non-local-variables.png)

实际上,非局部变量被拷贝成常量意味着一个block不仅有权访问这些非局部变量-它(还)创建了这些非局部变量的快照(副本).非局部变量在block定义时,就把它们自身携带的值冻结到这个block中了(Non-local variables are frozen at whatever value they contain when the block is defined),并且这个block始终用的都是被冻结(拷贝)的值,即使这个非局部变量随后在程序中更改了.让我们来一下,在创建block之后改变make变量发生了什么事情.

```
NSString *make = @"Honda";
NSString *(^getFullCarName)(NSString *) = ^(NSString *model) {
    return [make stringByAppendingFormat:@" %@", model];
};
NSLog(@"%@", getFullCarName(@"Accord"));    // Honda Accord

// Try changing the non-local variable (it won't change the block)
make = @"Porsche";
NSLog(@"%@", getFullCarName(@"911 Turbo")); // Honda 911 Turbo
```
闭包是一个非常方便的能与周围状态(上下文)一起工作的方式,因为它消除了将外部的值当做参数传递的需求-你只要简单将非局部变量当做在这个block中定义的变量来用就行.

#####可变的非局部变量.
将非局部变量冻结(存储)成常量值是一个安全的默认行为,以阻止你不经意的在block中修改这些变量(引起非局部变量值的更改).然而,也有你想在block中修改这些变量的情况.那么,你可以通过使用__block存储修饰符来声明一个变量以重写它的const复制行为.

```
__block NSString *make = @"Honda";
```
这种(声明方式)告诉block通过引用来捕获这个变量,即,在block内部使用的make变量与其之外的使用的make变量之间创建一个直接关系.现在,你就可以在block外部给make赋新值,它会反映在block中,反之亦然.

![Accessing non-local variables by reference](http://rypress.com/tutorials/objective-c/media/blocks/mutable-non-local-variables.png)

像普通函数中的静态局部变量一样,__block修饰符可以充当多次调用block时使用的记忆体.This makes it possible to use blocks as generators(**这句太蛋疼了,理解为把block当做一个生成器使用**).例如下面的代码段,它创建了一个可以在随后多次执行中记住i值的block.

```
__block int i = 0;
int (^count)(void) = ^ {
    i += 1;
    return i;
};
NSLog(@"%d", count());    // 1
NSLog(@"%d", count());    // 2
NSLog(@"%d", count());    // 3
```
####作为方法参数的Block
将block当做变量存储并不常用,在实际情况下,blocks更多的被当做方法参数使用.它们被作为函数指针来解决同样的问题[**此处不是很明白这个same question指代的是什么**],但因为它们被定义成内联的,所以生成的代码更易阅读.

下面的例子中,Car接口声明了一个记录汽车行驶距离的方法.它不用强迫调用者必须传递一个速度常量,而是接收一个block,这个block利用时间来计算速度(the block defines the car’s speed as a function of time).

```
// Car.h
#import <Foundation/Foundation.h>

@interface Car : NSObject

@property double odometer;

- (void)driveForDuration:(double)duration
       withVariableSpeed:(double (^)(double time))speedFunction
                   steps:(int)numSteps;

@end
```
这个block的数据类型是double(^)(double time),标识着无论调用者传递什么样的block,这个block的返回值必须是double类型,并且只接收单一的double类型参数.注意一点,此处的block跟该章节刚开始讨论的block变量声明基本是一样的语法,除了没有变量名.

实现文件中可以通过speedFunction来调用这个block.下面的例子使用单纯的黎曼加法来近似得到持续时间中汽车行驶的距离(The following example uses a naïve right-handed Riemann sum to approximate the distance traveled over duration).steps参数被用来让调用者决定估算的精准度.

```
// Car.m
#import "Car.h"

@implementation Car

@synthesize odometer = _odometer;

- (void)driveForDuration:(double)duration
       withVariableSpeed:(double (^)(double time))speedFunction
                   steps:(int)numSteps {
    double dt = duration / numSteps;
    for (int i=1; i<=numSteps; i++) {
        _odometer += speedFunction(i*dt) * dt;
    }
}

@end
```
正如你在下面的main()函数中所见,block可以在方法调用中定义.尽管这需要时间解析这种语法,但是这比创建一个专用的函数来定义withVariableSped参数更直观.

```
// main.m
#import <Foundation/Foundation.h>
#import "Car.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Car *theCar = [[Car alloc] init];
        
        // Drive for awhile with constant speed of 5.0 m/s
        [theCar driveForDuration:10.0
               withVariableSpeed:^(double time) {
                           return 5.0;
                       } steps:100];
        NSLog(@"The car has now driven %.2f meters", theCar.odometer);
        
        // Start accelerating at a rate of 1.0 m/s^2
        [theCar driveForDuration:10.0
               withVariableSpeed:^(double time) {
                           return time + 5.0;
                       } steps:100];
        NSLog(@"The car has now driven %.2f meters", theCar.odometer);
    }
    return 0;
}
```

这是block多功能用处的其中一个例子,标准框架中充满了其他block的用例.[NSArray](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSArray_Class/NSArray.html)允许使用带有block的sortedArrayUsingComparator:方法来对元素进行排序,而[UIView](http://developer.apple.com/library/ios/#documentation/uikit/reference/uiview_class/uiview/uiview.html)通过animateWithDuration:animations:方法中的一个block来决定动画的最终状态.

另外,[NSOpenPanel](https://developer.apple.com/Library/mac/documentation/Cocoa/Reference/ApplicationKit/Classes/NSOpenPanel_Class/index.html)类可以在用户选中一个文件后执行一个block.这种便捷行为在[Ry’s Cocoa Tutorial]的[ Persistent Data]章节有探讨.
####定义Block类型
因为block的数据类型语法会让你的方法变得混乱,所以typedef一个公用的block签名很有用.如下面的代码,创建了一个被称作SpeedFunction的新类型,以便我们能在语义上更容易理解withVariable参数.

```
// Car.h
#import <Foundation/Foundation.h>

// Define a new type for the block
typedef double (^SpeedFunction)(double);

@interface Car : NSObject

@property double odometer;

- (void)driveForDuration:(double)duration
       withVariableSpeed:(SpeedFunction)speedFunction
                   steps:(int)numSteps;

@end
```
很多OC中的标准框架都使用这个技术(比如:[NSComparator](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_DataTypes/Reference/reference.html#//apple_ref/doc/c_ref/NSComparator))
####总结
Block提供了很多与C函数相同的功能,但是他们使用起来更直观(在你习惯了它的语法后).由于他们可以被定义成内联的,所以很容易在方法调用中使用,而且它们是闭包的,所以它们不费吹飞之力就可获取周围(上下文)的变量值.

下个模块,我们安排一些有关iOS以及OS X的错误处理能力的内容.我们会探究两种用来表示错误的类:NSException和NSError.
***
写于15年09月11日 - 希望世界不再有恐怖袭击
