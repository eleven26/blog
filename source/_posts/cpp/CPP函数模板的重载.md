---
title: C++ 函数模板的重载
date: 2020-04-20 18:09:00
tags: [C++]
---

当需要对不同的类型使用同一种算法（同一个函数体）时，为了避免定义多个功能重复的函数，可以使用模板。然而，并非所有的类型都使用同一种算法，有些特定的类型需要单独处理，为了满足这种需求，C++ 允许对函数模板进行重载，程序员可以像重载常规函数那样重载模板定义。

比如，交换两个变量的值，有两个方法，一个是使用指针，另一个是使用引用：

```C++
// 方案1：使用指针
template<typename T> void Swap(T *a, T *b) {
    T temp = *a;
    *a = *b;
    *b = temp;
}

// 方案2：使用引用
template<class T> void Swap(T &a, T &b) {
    T temp = a;
    a = b;
    b = temp;
}
```

这两种方案都可以交换 int、float、char、bool 等基本类型变量的值，但是却不能交换两个数组。

对于方案1，调用阿函数时传入的是数组指针，或者说是指向第 0 个元素的指针，这样交换的仅仅是数组的第 0 个元素，而不是整个数组。数组和指针本来是不等价的，只是当函数的形参为指针时，传递的数组也会隐式地转换为指针。

对于方案2，假设传入的是一个长度为 3 的 int 类型数组（该数组的类型是 `int[3]`），那么 T 的真实类型为 `int[3]`，`T temp` 会被替换为 `int [3]temp`，这显然是错误的。另外一方面，语句 `a=b;` 尝试对数组 a 赋值，而数组名是常量，它的值不允许被修改，所以也会产生错误。

交换两个数组唯一的办法就是逐个交换所有的数组元素，请看下面的代码：

```C++
template<typename T> void Swap(T a[], T b[], int len) {
    T temp;
    for (int i = 0; i < len; i++) {
        temp = a[i];
        a[i] = b[i];
        b[i] = temp;
    }
}
```

在该函数模板中，最后一个参数的类型为具体类型（int），而不是泛型。并不是所有的模板参数都必须被泛型化。

下面是一个重载函数模板的完整示例：

```C++
#include<iostream>
using namespace std;

template<class T> void Swap(T &a, T &b); // 模板1：交换基本类型的值
template<typename T> void Swap(T a[], T b[], int len); // 模板2：交换两个数组

template<typename T> void printArray(T arr[], int len); // 打印数组元素

int main() {
    // 交换基本类型的值
    int m = 10, n = 99;
    Swap(m, n); // 匹配模板1
    cout << m << ", " << n << endl;

    // 交换两个数组
    int a[3] = {1, 2, 3};
    int b[3] = {4, 5, 6};
    int len = sizeof(a) / sizeof(int); // 数组长度
    Swap(a, b, len); // 匹配模板2
    printArray(a, len);
    printArray(b, len);

    return 0;
}

template<class T> void Swap(T &a, T &b) {
    T temp = a;
    a = b;
    b = temp;
}

template<typename T> void Swap(T a[], T b[], int len) {
    T temp;
    for (int i = 0; i < len; i++) {
        temp = a[i];
        a[i] = b[i];
        b[i] = temp;
    }
}

template<typename T> void printArray(T arr[], int len) {
    for (int i = 0; i < len; i++) {
        cout << arr[i] << ", ";
    }
    cout << endl;
}
```

输出：

```
99, 10
4, 5, 6, 
1, 2, 3,
```
