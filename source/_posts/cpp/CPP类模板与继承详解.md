---
title: C++ 类模板与继承详解
date: 2020-04-22 20:09:00
tags: [C++]
---

类模板和类模板之间、类模板和类之间可以互相继承。它们之间的派生关系有以下四种情况。

## 类模板从类模板派生

```C++
template<class T1, class T2>
class A {
    T1 v1;
    T2 v2;
};

template<class T1, class T2>
class B : public A<T2, T1> {
    T1 v3;
    T2 V4;
};

template<class T>
class C : public B<T, T> {
    T v5;
};

int main() {
    B<int, double> obj1;
    C<int> obj2;

    return 0;
}
```

编译到第 30 行，编译器用 int 替换类模板 B 中的 T1，用 double 替换 T2，生成 B<int, double> 类如下：

```C++
class B<int, double>: public A <double, int>
{
    int v3;
    double v4;
};
```

B<int, double> 的基类是 A<double, int>。于是编译器就要用 double 替换类模板 A 中的 T1，用 int 替换 T2，生成 A<double, int> 类如下：

```C++
class A<double, int> {
    double v1;
    int v2;
};
```

编译到第 31 行，编译器生成类 C<int>，还有 C<int> 的直接基类 B<int, int>，以及 B<int, int> 的基类 A<int, int>。


## 类模板从模板类派生

```C++
template<class T1, class T2>
class A {
    T1 v1;
    T2 v2;
};
template<class T>
class B: public A<int, double>{
    T v;
};
int main() {
    B<char> obj1;
    return 0;
}
```

A<int, double> 是一个具体类的名字，而且它是一个模板类，因此说模板 B 是从模板类派生而来的。

编译器编译到 `B<char> obj1;` 时会自动生成两个模板类：`A<int, double>` 和 `B<char>`。


## 类模板从普通类派生

```C++
class A{ int v1; };
template<class T>
class B: public A {T v;};
int main() {
    B<char> obj1;
    return 0;
}
```


## 普通类从模板类派生

```C++
template<class T>
class A{
    T v1;
    int n;
};
class B: public A<int> {
    double v;
};
int main() {
    B obj1;
    return 0;
}
```
