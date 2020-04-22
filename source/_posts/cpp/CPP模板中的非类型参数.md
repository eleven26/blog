---
title: C++ 模板中的非类型参数
date: 2020-04-22 20:27:00
tags: [C++]
---

模板是一种泛型技术，目的是将数据的类型参数化，以增强 C++ 语言（强类型语言）的灵活性。C++ 对模板的支持非常自由，模板中除了可以包含类型参数，还可以包含非类型参数，例如：

```C++
template<typename T, int N> class Demo{ };
template<class T, int N> void func(T (&arr)[N]);
```

T 是一个类型参数，它通过 `class` 或 `typename` 关键字指定。N 是一个非类型参数，用来传递数据的值，而不是类型，它和普通函数的形参一样，都需要指明具体的类型。类型参数和非类型参数都可以用在函数体或者类体中。

当调用一个函数模板或者通过一个类模板创建对象时，非类型参数会被用户提供的、或者编译器推断出的值所取代。


## 在函数模板中使用非类型参数

在 《C++函数模板的重载》一节中，我们通过 Swap() 函数来交换两个数组的值，其原型为：

```C++
template<typename T> void Swap(T a[], T b[], int len);
```

形参 len 用来指明要交换的数组的长度，调用 Swap() 函数之前必须先通过 sizeof 求得数组长度再传递给它。

这是因为，数组作为参数传递给函数形参的时候，会被转换为指针类型。而单纯通过这个指针无法获取数组长度。

多出来的形参 len 给编码带来了不便，我们可以借助模板中的非类型参数将它消除，请看下面的代码：

```C++
template<typename T, unsigned N> void Swap(T (&a)[N], T(&b)[N]) {
    T temp;
    for (int i=0; i<N; i++) {
        temp = a[i];
        a[i] = b[i];
        b[i] = temp;
    }
}
```

`T (&a)[N]` 表明 a 是一个引用，它引用的数据类型是 `T[N]`，也即一个数组；`T (&b)[N]` 也是类似的道理。分析一个引用和分析一个指针的方法类似，编译器总是从它的名字开始读取，然后按照优先级顺序依次解析。

调用 Swap() 函数时，需要将数组名字传递给它：

```
int a[5] = {1, 2, 3, 4, 5};
int b[5] = {10, 20, 30, 40, 50};
Swap(a, b);
```

编译器会使用数组类型 int 来代替类型参数 T，使用数组长度 5 来代替非类型参数 N。

下面是一个完整的示例：

```C++
#include <iostream>
using namespace std;

template<class T> void Swap(T &a, T &b); // 模板1：交换基本类型的值
template<typename T, unsigned N> void Swap(T (&a)[N], T (&b)[N]); // 模板2：交换两个数组

template<typename T, unsigned N> void printArray(T (&arr)[N]); // 打印数组元素

int main()
{
    // 交换基本类型的值
    int m = 10, n = 99;
    Swap(m, n); // 匹配模板1
    cout << m << ", " << n << endl;

    // 交换两个数组
    int a[5] = {1, 2, 3, 4, 5};
    int b[5] = {10, 20, 30, 40, 50};
    Swap(a, b); // 匹配模板2
    printArray(a);
    printArray(b);

    return 0;
}

template<class T> void Swap(T &a, T &b) {
    T temp = a;
    a = b;
    b = temp;
}

template<typename T, unsigned N> void Swap(T (&a)[N], T (&b)[N]) {
    T temp;
    for (int i = 0; i < N; ++i) {
        temp = a[i];
        a[i] = b[i];
        b[i] = temp;
    }
}

template<typename T, unsigned N> void printArray(T (&arr)[N]) {
    for (int i = 0; i < N; ++i) {
        if (i == N - 1) {
            cout << arr[i] << endl;
        } else {
            cout << arr[i] << ", ";
        }
    }
}
```

## 在模板中使用非类型参数

C/C++ 规定，数组一旦定义后，它的长度就不能改变了；换句话说，数组容量不能动态地增大或者减小。这样的数组称为静态数组（static array）。静态数组有时候会给编写代码带来不便，我们可以通过自定义的 Array 类来实现动态数组（dynamic array）。所谓动态数组，是指数组容量能够在使用的过程中随时增大或减小。

