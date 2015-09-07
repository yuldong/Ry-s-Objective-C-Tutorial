来源于 Ry’s Objective-C Tutorial - RyPress

一个学习Objective-C基础知识的网站.
 
个人觉得很棒,所以决定抽时间把章节翻译一下.

本人的英语水平有限,有让大家误解或者迷惑的地方还请指正.
***
正如其他许多面向对象的编程语言,OC类提供了一个创建对象的设计(图).首先,你在类中定义一组(系列)可重用的属性以及行为(方法).随后,使用该类实例化(一个)对象来与这些属性和行为交互.

OC与C++在从类实现中抽象出它的接口(这个方面)很相似.一个接口声明了类中的公共属性和方法,并在对应的实现文件中定义这些属性与方法实际执行的代码.这跟我们之前所见的函数声明实现分离关注点一样.

![A class’s interface and implementation](http://rypress.com/tutorials/objective-c/media/classes/interface-vs-implementation.png)

在这个模块,我们将探讨类的基础语法,涉及接口,实现,属性和方法,并探讨实例化对象的规范方法.我们也将会介绍一些OC的内省和反射的能力.
####创建类 - 使用Xcode
略
####接口
Car.h包含了一些模板代码,但我们将要把它改成下面所示的.(Car.h)声明了一个model的属性以及drive方法.

``` 
// Car.h
#import <Foundation/Foundation.h>

@interface Car : NSObject {
    // Protected instance variables (not recommended)
}

@property (copy) NSString *model;

 - (void)drive;

@end
```
用@interface指令来创建一个接口,跟在(@interface)后面是这个类以及其超类的名称,通过冒号分隔.受保护的变量可以定义在花括号内,但大多数开发人员把实例变量当做实现的细节并倾向于把它们放在.m(实现)文件中.

@property指令则声明了一个公共属性,并且(copy)决定该属性(property)的内存管理方式.在这种情况下,给模型分配的值会被作为副本存储,而不是被直接指向.[属性](http://rypress.com/tutorials/objective-c/properties.html)模块(对这方面)论述更多细节.紧跟(@property)其后的是该属性的数据类型以及名称,跟常规的变量声明一样.

-(void)drive 这一行声明了一个无参的drive方法,(void)这部分规定了它的返回值类型.预置的减号(-)标识该方法是一个实例方法(与[类方法](#jump)对应).

####实现
任何一个类的实现第一件事是导入相应的接口.@implementation指令与@interface指令除了不用包含它的超类之外,其他都类似.私有变量可以存储在跟在类名之后的花括号内.

``` 
// Car.m
#import "Car.h"

@implementation Car {
    // Private instance variables
    double _odometer;
}

@synthesize model = _model;    // Optional for Xcode 4.4+

- (void)drive {
    NSLog(@"Driving a %@. Vrooooom!", self.model);
}

@end
```
@synthesize是为属性生成存取方法的便捷指令(在Xcode4.4之后会自动生成存取(方法),不使用这个指令也行,但对于属性起别名还是有用的,不过很少人这么做).默认情况下,getter(获取)方法就是属性名称,setter(设置)方法则是在属性名加上set前缀并将属性名第一个字母大写(setModel).这比手动创建属性的存取(方法)容易得多.synthesize这个语句中的_model决定该属性要使用的私有实例变量名.

在Xcode4.4中,通过@property声明的属性会自动合成(存取器),所以如果你对默认的实例变量的命名规约习惯的话,你可以忽略@synthesize这一行.

drive的实现与接口中的定义有着相同的签名,但在其后跟着该方法调用是需要执行的代码.请注意我们是如何通过 self.model 来代替 _model 访问实例变量的值(的方式).这是一个很好的实践,因为它(self.model)利用了属性的存取方法.一般来说,唯一你要直接访问实例变量的地方是在init方法以及dealloc方法.

self关键字引用着(指着)调用方法的实例(类似C++与Java中的this关键字).除了访问属性之外,self也可以用来调用在同一个类中定义的方法(比如,[self anotherMethod]).我们将在该教程中看到很多这样的例子.

####实例与使用
任何需要访问一个类的文件都必须导入那个类的头(接口)文件 - 决不能尝试直接导入类的实现文件.那样有悖于公有API与其底层实现分离的目标.为了查看Car类的行为,将main.m修改如下:

``` 
// main.m
#import <Foundation/Foundation.h>
#import "Car.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Car *toyota = [[Car alloc] init];
        
        [toyota setModel:@"Toyota Corolla"];
        NSLog(@"Created a %@", [toyota model]);
        
        toyota.model = @"Toyota Camry";
        NSLog(@"Changed the car to a %@", toyota.model);
        
        [toyota drive];
        
    }
    return 0;
}
```
在使用#import指令导入接口之后,便可以使用上面写的alloc/init方式实例化对象.正如你所见,实例(一个对象)分成两步:首先,你必须通过调用alloc方法给该对象分配内存空间,然后初始化该对象以便使用.你绝不能使用一个未初始化的对象.

必须将对象存储为指针是值得重复(的话题).这也是为什么我们声明变量使用Car *toyota而不是Car toyota.

想调用OC对象的方法,可以通过在方括号内放置(该对象的)实例与方法,并且用空格隔开实现.参数在跟在方法名之后,且前面放置冒号.所以,如果你有从事C++,Java或者Python的背景,那么这种调用方式[toyota setModel:@"Toyota Corolla"]可以转化(理解)为:

``` 
toyota.setModel("Toyota Corolla");
```
对刚转到这门语言(OC)的人来说,这种方括号语法可能会不习惯,但请放心,在你阅读完[Methods]模块后,你会对OC方法的约定感到非常舒服(适应,不仅仅是舒服).

这个例子也向你展示了两种使用对象属性的方式.既可通过synthesize又可通过setModel存取方法,或者使用便捷的点语法,这对那些曾使用过Simula-style语言的开发人员来说,应该更熟悉.

<h4 id="jump">类方法与变量</h4>
上面的代码段定义了实例层次的属性以及方法,但也能定义类层次的.在其他编程语言中被统称为"static"方法/属性(不要与[static](http://www.jianshu.com/p/4fe8df9e6b1e)关键字混淆).

类方法的声明跟实例方法的很像,除了它们的前缀用加号替换了减号.比如,给Car.h如下增加一个类层次的方法:

``` 
// Car.h
+ (void)setDefaultModel:(NSString *)aModel;
```
同样地,类方法的实现也在方法前有个加号.尽管OC中的类层次变量在技术没有这种(写法),但你可以在定义实现之前声明一个静态变量来与之对应(模拟):

``` 
// Car.m
#import "Car.h"

static NSString *_defaultModel;

@implementation Car {
...

+ (void)setDefaultModel:(NSString *)aModel {
    _defaultModel = [aModel copy];
}

@end
```
[aModel copy]这种调用方式创建了这个参数的副本,而不是直接分配(给_defaultModel).这就是我们在给model属性使用copy特性时真正做的事情.

类方法的使用与实例方法语法相同,但是它们必须是类直接调用,正如下面展示的那样.它们不能通过类的实例调用([toyota setDefaultModel:@"Model T"]这种方式会报错).

``` 
// main.m
[Car setDefaultModel:@"Nissan Versa"];
```
####"构造"方法
(准确来说)OC中并没有构造方法.相应的,OC中,一个对象在申请(分配内存)之后就立即调用init方法来完成初始化操作.这也是为什么实例化操作总是两步过程:分配(内存空间),然后初始化.稍后将讨论一下类层次的初始化方法.

init是默认的用来初始化的方法,但你仍可以定义你自己的可以接受配置参数的初始化版本(方法).即使自定义的初始化方法也没有什么特写-除了方法名总是用init开头之外,它们也只是普通的实例方法.下面展示了一个"构造"方法原型(模板):

``` 
// Car.h
- (id)initWithModel:(NSString *)aModel;
```
若要实现上述的方法,你应该遵循initWithModel:所示的公认的(权威的)初始化模式.其中super关键字引用着父类,再(强调)一次,self关键字引用着调用该方法的实例.下一步,我们将下述的方法添加到Car.m中.

``` 
// Car.m
- (id)initWithModel:(NSString *)aModel {
    self = [super init];
    if (self) {
        // Any custom setup work goes here
        _model = [aModel copy];
        _odometer = 0;
    }
    return self;
}

- (id)init {
    // Forward to the "designated" initialization method
    return [self initWithModel:_defaultModel];
}
```
初始化方法应永远都是返回一个指向对象本身的引用,并且如果未能完成初始化,应该返回nil.这就是为什么我们在使用对象之前要核查self是否存在的原因.通常也只有初始化方法才需要做这件事,剩下的只需要调用(该类)**指定的初始化器**就行了.这样的话,当你有几个自定义的init方法时便可忽略样板代码(更加通用)了.

请注意我们是怎样在initWithModel:方法给_model和_odometer赋值的.记住,这是唯一的地方(初始化方法),你应该这么做-而在其他的方法中,你应该这么用-self.model,self.odometer.
#####类层次的初始化
initialize方法是类层次中与init等同的(初始化方法).它能让你在使用类之前对类进行设置.比如,我们可以(通过这方法)来给_defaultModel填充一个合法的值,就像下面这样:

``` 
// Car.m
+ (void)initialize {
    if (self == [Car class]) {
        // Makes sure this isn't executed more than once
        _defaultModel = @"Nissan Versa";
    }
}
```
initialize类方法对于每个类来说,都是在使用之前仅调用一次.这种情况包括Car的所有子类,也就意味着假如Car的其中一个子类没有重新实现(重写)initialize方法,那么Car将会被调用两次.所以,使用self == [Car class]条件来保证initialization的代码只执行一次是很好的做法.也需要注意,在类方法中,self关键字引用着类本身,而不是一个实例.

OC不强制要求标识一个方法是被重写的.虽然init和initialize都是在它们的超类NSObject中定义的,但你在Car.m中重定义它们,编译器也不会抱怨.

下一个重复的main.m将展示我们自定义的初始化方法.在第一次使用类之前,[Car initialize]会被自动调用,设置_defaultModel为@"Nissan Versa".可以在第一个NSLog()看到(结论).当然,也可以第二个日志输出中看到自定义方法的结果.

``` 
// main.m
#import <Foundation/Foundation.h>
#import "Car.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
        // Instantiating objects
        Car *nissan = [[Car alloc] init];
        NSLog(@"Created a %@", [nissan model]);
        
        Car *chevy = [[Car alloc] initWithModel:@"Chevy Corvette"];
        NSLog(@"Created a %@, too.", chevy.model);
        
    }
    return 0;
}
```
####动态类型
类本身就代表着对象,这也就是的它们可以查看自己的属性(内省),甚至改变自己的行为(通过反射).这些时非常强大的动态类型能力,因为这种能力可以让你即使在不知道对象是什么类型的情况下调用对象的方法或者设置对象的属性.

最容易得到一个类对象的方式时通过类层次的方法(对(经常使用)这些冗余术语标识歉意).例如,[Car class]返回一个代表Car本身的对象.你可以将该对象传递给类似isMemberOfClass:以及isKindOfClass这样的方法来获取其他实例的信息.下面给出了一个综合的例子.

``` 
// main.m
#import <Foundation/Foundation.h>
#import "Car.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Car *delorean = [[Car alloc] initWithModel:@"DeLorean"];
        
        // Get the class of an object
        NSLog(@"%@ is an instance of the %@ class",
              [delorean model], [delorean class]);
        
        // Check an object against a class and all subclasses
        if ([delorean isKindOfClass:[NSObject class]]) {
            NSLog(@"%@ is an instance of NSObject or one "
                  "of its subclasses",
                  [delorean model]);
        } else {
            NSLog(@"%@ is not an instance of NSObject or "
                  "one of its subclasses",
                  [delorean model]);
        }
        
        // Check an object against a class, but not its subclasses
        if ([delorean isMemberOfClass:[NSObject class]]) {
            NSLog(@"%@ is a instance of NSObject",
                  [delorean model]);
        } else {
            NSLog(@"%@ is not an instance of NSObject",
                  [delorean model]);
        }
        
        // Convert between strings and classes
        if (NSClassFromString(@"Car") == [Car class]) {
            NSLog(@"I can convert between strings and classes!");
        }
    }
    return 0;
}
```
NSClassFormString()函数时其中一种让你触摸(获取)到类对象的方式.这种方式很灵活,因为它能让你在运行时动态的请求类对象;然而,它也是相当低效.由于这个原因,你应该尽可能地选择类方法(来请求).

如果你对动态类型感兴趣,请务必查阅[选择器](http://rypress.com/tutorials/objective-c/methods.html#selectors)以及[id类型](http://rypress.com/tutorials/objective-c/data-types/primitives.html#the-id-type).
####总结
在这个模块,我们学习了怎么创建类,实例化对象,定义初始化方法,以及操作类层次的方法以及变量.我们也简单的看了动态类型.

[前一个模块](http://www.jianshu.com/p/4fe8df9e6b1e)提到OC不支持命名空间,那也是Cocoa函数需要像NS,CA,AV等这样的前缀去避免命名冲突.这(使用前缀)对类也有同样要求.推荐的规约是使用三个字母前缀用于你应用程序中的特定类(比如,XYZCar).

即使你对属性和方法感觉还不是很舒服也请不要担心,因为这些(上述的内容)基本上是你在开始写自己类时要知道的了,尽管我们略过了一些重要的细节.在下一个模块,我们将通过详细的地查看@property指令以及影响其行为的所有特性来填补这些孔(略过的重要细节).
***
写于15年09月05号



