来源于 Ry’s Objective-C Tutorial - RyPress

一个学习Objective-C基础知识的网站.

个人觉得很棒,所以决定抽时间把章节翻译一下.

本人的英语水平有限,有让大家误解或者迷惑的地方还请指正.

原文地址:http://rypress.com/tutorials/objective-c/exceptions

仅供学习,如有转摘,请注明出处.
***
iOS或者OS X程序运行中,会产生两种不同类型的问题.其中,**异常**代表着编程人员级别的bugs,比如尝试访问不存在的数组元素.它们被设计的目的就是告知开发人员 - 有意外的情况发生.异常一般很少在你写代码时发生,却通常导致程序崩溃.
相对于异常,**错误**则是用户级别的问题,比如尝试加载一个不存在的文件.因为错误是正常程序执行时预期的,所以在这类错误发生时,你应该手动核查这类情况并告知用户.大部分情况下,错误不会引起程序崩溃.

![Exceptions vs. errors](http://rypress.com/tutorials/objective-c/media/exceptions/exceptions-vs-errors.png)

该部分将对异常和错误进行深入的介绍.从概念上来说,处理异常与处理错误很相似.首先,你得检测到问题,然后再处理它.随后我们会看到它们之间的不同.
####异常
异常对应的类是[NSException](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSException_Class/Reference/Reference.html).它被设计成(用)一个通用的方式来封装了异常数据,所以你基本上不用子类化它或者定义一个自定义的异常对象.下面列出了NSException的三个主要属性.

属性 		 | 描述 |
------------ | ------------- |
name | NSString类型,唯一标识该异常  |
reason | NSString类型,可读的异常信息描述  |
userinfo | NSDictionary类型,其中的键值对包含有关异常的额外信息,取决于异常类型 |

异常仅用于严重的程序错误,理解这点很重要.这个是让你知道,一些问题在开发周期中产生,在你去解决这个问题之后,是不会再发生了.而如果处理一个可预测的问题,那你应该用[错误](http://rypress.com/tutorials/objective-c/exceptions#errors),而不是异常.
#####异常处理
在大多数的高级编程语言中,都能通过使用常规的try-catch-finally方式来处理异常.首先,将可能产生异常的代码放在@try块中,随后,如果抛出异常,对应的@catch()块便会执行以便处理发生的问题.@finally块在最后执行,无论是否产生异常,@finally块都会执行.

下面的main.m文件通过访问一个数组不存在元素来触发一个异常.在@catch()块中,我们仅简单的显示了异常详细内容.括号里的NSException对象*theException是包含异常对象的变量名称.

```
// main.m
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSArray *inventory = @[@"Honda Civic",
                               @"Nissan Versa",
                               @"Ford F-150"];
        int selectedIndex = 3;
        @try {
            NSString *car = inventory[selectedIndex];
            NSLog(@"The selected car is: %@", car);
        } @catch(NSException *theException) {
            NSLog(@"An exception occurred: %@", theException.name);
            NSLog(@"Here are some details: %@", theException.reason);
        } @finally {
            NSLog(@"Executing finally block");
        }
    }
    return 0;
}
```
在实际情况下,你会想在@catch()块中,通过打印问题并修正它来处理异常,或者构造一个错误对象并展示给用户.(对于)处理未捕获的异常,默认的行为是在控制台输出(相关)信息并退出程序.

OC的异常处理能力并不是最高效的,所以你应该仅在真正特殊(例外)的情况下使用@try/@catch()块.不要用它来代替普通的的控制流.相反的,使用标准的if语句来核查可预测的情况.

这意味着上述的代码对异常的使用很不恰当.一个更好的方式是应该使用惯用的比较方式来确认selectedIndex得比[inventory count]小.

```
if (selectedIndex < [inventory count]) {
    NSString *car = inventory[selectedIndex];
    NSLog(@"The selected car is: %@", car);
} else {
    // Handle the error
}
```
#####内置的异常(对象)
标准的iOS和OS X框架中定义了几种内置的异常.完整的(异常)列表可在[这里](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Exceptions/Concepts/PredefinedExceptions.html#//apple_ref/doc/uid/20000057-BCIGHECA)查看,但最常用的一些如下所示:

异常名称 		 | 描述 |
------------ | ------------- |
NSRangeException | 访问集合外界元素时产生  |
NSInvalidArgumentException | 给方法传递非法参数时产生  |
NSInternalInconsistencyException | 内部发生意外情况产生 |
NSGenericException | 当你不知道是什么触发异常时产生 |

请注意,这些值都是strings,不是NSException的子类.所以,当你想查看异常的特定类型时,你需要像下面这样查看(异常的)name属性才行:

```
...
} @catch(NSException *theException) {
    if (theException.name == NSRangeException) {
        NSLog(@"Caught an NSRangeException");
    } else {
        NSLog(@"Ignored a %@ exception", theException);
        @throw;
    }
} ...
```
在@catch块中,@throw指令重新产生一个已捕获的异常.在上述代码中,我们使用@throw来将异常抛到更高层的@try块中,以便我们忽略我们不想处理的异常.一个简单的if语句还是很有必要的.
#####自定义异常
可以使用@throw来抛出包含自定义数据的异常对象.最容易的方式就是通过 exceptionWithName:reason:userinfo 工厂方法创建一个异常实例.下面的例子是在top-level函数中抛出异常并在mian()函数中捕获:

```
// main.m
#import <Foundation/Foundation.h>

NSString *getRandomCarFromInventory(NSArray *inventory) {
    int maximum = (int)[inventory count];
    if (maximum == 0) {
        NSException *e = [NSException
                          exceptionWithName:@"EmptyInventoryException"
                          reason:@"*** The inventory has no cars!"
                          userInfo:nil];
        @throw e;
    }
    int randomIndex = arc4random_uniform(maximum);
    return inventory[randomIndex];
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        @try {
            NSString *car = getRandomCarFromInventory(@[]);
            NSLog(@"The selected car is: %@", car);
        } @catch(NSException *theException) {
            if (theException.name == @"EmptyInventoryException") {
                NSLog(@"Caught an EmptyInventoryException");
            } else {
                NSLog(@"Ignored a %@ exception", theException);
                @throw;
            }
        }
    }
    return 0;
}
```
除非必要,否则你不应该在常规程序中这样去抛出自定义异常.其中一个原因就是,异常代表了编程人员的错误,而且应该在当你为严重的编码错误做打算时,才考虑使用它.其二,@throw是一个昂贵的操作,尽可能的使用errors更好.
####错误
错误代表着可预料的问题,并且有很多类型的操作可以在不引起程序崩溃的的情况下失败,它们比异常更常见.与异常不同,这种错误核查是高质量代码的常规项.
[NSError](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/Foundation/Classes/NSError_Class/Reference/Reference.html)类封装了失败操作的详细内容.它的主要属性与NSException类似.

属性 		 | 描述 |
------------ | ------------- |
domain | NSString类型,包含了错误的domain.被用来将错误组织成层级结构并且保证错误码不会冲突  |
code | NSInteger类型,标识了error的ID.在相同domain中的每个error都有一个唯一的值  |
userInfo | NSDictionary类型,其中的key-value对包含了错误的额外信息, (键值对内容)取决与错误类型|

NSError对象的userInfo字典比NSException的字典版本提供了更多内容.一些预定义的键被定义为常量,如下表:

键 		 | 值 |
------------ | ------------- |
NSLocalizedDescriptionKey | NSString类型,代表着错误的全部描述.通常也包含了失败原因|
NSLocalizedFailureReasonErrorKey | NSString类型,简洁的错误原因描述|
NSUnderlyingErrorKey | 对代表着下一高层次的domain中的错误的另一个NSError引用 |

根据错误(情况), 这个字典也包含其他特殊的domain信息.比如, 文件加载错误对应的key是NSFilePathErrorKey,它(对应的value)包含了所请求文件的路径.

注意,localizedDescription和localizedFailureReason方法是分别访问头两个key的可选方式.下面的章节使用了它们.
#####错误处理
错误不需要任何像@try,@catch这样的专用的语言指令.相反地,函数或者方法失败之后会接受到一个额外的参数(通常被称作Error),(这个error)指向了NSError对象.如果一个操作失败了,一般会返回NO或者nil来标明错误并且把这个(额外的)参数填充到错误详情中.如果成功,则简单返回正常的请求值.

很多方法都被配置成能接受一个间接的NSError对象引用.一个间接的引用是一个指针的指针,它允许方法的这个参数指向一个全新的NSError实例.你可以通过两个指针记号[(NSError **)error]来决定一个方法的error参数是否接受一个间接的引用.

随后的代码段,通过NSString类的stringWithContentsOfFile:encoding:error:方法来尝试加载不存在文件以证明这种错误处理模式.当文件加载成功,这个方法以NSString类型返回文件的内容,但当加载失败,便会直接返回nil,同时返回已填充新的NSError对象的error参数.

```
// main.m
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSString *fileToLoad = @"/path/to/non-existent-file.txt";
        
        NSError *error;
        NSString *content = [NSString stringWithContentsOfFile:fileToLoad
                                                      encoding:NSUTF8StringEncoding
                                                         error:&error];
        
        if (content == nil) {
            // Method failed
            NSLog(@"Error loading file %@!", fileToLoad);
            NSLog(@"Domain: %@", error.domain);
            NSLog(@"Error Code: %ld", error.code);
            NSLog(@"Description: %@", [error localizedDescription]);
            NSLog(@"Reason: %@", [error localizedFailureReason]);
        } else {
            // Method succeeded
            NSLog(@"Content loaded!");
            NSLog(@"%@", content);
        }
    }
    return 0;
}
```
注意我们是怎样被迫通过引用操作来将error变量传递给这个方法的.因为error参数接收一个指针引用(双指针).也注意我们是怎样使用普通的if语句来核查方法成功情况下的返回值.你应该只在方法直接返回nil的时候再去访问NSError引用,而且,你永远都不应该使用NSError对象的存在来判断(方法调用的)成功或失败.

当然,如果你仅关注的操作的成功而不考虑为何失败,那你只要给error参数传递一个NULL,它就会被忽略了.
#####内置错误
与NSException类似,NSError也被设计成一个用来表示错误的通用对象.与子类化它相反,各种iOS与OS X框架都为domain和code fields定义了它们自己的常量.有很多内置的错误domain,主要的四个如下:

- NSMachErrorDomain
- NSPOSIXErrorDomain
- NSOSStatusErrorDomain
- NSCocoaErrorDomain

大部分你遇到的错误都在NSCocoaErrorDomain中,但如果你继续往低层次的domain中探究,你会看到其他一些.比如,如果你把下面一行加入到main.m中,你将会发现一个NSPOSIXErrorDomain的错误.

```
NSLog(@"Underlying Error: %@", error.userInfo[NSUnderlyingErrorKey]);
```
对于大多数应用来说,你不必这么做,但是当你需要知道引起错误的根源时,它就能派上用场了.

在你确定了错误domain后,你可以核查一下具体的错误码.[Foundation Constants Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_Constants/Reference/reference.html#//apple_ref/doc/uid/TP40003793)描述了一些枚举,其中的大部分错误码都在NSCocoaErrorDomain中定义了.比如下面的代码,用来判断错误是否是NSFileReadNoSuchFileError.

```
...
if (content == nil) {
    if ([error.domain isEqualToString:@"NSCocoaErrorDomain"] &&
        error.code == NSFileReadNoSuchFileError) {
        NSLog(@"That file doesn't exist!");
        NSLog(@"Path: %@", [[error userInfo] objectForKey:NSFilePathErrorKey]);
    } else {
        NSLog(@"Some other kind of read occurred");
    }
} ...
```
其他框架应该在他们的文档中包含任何自定义的domain和错误码.
#####自定义错误
如果你正在参与一个大的项目,你很可能会有一些函数或者方法导致错误.这部分章节将说明如何使用上述的典型的错误处理模式来配置它们.

作为最佳实践,你应该在专门的头文件中定义你的错误.举例来说,InventoryErrors.h文件可以定义一个domain,包含了与inventory中项目相关的各类的不同错误码.

```
// InventoryErrors.h

NSString *InventoryErrorDomain = @"com.RyPress.Inventory.ErrorDomain";

enum {
    InventoryNotLoadedError,
    InventoryEmptyError,
    InventoryInternalError
};
```
从技术上来说,自定义错误domain可以定义成任何你想的,但推荐的形式是com.<Company>.<Framework-or-project>.ErrorDomain,正如在InventoryErrorDomina.h中所示的.(并利用)枚举定义了错误码常量.

关于函数和方法的区别就在于是否支持额外的error参数.它是特定的NSError **类型,如下所示的getRandomCarFromInventory().当发生一个错误,你会将这个参数指向一个新的NSError对象.需要注意我们是怎样手动将NSLocalizedDescriptionKey添加到userInfo字典中来定义localizedDescription的.

```
// main.m
#import <Foundation/Foundation.h>
#import "InventoryErrors.h"

NSString *getRandomCarFromInventory(NSArray *inventory, NSError **error) {
    int maximum = (int)[inventory count];
    if (maximum == 0) {
        if (error != NULL) {
            NSDictionary *userInfo = @{NSLocalizedDescriptionKey: @"Could not"
            " select a car because there are no cars in the inventory."};
            
            *error = [NSError errorWithDomain:InventoryErrorDomain
                                         code:InventoryEmptyError
                                     userInfo:userInfo];
        }
        return nil;
    }
    int randomIndex = arc4random_uniform(maximum);
    return inventory[randomIndex];
}

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSArray *inventory = @[];
        NSError *error;
        NSString *car = getRandomCarFromInventory(inventory, &error);
        
        if (car == nil) {
            // Failed
            NSLog(@"Car could not be selected");
            NSLog(@"Domain: %@", error.domain);
            NSLog(@"Error Code: %ld", error.code);
            NSLog(@"Description: %@", [error localizedDescription]);
            
        } else {
            // Succeeded
            NSLog(@"Car selected!");
            NSLog(@"%@", car);
        }
    }
    return 0;
}
```
因为从技术上它是一个错误,而不是异常,这个getRandomCarFromInventory()版本是一个更"适当"的方式来处理它.(与[Custom Exceptions](http://rypress.com/tutorials/objective-c/exceptions#custom-exceptions)对应).
####总结
错误代表着iOS或者OS X应用的失败操作.它是一个标准的方式,用来记录检测点的相关信息并且将它传给处理代码.异常(与错误)也比较类似,但被设计成更多用于开发辅助.它们通常都不应该用于 production-ready[已成品的] 程序中.

怎么处理错误或者异常很大程度上都得根据问题类型以及你的应用程序才能决定.但是,大多数情况都会使用像 [UIAlertView](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIAlertView_Class/UIAlertView/UIAlertView.html#//apple_ref/occ/cl/UIAlertView) (iOS). or [NSAlert](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ApplicationKit/Classes/NSAlert_Class/Reference/Reference.html) (OS X)(控件)这种来告知用户信息.之后,你很可能想通过检查NSError或者NSExcepiton对象来发掘问题所在,从而尝试修复它.

下个章节探讨一下关于OC运行时的比较偏概念的东西.我们将学习有关对象背后的内存是怎样通过手动retain,release进行管理的(目前已过时),以及目前新的ARC实际含义.

---
写于15年09月29号, 完成于15年10月20号
**这一片感觉翻译的很烂, 请留情...不甚感激**