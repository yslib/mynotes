
# Cpp 的一些坑

#### 特殊成员函数的生成机制

复制操作相关函数，移动操作相关函数，以及析构函数中的任何一个如果被声明，那么其他的函不会默认再被提供。其背后所依据的思想是，如果定义了其中一个，那么对于这个类来说默认管理资源的方式可能不再适用，需要用户自己给出定义。

#### 智能指针的make_xxx

一般情况下make_xxx可以提高效率并且减少出错的可能。但是在某些极端的情况下这种方法不好（内存占用多）。并且这种函数不能使用自定义删除器。


#### delete 删除函数

这个除了用于禁止类的特殊的成员函数，还可以用于一般的函数。主要用来限制隐式类型转换和模板特化。

#### 为什么模板参数依赖类型需要用typename

比如对于typename Class<T>::type，如果不用typename，那么Class<T>::type有可能是一个特化，这个特化里面的type可能不是个type，可能是一个成员变量。

#### c++11中，如果函数参数一定要复制，并且这个参数有移动寓意，那么直接传值是最好的选择。
>
```
void foo(std::string name)
{
    std::string copy = std::move(name);
    // do others
}
```

#### constexpr 关键字在c++11 和 c++14中的区别

在c++11中，constexpr的关键字的限制非常严格。当无脑的把constexpr用于构造函数和成员函数的时候，在c++11中就会出错。因为被constexpr所修饰的成员函数具有const性质，是不能改变成员变量的。然而constexpr的这种使用方法在很多情况下并不表示这种寓意，比如编译期的常量构造，构造函数中一定要初始化变量的，但这并不是const。所以在c++11中constexpr受到了限制，并且constexpr的函数只能有一行语句。但是在c++14中就没有这个限制了。