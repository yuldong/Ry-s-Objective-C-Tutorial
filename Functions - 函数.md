来源于 Ry’s Objective-C Tutorial - RyPress

一个学习Objective-C基础知识的网站.

个人觉得很棒,所以决定抽时间把章节翻译一下.

本人的英语水平有限,有让大家误解或者迷惑的地方还请指正.
***
与变量,条件语句,循环一道,函数也是任何现代编程语言的基本组成部分.它们可以让你在应用程序中复用任意的代码块,这对你来组织和维护大部分零散的代码来说很有必要.在iOS和OS X框架中,你会发现很多关于函数的例子[many example](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_Functions/Reference/reference.html#//apple_ref/doc/uid/TP40003774).

与其他基本结构一样,OC依赖(可使用)C语言中的全部函数.该模块介绍C函数中的最重要的方面,包括基础语法,其声明与实现的分离,适用范围(被访问作用域)以及库函数(使用)注意点.

####基础语法
C语言函数由四部分组成:返回值,名称,参数以及相关联的代码块.在定义完一个函数后,你可以通过传递必备的参数(在括号之间)来调用函数来执行它相关联的代码块.

如下所写的例子,该小段代码定义了一个getRandomInteger名称的函数,它的参数接收两个int值,并且换回另一个(新的)int值.在该函数内,我们通过minimum和maximum参数接收输入,并通过return关键字返回一个计算好的值.然后,我们在main()中通过传入相应的-10 和 10作为参数调用这个函数.

```
// main.m
#import <Foundation/Foundation.h>

int getRandomInteger(int minimum, int maximum) {
    return arc4random_uniform((maximum - minimum) + 1) + minimum;
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        int randomNumber = getRandomInteger(-10, 10);
        NSLog(@"Selected a random number between -10 and 10: %d",
              randomNumber);
    }
    return 0;
}
```
其中内置的[arc4random_uniform()](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man3/arc4random.3.html)函数返回一个0到任意指定数值之间的随机数(它比原有的rand()和random()算法更优).

函数允许你使用指针引用来充当返回值或参数,这也就意味着我们可以无缝与OC对象集成(有一点要记住,所有的对象都是指针)美,例如,吧main.m修改如下:

``` 
// main.m
#import <Foundation/Foundation.h>

NSString *getRandomMake(NSArray *makes) {
    int maximum = (int)[makes count];
    int randomIndex = arc4random_uniform(maximum);
    return makes[randomIndex];
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSArray *makes = @[@"Honda", @"Ford", @"Nissan", @"Porsche"];
        NSLog(@"Selected a %@", getRandomMake(makes));
    }
    return 0;
}
```
其中的getRandomMake()函数接收NSArray对象作为参数,并且返回一个NSString对象.注意它们使用了相同的星号语法 [pointer variable declarations](http://rypress.com/tutorials/objective-c/c-basics.html#pointers)(指针变量的声明).

####声明与实现
函数必须在使用之前定义.在上面的例子中,如果你在main()函数之后定义getRandomMake(),那当你在main()中尝试调用该方法时,编译器是找不到的.这给开发人员施加了严格的(编码)结构约束,也使得很难组织更大的程序.为了解决这个问题,C语言允许你将函数的声明与实现分离.
![Function declarations vs. implementations](http://rypress.com/tutorials/objective-c/media/functions/declarations-vs-implementations.png)

函数声明是用来告诉编译器这个函数的输入,输出情况.函数通过提供返回值和参数类型,使得编译器确保你在正确使用该函数,并非在不知晓该函数干吗(使用它).对应的实现会对已声明的函数附加一个代码块.在一起,便是一个完整的函数定义.

下面的例子声明了getRandomMake()函数以便允许你在实现它之前在main()中使用.注意,声明函数时,参数可以只有类型信息 - 参数名字可以忽略(如果有需要).

``` 
// main.m
#import <Foundation/Fo undation.h>

// Declaration
NSString *getRandomMake(NSArray *);

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSArray *makes = @[@"Honda", @"Ford", @"Nissan", @"Porsche"];
        NSLog(@"Selected a %@", getRandomMake(makes));
    }
    return 0;
}

// Implementation
NSString *getRandomMake(NSArray *makes) {
    int maximum = (int)[makes count];
    int randomIndex = arc4random_uniform(maximum);
    return makes[randomIndex];
}
```

我们将在 [Function Libraries](http://rypress.com/tutorials/objective-c/functions#function-libraries)看到,把函数的声明和实现分离对组织大型框架是很有用的.
####static关键字
static关键字允许你调整函数或者变量的可用性.不幸的是,它的不同效果取决于你的使用.该部分将从两个通用的用例来解释static关键字.
#####Static Functions
默认情况下,所有的函数都是全局的.意味着,只要你在文件中定义了一个函数,它在任意位置立刻变的可用.static标示符则让你将函数的作用范围限制在当前文件.这种做法对创建"私有"函数以及避免名称冲突很有帮助.

![Globally-scoped functions vs. statically-scoped functions](http://rypress.com/tutorials/objective-c/media/functions/static-functions.png)

下面的例子向你展示如何创建static函数.如果你将这些代码添加到另一个文件(比如,一个专用的函数库),那么你在main.m中就不能访问getRandomInteger().注,static关键字在函数的声明和实现都要使用.

``` 
// Static function declaration
static int getRandomInteger(int, int);

// Static function implementation
static int getRandomInteger(int minimum, int maximum) {
    return arc4random_uniform((maximum - minimum) + 1) + minimum;
}
```
#####Static 局部变量
声明在函数内的变量(也称作自动局部变量)在每次函数调用时都会被重设.这是一个默认的行为,无论时调用多少次,函数还是这样一贯的执行.然而,当时使用static修饰一个局部变量时,函数在调用过程中就会"记住"该变量的值.
![Independent automatic variables vs. shared static variables](http://rypress.com/tutorials/objective-c/media/functions/static-local-variables.png)
举个例子,在下面的小段代码中,我们不在main()中存储count,而是让countByTwo()函数帮我们记录,currentCount变量也不会被重设.

``` 
// main.m
#import <Foundation/Foundation.h>

int countByTwo() {
    static int currentCount = 0;
    currentCount += 2;
    return currentCount;
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSLog(@"%d", countByTwo());    // 2
        NSLog(@"%d", countByTwo());    // 4
        NSLog(@"%d", countByTwo());    // 6
    }
    return 0;
}
```
但有一点,这与之前讨论过的修饰函数不同,它(使用static关键字修饰变量)不会影响局部变量的作用域.也就是说,局部变量还是只能在函数体内访问.
#####函数库
OC不支持命名空间的,所以为了防止与其他全局函数,大型框架命名冲突,需要给函数(以及类)添加唯一的标示符前缀.这就是你看到内置函数都是[NSMakeRange()](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_Functions/index.html#//apple_ref/c/func/NSMakeRange)和[CGImageCreate()](https://developer.apple.com/library/ios/documentation/GraphicsImaging/Reference/CGImage/index.html#//apple_ref/c/func/CGImageCreate)这样,而不是makeRange()和imageCreate()这种样子.
当你创建自己的函数库,你应该在专用的头文件中声明函数,并在另一个分离的文件中实现对应的函数(就像[Objective-C classes](http://rypress.com/tutorials/objective-c/classes.html)).这样能让你通过导入头文件来使用这个库,而不用担心其内部的函数实现.例如,CarUtilities库的头文件看起来像下面这样:

``` 
// CarUtilities.h
#import <Foundation/Foundation.h>

NSString *CUGetRandomMake(NSArray *makes);
NSString *CUGetRandomModel(NSArray *models);
NSString *CUGetRandomMakeAndModel(NSDictionary *makesAndModels);
```
对应的实现文件定义这些函数的真实操作,因为其他文件不能(应当)导入实现的文件,这样你就可以通过static修饰符来创建仅供库内部使用的"私有"函数了.

``` 
// CarUtilities.m
#import "CarUtilities.h"

// Private function declaration
static id getRandomItemFromArray(NSArray *anArray);

// Public function implementations
NSString *CUGetRandomMake(NSArray *makes) {
    return getRandomItemFromArray(makes);
}
NSString *CUGetRandomModel(NSArray *models) {
    return getRandomItemFromArray(models);
}
NSString *CUGetRandomMakeAndModel(NSDictionary *makesAndModels) {
    NSArray *makes = [makesAndModels allKeys];
    NSString *randomMake = CUGetRandomMake(makes);
    NSArray *models = makesAndModels[randomMake];
    NSString *randomModel = CUGetRandomModel(models);
    return [randomMake stringByAppendingFormat:@" %@", randomModel];
}

// Private function implementation
static id getRandomItemFromArray(NSArray *anArray) {
    int maximum = (int)[anArray count];
    int randomIndex = arc4random_uniform(maximum);
    return anArray[randomIndex];
}
```
现在,main.m可以导入(CarUtilities头文件,并调用其中的函数,就像这些函数在main.m中定义的一样.需要注意的是,如果试图调用其中的static getRandomItemFromArray()函数,则会报一个编译错误.

``` 
// main.m
#import <Foundation/Foundation.h>
#import "CarUtilities.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSDictionary *makesAndModels = @{
            @"Ford": @[@"Explorer", @"F-150"],
            @"Honda": @[@"Accord", @"Civic", @"Pilot"],
            @"Nissan": @[@"370Z", @"Altima", @"Versa"],
            @"Porsche": @[@"911 Turbo", @"Boxster", @"Cayman S"]
        };
        NSString *randomCar = CUGetRandomMakeAndModel(makesAndModels);
        NSLog(@"Selected a %@", randomCar);
    }
    return 0;
}
```
####概述
该部分已经介绍完了C语言中function.学习了怎么声明和实现函数,调整它们的作用域,(使用static)使得它们能记住局部变量,并且组织大的函数库.

大多数在Cocoa和Cocoa Touch框架中的功能都被封装程OC类,所以不缺内置函数.你今后(编程)最可能遇到的便是像NSLog()这样的通用函数以及像NSMakeRect()这样利用友好的API创建和配置复杂对象的便捷函数.

我们现在准备开始拥抱(对付)OC的其他面向对象方面.在下个模块,我们将会学习怎样定义类,实例化对象,设置属性以及调用方法(发送消息).
***
写于15年09月04号




