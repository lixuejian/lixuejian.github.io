---
layout: post
title:  "C++ 类型转换及cast用法"
categories: C++
tags:  cast 
author: 李学健
---

* content
{:toc}
## C++ 类型转换及cast用法
C++类型转换分为：

 - 隐式类型转换
 - 显式类型转换    
----
### 隐式类型转换
以下为常见的隐式类型转换，将3.14隐式类型转换为int类型。

    int a =（int）3.14
   ----
### 显式类型转换
显示类型转换按适用范围从大到小分为以下四种：

 - static_cast
 - dynamic_cast
 - const_cast
 - reinterpret_cast

 #### static_cast
 用法：**static_cast** < type-id > ( expression )
 说明：该运算符把expression转换为type-id类型，但没有运行时类型检查来保证转换的安全性。
 
 来源：为什么需要static_cast强制转换？  
情况1：void指针->其他类型指针  
情况2：改变通常的标准转换  
情况3：避免出现可能多种转换的歧义

它主要有如下几种用法：

-   用于类层次结构中基类和子类之间指针或引用的转换。进行上行转换（把子类的指针或引用转换成基类表示）是安全的；进行下行转换（把基类指针或引用转换成子类指针或引用）时，由于没有动态类型检查，所以是不安全的。
-   用于基本数据类型之间的转换，如把int转换成char，把int转换成enum。这种转换的安全性也要开发人员来保证。
-   把void指针转换成目标类型的指针(不安全!!)
-   把任何类型的表达式转换成void类型。

注意：static_cast不能转换掉expression的const、volitale、或者__unaligned属性。将int(4字节)显示转换成short(2字节)，用vs调试一下，发现static_cast的作用就是将高位截断。
```cpp
//使用static_dynamic
double result=static_cast<double> (firstnumber/secondnumber);
```
 #### dynamic_cast
用法：**dynamic_cast** < type-id > ( expression )
说明：该运算符把expression转换成type-id类型的对象。Type-id必须是类的指针、类的引用或者void *；

如果type-id是类指针类型，那么exdivssion也必须是一个指针，如果type-id是一个引用，那么exdivssion也必须是一个引用。

dynamic_cast主要用于类层次间的上行转换和下行转换，还可以用于类之间的交叉转换。

在类层次间进行上行转换时，dynamic_cast和static_cast 的效果是一样的；

在进行下行转换时，dynamic_cast具有类型检查的功能，比static_cast 更安全。
    
**上行转换**，比如B继承自A，B转换为A，进行上行转换时，是安全的，如下：
```cpp
#include <iostream>
using namespace std;
class A
{
     // ......
};
class B : public A
{
     // ......
};
int main()
{
     B *pB = new B;
     A *pA = dynamic_cast<A *>(pB); // Safe and will succeed
}
```

**下行转换**，如果expression是type-id的基类，使用dynamic_cast进行转换时，在运行时就会检查expression是否真正的指向一个type-id类型的对象，如果是，则能进行正确的转换，获得对应的值；否则返回NULL，如果是引用，则在运行时就会抛出异常；例如：
```cpp
class B
{
     virtual void f(){};
};
class D : public B
{
     virtual void f(){};
};
void main()
{
     B* pb = new D;   // unclear but ok
     B* pb2 = new B;
     D* pd = dynamic_cast<D*>(pb);   // ok: pb actually points to a D
     D* pd2 = dynamic_cast<D*>(pb2);   // pb2 points to a B not a D, now pd2 is NULL
}
```
复杂的继承关系暂不讨论。

### const_cast
用法：**const_cast**<type_id> (expression)
说明：该运算符用来修改类型的const或volatile属性。除了const 或volatile修饰之外， type_id和expression的类型是一样的。

常量指针被转化成非常量指针，并且仍然指向原来的对象；常量引用被转换成非常量引用，并且仍然指向原来的对象；常量对象被转换成非常量对象。

```cpp
class B{  
  public:  
	int m_iNum;  
}  
  
void foo(){  
	const B b1;  
	b1.m_iNum = 100; //comile error  
	B b2 = const_cast<B>(b1);  
	b2. m_iNum = 200; //fine  
}
```
上面的代码编译时会报错，因为b1是一个常量对象，不能对它进行改变；使用const_cast把它转换成一个常量对象，就可以对它的数据成员任意改变。注意：b1和b2是两个不同的对象。

### reinterpret_cast
用法：**reinterpret_cast**<type_id> (expression)
说明：type-id必须是一个指针、引用、算术类型、函数指针或者成员指针。它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针(先把一个指针转换成一个整数，在把该整数转换成原类型的指针，还可以得到原先的指针值)。

IBM的C++指南里明确告诉了我们reinterpret_cast可以，或者说应该在什么地方用来作为转换运算符：

-   从指针类型到一个足够大的整数类型
-   从整数类型或者枚举类型到指针类型
-   从一个指向函数的指针到另一个不同类型的指向函数的指针
-   从一个指向对象的指针到另一个不同类型的指向对象的指针
-   从一个指向类函数成员的指针到另一个指向不同类型的函数成员的指针
-   从一个指向类数据成员的指针到另一个指向不同类型的数据成员的指针

总结来说：reinterpret_cast用在任意指针（或引用）类型之间的转换；以及指针与足够大的整数类型之间的转换；从整数类型（包括枚举类型）到指针类型，无视大小。
```cpp
int main()
{
   int a = 49;
   int *pi = &a;
   char *pc = reinterpret_cast<char*>(pi);//int * 到char *,用户自己安全
   cout<<*pc<<endl; //输出字符"1"
   unsigned long b = reinterpret_cast<unsigned long>(pc);//char * 转 unsigned long
   cout<<b<<endl;//输出pc指向地址（即a的地址）对应的整数
   int *pi2 = reinterpret_cast<int *>(b);//unsigned long 转 int*
   cout<<*pi2<<endl; //输出49
}
```  
    
  ---
引用
> [https://www.jianshu.com/p/5163a2678171](https://www.jianshu.com/p/5163a2678171)
> [https://www.cnblogs.com/TenosDoIt/p/3175217.html](https://www.cnblogs.com/TenosDoIt/p/3175217.html)
> [https://www.cnblogs.com/chio/archive/2007/07/18/822389.html](https://www.cnblogs.com/chio/archive/2007/07/18/822389.html)

