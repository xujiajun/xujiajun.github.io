---
layout: post
title: Go切片与技巧（附图解）
disqus: y
---



以下摘自我公众号『太白技术』的内容：

![太白技术公号二维码1](https://user-images.githubusercontent.com/6065007/156370814-fa630b98-31f0-4de8-85fb-8075b535ff7c.jpg)



## 1 前言

切片是Go语言中最多被使用的数据结构之一。本文将介绍切片的创建、操作、表达式以及使用的技巧。

## 2 数组

slice类型是建立在 Go 的数组类型之上的抽象，因此要理解 slice，我们必须首先理解数组。

先看一段代码：

```go
	var a [4]int // 数组类型: 定义指定长度和元素类型
	a[0] = 1     // 数组a的第一个索引的值设置为1
	i := a[0]
	fmt.Println(i)    // 输出：1， 说明已经设置成功
	fmt.Println(a[1]) // 输出：0，说明数组不需要显式初始化，其元素本身为零
	fmt.Println(a[4]) // 报错：invalid array index 4 (out of bounds for 4-element array)
```
（代码来自 [Go Slices: usage and internals](https://go.dev/blog/slices-intro "Go Slices: usage and internals")）

说明：

* 1、代码中 `var a [4]int` 完成了数组的初始化，表示一个由四个整数组成的数组。
* 2、默认值为0。当我们打印`a[1]`的时候输出0。
* 3、设置数组索引的值，可以直接指定下标的索引进行赋值，例子中是`a[0] = 1`。
* 3、Go数组的索引和其他语言一样，从0开始，数组的大小是固定的，上面例子中最大是4的的大小，所以下标索引最大是3，所以当我们访问`a[4]`会报数组越界的错误。

`Go的数组是值类型`。不像C语言数组变量是指向第一个元素的指针，所以当我们把数组变量传递或者赋值的时候，其实是做copy的操作。比如下面的例子a赋值给b，修改a中的元素，并没有影响b中的元素：

```go
	a := [2]string{"johnny", "太白技术"} // 长度`2`，可以改成`...`，编译器会自动计算数组的长度
	b := a
	a[0] = "xujiajun"
	fmt.Println(a) // 输出：[xujiajun 太白技术]
	fmt.Println(b) // 输出：[johnny 太白技术]
```

为了避免复制，你可以传递一个指向数组的指针，例如：

```go
func double(arr *[3]int) {
	for i, num := range *arr {
		(*arr)[i] = num * 2
	}
}
a := [3]int{1, 2, 3}
double(&a)
fmt.Println(a) // [2 4 6]
```
（代码参考 [Effective Go](https://go.dev/doc/effective_go#arrays "Effective Go")）

## 3 切片的创建

由于数组需要固定长度，很多时候不是很灵活，而切片更灵活、更强大。`切片包含对底层数组的引用`，如果将一个切片分配给另一个切片，则两者都引用同一个数组。

切片底层的数据结构（src/runtime/slice.go）：

```go
type slice struct {
	array unsafe.Pointer // 指针指向底层数组
	len   int  // 切片长度
	cap   int  // 底层数组容量
}
```

切片的类型规范是[]T，其中T是切片元素的类型。`与数组类型不同，切片类型没有指定长度`。

创建切片有多种方式：`变量声明`、`切片字面量`、`make创建`、`new创建`、`从切片/数组截取`。

#### 1、变量声明
```go
	var s []byte
```
这种声明的切片变量，默认值是nil，容量和长度默认都是0。

```go
	var x []byte
	fmt.Println(cap(x))   // 0
	fmt.Println(len(x))   // 0
	fmt.Println(x == nil) // true
```

#### 2、切片字面量

当使用字面量来声明切片时，其语法与使用字面量声明数组非常相似：

```go
a := []string{"johnny", "太白技术"} // 这是切片声明，少了长度的指定
b := [2]string{"johnny", "太白技术"} // 这是数组声明，多了长度的指定
```

#### 3、make创建：

除了上面创建方式外，使用内置函数`make`可以指定**长度**和**容量**来创建（具体函数：func make([]T, len, cap) []T）

其中 T 代表要创建的切片的元素类型。该make函数采用类型、长度和可选容量。调用时，make分配一个数组并返回一个引用该数组的切片。

首先看下空切片：

```go
	y := make([]int, 0)
	fmt.Println(cap(y))   // 0
	fmt.Println(len(y))   // 0
	fmt.Println(y == nil) // false
``` 
下面这个例子中创建了长度为5，容量等于5的切片

```go
	var s []byte
	s = make([]byte, 5, 5) // 指定了长度和容量
	fmt.Println(s) // 输出：[0 0 0 0 0]

```
当容量参数被省略时，它默认为指定的长度：

```go
s := make([]byte, 5)
fmt.Println(s) // 输出也是 [0 0 0 0 0]
```

内置len和cap函数检查切片的长度和容量：

```go
	fmt.Println(len(s)) // 5
	fmt.Println(cap(s)) // 5
```

#### 4、使用new

使用new创建的切片是个nil切片。

```go
	n := *new([]int)
	fmt.Println(n == nil) // true
```

#### 5、从切片/数组截取

切片可以基于数组和切片来创建。这边会用到切片表达式，3.3会详细说明。
```go
	n := [5]int{1, 2, 3, 4, 5}
	n1 := n[1:]     // 从n数组中截取
	fmt.Println(n1) // [2 3 4 5]
	n2 := n1[1:]    // 从n1切片中截取
	fmt.Println(n2) // [3 4 5]
```
切片与原数组或切片是共享底层空间的，接着上面例子，把n2的元素修改之后，会影响原切片和数组：

```go
	n2[1] = 6 // 修改n2，会影响原切片和数组
	fmt.Println(n1) // [2 3 6 5]
	fmt.Println(n2) // [3 6 5]
	fmt.Println(n)  // [1 2 3 6 5]
```

## 4 切片操作
内置函数`append()`用于向切片中追加元素。

```go
	n := make([]int, 0)
	n = append(n, 1)                 // 添加一个元素
	n = append(n, 2, 3, 4)           // 添加多个元素
	n = append(n, []int{5, 6, 7}...) // 添加一个切片
	fmt.Println(n)                   // [1 2 3 4 5 6 7]
```
当append操作的时候，切片容量如果不够，会触发扩容，接着上面的例子：

```go
	fmt.Println(cap(n)) // 容量等于8
	n = append(n, []int{8, 9, 10}...)
	fmt.Println(cap(n)) // 容量等于16，发生了扩容
```
当一开始容量是8，后面追加了切片[]int{8, 9, 10}之后，容量变成了16。关于详细的扩容机制，我们后面文章再聊。


## 5 切片表达式

Go语言提供了两种切片的表达式：

* 1、简单表达式
* 2、扩展表达式

### 1、简单的表达式

简单切片表达式的格式`[low:high]`。

如下面例子，n为一个切片，当用这个表达式`[1:4]`表示的是左闭右开`[low, high)区间`截取一个新的切片（例子中结果是[2 3 4]），切片被截取之后，截取的长度是`high-low`。 

```go
	n := []int{1, 2, 3, 4, 5, 6}
 fmt.Println(n[1:4])      // [2 3 4]
```

切片表达式的开始low和结束索引high是可选的；它们分别默认为零和切片的长度：
```go
n := []int{1, 2, 3, 4, 5, 6}
fmt.Println(n[:4])      // [1 2 3 4]，:前面没有值，默认表示0
fmt.Println(n[1:])       // [2 3 4 5 6]，:后面没有值，默认表示切片的长度
```

#### 边界问题

* 1、当n为数组或字符串表达式n[low:high]中low和high的取值关系：
> 0 <= low <=high <= len(n)
* 2、当n为切片的时候，表达式n[low:high]中high最大值变成了cap(n)，low和high的取值关系：
> 0 <= low <=high <= cap(n)

不满足以上条件会发送越界panic。

#### 截取字符串

注意截取字符串，产生的也是新的字符串。

```go
	s := "hello taibai"
	s1 := s[6:]
	fmt.Println(s1)                 // taibai
	fmt.Println(reflect.TypeOf(s1)) // string
```

### 2、扩展表达式

简单表达式生产的新切片与原数组或切片会共享底层数组，虽然避免了copy，但是会带来一定的风险。
下面这个例子当新的n1切片append添加元素的时候，覆盖了原来n的索引位置4的值，导致你的程序可能是非预期的，从而产生不良的后果。

```
	n := []int{1, 2, 3, 4, 5, 6}
	n1 := n[1:4]
	fmt.Println(n)       // [1 2 3 4 5 6]
	fmt.Println(n1)      // [2 3 4]
	n1 = append(n1, 100) // 把n的索引位置4的值从原来的5变成了100
	fmt.Println(n)       // [1 2 3 4 100 6]
```
[Go 1.2](https://tip.golang.org/doc/go1.2#three_index "go1.2#three_index")中提供了一种可以限制新切片容量的表达式：

> n[low:high:max]

max表示新生成切片的容量，新切片容量等于`max-low`，表达式中low、high、max关系：
 
> 0 <= low <= high <= max <= cap(n)
 

继续刚才的例子，当计算`n[1:4]`的容量，用cap得到值等于5，用扩展表达式`n[1:4:5]`，用cap重新计算得到新的容量值（5-1）等于4：

```go
	fmt.Println(cap(n[1:4])) // 5
	fmt.Println(cap(n[1:4:5])) // 4
```

#### 关于容量

n[1:4]的长度是3好理解（4-1），容量为什么是5？

<b>因为切片n[1:4]和切片n是共享底层空间</b>，所以它的容量并不等于他的长度3，根据1等于索引1的位置（等于值2），从值2这个元素开始到末尾元素6，共5个，所以`n[1:4]`容量是5。

如果append超过切片的长度会重新生产一个全新的切片，不会覆盖原来的：
```
	n2 := n[1:4:5]         // 长度等于3，容量等于4
	fmt.Printf("%p\n", n2) // 0xc0000ac068
	n2 = append(n2, 5)
	fmt.Printf("%p\n", n2) // 0xc0000ac068
	n2 = append(n2, 6)
	fmt.Printf("%p\n", n2) // 地址发生改变，0xc0000b8000
```

## 6 切片技巧

Go在Github的官方Wiki上介绍了切片的技巧[SliceTricks](https://github.com/golang/go/wiki/SliceTricks "SliceTricks")，另外这个项目[Go Slice Tricks Cheat Sheet](https://ueokande.github.io/go-slice-tricks/ "Go Slice Tricks Cheat Sheet")基于SliceTricks做了一系列的图，比较直观。**Go Slice Tricks Cheat Sheet 做的图有些缺失，我也做了补充**

**说明：官方Wiki最后更新时间是2022.1.22，下文是基于这个版本来描述。**

#### AppendVector

这个是添加一个切片的操作，上面我们在切片操作中已经介绍过。

```go
a = append(a, b...)
```

#### Copy

这边给我们展示了三种copy的写法：

```go
b := make([]T, len(a))
copy(b, a)

// 效率一般比上面的写法慢，但是如果有更多，他们效率更好
b = append([]T(nil), a...)
b = append(a[:0:0], a...)

// 这个实现等价于make+copy。
// 但在Go工具链v1.16上实际上会比较慢。
b = append(make([]T, 0, len(a)), a...)
```

### Cut

截掉切片`[i,j）`之间的元素：

```go
a = append(a[:i], a[j:]...)
```

### Cut（GC）

上面的`Cut`如果元素是指针的话，会存在内存泄露，所以我们要对删除的元素设置`nil`，等待GC。

```go

copy(a[i:], a[j:])
for k, n := len(a)-j+i, len(a); k < n; k++ {
	a[k] = nil // or the zero value of T
}
a = a[:len(a)-j+i]
```


### Delete

删除索引位置i的元素：

```go
a = append(a[:i], a[i+1:]...)
// or
a = a[:i+copy(a[i:], a[i+1:])]
```

### Delete（GC）

删除索引位置i的元素：

```go
copy(a[i:], a[i+1:])
a[len(a)-1] = nil // or the zero value of T
a = a[:len(a)-1]
```

### Delete without preserving order

删除索引位置i的元素，把最后一位放到索引位置i上，然后把最后一位元素删除。这种方式底层并没有发生复制操作。

```go
a[i] = a[len(a)-1] 
a = a[:len(a)-1]
```

### Delete without preserving order（GC）

上面的删除操作，元素是一个指针的类型或结构体指针字段，会存在最后一个元素不能被GC掉，造成泄露，把末尾的元素设置nil，等待GC。

```go
a[i] = a[len(a)-1]
a[len(a)-1] = nil
a = a[:len(a)-1]
```

### Expand

这个本质上是多个append的组合操作。

```go
a = append(a[:i], append(make([]T, j), a[i:]...)...)

```

### Extend

用新列表扩展原来的列表

```go
a = append(a, make([]T, j)...)
```

### Filter (in place)

下面代码演示原地删除Go切片元素：

```go
n := 0
for _, x := range a {
	if keep(x) {
		a[n] = x
		n++
	}
}
a = a[:n]
```


### Insert


```go
a = append(a[:i], append([]T{x}, a[i:]...)...)
```
第二个`append`会产生新的切片，产生一次copy，可以用以下代码方式，可免去第二次的copy：

 
 
```go
s = append(s, 0 /* 先添加一个0值*/)
copy(s[i+1:], s[i:])
s[i] = x
```

### InsertVector

下面代码演示插入向量（封装了动态大小数组的顺序容器）的实现：

```go
a = append(a[:i], append(b, a[i:]...)...)
```

```go
func Insert(s []int, k int, vs ...int) []int {
	if n := len(s) + len(vs); n <= cap(s) {
		s2 := s[:n]
		copy(s2[k+len(vs):], s[k:])
		copy(s2[k:], vs)
		return s2
	}
	s2 := make([]int, len(s) + len(vs))
	copy(s2, s[:k])
	copy(s2[k:], vs)
	copy(s2[k+len(vs):], s[k:])
	return s2
}

a = Insert(a, i, b...)
```

### Push

```go
a = append(a, x)
```
### Pop
```
 x, a = a[len(a)-1], a[:len(a)-1]
```

### Push Front/Unshift

```go
a = append([]T{x}, a...)
```

### Pop Front/Shift

```go
x, a = a[0], a[1:]

```

## 7 切片额外技巧

### Filtering without allocating

下面例子演示数据过滤的时候，b基于原来的a存储空间来操作，并没有重新生成新的存储空间。

```go
b := a[:0]
for _, x := range a {
	if f(x) {
		b = append(b, x)
	}
}
```

为了让截取之后没有使用的存储被GC掉，需要设置成nil：

```go
for i := len(b); i < len(a); i++ {
	a[i] = nil // or the zero value of T
}
```

### Reversing

反转操作演示：

```go
for i := len(a)/2-1; i >= 0; i-- {
	opp := len(a)-1-i
	a[i], a[opp] = a[opp], a[i]
}
```

还有一种方法：

```go
for left, right := 0, len(a)-1; left < right; left, right = left+1, right-1 {
	a[left], a[right] = a[right], a[left]
}
```

### Shuffling

洗牌算法。算法思想就是从原始数组中随机抽取一个新的数字到新数组中。


```go
for i := len(a) - 1; i > 0; i-- {
    j := rand.Intn(i + 1)
    a[i], a[j] = a[j], a[i]
}
```
go1.10之后有内置函数[Shuffle](https://pkg.go.dev/math/rand#Shuffle "Shuffle")
 
### Batching with minimal allocation
做批处理大的切片的时候，这个技巧可以了解下：

```go
actions := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
batchSize := 3
batches := make([][]int, 0, (len(actions) + batchSize - 1) / batchSize)

for batchSize < len(actions) {
    actions, batches = actions[batchSize:], append(batches, actions[0:batchSize:batchSize])
}
batches = append(batches, actions)

// 结果：
// [[0 1 2] [3 4 5] [6 7 8] [9]]

```

### In-place deduplicate (comparable)

删除有序数组中的重复项：

```go
import "sort"

in := []int{3,2,1,4,3,2,1,4,1} // any item can be sorted
sort.Ints(in)
j := 0
for i := 1; i < len(in); i++ {
	if in[j] == in[i] {
		continue
	}
	j++
	// preserve the original data
	// in[i], in[j] = in[j], in[i]
	// only set what is required
	in[j] = in[i]
}
result := in[:j+1]
fmt.Println(result) // [1 2 3 4]
```

### Move to front, or prepend if not present, in place if possible.

下面代码演示移动指定元素到头部：

```go
// moveToFront moves needle to the front of haystack, in place if possible.
func moveToFront(needle string, haystack []string) []string {
	if len(haystack) != 0 && haystack[0] == needle {
		return haystack
	}
	prev := needle
	for i, elem := range haystack {
		switch {
		case i == 0:
			haystack[0] = needle
			prev = elem
		case elem == needle:
			haystack[i] = prev
			return haystack
		default:
			haystack[i] = prev
			prev = elem
		}
	}
	return append(haystack, prev)
}

haystack := []string{"a", "b", "c", "d", "e"} // [a b c d e]
haystack = moveToFront("c", haystack)         // [c a b d e]
haystack = moveToFront("f", haystack)         // [f c a b d e]
```

### Sliding Window

下面实现根据size的滑动窗口输出：

```go
func slidingWindow(size int, input []int) [][]int {
	// returns the input slice as the first element
	if len(input) <= size {
		return [][]int{input}
	}

	// allocate slice at the precise size we need
	r := make([][]int, 0, len(input)-size+1)

	for i, j := 0, size; j <= len(input); i, j = i+1, j+1 {
		r = append(r, input[i:j])
	}

	return r
}

func TestSlidingWindow(t *testing.T) {
	result := slidingWindow(2, []int{1, 2, 3, 4, 5})
	fmt.Println(result) // [[1 2] [2 3] [3 4] [4 5]]
}
```


## 总结
* 1、Go切片本质上是一个结构体，保存了其长度、底层数组的容量、底层数组的指针。
* 2、Go切片创建方式比较多样：`变量声明`、`切片字面量`、`make创建`、`new创建`、`从切片/数组截取`。
* 3、Go切片使用`len()`计算长度、`cap()`计算容量、`append()`来添加元素。
* 4、Go切片相比数组更灵活，有很多技巧，也正因为灵活，容易发生类似内存泄露的问题，需要注意。
