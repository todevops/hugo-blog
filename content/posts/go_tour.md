---
title: Go Tour 练习
date: 2019-12-15
categories: ["go"]
tags:
  - golang
---

Go 指南练习，题解
<!--more-->

## 练习 1：[循环与函数](https://tour.go-zh.org/flowcontrol/8)

为了练习函数与循环，我们来实现一个平方根函数：用牛顿法实现平方根函数。  
计算机通常使用循环来计算 x 的平方根。从某个猜测的值 z 开始，我们可以根据 z² 与 x 的近似度来调整 z，产生一个更好的猜测：  
`z -= (z*z - x) / (2*z)`  
重复调整的过程，猜测的结果会越来越精确，得到的答案也会尽可能接近实际的平方根。  
在提供的 func Sqrt 中实现它。无论输入是什么，对 z 的一个恰当的猜测为 1。 要开始，请重复计算 10 次并随之打印每次的 z 值。观察对于不同的值 `x（1、2、3 ...）`， 你得到的答案是如何逼近结果的，猜测提升的速度有多快。  
提示：用类型转换或浮点数语法来声明并初始化一个浮点数值：  
`z := 1.0`  
`z := float64(1)`  
然后，修改循环条件，使得当值停止改变（或改变非常小）的时候退出循环。观察迭代次数大于还是小于 10。 尝试改变 z 的初始猜测，如 x 或 x/2。你的函数结果与标准库中的 math.Sqrt 接近吗？  
（*注：* 如果你对该算法的细节感兴趣，上面的 `z² − x` 是 `z²` 到它所要到达的值（即 x）的距离， 除以的 2z 为 z² 的导数，我们通过 z² 的变化速度来改变 z 的调整量。 这种通用方法叫做牛顿法。 它对很多函数，特别是平方根而言非常有效。）

```go
package main

import (
	"fmt"
)

func Sqrt(x float64) float64 {
}

func main() {
	fmt.Println(Sqrt(2))
}
```

### 实现

```go
package main

import (
	"fmt"
	"math"
)

func Sqrt(x float64) float64 {
	z := 1.0	// 初始化z
	temp := 0.0	// 初始化temp，用于临时存放结果
	for {
		z = z - (z*z-x)/(2*z)	// 定义牛顿法公式求出新的z值
		fmt.Println(z)
		if math.Abs(z-temp) < 0.000000000000001 {	// 判断z是否达到精度
			break
		} else {
			temp = z	// 临时存放计算得出的z
		}
	}
	return z
}

func main() {
	fmt.Println("牛顿法:")
	fmt.Println(Sqrt(3))
	fmt.Println("math.Sqrt:")
	fmt.Println(math.Sqrt(3))
}

```

### 结果
```
牛顿法:
2
1.75
1.7321428571428572
1.7320508100147276
1.7320508075688772
1.7320508075688774
1.7320508075688774
math.Sqrt:
1.7320508075688772
```



## 练习 2：[切片](https://tour.go-zh.org/moretypes/18)

实现 Pic。它应当返回一个长度为 dy 的切片，其中每个元素是一个长度为 dx，元素类型为 uint8 的切片。当你运行此程序时，它会将每个整数解释为灰度值（好吧，其实是蓝度值）并显示它所对应的图像。

图像的选择由你来定。几个有趣的函数包括 (x+y)/2, x*y, x^y, x*log(y) 和 x%(y+1)。

（提示：需要使用循环来分配 [][]uint8 中的每个 []uint8；请使用 uint8(intValue) 在类型之间转换；你可能会用到 math 包中的函数。）

```go
package main

import "golang.org/x/tour/pic"

func Pic(dx, dy int) [][]uint8 {
}

func main() {
	pic.Show(Pic)
}

```

### 实现

```go
package main

import "golang.org/x/tour/pic"

func Pic(dx, dy int) [][]uint8 {
	a := make([][]uint8, dy)	// make 一个长度为 dy 的二维数组切片
  for x := range a {	// index, value := range a，x 为数组索引
		b := make([]uint8, dx)	// make 一个长度为 dx 的一维数组切片
		for y := range b {	// y 为数组索引
			b[y] = uint8(x % (y + 1))	// 修改数组 b 对应索引位置的值
		}
		a[x] = b	// 修改数组 a 对应索引位置的值
	}
	return a
}

func main() {
	pic.Show(Pic)
}

```



## 练习 3：[映射](https://tour.go-zh.org/moretypes/23)

