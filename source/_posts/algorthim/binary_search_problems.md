---
title: 二分查找法变形问题
date: 2020-10-27 14:48:02
tags: [算法, 二分查找法]
---

#### 问题

1. 在有序数组中查找第一个值等于 $value 的元素
2. 在有序数组中查找最后一个值等于 $value 的元素
3. 在有序数组中查找第一个值大于等于 $value 的元素
4. 在有序数组中查找最后一个值小于等于 $value 的元素


#### 在有序数组中查找第一个值等于 $value 的元素

```PHP
$array = [1, 2, 3, 4, 5, 5, 5, 5, 7];

// 1. 在有序数组中查找第一个值等于 $value 的元素
function bsearch1($arr, $value)
{
    $low = 0;
    $high = count($arr) - 1;

    while ($low <= $high) {
        // 最终只剩两个元素的时候，$mid 指向小的元素
        $mid = intval(($low + $high) / 2);

        if ($arr[$mid] < $value) {
            $low = $mid + 1;
        } else if ($arr[$mid] > $value) {
            $high = $mid - 1;
        } else {
            // 如果 mid == 0，则说明只有一个元素，则直接返回 0。
            // 判断 mid 的前一个元素是否不等于 value，如果是，则说明 mid 就是第一个等于 value 的元素
            if ($mid == 0 || $arr[$mid - 1] != $value) {
                return $mid;
            }
            $high = $mid - 1;
        }
    }

    return -1;
}

var_dump(bsearch1($array, 5));
```

#### 在有序数组中查找最后一个值等于 $value 的元素

```PHP
$array = [1, 2, 3, 4, 5, 5, 5, 5, 7];

// 2. 在有序数组中查找最后一个值等于 $value 的元素
function bsearch2($arr, $value)
{
    $low = 0;
    $high = count($arr) - 1;

    while ($low <= $high) {
        // 最终只剩两个元素的时候，$mid 指向小的元素
        $mid = intval(($low + $high) / 2);

        if ($arr[$mid] < $value) {
            $low = $mid + 1;
        } else if ($arr[$mid] > $value) {
            $high = $mid - 1;
        } else {
            // 如果 mid 是最后一个元素，则直接返回 count($arr) - 1。
            // 如果不是，判断 $arr[mid + 1] 的值是否不等于 value，如果不等于 value，则说明 $arr[$mid] 就是最后一个等于 value 的元素了
            if ($mid == count($arr) - 1 || $arr[$mid + 1] != $value) {
                return $mid;
            }

            $low = $mid + 1;
        }
    }

    return -1;
}

var_dump(bsearch2($array, 5));
```

#### 在有序数组中查找第一个值大于等于 $value 的元素

```PHP
$array = [1, 2, 3, 3, 5, 5, 5, 5, 7];

// 3. 在有序数组中查找第一个值大于等于 $value 的元素
function bsearch3($arr, $value)
{
    $low = 0;
    $high = count($arr) - 1;

    while ($low <= $high) {
        // 最终只剩两个元素的时候，$mid 指向小的元素
        $mid = intval(($low + $high) / 2);

        if ($arr[$mid] < $value) {
            $low = $mid + 1;
        } else {
            // 如果 mid == 0，说明只有一个元素，直接返回 0；
            // 否则，判断 mid 的前一个元素是否小于 value，如果是，则返回 mid。
            if ($mid == 0 || $arr[$mid - 1] < $value) {
                return $mid;
            }

            $high = $mid - 1;
        }
    }

    return -1;
}

var_dump(bsearch3($array, 3));
```

#### 在有序数组中查找最后一个值小于等于 $value 的元素

```PHP
$array = [1, 2, 3, 3, 5, 5, 5, 5, 7];

// 4. 在有序数组中查找最后一个值小于等于 $value 的元素
function bsearch4($arr, $value)
{
    $low = 0;
    $high = count($arr) - 1;

    while ($low <= $high) {
        // 最终只剩两个元素的时候，$mid 指向小的元素
        $mid = intval(($low + $high) / 2);

        if ($arr[$mid] > $value) {
            $high = $mid - 1;
        } else {
            // 如果是最后一个元素，则直接返回 count($arr) - 1；
            // 否则，判断下一个元素是否大于 value，如果是，则返回 mid。
            if ($mid == count($arr) - 1 || $arr[$mid + 1] > $value) {
                return $mid;
            }

            $low = $mid + 1;
        }
    }

    return -1;
}

var_dump(bsearch4($array, 3));
```
