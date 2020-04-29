---
title: C++ 模板的显式具体化
date: 2020-04-20 20:27:00
tags: [C++]
---

C++ 没有办法限制类型参数的范围，我们可以使用任意一种类型来实例化模板。但是模板中的语句（函数体或者类体）不一定就能适应所有的类型，可能会有个别的类型没有意义，或者会导致语法错误。

例如有下面的函数模板，它用来获取两个变量中较大的一个：

```C++
template<class T> const T& Max(cosnt T& a, const T& b) {
    return a > b ? a : b;
}
```

请读者注意 `a > b` 这条语句，`>` 能够用来比较 int、float、char 等基本类型数据的大小，但是却不能用来比较结构体变量、对象以及数组的大小，因为我们并没有针对结构体、类和数组重载 `>`。

另外，该函数模板虽然可以用于指针，但比较的是地址大小，而不是指针指向的数据，所以也没有现实的意义。

除了 `>`，`+` `-` `*` `/` 等运算符也只能用于基本类型，不能用于结构体、类、数组等复杂类型。总之，编写的函数模板很可能无法处理某些类型，我们必须对这些类型进行单独处理。

模板是一种泛型技术，它能接受的类型是宽泛的、没有限制的，并且对这些类型使用的算法都是一样的（函数体或类体一样）。但是现在我们希望改变这种 "游戏规则"，让模板能够针对某种具体的类型使用不同的算法（函数体或类体不同），这在 C++ 中是可以做到的，这种技术称为模板的显式具体化（Explicit Specialization）。

函数模板和类模板都可以显式具体化，下面我们线讲解函数模板的显式具体化，再讲解类模板的显式具体化。


## 函数模板的显式具体化

在讲解函数模板的显式具体化语法之前，我们先来看一个显式具体化的例子：

```C++
#include<iostream>
#include<string>
using namespace std;

typedef struct {
    string name;
    int age;
    float score;
} STU;

// 函数模板
template<class T> const T& Max(const T& a, const T& b);
// 函数模板的显式具体化（针对 STU 类型的显式具体化）
template<> const STU& Max<STU>(const STU& a, const STU& b);
// 重载 <<
ostream & operator<<(ostream &out, const STU &stu);

int main()
{
    int a = 10;
    int b = 20;
    cout << Max(a, b) << endl;

    STU stu1 = {"张三", 16, 95.5};
    STU stu2 = {"李四", 18, 90.0};
    cout << Max(stu1, stu2) << endl;

    return 0;
}

template<class T> const T& Max(const T& a, const T& b) {
    return a > b ? a : b;
}

template<> const STU& Max<STU>(const STU &a, const STU& b) {
    return a.score > b.score ? a : b;
}

ostream & operator<<(ostream &out, const STU &stu) {
    out << stu.name << " , " << stu.age << " , " << stu.score;
    return out;
}
```

运行结果：

```
20
张三 , 16 , 95.5
```

本例中，STU 结构体用来表示一名学生（Student），它有三个成员，分别是姓名（name）、年龄（age）、成绩（score）；Max() 函数用来获取两份数据中较大的一份。

要想获取两份数据中较大的一份，必然会涉及到对两份数据的比较。对于 int、float、char 等基本类型的数据，直接比较它们本身的值即可，而对于 STU 类型的数据，直接比较它们本身的值不但有语法错误，而且毫无意义，这就要求我们设计一套不同的比较方案，从语法和逻辑上都能行得通，所以本例中我们比较的是两名学生的成绩（score）。

不同的比较方案最终导致了算法（函数体）的不同，我们不得不借助模板的显式具体化技术对 STU 类型进行单独处理。第 14 行代码就是显式具体化的声明，第 34 行代码进行了定义。

请读者注意第 34 行代码， `Max<STU>` 中的 `STU` 表明了要将类型参数 T 具体化为 STU 类型，原来使用 T 的位置都应该使用 STU 替换，包括返回值类型、形参类型、局部变量的类型。

Max 只有一个类型参数 T，并且已经被具体化为 STU 了，这样整个模板就不再有类型参数了，类型参数列表也就为空了，所以模板头应该写做 `template<>`。

另外，`Max<STU>` 中的 `STU` 是可选的，因为函数的形参已经表明，这是 STU 类型的一个具体化，编译器能够逆推出 T 的具体类型。简化后的函数声明为：

