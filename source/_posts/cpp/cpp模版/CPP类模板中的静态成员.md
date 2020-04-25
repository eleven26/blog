---
title: C++ 类模板中的静态成员
date: 2020-04-23 20:09:00
tags: [C++]
---

类模板中可以定义静态成员，从该类模板实例化得到的所有类都包含同样的静态成员。

```
#include <iostream>
using namespace std;
template<class T>
class A
{
private:
    static int count;
public:
    A() { count++; }
    ~A() { count--; }
    A(A&) { count++; }
    static void PrintCount() { cout << count << endl; }
};
template<> int A<int>::count = 0;
template<> int A<double>::count = 0;
int main()
{
    A<int> ia;
    A<double> da;
    ia.PrintCount();
    da.PrintCount();
    return 0;
}
```

输出：

```
1
1
```

对静态成员在类外部加以声明是必需的。

A<int> 和 A<double> 是两个不同的类。虽然它们都有静态成员变量 count，但是显然，A<int> 的对象 ia 和 A<double> 的对象 da 不会共享一份 count。
