# 从 *#include* 到 *import*

这次在GTest中遇到一个Bug，new出来的对象被析构掉的时候出现了堆损坏(HEAP CORRUPTION)的异常。这种错误只会在Debug上出现，因为Relase版本是不会记录内存分配的状态的。一般情况下，这种错误的根本原因是修改了分配出来的内存的范围之外的部分，也就是越界修改。

这个Bug当时的表现有以下几种特征：

+ 这个功能在项目里一切正常
+ 通过调试观察内存数据正常
+ 在释放时候的栈帧正常
+ GTest时释放的时候内存堆损坏
+ 刚写单元测试的时候也没问题，随着单元测试的内容增加才出现问题
+ Linux和Windows下Bug都会复现
+ 复现概率几乎100%
    

测试代码是测试一个自己实现的智能指针，这个智能指针的实现具有一定的复杂性，里面重载了```operator new```等改变内存正常分配流程的东西，所以我不敢保证我本身写的正确，因此以为是自己代码的原因，排查了很久没有结果。
下面是Debug的过程：

+ 一开始以为是自己不小心修改了new 出来的内存前面的部分，因为一般编译器都会在之前加上记录分配长度的数据。但实际上长度并没有存储在这里，而且这块内存看起来也无意义。分配完之后的数据和释放之前的数据是相同的。

+ 因为堆损坏崩溃的时候会给定内存块，如果用```_CrtSetBreakAlloc```是可以在对这块内存进行修改时触发断点的，然而没有效果。（其实如果这一步成功了就能知道问题所在了）

+ 在Linux下用valgrind 看到调用栈突然跳到了其他无关的结构体的构造函数里，并且是个vector写入错误。因此以为是这个结构体对应的代码出错了。然而这段代码由于使用了元编程，编译之后没有对应函数的栈帧，不太好直接定位到问题所在。（其实到这里如果经验丰富的话已经明白为什么了，只是由于之前没有遇到过这种bug，valgrind第一次用，输出的log的具体意思也不是很明白，而且基本上还是认为我自己写的代码有问题，因此这里错过了最重要的提示）

+ 把这段无关的代码注释掉发现我的那段代码没问题了，于是又认为是这段代码问题。当时在这里又很多问好，为什么两段不相关的代码会相互影响呢？这两段代码都在单元测试里。
  
+ 只能自己调试这段代码了，因为这段代码不是我写的，要想debug只能自己看了。为了不触发异常影响debug的效率，只能先把我的代码注释掉。看了很久之后调了半天也没什么问题。

+ 机缘巧合下再测试我的代码时发现断点断在了那段出现问题的代码里了。到这里才发现为什么函数会跳到一个不相关的代码里呢? 此时并没有意识到有两个内存布局不同但同名的结构体。由于没经验，根本没去留意结构体的名字。


其实到这里的话问题就很清楚了，根本原因就是不小心在单元测试里定义了两个相同名字的类A，我的代码里分配A的时候分配了a大小的空间，但是构造函数却调用了另一段代码里的A的构造函数，那个构造函数栈比较大，大于a，所以破坏了a的堆。然而因为我的代码里的A是一个trivial的基类，构造完之后又被父类的构造函数填充了正常数据。所以只要A构造完成后内存里的内容并没有任何问题。
这个问题违反了C++里面的 One Definition Rule的原则。也就一个定义规则的问题。出现了两个名称相同但内容不同的定义。

### C++03 3.2 One definition rule

>There can be more than one definition of a class type (clause 9), enumeration type (7.2), inline function with external linkage (7.1.2), class template (clause 14), non-static function template (14.5.5), static data member of a class template (14.5.1.3), member function of a class template (14.5.1.1), or template specialization for which some template parameters are not specified (14.7, 14.5.4) in a program provided that each definition appears in a different translation unit, and provided the definitions satisfy the following requirements. Given such an entity named D defined in more than one translation unit, then
— each definition of D shall consist of the same sequence of tokens; and ...


直觉上来讲，如果出现了两个同名定义，至少应该在链接的时候报错。但那时不对的，因为c++的#include的机制本质上就是给了不同的编译单元两个相同名字的定义，但#include能保证内存布局相同，此时链接到那个上都无所谓了。所以，include机制导致了没有办法去处理这种情况。在测试的GTest中，两个编译单元各测试自己的逻辑，都定义了一个名为A的简单的测试结构体，但是结构体却是不同的。这就导致了未定义的行为。

总的来说，这就是两个人写的代码都没问题，放在了一起便出了问题。这只能算是语言的坑。像这种坑在c++里面很多，而且都非常难以察觉。


这个问题难于调试的原因其实是各种机缘巧合综合的结果。只要巧合少了一个，就容易很多。首先，如果不是在GTest里搞测试，这种问题不会出现，GTest把所有测试用例都编译在了一起，逻辑上讲每个Test都应该是独立的，但c++的机制并没有完全把代码环境隔离开。其次，如果我写的类A不是作为一个trivial的基类，而是直接作为有一个non-trival的构造函数的普通类，就会造成两个非常明显的现象，一是这个类的构造函数不会调用，二是类的内容不对。最后，这个类的名字起的太简单了，就是一个A，很难引起人的注意。只要是一个复杂一点的名字，重名的概率就会小很多。
这种问题就是C++的天坑，让其中的某一个人去背锅真的不是很公平。本质上是include这种没有编译状态机制的问题。(未完待续 import部分)