实现 WordCount。它应当返回一个映射，其中包含字符串 s 中每个“单词”的个数。函数 wc.Test 会对此函数执行一系列测试用例，并输出成功还是失败。  
你会发现 strings.Fields 很有帮助。

```go
package main

import (
	"golang.org/x/tour/wc"
)

func WordCount(s string) map[string]int {
	return map[string]int{"x": 1}
}

func main() {
	wc.Test(WordCount)
}

```

### 实现

```go
package main

import (
	"strings"

	"golang.org/x/tour/wc"
)

func WordCount(s string) map[string]int {
	a := strings.Fields(s)	// 使用 strings.Fields 将字符串分割成数组
	map_s := make(map[string]int)	// 映射单词和出现次数
	for _, v := range a {	// 遍历 a 的值
		_, ok := map_s[v]	// 获取状态值
		if ok {	// 判断map中是否存在当前字符串为下标的值
			map_s[v]++	// 有则出现次数+1
		} else {
			map_s[v] = 1	// 无则=1
		}
	}
	return map_s
}

func main() {
	wc.Test(WordCount)
}

```

### 结果

```
PASS
 f("I am learning Go!") = 
  map[string]int{"Go!":1, "I":1, "am":1, "learning":1}
PASS
 f("The quick brown fox jumped over the lazy dog.") = 
  map[string]int{"The":1, "brown":1, "dog.":1, "fox":1, "jumped":1, "lazy":1, "over":1, "quick":1, "the":1}
PASS
 f("I ate a donut. Then I ate another donut.") = 
  map[string]int{"I":2, "Then":1, "a":1, "another":1, "ate":2, "donut.":2}
PASS
 f("A man a plan a canal panama.") = 
  map[string]int{"A":1, "a":2, "canal":1, "man":1, "panama.":1, "plan":1}
```



## 练习 4：[斐波纳契闭包](https://tour.go-zh.org/moretypes/26)

实现一个 `fibonacci` 函数，它返回一个函数（闭包），该闭包返回一个斐波纳契数列 `(0, 1, 1, 2, 3, 5, ...)`。

```go
package main

import "fmt"

// 返回一个“返回int的函数”
func fibonacci() func() int {
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}

```

### 实现

```go
package main

import "fmt"

// 返回一个“返回int的函数”
func fibonacci() func() int {
	a, b := 0, 1
	return func() int {
		temp := a
		a, b = b, a+b
		return temp
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}

```

### 结果

```
0
1
1
2
3
5
8
13
21
34
```



## 练习 5： [Stringer](https://tour.go-zh.org/methods/18)

1. 通过让 ```IPAddr``` 类型实现 ```fmt.Stringer``` 来打印点号分隔的地址。  
> 例如，```IPAddr{1, 2, 3, 4}``` 应当打印为 ```"1.2.3.4"```。

```go
package main

import "fmt"

type IPAddr [4]byte

// TODO: 给 IPAddr 添加一个 "String() string" 方法

func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}

```

### 实现

```go
package main

import "fmt"

type IPAddr [4]byte

// TODO: 给 IPAddr 添加一个 "String() string" 方法
func (ip IPAddr) String() string {
	return fmt.Sprintf("%v.%v.%v.%v", ip[0], ip[1], ip[2], ip[3])
}

func main() {
	hosts := map[string]IPAddr{
		"loopback":  {127, 0, 0, 1},
		"googleDNS": {8, 8, 8, 8},
	}
	for name, ip := range hosts {
		fmt.Printf("%v: %v\n", name, ip)
	}
}

```

### 结果
```
loopback: 127.0.0.1
googleDNS: 8.8.8.8
```



## 练习 6：[错误](https://tour.go-zh.org/methods/20)

Sqrt 接受到一个负数时，应当返回一个非 nil 的错误值。复数同样也不被支持。  
+ 创建一个新的类型
  `type ErrNegativeSqrt float64`
  并为其实现
  `func (e ErrNegativeSqrt) Error() string`
  方法使其拥有 `error` 值，通过 ```ErrNegativeSqrt(-2).Error()``` 调用该方法应返回 `"cannot Sqrt negative number: -2"`。  

> 注意: 在 Error 方法内调用 ```fmt.Sprint(e)``` 会让程序陷入死循环。可以通过先转换 e 来避免这个问题：```fmt.Sprint(float64(e))```。这是为什么呢？  
+ 修改 ```Sqrt``` 函数，使其接受一个负数时，返回 ```ErrNegativeSqrt``` 值。

```go
package main

import (
	"fmt"
)

func Sqrt(x float64) (float64, error) {
	return 0, nil
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}

```

