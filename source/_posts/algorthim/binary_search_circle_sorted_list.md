---
title: 循环有序数组二分查找
date: 2020-10-30 11:48:02
tags: [算法, 二分查找法]
---

### 实现思路

我们按照普通二分查找的方式获取中点，这个时候和普通有序数组有一个不一样的地方，有可能这个数组不是单调递增的。
1. 如果是单调递增的，按照普通二分查找的方式查找即可
2. 如果不是单调递增的，判断 `value` 是否在这个区间内，如果不在，返回 -1 即可。判断方法比较简单，如果 `$value < $array[$low]` 并且 `$value > $array[$high]` 那就说明不在该区间内（因为是循环有序，比如`[5,6,1]`里面查找2）。
3. 如果在这个区间内，还需要判断是否等于临界点，因为 `$array[$low]` 或者 `$array[$high]` 有可能等于 `value`，比如 `[5, 6, 1]` 里面查找 1


### PHP 实现

```PHP
$arr1 = [4, 5, 6, 1, 2, 3];

function bsearch($array, $value)
{
    $low = 0;
    $high = count($array) - 1;

    while ($low <= $high) {
        $mid = intval(($low + $high) / 2);

        if ($array[$low] <= $array[$high]) {
            if ($value > $array[$mid]) {
                $low = $mid + 1;
            } else if ($value < $array[$mid]) {
                $high = $mid - 1;
            } else {
                return $mid;
            }
        } else {
            // 不是递增区间
            if ($value == $array[$high]) return $high;
            if ($value == $array[$low]) return $low;

            if ($value > $array[$high] && $value < $array[$low]) {
                return -1;
            } else if ($value < $array[$high]) {
                $high--;
            } else {
                $low++;
            }
        }
    }

    return -1;
}

var_dump(bsearch($arr1, 2));
```
