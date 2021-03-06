---
title: 归并排序、快速排序
date: 2020-09-03 12:31:02
tags: [算法, 排序算法]
---

> 冒泡排序、插入排序、选择排序这三种排序算法，它们的时间复杂度都是 `O(n²)`，比较高，适合小规模数据的排序。归并排序和快速排序时间复杂度为 `O(nlogn)`，适合大规模的数据排序。

> 归并排序和快速排序都用到了分治思想。

## 归并排序

### 归并排序的原理

如果要排序一个数组，先从中间的位置将数组分割成两个子数组，然后再对这两个子数组从中间位置分割，如此直到分割之后的子数组只有一个元素。然后将最终分割的数组按大小顺序合并到一个新的数组中，如此递归地合并直到所有子数组合并完毕，就可以得到一个有序的数组了。

### 例子

假设要对下面这组数据利用归并排序方法进行排序：

```
$arr = [3, 5, 100, 6, 78];
```

第一次分割成下面两个数组：

```
$arr1 = [3, 5, 100]; $arr2 = [6, 78];
```

第二次分割：

```
// $arr1 划分成
[3] [5, 100]
// $arr2 划分成
[6] [78]
```

按递归的思想会先处理 `$arr1` 分割出来的两个数组：

`[3]` 里面只有一个元素，无需再分割，`[5, 100]` 还需要再次分割成 `[5]` 和 `[100]`，分割完成就开始合并了，合并的时候，需要创建一个大小等于两个需要合并的数组大小总和的数组 `temp`，然后将小的元素先放入 `temp`，如此直到所有元素都放进了 `temp` 之后，这次合并操作就完成了，就得到了一个有序的数组 `temp`。

所以 `$arr1` 最终合并得到：

```
[3, 5, 100]
```

`$arr2` 最终合并得到：

```
[6, 78]
```

再将 `$arr1` 和 `$arr2` 处理的结果合并，也就是将 `[3, 5, 100]` 和 `[6, 78]` 合并，即可得到：

```
[3, 5, 6, 78, 100]
```

### PHP 实现

```
function merge_sort($arr, $p, $r)
{
    if ($p >= $r) return [$arr[$r]];

    $q = intval(($p + $r) / 2);

    $arr1 = merge_sort($arr, $p, $q);
    $arr2 = merge_sort($arr, $q + 1, $r);

    return merge($arr1, $arr2);
}

function merge($arr1, $arr2)
{
    $arr = [];
    $n1 = count($arr1);
    $n2 = count($arr2);

    $k = 0;
    $i = 0;
    $j = 0;
    while ($i < $n1 && $j < $n2) {
        if ($arr1[$i] <= $arr2[$j]) {
            $arr[$k++] = $arr1[$i++];
        } else {
            $arr[$k++] = $arr2[$j++];
        }
    }

    while ($i < $n1) {
        $arr[$k++] = $arr1[$i++];
    }

    while ($j < $n2) {
        $arr[$k++] = $arr2[$j++];
    }

    return $arr;
}
```


### 算法分析

1. 归并排序是否稳定？

在 `merge` 实现里面对相等的元素，我们可以将 `$arr1` 的元素先合并到 `$arr`，这样即可实现排序算法的稳定性。

2. 时间复杂度是多少？

归并排序的排序过程中不会受原始数据的有序度影响，都是固定的不断拆分成两个数组，直到最终拆分到只有一个元素的时候进行合并，小的在前面，大的在后面。也就是说，归并排序的时间复杂度是固定的 `O(nlogn)`，非常稳定。

3. 空间复杂度是多少？

在将原始数据一分为二的时候，我们不需要开辟额外的空间来存储数据，但是在合并的过程中，我们需要开辟一个临时数组来存储排序后的数据，然后将这个数组覆盖掉原始数组的那一段范围的数据。

但由于某一时刻 CPU 中最多只有一个合并的操作在进行，所以最大的空间复杂度为 `O(n)`。



## 快速排序


### 快速排序原理

快速排序的过程中，我们会先从原始数组中随机选一个数 pivot，以这个数为中点，将剩余的其他 n-1 个数划分为两个部分，小于 pivot 的交换到 pivot 的左边，大于 pivot 的交换到 pivot 的右边。然后再对左边和右边两部分的子数组递归进行此操作，直到最后只有一个元素为止，这个时候我们的整个数组就排序完毕了。


### 例子

利用快速排序对以下数组进行排序：

```
$arr = [5, 1, 7, 8, 6];
```

1. 选择最后一个数（6）作为中点，小于 6 的交换到 `$arr` 的左边，中间为 6，大于 6 的交换到 `$arr` 的右边。

2. 经过上一步的交换后，`$arr` 为 `[5, 1, 6, 8, 7]`，这个时候 6 其实就是最终排序完的位置了。

3. 再对 6 左边和右边的子数组 `[5, 1]`、`[8, 7]` 执行第一步操作，如此直到子数组只有一个元素，结束排序。

> 我们可以发现，快速排序的过程，其实就是一次次地确定最终排序数组中间元素的过程。


### PHP 实现

```
function quick_sort(&$arr, $p, $r)
{
    if ($p >= $r) return;

    $q = partition($arr, $p, $r);

    quick_sort($arr, $p, $q - 1);
    quick_sort($arr, $q + 1, $r);
}

$array = [1, 5, 3, 7, 9, 4, 20, 100, 55, 67, 66];

// 分区+排序
function partition(&$arr, $p, $r)
{
    // 返回分区点
    $pivot = $arr[$r];

    $i = $p;
    for ($j = $p; $j < $r - 1; $j++) { // $r - 1 的值为 pivot，中间值
        if ($arr[$j] < $pivot) {
            $temp = $arr[$i];
            $arr[$i] = $arr[$j];
            $arr[$j] = $temp;
            $i++;
        }
    }

    // $i 永远不大于 $j，$i <= $j
    // $i == $j 的情况下不交换位置
    // 情况一：只有一个数，不影响
    // 情况二：数组里面全部都是小于 $pivot 的元素，这种情况不需要再互换了
    if ($i < $j) {
        $arr[$r] = $arr[$i];
        $arr[$i] = $pivot;
    }

    return $i;
}

quick_sort($array, 0, count($array) - 1);
var_export($array);
```

### 算法分析

1. 快速排序是否是稳定的排序算法？

由于快速排序的过程中，会发生元素前后的交换，所以不能保证稳定性，所以是不稳定的排序算法。

2. 快速排序的时间复杂度是多少？

平均时间复杂度是 `O(nlogn)`，但可能比这个大

3. 快速排序的空间复杂度是多少？

快速排序是原地排序算法，排序过程不需要开辟新的空间来处理。


## 归并排序和快速排序对比

1. 归并排序是稳定的排序算法，快速排序不是稳定的排序算法

2. 归并排序的过程需要开辟临时的空间来进行合并操作，快速排序是原地排序算法，归并排序的空间复杂度太高

3. 归并排序的时间复杂度是固定的，快速排序的时间复杂度会受实际数据有序度影响

4. 实际使用中，快速排序用得更多，因为是原地排序算法，但是不能保证排序结果的稳定性
