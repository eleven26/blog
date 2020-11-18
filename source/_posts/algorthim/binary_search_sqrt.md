---
title: 二分查找法计算一个数的平方根
date: 2020-10-26 15:52:02
tags: [算法, 二分查找法]
---

> 如何使用二分查找法求 8 的平方根？保留 6 位小数。

### 假设只精确到整数部分

如果我们只需要计算整数部分的话，那很好实现，假设要开平方的数为 `value`，依次二分，直到找到某一个整数的平方小于等于 `value`，然后这个整数加1的平方大于 `value`，那这个数就是开平方结果的整数部分了。

下面为 PHP 实现代码： 

```PHP
function calcInteger($value, $low, $high)
{
    $mid = round(($low + $high) / 2);
    $mid2 = $mid * $mid;

    if ($mid2 > $value) {
        if (($mid - 1) * ($mid - 1) <= $value) return $mid - 1;
        return calcInteger($value, 0, $mid);
    } else if ($mid2 < $value) {
        if (($mid + 1) * ($mid + 1) >= $value) return $mid;
        return calcInteger($value, $mid, $high);
    } else {
        return $mid;
    }
}

var_dump(calcInteger(8, 0, 8)); // 2
```

实现思路：

1. 如果二分的中点的平方大于 `value`，说明整数部分肯定位于中点之前。这个时候需要特别注意的是，不能直接 `calcInteger($value, 0, $mid - 1)` 这样使用 `$mid - 1` 作为下一次二分查找区间的上限，因为如果开平方结果带有小数的情况下，最终会陷入死循环（具体表现为，不断地在两个区间之间循环，比如 `[0, 3], [2, 3]`）。
2. 如果位于中点之前，那么先判断 中点-1 的平方是否不大于 `value`，那说明这个 中点-1 就是开平方结果的整数部分了
3. 中点的平方小于 `value` 的情况也类似
4. 如果中点的平方恰好等于 `value`，那这个中点就是整数部分了。


### 再假设一下，精确到 1 位小数

这也好办，只要我们做平方操作的时候精确到 1 位小数就行了。如果中位数的平方大于 `value`，这个时候就计算一下 `$mid - 0.1` 的平方，跟 `value` 做比较。我们在这种情况下，假设下一个比 `$mid` 小的数字为 `$mid - 0.1`。

> 将一个整数之间的区间再细分成十份就很好理解了。

下面为 PHP 实现代码：

```PHP
function calcInteger($value, $low, $high)
{
    $mid = round(($low + $high) / 2, 1);
    $mid2 = $mid * $mid;

    if ($mid2 > $value) {
        if (($mid - 0.1) * ($mid - 0.1) <= $value) return $mid - 0.1;
        return calcInteger($value, 0, $mid);
    } else if ($mid2 < $value) {
        if (($mid + 0.1) * ($mid + 0.1) >= $value) return $mid;
        return calcInteger($value, $mid, $high);
    } else {
        return $mid;
    }
}

var_dump(calcInteger(8, 0, 8)); // 2.8
```


### 回到正题

按上一小节说的，我们如果要保留 6 位小数，则可以将原来一个整数的区间想象成有 1000000（10的6次方） 个小区间，比较的过程加减的粒度就是 10^-6 了。

下面为 PHP 实现代码：

```PHP
// 求 3 的平方根，保留 6 位小数
$value = 8;
// 2.8284271247462

function calc($value, $low, $high, $float)
{
    $mid = round(($low + $high) / 2, $float);
    $mid2 = $mid * $mid;

    // 依次确定每一位小数
    $v = 1 / pow(10, $float);

    if ($mid2 > $value) {
        if (($mid - $v) * ($mid - $v) <= $value) return $mid - $v;
        return calc($value, 0, $mid, $float);
    } else if ($mid2 < $value) {
        if (($mid + $v) * ($mid + $v) >= $value) return $mid;
        return calc($value, $mid, $high, $float);
    } else {
        return $mid;
    }
}

var_dump(calc($value, 0, $value, 6)); // 2.828427
```


### 非递归实现

非递归实现的方式也挺简单的，只要在 `$mid * $mid > $value` 和 `$mid * $mid < $value` 两种情况下，分别修改一下区间的上限、下限即可。

```PHP
// 求 8 的平方根，保留 6 位小数
$value = 8;
// 2.8284271247462

function calc($value, $low, $high, $float)
{
    // 依次确定每一位小数
    $v = 1 / pow(10, $float);

    while (true) {
        $mid = round(($low + $high) / 2, $float);
        $mid2 = $mid * $mid;

        if ($mid2 > $value) {
            if (($mid - $v) * ($mid - $v) <= $value) return $mid - $v;
            $high = $mid;
        } else if ($mid2 < $value) {
            if (($mid + $v) * ($mid + $v) >= $value) return $mid;
            $low = $mid;
        } else {
            return $mid;
        }
    }
}

var_dump(calc($value, 0, $value, 6)); // 2.828427
```


### 总结

本文求平方根的思想在于，将原来一个整数的区间拆分成多个小的区间，然后在所有的这些小区间里面依次二分找到最终合适的那个数。