```C++
#include <iostream>
#include <cstring>
#include <cstdlib>
using namespace std;

template<typename T, int N>
class Array {
public:
    Array();
    ~Array();

public:
    T& operator[](int i); // 重载下标运算符 []
    int length() const { return m_length; } // 获取数组长度
    bool capacity(int n); // 改变数组容量
    void print();
private:
    int m_length; // 数组的当前长度
    int m_capacity; // 当前内存的容量（能容纳的元素的个数）
    T *m_p; // 指向数组内存的指针
};

template<typename T, int N>
Array<T, N>::Array() {
    m_p = new T[N];
    m_capacity = m_length = N;
}

template<typename T, int N>
Array<T, N>::~Array() {
    delete []m_p;
}

template<typename T, int N>
T & Array<T, N>::operator[](int i) {
    if (i < 0 || i > m_length -1) {
        cout << "Exception: index out of range." << endl;
    }

    return m_p[i];
}

template<typename T, int N>
bool Array<T, N>::capacity(int n) {
    if (n > 0) {
        // 容量增长
        int len = m_length + n; // 增大后的数组长度
        if (len < m_capacity) { // 现有内存足以容纳增大后的数组
            m_length = len;
            return true;
        } else { // 现有内存不能容纳增大后的数组
            T *pTemp = new T[m_length + 2 * n * sizeof(T)]; // 增加的内存足以容纳 2*n 个元素
            if (pTemp == NULL) { // 内存分配失败
                cout << "Exception: Failed to allocate memory" << endl;
                return false;
            } else { // 内存分配成功
                memcpy(pTemp, m_p, m_length * sizeof(T));
                delete []m_p;
                m_p = pTemp;
                m_capacity = m_length = len;
                return true;
            }
        }
    } else {
        // 容量减小
        int len = m_length - abs(n); // 收缩后的数组长度
        if (len < 0) {
            cout<<"Exception: Array length is too small!"<<endl;
            return false;
        } else {
            m_length = len;
            return true;
        }
    }
}

template<typename T, int N>
void Array<T, N>::print() {
    for (int i = 0; i < m_length; ++i) {
        cout << m_p[i] << " ";
    }
    cout << endl;
}

int main()
{
    Array<int, 5> arr;

    // 为数组元素赋值
    for (int i = 0; i < arr.length(); ++i) {
        arr[i] = 2 * i;
    }

    // 第一次打印数组
    arr.print();

    // 扩大容量并为增加的元素赋值
    arr.capacity(8);
    for (int j = 5; j < arr.length(); ++j) {
        arr[j] = 2 * j;
    }

    // 第二次打印数组
    arr.print();

    // 收缩容量
    arr.capacity(-4);
    // 第三次打印数组
    arr.print();
}
```

运行结果：

```
0 2 4 6 8
0 2 4 6 8 10 12 14 16 18 20 22 24
0 2 4 6 8 10 12 14 16
```

Array 是一个类模板，它有一个类型参数 T 和一个非类型参数 N，T 指明了数组元素的类型，N 指明了数组长度。

capacity() 成员函数是 Array 类的关键，它使得数组容量可以动态地增加或者减小。传递给它一个正数时，数组容量增大；传递给它一个负数时，数组容量减小。

之所以能通过 [] 来访问数组元素，是因为在 Array 类中以成员函数的形式重载了 [] 运算符，并且返回值是数组元素的引用。如果直接返回数组元素的值，那么将无法给数组元素赋值。


## 非类型参数的限制

非类型参数不能随意指定，它受到了严格的限制，只能是一个整数，或者是一个指向对象或函数的指针（也可以是引用）。引用和指针在本质上是一样的。

1) 当非类型参数是一个整数时，传递给它的实参，或者由编译器推导出的实参必须是一个常量表达式，例如 10、 2 * 30 等，但不能是 n、10 + n 等（n 是变量）。

对于上面的 Swap() 函数，下面的调用就是错误的：

```C++
int len;
cin >> len;
int a[len];
int b[len];
Swap(a, b);
```

对上面的 Array 类，以下创建对象的方式是错误的：

```
int len;
cin >> len;
Array<int, len> arr;
```

这两种情况，编译器推导出来的实参是 len，是一个变量，而不是常量。

2) 当非类型参数是一个指针（引用）时，绑定到该指针的实参必须具有静态的生存期；换句话说，实参必须存储在虚拟地址空间中的静态数据区。局部变量位于栈区，动态创建的对象位于堆区，它们都不能用作实参。
