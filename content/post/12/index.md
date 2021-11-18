---
title: 使用 Kotlin 语言实现一个快速排序算法
author: 星野玲
date: 2023-01-08T00:00:00+08:00
description: 今天小玲来教大家用 Kotlin 语言实现一个快速排序算法。将一个元素是 `Double` 类型的 `List` 从小到大排序。整个函数代码仅仅只有 11 行。
draft: false
tags:
  - Kotlin
  - 算法
 
---

今天小玲来教大家用 Kotlin 语言实现一个快速排序算法。将一个元素是 `Double` 类型的 `List` 从小到大排序。整个函数代码仅仅只有 11 行。快速排序算法的原理小玲就不再赘述了，这里小玲只讲实现。

## 安装 Intellij IDEA

咱们要采用 Intellij IDEA 这款 IDE 来编写 Kotlin 程序。如果你已经有了可以跳过。

因为目前小玲还在使用 Windows，所以给出的安装命令都是 Windows 的。如果你使用 macOS、Linux，还请自己学习如何安装。

这是使用 [Scoop]({{< ref "11.md" >}}) 安装 Intellij IDEA 的命令。

```powershell
scoop bucket add extras
scoop install extras/idea
```

开发 Kotlin Native 程序还是需要一个 JDK 的。如果你没有 JDK，可以安装一个。小玲就用 graalvm-17 这个 JDK 好了。

```powershell
scoop bucket add java
scoop install java/graalvm-jdk17
```

## 创建项目

因为咱们只使用 Kotlin 语言，所以要创建一个 Kotlin Native 项目。Kotlin Native 项目要在 Kotlin Multiplatform 里面创建。

{{< figure src="Snipaste_2023-01-07_23-44-45.png" position="center">}}

这一步咱们不需要修改，直接点击 `Finish` 即可。

{{< figure src="Snipaste_2023-01-07_23-44-52.png" position="center">}}

## 开始编写

创建完成后，`src/nativeMain/kotlin/Main.kt` 文件默认如下。

```kotlin
fun main() {
    println("Hello, Kotlin/Native!")
}
```

咱们在 `main()` 函数的下面写上快速排序函数的大概框架。首先它的函数名为 `quickSort`，接收一个类型为 `List<Double>`、名称为 `list` 的形参，返回值类型为 `List<Double>`。

```kotlin
fun quickSort(list: List<Double>): List<Double> {

}
```

咱们采用递归加分治的方法来写。因为是递归了，所以一定要有一个递归出口，否则会无限循环。

这里咱们就判断形参 `list` 有多少个元素，如果元素个数小于或等于 1 了，就直接返回这个形参。这也适用于用 `quickSort()` 函数处理只有一个元素的 `List` 的情况。因为只有 1 个元素了，所以就没有排序的必要了，直接返回它本身就好。

```kotlin
fun quickSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {

    }
}
```

`if` 语句的 `else` 块里就要写递归和分治的代码了。

咱们需要在这个 `List` 里先找一个基准点。把 `List` 里除了基准点所在的元素的剩余所有元素划分为两个部分。

一个部分里的所有元素都比基准点所在的元素小，这个部分咱们称作 `leftList`，意思是从小到大排序的话，这个部分都会在基准点左边。

另一个部分里的所有元素都比基准点所在的元素大或等于基准点所在的元素，这个部分咱们称作 `rightList`，意思是从小到大排序的话，这个部分都会在基准点的右边。

然后对 `leftList` 和 `rightList` 递归调用这个函数本身，它会不断地划分 `leftList` 的 `leftList`、`leftList` 的 `rightList`、`rightList` 的 `leftList`、`rightList` 的 `rightList`，直到不可划分为止。再按 `leftList`、基准点、`rightList` 的顺序合并 List，把合并的结果返回给上一层，直到不可合并为止。最后合并这个结果就是整个 `list` 形参排序之后的结果了。

下面是代码实现，现在咱们就写完 `quickSort()` 函数了。现在这个函数除了注释就只有 11 行。

```kotlin
fun quickSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {
        // 定义基准点位于 list 中的索引
        val pivotIndex = 0
        // 将 list 中除了基准点的剩余元素找出来，并存放在 listWithoutPivot 变量里。
        val listWithoutPivot = list.filterIndexed{it, _ -> it != pivotIndex}
        // 将 listWithoutPivot 里小于基准点的元素找出来，并存放在 leftList 变量里。
        val leftList = listWithoutPivot.filter { it < list[pivotIndex] }
        // 将 listWithoutPivot 里大于或等于基准点的元素找出来，并存放在 rightList 变量里。
        val rightList = listWithoutPivot.filter { it >= list[pivotIndex] }
        // 递归调用和合并
        listOf(quickSort(leftList), listOf(list[pivotIndex]), quickSort(rightList)).flatten()
    }
}
```

至于将哪个元素作为基准点其实都可以，它不会影响最终排序的结果。这里咱们就将 `list` 里的第 1 个元素作为基准点，你也可以将 `list` 的最后一个元素作为基准点。

```kotlin
val pivotIndex = list.size - 1
```

也可以将中间那个元素作为基准点。

```kotlin
val pivotIndex = if (list.size % 2 == 0) list.size / 2 - 1 else list.size / 2
```

不过要注意索引不要溢出，比如 `list` 里只有 2 个元素，取第 3 个元素作为基准点就溢出了。

## 测试

写完了以后，咱们来测试一下吧。把 `main()` 函数修改一下。生成 10 个小数点后有 1 位的小数，再把它们添加到一个 `List` 里，然后对这个 `List` 进行排序，观察排序的结果。

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
    println("Sorted list from quickSort is ${quickSort(list)}")
}

fun quickSort(list: List<Double>): List<Double> {
    return if (list.size <= 1) {
        list
    } else {
        val pivotIndex = 0
        val listWithoutPivot = list.filterIndexed{it, _ -> it != pivotIndex}
        val leftList = listWithoutPivot.filter { it < list[pivotIndex] }
        val rightList = listWithoutPivot.filter { it >= list[pivotIndex] }
        listOf(quickSort(leftList), listOf(list[pivotIndex]), quickSort(rightList)).flatten()
    }
}
```

执行结果，很完美地从小到大排序了。

```text
Original list is [64.8, 74.6, 96.7, 57.1, 58.3, 59.4, 18.8, 60.2, 4.9, 17.4]
Sorted list from quickSort is [4.9, 17.4, 18.8, 57.1, 58.3, 59.4, 60.2, 64.8, 74.6, 96.7]
```

## 结尾

好了，这篇文章就到这里。这是小玲首次写编程相关的文章。可能会有一些错误，如果有，还请指正，谢谢！