```C++
template<> const STU& Max(const STU& a, const STU& b);
```

函数的调用规则：

对于给定的函数名，可以有非模板函数、模板函数、显式具体化模板函数以及它们的重载版本。在调用函数时，显式具体化优先于常规模板，而非模板函数优先于显式具体化和常规模板。


## 类模板的显式具体化

除了函数模板，类模板也可以显式具体化，并且它们的语法是类似的。

```C++
#include <iostream>
using namespace std;

// 类模板
template<class T1, class T2> class Point {
public:
    Point(T1 x, T2 y): m_x(x), m_y(y) {}
    void display() const;
private:
    T1 m_x;
    T2 m_y;
};

template<class T1, class T2> // 这里要带上模板头
void Point<T1, T2>::display() const {
    cout << "x=" << m_x << ", y=" << m_y << endl;
}

// 类模板的显式具体化
template<> class Point<char*, char*> {
public:
    Point(char *x, char *y): m_x(x), m_y(y) {}
    void display() const;
private:
    char *m_x; // x 坐标
    char *m_y; // y 坐标
};

// 这里不能带模板头 template<>
void Point<char*, char*>::display() const {
    cout << "x=" << m_x << " | y=" << m_y << endl;
}

int main()
{
    (new Point<int, int>(10, 20))->display();
    (new Point<int, char*>(10, "东经180度"))->display();
    (new Point<char*, char*>("东经180度", "北纬210度"))->display();
    return 0;
}
```

运行结果：

```
x=10, y=20
x=10, y=东经180度
x=东经180度 | y=北纬210度
```

`Point<char*, char*>` 表明了要将类型参数 T1、T2 都具体化为 `char*` 类型，原来使用 T1、T2 的位置都应该使用 `char*` 替换。Point 类有两个类型参数 T1、T2，并且都已经被具体化了，所以整个类模板就不再有类型参数了，模板头应该写作 `template<>`。

可以发现，当在类的外部定义成员函数时，普通类模板的成员函数前面要带上模板头，而具体化的类模板的成员函数不能带模板头。


## 部分显式具体化

在上面的显式具体化例子中，我们为所有的类型参数提供了实参，所以最后的模板头为空，也即 `template<>`。另外 C++ 还允许只为一部分类型参数提供实参，这称为部分显式具体化。

部分显式具体化只能用于类模板，不能由于函数模板。

仍然以 Point 为例，假设我们现在希望 "只要横坐标 x 是字符串类型" 就以 `|` 来分隔输出结果，而不管纵坐标 y 是什么类型，这种要求就可以使用部分显式具体化技术来满足。请看下面的代码：

```C++
#include <iostream>

using namespace std;

// 类模板
template<class T1, class T2>
class Point {
public:
    Point(T1 x, T2 y) : m_x(x), m_y(y) {}
    void display() const;

private:
    T1 m_x;
    T2 m_y;
};

template<class T1, class T2>
// 这里需要带上模板头
void Point<T1, T2>::display() const {
    cout << "x=" << m_x << ", y=" << m_y << endl;
}

// 类模板部分显式具体化
template<typename T2>
class Point<char *, T2> {
public:
    Point(char *x, T2 y) : m_x(x), m_y(y) {}
    void display() const;

private:
    char *m_x; // x 坐标
    T2 m_y; // y 坐标
};

template<typename T2>
// 这里需要带上模板头
void Point<char *, T2>::display() const {
    cout << "x=" << m_x << " | y=" << m_y << endl;
}

int main() {
    (new Point<int, int>(10, 20))->display();
    (new Point<char *, int>("东经180度", 10))->display();
    (new Point<char *, char *>("东经180度", "北纬210度"))->display();
    return 0;
}
```

输出：

```
x=10, y=20
x=东经180度 | y=10
x=东经180度 | y=北纬210度
```

本例中，T1 对应横坐标 x 的类型，我们将 T1 具体化为 `char*`。

模板头 `template<typename T2>` 中声明的是没有被具体化的类型参数；类名 `Point<char*, T2>` 列出了所有类型参数，包括未被具体化和已经被具体化的。

类名后面之所以要列出所有的类型参数，是为了让编译器确认 "到底是第几个类型参数被具体化了"，如果写作 `template<typename T2> class Point<char*>`，编译器就不知道 `char*` 代表的是第一个类型参数，还是第二个类型参数。


