---
title: C 结构体传参
date: 2020-01-15 08:43:00
tags: [C]
---

我们知道，在 C 中，函数参数传递只有两种方式，值传递、引用传递。

值传递的时候，会把对应的值复制一份传到函数中，形参和原变量是不同的两个变量，而引用传递只是把原变量的地址传递给形参，形参根据这个地址拿到对应的变量。

引用传递的时候，我们需要通过解引用 (*xxx) 的方式拿到变量，这个时候我们就拿到了原始变量，我们在函数中所有针对该变量的操作都是针对原始变量的操作，因为它们是同一个变量。

> 那这个规则对结构体变量来说是否还适用呢？

```C
#include <stdio.h>

typedef struct User {
    char *name;
    int age;
} User;

void test1(User user)
{
    user.name = "name1";
    printf("test1: user address = %p\n", (void*)&user);      // 0x7ffee7aae760
    printf("test1: name address = %p\n", (void*)&user.name); // 0x7ffee7aae760
}

void test2(User *user)
{
    user->name = "name2";
    printf("test2: user address = %p\n", (void*)&user);       // 0x7ffee7aae768 (这个是参数变量的地址)
    printf("test2: name address = %p\n", (void*)&user->name); // 0x7ffee7aae798
}

int main()
{
    User user = {.age = 23, .name = "name"};
    printf("origin: user address = %p\n", (void*)&user);      // 0x7ffee7aae798
    printf("origin: name address = %p\n", (void*)&user.name); // 0x7ffee7aae798 (因为 name 是结构体的第一个成员，所以和结构体的地址一样)

    test1(user);
    printf("test1: user.name = %s\n", user.name);             // name

    test2(&user);
    printf("test2: user.name = %s\n", user.name);             // name2

    return 0;
}
```

输出：

```
origin: user address = 0x7ffee7aae798
origin: name address = 0x7ffee7aae798
test1: user address = 0x7ffee7aae760
test1: name address = 0x7ffee7aae760
test1: user.name = name
test2: user address = 0x7ffee7aae768
test2: name address = 0x7ffee7aae798  // 和原始 user 的地址一致
test2: user.name = name2
```

有人会发现 `printf("test2: user address = %p\n", (void*)&user);` 输出的地址并不是原始 `user` 的地址，这是为什么呢？

其实实际上 `void test2(User *user)` `test2(&user);` 这两句还是值传递，只是传递的值是一个指针的地址。而上面的打印 `&user` 的地址实际上
是打印保存这个指针地址变量的地址，听起来有点拗口，但实际上就是这样，我们在 `test2` 内拿到了一个地址，我们会根据这个地址拿到原始的变量。

> 如何证实这一点呢？

在 `test2` 里面加上这一行，获取解引用后的 `user` 的地址，发现指向是原始 `user` 的地址。

```
printf("test2: user address = %p\n", (void*)&(*user)); // 0x7ffee7aae798
```

结论：值传递、引用传递的规则对于结构体变量传参同样适用。
