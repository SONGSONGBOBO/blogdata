---
title: Golang学习笔记note
date: 2018-10-04 15:27:52
tags:
 - Golang
 - 学习笔记

toc: ture
---
### 总览
* =和:=
* 切片的生长（copy and append 函数）
* 位运算符
* 运算符优先级
* map(集合)
* Comma-ok

<!--more-->

## =和:=

```bash
# = 使用必须使用先var声明例如：
var a
a=100 或 var b = 100 或
var c int = 100
# := 是声明并赋值，并且系统自动推断类型，不需要var关键字
d := 100
```

**:= 只能用在函数内部 **

## 切片的生长（copy and append 函数）
要增加切片的容量必须创建一个新的、更大容量的切片，然后将原有切片的内容复制到新的切片。 整个技术是一些支持动态数组语言的常见实现。下面的例子将切片 s 容量翻倍，先创建一个2倍 容量的新切片 t ，复制 s 的元素到 t ，然后将 t 赋值给 s 

```bash
t := make([]byte, len(s), (cap(s)+1)*2) // +1 in case cap(s) == 0
for i := range s {
        t[i] = s[i]
}
s = t
```
循环中复制的操作可以由 copy 内置函数替代。copy 函数将源切片的元素复制到目的切片。 它返回复制元素的数目。

```bash
func copy(dst, src []T) int
```

copy 函数支持不同长度的切片之间的复制（它只复制较短切片的长度个元素）。 此外， copy 函数可以正确处理源和目的切片有重叠的情况。

使用 copy 函数，我们可以简化上面的代码片段：

```bash
t := make([]byte, len(s), (cap(s)+1)*2)
copy(t, s)
s = t
```

一个常见的操作是将数据追加到切片的尾部。下面的函数将元素追加到切片尾部， 必要的话会增加切片的容量，最后返回更新的切片：

```bash
func AppendByte(slice []byte, data ...byte) []byte {
    m := len(slice)
    n := m + len(data)
    if n > cap(slice) { // if necessary, reallocate
        // allocate double what's needed, for future growth.
        newSlice := make([]byte, (n+1)*2)
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:n]
    copy(slice[m:n], data)
    return slice
}
```

下面是 AppendByte 的一种用法：

```bash
p := []byte{2, 3, 5}
p = AppendByte(p, 7, 11, 13)
# p == []byte{2, 3, 5, 7, 11, 13}
```

类似 AppendByte 的函数比较实用，因为它提供了切片容量增长的完全控制。 根据程序的特点，可能希望分配较小的活较大的块，或则是超过某个大小再分配。

但大多数程序不需要完全的控制，因此Go提供了一个内置函数 append ， 用于大多数场合；它的函数签名：

```bash
func append(s []T, x ...T) []T
append 函数将 x 追加到切片 s 的末尾，并且在必要的时候增加容量。

a := make([]int, 1)
# a == []int{0}
a = append(a, 1, 2, 3)
# a == []int{0, 1, 2, 3}
```

如果是要将一个切片追加到另一个切片尾部，需要使用 ... 语法将第2个参数展开为参数列表。

```bash
a := []string{"John", "Paul"}
b := []string{"George", "Ringo", "Pete"}
a = append(a, b...) // equivalent to "append(a, b[0], b[1], b[2])"
# a == []string{"John", "Paul", "George", "Ringo", "Pete"}
```

由于切片的零值 nil 用起来就像一个长度为零的切片，我们可以声明一个切片变量然后在循环 中向它追加数据：

```bash
# Filter returns a new slice holding only
# the elements of s that satisfy f()
func Filter(s []int, fn func(int) bool) []int {
    var p []int // == nil
    for _, v := range s {
        if fn(v) {
            p = append(p, v)
        }
    }
    return p
}
```

可能的“陷阱”
正如前面所说，切片操作并不会复制底层的数组。整个数组将被保存在内存中，直到它不再被引用。 有时候可能会因为一个小的内存引用导致保存所有的数据。

例如， FindDigits 函数加载整个文件到内存，然后搜索第一个连续的数字，最后结果以切片方式返回。

