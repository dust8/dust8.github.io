---
title: a-tour-of-go-exercise-slices.go实现
date: "2014/12/13 20:00:00"
tags:
  - go
---

我用的是官网中文网站练习的。<br>
中文练习网站： <http://go-tour-zh.appspot.com/><br>
中文翻译害死人，不知道是不是机翻的。<br>
我按我的理解翻译一遍。

# 意译

练习：`Slices`

实现 `Pic`。 它返回一个长度为 `dy` 的 `slice` 类型。<br>
而这个长度为 `dy` 的 `slice` 类型里面的每一个元素是一个长度为 `dx` 的 `slice` 类型，<br>
它的值是一个 `8` 位的无符号整数。 当你运行程序，它将会显示你的图像，<br>
这个图像是由你返回的那个 `8` 位无符号整数作为图像的灰度(或者是蓝度)值生成的。

由你决定函数去生成图片。<br>
下面是一些有趣的函数，有 `(x+y)/2` , `x*y`, 和 `x^y` (最后一个函数可以用 `math.Pow` 实现计算方值)。

（你需要使用循环来分配 `[][]uint8` 中的每一个 `[]uint8`。）<br>
（使用 `uint8(intValue)` 在类型之间进行转换。）

# 解题

本题的本意其实就是要生成一个 2 维的整数数组。<br>
那些函数只是决定生成的图像的样子，并不是考察的目的所在。

代码实现如下：

```
package main

import "code.google.com/p/go-tour/pic"

func Value(x, y int) uint8 {
    // 有趣的函数有 (x+y)/2, x*y, and x^y
    return uint8((x + y) / 2)
}

func Pic(dx, dy int) [][]uint8 {
    ret := make([][]uint8, dy)
    for i := 0; i < dy; i++ {
        ret[i] = make([]uint8, dx)
        for j := 0; j < dx; j++ {
            ret[i][j] = Value(i, j)
        }
    }
    return ret
}

func main() {
    pic.Show(Pic)
}
```

# 有趣的函数生成的图像

1. `(x+y)/2`<br>
   ![(x+y)/2](/blog/assert/2014-12-13-exercise-slices-1.png)

2. `x*y`<br>
   ![x*y](/blog/assert/2014-12-13-exercise-slices-2.png)

3. `x^y`<br>
   ![x^y](/blog/assert/2014-12-13-exercise-slices-3.png)
