---
title: 使用 Kotlin 语言实现一个归并排序算法
author: 星野玲
date: 2023-01-15T00:00:00+08:00
description: 上周小玲讲了快速排序算法。这周小玲来将归并排序算法的实现。
draft: false
tags:
  - Kotlin
  - 算法
 
---

上周小玲讲了快速排序算法。这周小玲来讲归并排序算法的实现。但是从这次开始不再讲 IDE 的安装和创建项目了。需要的话请看 [这里]({{< ref "12.md#安装-intellij-idea" >}})。

## 开始编写

这里从创建好了项目开始。首先写出归并排序函数的大概框架。它的函数名为 `mergeSort`，接收一个类型为 `List<Double>`、名称为 list 的形参，返回值类型为 `List<Double>`。注意它的返回值永远都是已经排好序的 `List`。

```kotlin
fun mergeSort(list: List<Double>): List<Double> {

}
```

归并排序算法和快速排序算法一样是采用递归加分治的方法来写的。判断小于或等于 1 的原因和快速排序算法一样，小玲就不再赘述了。

```kotlin
fun mergeSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {
        
    }
}
```

`if` 语句的 `else` 块里就像写快速排序算法一样写递归和分治的代码。

首先咱们需要把这个长度大于 1 的 list，从中间分成两个部分——`leftList` 和 `rightList`。

如果这个长度大于 1 的 list 的长度是偶数，`leftList` 和 `rightList` 的长度就是相等的。

如果这个长度大于 1 的 list 的长度是奇数，`leftList` 将比 `rightList` 多一个元素。

```kotlin
fun mergeSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {
        val middleIndex = if (list.size % 2 == 0) list.size / 2 else list.size / 2 + 1
        val leftList = list.subList(0, middleIndex)
        val rightList = list.subList(middleIndex, list.size)
    }
}
```

然后分别对 `leftList` 和 `rightList` 递归调用这个函数本身。调用后的返回值分别赋给 `sortedLeftList` 和 `sortedRightLeft`。因为 `mergeSort` 函数的返回值永远都是已经排好序的 `List`。所以现在咱们默认 `sortedLeftList` 和 `sortedRightList` 是已经排好序的 `List`。

```kotlin
fun mergeSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {
        val middleIndex = if (list.size % 2 == 0) list.size / 2 else list.size / 2 + 1
        val leftList = list.subList(0, middleIndex)
        val rightList = list.subList(middleIndex, list.size)

        val sortedLeftList = mergeSort(leftList)
        val sortedRightList = mergeSort(rightList)
    }
}
```

现在咱们来编写将两个已经排好序的 list 合并成一个已经排好序的 list 的代码。

定义一个可变的 list 并将它赋值给变量 `sortedList`。

```kotlin
fun mergeSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {
        val middleIndex = if (list.size % 2 == 0) list.size / 2 else list.size / 2 + 1
        val leftList = list.subList(0, middleIndex)
        val rightList = list.subList(middleIndex, list.size)

        val sortedLeftList = mergeSort(leftList)
        val sortedRightList = mergeSort(rightList)

        val sortedList = mutableListOf<Double>()
    }
}
```

定义变量 `tempIndexInSortedLeftList` 和 `tempIndexInSortedRightList`，并将它们的初始值设为 0，这两个变量是用于接下来的 while 循环的。

```kotlin
fun mergeSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {
        val middleIndex = if (list.size % 2 == 0) list.size / 2 else list.size / 2 + 1
        val leftList = list.subList(0, middleIndex)
        val rightList = list.subList(middleIndex, list.size)

        val sortedLeftList = mergeSort(leftList)
        val sortedRightList = mergeSort(rightList)

        val sortedList = mutableListOf<Double>()

        var tempIndexInSortedLeftList = 0
        var tempIndexInSortedRightList = 0
    }
}
```

接下来咱们写一个 while 循环来遍历 `sortedLeftList` 和 `sortedRightList`。这个 while 循环的作用是把 `sortedLeftList` 和 `sortedRightList` 合并成一个已经排好序的 list——`sortedList`。