```bash
var digitRegexp = regexp.MustCompile("[0-9]+")

func FindDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    return digitRegexp.Find(b)
}
```

这段代码的行为和描述类似，返回的 []byte 指向保存整个文件的数组。因为切片引用了原始的数组， 导致 GC 不能释放数组的空间；只用到少数几个字节却导致整个文件的内容都一直保存在内存里。

要修复整个问题，可以将感兴趣的数据复制到一个新的切片中：

```bash
func CopyDigits(filename string) []byte {
    b, _ := ioutil.ReadFile(filename)
    b = digitRegexp.Find(b)
    c := make([]byte, len(b))
    copy(c, b)
    return c
}
```

可以使用 append 实现一个更简洁的版本。这留给读者作为练习。

## 位运算符

{% asset_img weiyunsuan.png test %}

```bash
package main

import "fmt"

func main() {

   var a uint = 60    /* 60 = 0011 1100 */  
   var b uint = 13    /* 13 = 0000 1101 */
   var c uint = 0          

   c = a & b       /* 12 = 0000 1100 */ 
   fmt.Printf("第一行 - c 的值为 %d\n", c )

   c = a | b       /* 61 = 0011 1101 */
   fmt.Printf("第二行 - c 的值为 %d\n", c )

   c = a ^ b       /* 49 = 0011 0001 */
   fmt.Printf("第三行 - c 的值为 %d\n", c )

   c = a << 2     /* 240 = 1111 0000 */ 
   fmt.Printf("第四行 - c 的值为 %d\n", c )

   c = a >> 2     /* 15 = 0000 1111 */
   fmt.Printf("第五行 - c 的值为 %d\n", c )
}
# 第一行 - c 的值为 12
# 第二行 - c 的值为 61
# 第三行 - c 的值为 49
# 第四行 - c 的值为 240
# 第五行 - c 的值为 15
```

## 运算符优先级
有些运算符拥有较高的优先级，二元运算符的运算方向均是从左至右。下表列出了所有运算符以及它们的优先级，由上至下代表优先级由高到低：

{% asset_img yunsuanyouxianji.png test %}

当然，你可以通过使用括号来临时提升某个表达式的整体运算优先级。

```bash
package main

import "fmt"

func main() {
   var a int = 20
   var b int = 10
   var c int = 15
   var d int = 5
   var e int;

   e = (a + b) * c / d;      // ( 30 * 15 ) / 5
   fmt.Printf("(a + b) * c / d 的值为 : %d\n",  e );

   e = ((a + b) * c) / d;    // (30 * 15 ) / 5
   fmt.Printf("((a + b) * c) / d 的值为  : %d\n" ,  e );

   e = (a + b) * (c / d);   // (30) * (15/5)
   fmt.Printf("(a + b) * (c / d) 的值为  : %d\n",  e );

   e = a + (b * c) / d;     //  20 + (150/5)
   fmt.Printf("a + (b * c) / d 的值为  : %d\n" ,  e );  
}

# (a + b) * c / d 的值为 : 90
# ((a + b) * c) / d 的值为  : 90
# (a + b) * (c / d) 的值为  : 90
# a + (b * c) / d 的值为  : 50
```

## map
Map 是一种无序的键值对的集合。Map 最重要的一点是通过 key 来快速检索数据，key 类似于索引，指向数据的值。
Map 是一种集合，所以我们可以像迭代数组和切片那样迭代它。不过，Map 是无序的，我们无法决定它的返回顺序，这是因为 Map 是使用 hash 表来实现的。

### 定义 Map
可以使用内建函数 make 也可以使用 map 关键字来定义 Map:

```bash
# 声明变量，默认 map 是 nil 
var map_variable map[key_data_type]value_data_type

# 使用 make 函数 
map_variable := make(map[key_data_type]value_data_type)
```

如果不初始化 map，那么就会创建一个 nil map。nil map 不能用来存放键值对

### 实例
下面实例演示了创建和使用map:

```bash
package main

import "fmt"

func main() {
    var countryCapitalMap map[string]string /*创建集合 */
    countryCapitalMap = make(map[string]string)

    /* map插入key - value对,各个国家对应的首都 */
    countryCapitalMap [ "France" ] = "Paris"
    countryCapitalMap [ "Italy" ] = "罗马"
    countryCapitalMap [ "Japan" ] = "东京"
    countryCapitalMap [ "India " ] = "新德里"

    /*使用键输出地图值 */ for country := range countryCapitalMap {
        fmt.Println(country, "首都是", countryCapitalMap [country])
    }

    /*查看元素在集合中是否存在 */
    captial, ok := countryCapitalMap [ "美国" ] /*如果确定是真实的,则存在,否则不存在 */
    /*fmt.Println(captial) */
    /*fmt.Println(ok) */
    if (ok) {
        fmt.Println("美国的首都是", captial)
    } else {
        fmt.Println("美国的首都不存在")
    }
}

# France 首都是 Paris
# Italy 首都是 罗马
# Japan 首都是 东京
# India  首都是 新德里
# 美国的首都不存在
```

### delete() 函数
delete() 函数用于删除集合的元素, 参数为 map 和其对应的 key。实例如下：

```bash
package main

import "fmt"

func main() {
    # 创建map 
    countryCapitalMap := map[string]string{"France": "Paris", "Italy": "Rome", "Japan": "Tokyo", "India": "New delhi"}

    fmt.Println("原始地图")

    # 打印地图 
    for country := range countryCapitalMap {
        fmt.Println(country, "首都是", countryCapitalMap [ country ])
    }

    # 删除元素 
    delete(countryCapitalMap, "France")
    fmt.Println("法国条目被删除")

    fmt.Println("删除元素后地图")

    # 打印地图
    for country := range countryCapitalMap {
        fmt.Println(country, "首都是", countryCapitalMap [ country ])
    }
}

# 原始地图
# India 首都是 New delhi
# France 首都是 Paris
# Italy 首都是 Rome
# Japan 首都是 Tokyo
# 法国条目被删除
# 删除元素后地图
# Italy 首都是 Rome
# Japan 首都是 Tokyo
# India 首都是 New delhi
```

## Comma-ok
Comma-ok断言的语法是：value, ok := element.(T)。element必须是接口类型的变量，T是普通类型。如果断言失败，ok为false，否则ok为true并且value为变量的值。来看个例子：

```bash
package main

import (
    "fmt"
)

type Html []interface{}

func main() {
    html := make(Html, 5)
    html[0] = "div"
    html[1] = "span"
    html[2] = []byte("script")
    html[3] = "style"
    html[4] = "head"
    for index, element := range html {
        if value, ok := element.(string); ok {
            fmt.Printf("html[%d] is a string and its value is %s\n", index, value)
        } else if value, ok := element.([]byte); ok {
            fmt.Printf("html[%d] is a []byte and its value is %s\n", index, string(value))
        }
    }
}
```

其实Comma-ok断言还支持另一种简化使用的方式：value := element.(T)。但这种方式不建议使用，因为一旦element.(T)断言失败，则会产生运行时错误。如：

```bash
package main

import (
    "fmt"
)

func main() {
    var val interface{} = "good"
    fmt.Println(val.(string))
    // fmt.Println(val.(int))
}
```

以上的代码中被注释的那一行会运行时错误。这是因为val实际存储的是string类型，因此断言失败。
还有一种转换方式是switch测试。既然称之为switch测试，也就是说这种转换方式只能出现在switch语句中。可以很轻松的将刚才用Comma-ok断言的例子换成由switch测试来实现：

```bash
package main

import (
    "fmt"
)

type Html []interface{}

func main() {
    html := make(Html, 5)
    html[0] = "div"
    html[1] = "span"
    html[2] = []byte("script")
    html[3] = "style"
    html[4] = "head"
    for index, element := range html {
        switch value := element.(type) {
        case string:
            fmt.Printf("html[%d] is a string and its value is %s\n", index, value)
        case []byte:
            fmt.Printf("html[%d] is a []byte and its value is %s\n", index, string(value))
        case int:
            fmt.Printf("invalid type\n")
        default:
            fmt.Printf("unknown type\n")
        }
    }
}
```














