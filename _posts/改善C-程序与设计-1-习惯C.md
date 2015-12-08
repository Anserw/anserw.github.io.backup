title: 改善C++程序与设计 「1」习惯C++
date: 2015-11-25 20:50:05
tags: 
- C++ 
- 读书笔记
- Effective C++
---
## 序
这一系列博客（如果还有后续的话）是我关于Effective C++这本书的笔记，我会将其中的内容做一些精简与梳理，发布在我的博客上，作为一种总结与备忘

## C++是一个语言联邦
C++主要的次语言有C, Object-Oriented C++, Template C++, STL. 对于不同的次语言，有着不同的规范与技巧.

## 以编译器替换预处理器
尽量避免使用`#define`，它会使程序员感到困惑. 推荐使用`const`, `enum`, `inline`.

- 全局变量
```c++
// 定义常数
const double Pi = 3.14;

//定义字符串常量，注意有两个const
const char* const authorName = "Anserw";

//对于字符串常量，更推荐这种方式
const std::string authorNameBetter("Anserw");
```
- class专属常量
```c++
class Foo {
private:
    static const int bar = 7;   //常量声明式
    int data[bar];              //使用该常量
    ...
}

const int Foo::bar;             //在class作用域外使用时可能需要定义式，无需再次赋值
```
- enum hack
 - 取`enum`、`#define`的地址不合法, 取`const`的地址合法
 - enum hack是template metaprogramming的基础技术
```c++
class Foo {
private:
    enum { bar = 7 };
    int data[bar];
    ...
}
```
- 用inline函数取代宏(macros)
```c++
//建议使用inline函数
template<typename T>
inline void Max(const T& a, const T& b)
{
    return a > b ? a : b;
}

//不推荐
//#define MAX(a, b) ((a) > (b) ? (a) : (b))
```

## 尽可能使用const
- const可修饰
 - 常量
 - 文件
 - 函数
 - 区块作用域中的static对象
 - classes内的成员变量
 - 指针自身     ` char* const p = foo;`
 - 指针所指物   `const char* p = foo;`
 - 两者都是     `const char* const p = foo;`
 
- 两个成员函数如果只是常量性不同，可以被重载.
```c++
class Foo {
public:
    const char& operator[](std::size_t position) const
    { return text[position]; }
    char& operator[](std::size_t position) 
    { return text[position]; }
private:
    std::string text;
};
```
const与non-const对象会调用不用的函数
```c++
Foo f("bar");
std::cout << f[0];          //调用non-const
const Foo cf("bar");
std::cout << cf[0];         //调用const
```
- const成员函数的两个流派
 - bitwise const阵营主张, const成员函数不更改对象内任何一个bit.(编译器亦然)
 - logical const阵营主张, 在客户端侦测不出的情况下, const成员函数可以修改对象内某些bits.

- 当const和non-const成员函数有着实质等价的实现时，令non-const版本调用const版本可避免代码重复.

## 确定对象被使用前已先被初始化
- C part of C++不保证内容被初始化
- 别混淆了赋值和初始化. 
```c++
//这些是赋值，而非初始化
Foo::Foo(Bar bar1, Bar bar2)
{
    m_bar1 = bar1;
    m_bar2 = bar2;
}

//这些才是初始化
Foo::Foo(Bar bar1, Bar bar2)
:m_bar1(bar1),
 m_bar2(bar2),
 m_bar3()       //调用default构造函数
{ }
```
- 通常初始化效率更高. 
- const或reference成员变量一定需要初值，且不能被赋值.
- base classes更早于derived classes被初始化.
- class的成员变量总是以其声明次序被初始化.
- C++对不同编译单元内定义之non-local static对象的初始化顺序并无明确定义, 需要将每个non-local static对象搬到自己的专属函数内, 并在其中被声明为static. 这是Singleton模式的一个常见实现手法.