```kotlin
fun mergeSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {
        val middleIndex = if (list.size % 2 == 0) list.size / 2 else list.size / 2 + 1
        val leftList = list.subList(0, middleIndex)
        val rightList = list.subList(middleIndex, list.size)

        val sortedLeftList = mergeSort(leftList)
        val sortedRightList = mergeSort(rightList)

        val sortedList = mutableListOf<Double>()

        var tempIndexInSortedLeftList = 0
        var tempIndexInSortedRightList = 0

        while (tempIndexInSortedLeftList < sortedLeftList.size || tempIndexInSortedRightList < sortedRightList.size) {
            if (tempIndexInSortedLeftList == sortedLeftList.size) {
                sortedList.add(sortedRightList[tempIndexInSortedRightList++])
            } else if (tempIndexInSortedRightList == sortedRightList.size){
                sortedList.add(sortedLeftList[tempIndexInSortedLeftList++])
            } else if (sortedLeftList[tempIndexInSortedLeftList] < sortedRightList[tempIndexInSortedRightList]) {
                sortedList.add(sortedLeftList[tempIndexInSortedLeftList++])
            } else {
                sortedList.add(sortedRightList[tempIndexInSortedRightList++])
            }
        }
    }
}
```

最后，返回 `sortedList`。到此为止，咱们的函数就写完啦。

```kotlin
fun mergeSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {
        val middleIndex = if (list.size % 2 == 0) list.size / 2 else list.size / 2 + 1
        val leftList = list.subList(0, middleIndex)
        val rightList = list.subList(middleIndex, list.size)

        val sortedLeftList = mergeSort(leftList)
        val sortedRightList = mergeSort(rightList)

        val sortedList = mutableListOf<Double>()

        var tempIndexInSortedLeftList = 0
        var tempIndexInSortedRightList = 0

        while (tempIndexInSortedLeftList < sortedLeftList.size || tempIndexInSortedRightList < sortedRightList.size) {
            if (tempIndexInSortedLeftList == sortedLeftList.size) {
                sortedList.add(sortedRightList[tempIndexInSortedRightList++])
            } else if (tempIndexInSortedRightList == sortedRightList.size){
                sortedList.add(sortedLeftList[tempIndexInSortedLeftList++])
            } else if (sortedLeftList[tempIndexInSortedLeftList] < sortedRightList[tempIndexInSortedRightList]) {
                sortedList.add(sortedLeftList[tempIndexInSortedLeftList++])
            } else {
                sortedList.add(sortedRightList[tempIndexInSortedRightList++])
            }
        }

        sortedList
    }
}
```

## 测试

写完了以后，咱们来测试一下吧。

下面是最终的 `Main.kt` 文件。

```kotlin
import kotlin.math.roundToInt
import kotlin.random.Random
import kotlin.system.getTimeMillis

fun main() {
    val list = mutableListOf<Double>()
    val random = Random(getTimeMillis())

    for (i in 0..9) {
        list.add((random.nextDouble(1.0, 100.0) * 10).roundToInt() / 10.0 )
    }

    println("Original list is $list")
    println("Sorted list from mergeSort is ${mergeSort(list)}")
}

fun mergeSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {
        val middleIndex = if (list.size % 2 == 0) list.size / 2 else list.size / 2 + 1
        val leftList = list.subList(0, middleIndex)
        val rightList = list.subList(middleIndex, list.size)

        val sortedLeftList = mergeSort(leftList)
        val sortedRightList = mergeSort(rightList)

        val sortedList = mutableListOf<Double>()

        var tempIndexInSortedLeftList = 0
        var tempIndexInSortedRightList = 0

        while (tempIndexInSortedLeftList < sortedLeftList.size || tempIndexInSortedRightList < sortedRightList.size) {
            if (tempIndexInSortedLeftList == sortedLeftList.size) {
                sortedList.add(sortedRightList[tempIndexInSortedRightList++])
            } else if (tempIndexInSortedRightList == sortedRightList.size){
                sortedList.add(sortedLeftList[tempIndexInSortedLeftList++])
            } else if (sortedLeftList[tempIndexInSortedLeftList] < sortedRightList[tempIndexInSortedRightList]) {
                sortedList.add(sortedLeftList[tempIndexInSortedLeftList++])
            } else {
                sortedList.add(sortedRightList[tempIndexInSortedRightList++])
            }
        }

        sortedList
    }
}
```

执行结果，很完美地从小到大排序了。

```kotlin
Original list is [4.9, 91.0, 41.9, 34.5, 49.5, 16.6, 9.7, 93.5, 84.9, 63.3]
Sorted list from mergeSort is [4.9, 9.7, 16.6, 34.5, 41.9, 49.5, 63.3, 84.9, 91.0, 93.5]
```
