来源于 Ry’s Objective-C Tutorial - RyPress

一个学习Objective-C基础知识的网站.

个人觉得很棒,所以决定抽时间把章节翻译一下.

本人的英语水平有限,有让大家误解或者迷惑的地方还请指正.
***
Objective-C(一下简称OC) 是为iOS以及Mac OS X编程的原生语言,它是一个编译的,广泛用途的语言,它能构建一切程序,包括命令行工具,可以动画实现GUI,可以编写特定领域的库.它也提供了很多用来维护大型的,可扩展的框架的工具.

![Types of programs written in Objective-C](http://rypress.com/tutorials/objective-c/media/introduction/obj-c-overview.png)

类似C++,OC也是被设计为C增加面向对象特征的语言,但这两种语言的实现方式却截然不同.OC更动态,更偏好在运行期间(Runtime)做决定而不是在编译期间.这个特性在很多iOS和OS X开发设计模式有体现.

OC也因它详细的(啰嗦的)命名约定而"闻名".这种情况下的代码基本上不可能会被误解或者误用.举个例子,下面代码段展示了与OC等同的C++方法调用方式.

```
// C++

john->drive("Corvette", "Mary's House")

// Objective-C

[john driveCar:@"Corvette" toDestination:@"Mary's House"]

```

如你所见,OC方法读起来更像人类语言.一旦你习惯了这种方式,(恭喜你),这样使你在使用第三方库和开始新的项目更容易. 如果你对这种方括号有点怕怕(蛋疼),别担心,在这个教程结束的时候,你肯定会觉得相当舒服.

####Frameworks
与大多数编程语言一样,OC有着广泛的标准库支持,语法也相对简单.这个教程主要专注于OC语言本身,但(知晓这些框架),有助于,至少让你在真实使用工具时有些想法.

有不少标准库,但Apple's [Cocoa](https://developer.apple.com/technologies/mac/cocoa.html)和[Cocoa Touch](https://developer.apple.com/technologies/ios/cocoa-touch.html)框架是目前最流行的.这些库分别定义了构建OS X和iOS的API.下面的列表重点介绍了Cocoa和Cocoa Touch中的关键库.对于更详尽的讨论,请访问[Mac Technology Overview](https://developer.apple.com/library/mac/documentation/MacOSX/Conceptual/OSX_Technology_Overview/About/About.html)或者[iOS Technology Overview](https://developer.apple.com/library/ios/documentation/miscellaneous/conceptual/iphoneostechoverview/Introduction/Introduction.html).

Framework | Description
----------| -----------
Foundation  |  Defines core object-oriented data types like strings, arrays, dictionaries, etc. We’ll explore the essential aspects of this framework in the Data Types module.
UIKit |             Provides dozens of classes for creating and controlling the user interface on iOS devices.
AppKit |            Same as UIKit, but for OS X devices.
CoreData |        Provides a convenient API for managing object relationships, supporting undo/redo functionality, and interacting with persistent storage.
MediaPlayer |    Defines a high-level API for playing music, presenting videos, and accessing the user’s iTunes library.
AVFoundation | Provides lower-level support for playing, recording, and integrating audio/video into custom applications.
QuartzCore  |  Contains two sub-frameworks for manipulating images. The CoreAnimation framework lets you animate UI components, and CoreImage provides image and video processing capabilities (e.g., filters).
CoreGraphics |   Provides low-level 2D drawing support. Handles path-based drawing, transformations, image creation, etc.

当你习惯了OC,这些都是你用来构建iOS和OS X程序的一部分工具.但,再说一次,这个教程不是一个app开发的指导教程-而是让你对使用上述框架有所准备.除了上述的Foundation框架,我们将不使用其中的任何库.

如果你对Mac App开发有兴趣,在你对OC有比较靠谱(牢固)的认知后,你可以参见一下Ry's Cocoa Tutorial.跟这个教程一样,它会向你展示如何构建OS X apps.

####Xcode
*注:该份教程使用的Xcode版本比较旧,这里就不介绍了.*

####Get Ready
接下来的两个章节将探讨基本的C语法.之后,我们将开始深入类,方法,协议,以及其他面向对象的构成.本教程有非常多的手写例子,我们鼓励你将这些例子粘贴到我们刚创建的​​模板项目,弄乱一些参数,看看会发生什么.
***
写于15年09月03号