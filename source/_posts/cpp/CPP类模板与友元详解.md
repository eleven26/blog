---
title: C++ 类模板与友元详解
date: 2020-04-23 20:09:00
tags: [C++]
---

## 函数、类、类的成员函数作为类模板的友元

```C++
void Func1(){}
class A{};
class B
{
public:
    void Func(){}
};
template<class T>
class Tmpl
{
    friend void Func1();
    friend class A;
    friend void B::Func();
};

int main()
{
    Tmpl<int> i;
    Tmpl<double> f;
    return 0;
}
```

类模板实例化时，除了类型参数被替换外，其他所有内容都原样保留，因此任何从 Tmpl 实例化得到的类都包含上面三条友元声明，因而也都会把 Func1、类A 和 B::Func 当作友元。


## 函数模板作为类模板的友元

```C++
#include<iostream>
#include<string>
using namespace std;
template<class T1, class T2>
class Pair
{
private:
    T1 key; // 关键字
    T2 value; // 值
public:
    Pair(T1 k, T2 v): key(k), value(v) {};
    bool operator<(const Pair<T1, T2> & p) const;
    template<class T3, class T4>
    friend ostream & operator<<(ostream & o, const Pair<T3, T4> &p);
};
template<class T1, class T2>
bool Pair<T1, T2>::operator<(const Pair<T1, T2> &p) const
{
    return key < p.key;
}
template<class T1, class T2>
ostream & operator<<(ostream &o, const Pair<T1, T2> &p)
{
    o << "(" << p.key << "," << p.value << ")";
    return o;
}
int main()
{
    Pair<string, int> student("Tom", 29);
    Pair<int, double> obj(12, 3.14);
    cout << student << " " << obj;
    return 0;
}
```

程序的输出结果是：

```
(Tom, 29)(12, 3.14)
```

第 13、14 行将函数模板 operator<< 声明为类模板 Pair 的友元。

编译本程序时，编译器自动生成了两个 operator<< 函数，它们的原型分别是：

```C++
ostream& operator<<(ostream &o, const Pair<string, int> &p);
ostream& operator<<(ostream &o, const Pair<int, double> &p);
```

前者是 Pair<string, int> 类的友元，但不是 Pair<int, double> 类的友元。后者相反。


## 函数模板作为类的友元

实际上，类也可以将函数模板声明为友元。

```C++
#include<iostream>
using namespace std;
class A
{
    int v;
public:
    A(int n): v(n) {}
    template<class T>
    friend void Print(const T & p);
};
template<class T>
void Print(const T &p)
{
    cout << p.v;
}
int main()
{
    A a(4);
    Print(a);
    return 0;
}
```

程序的输出结果是：

```
4
```

编译器编译到第 19 行 `Print(a);` 时，就从 Print 模板实例化出一个 Print 函数，原型如下：

```
void Print(const A & p);
```

这个函数本来不能访问 p 的私有成员。但是编译器发现，如果将类 A 的友元声明中的 T 换成 A，就能起到将该 Print 函数声明为友元的作用，因此编译器就认为该 Print 函数是类 A 的友元。


## 类模板作为类模板的友元

一个类模板还可以将另一个类模板声明为友元。

```C++
#include<iostream>
using namespace std;
template<class T>
class A
{
public:
    void Func(const T & p)
    {
        cout << p.v;
    }
};
template<class T>
class B
{
private:
    T v;
public:
    B(T n): v(n) {}
    template<class T2>
    friend class A; // 把类模板 A 声明为友元，这样 A 里面的成员函数都是 B 的友元函数
};
int main()
{
    B<int> b(5);
    A< B<int> > a; // 用 B<int> 替换 A 模板中的 T
    a.Func(b);
    return 0;
}
```

程序输出结果：

```
5
```

在本程序中，A< B<int> > 类称为 B<int> 类的友元。