### 实现

```go
package main

import (
	"fmt"
	"math"
)

type ErrNegativeSqrt float64

func (e ErrNegativeSqrt) Error() string {	// 重写 Error 函数
	return fmt.Sprintf("cannot Sqrt negative number:%v", float64(e))
}

func Sqrt(x float64) (float64, error) {
	if x < 0 {
		return 0, ErrNegativeSqrt(x)
	}
	return math.Sqrt(x), nil
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}

```

### 结果
```
1.4142135623730951 <nil>
0 cannot Sqrt negative number:-2
```



## 练习 7：[Reader](https://tour.go-zh.org/methods/22)

+ 实现一个 ```Reader``` 类型，它产生一个 ASCII 字符 'A' 的无限流。

```go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: 给 MyReader 添加一个 Read([]byte) (int, error) 方法

func main() {
	reader.Validate(MyReader{})
}

```

### 实现

```go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: 给 MyReader 添加一个 Read([]byte) (int, error) 方法
func (r MyReader) Read(b []byte) (int, error) {
	b[0] = 'A'
	return 1, nil
}

func main() {
	reader.Validate(MyReader{})
}

```



## 练习 8：[rot13Reader](https://tour.go-zh.org/methods/23)

有种常见的模式是一个 ```io.Reader``` 包装另一个 ```io.Reader```，然后通过某种方式修改其数据流。  
> 例如，```gzip.NewReader``` 函数接受一个 ```io.Reader```（已压缩的数据流）并返回一个同样实现了 ```io.Reader``` 的 ```*gzip.Reader```（解压后的数据流）。

+ 编写一个实现了 ```io.Reader``` 并从另一个 ```io.Reader``` 中读取数据的 ```rot13Reader```，通过应用 ```rot13``` 代换密码对数据流进行修改。

> ```rot13Reader``` 类型已经提供。实现 ```Read``` 方法以满足 ```io.Reader```。

```go
package main

import (
	"io"
	"os"
	"strings"
)

type rot13Reader struct {
	r io.Reader
}

func main() {
	s := strings.NewReader("Lbh penpxrq gur pbqr!")
	r := rot13Reader{s}
	io.Copy(os.Stdout, &r)
}

```

### 实现


```go
package main

import (
	"io"
	"os"
	"strings"
)

type rot13Reader struct {
	r io.Reader
}

// 实现 rot13 函数
func rot13(out byte) byte {		//字母转换
	switch {
	case out >= 'A' && out <= 'M' || out >= 'a' && out <= 'm':
		out += 13
	case out >= 'N' && out <= 'Z' || out >= 'n' && out <= 'z':
		out -= 13
	}
	return out
}

func (fz rot13Reader) Read(b []byte) (int, error) {
	n, e := fz.r.Read(b)
	for i := 0; i < n; i++ {
		b[i] = rot13(b[i])
	}
	return n, e
}

func main() {
	s := strings.NewReader("Lbh penpxrq gur pbqr!")
	r := rot13Reader{s}
	io.Copy(os.Stdout, &r)
}

```

### 结果

```
You cracked the code!%   
```



## 练习 9：[图像](https://tour.go-zh.org/methods/25)
还记得之前编写的图片生成器 吗？我们再来编写另外一个，不过这次它将会返回一个 image.Image 的实现而非一个数据切片。  
定义你自己的 Image 类型，实现必要的方法并调用 pic.ShowImage。  
Bounds 应当返回一个 image.Rectangle ，例如 image.Rect(0, 0, w, h)。  
ColorModel 应当返回 color.RGBAModel。  
At 应当返回一个颜色。上一个图片生成器的值 v 对应于此次的 color.RGBA{v, v, 255, 255}。

```go
package main

import "golang.org/x/tour/pic"

type Image struct{}

func main() {
	m := Image{}
	pic.ShowImage(m)
}
```

### 实现

```go
package main

import (
	"image"
	"image/color"

	"golang.org/x/tour/pic"
)

type Image struct{}

func (i Image) ColorModel() color.Model {	//实现 Image 包中颜色模式的方法
	return color.RGBAModel
}

func (i Image) Bounds() image.Rectangle {	//实现 Image 包中生成图片边界的方法
	return image.Rect(0, 0, 200, 200)
}

func (i Image) At(x, y int) color.Color {	// 实现 Image 包中生成图像某个点的方法
	return color.RGBA{uint8(x), uint8(y), uint8(255), uint8(255)}
}

func main() {
	m := Image{}
	pic.ShowImage(m)
}

```