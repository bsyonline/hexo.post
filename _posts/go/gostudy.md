---
title: GC
tags:
  - Interview
category:
  - JVM
author: bsyonline
lede: 没有摘要
date: 2019-11-04 14:30:42
thumbnail:
---

-



从国内镜像下载 [golang.google.cn/dl/](https://golang.google.cn/dl/) 下载后安装。

环境配置

windows

安装好 go 之后配置环境变量 GOROOT、GOPATH、PATH 。

修改 go env

```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```



linux 

```
tar -zxf go1.21.3.linux-amd64.tar.gz -C /usr/local
```

配置环境变量

```
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
```





函数类型

在 Go 语言中，函数也是一种类型，被称为函数类型，可以用 type 关键字声明。函数类型可以显式声明，也可以隐式声明。

显式声明函数类型的方式如下：

```go
type myFuncType1 func(a int, b int) string
```

其中，``myFuncType1`` 是函数类型的名称，`func` 是关键字，`a` 和 `b` 是函数的参数，`int` 是参数的类型，`string` 是函数返回值的类型。

隐式声明函数类型的方式是直接使用函数名作为类型名。例如：

```go
type myFuncType2 = func(int, int) string
```

其中 `func` 关键字和函数参数列表来隐式声明了一个名为 `myFuncType2` 的函数类型。这个函数类型接受两个 `int` 类型的参数，并返回一个 `string` 类型的值。

下面是一个示例：

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {
	type myFuncType1 func(a int, b int) string
	var f1, f2 myFuncType1
	f1 = add
	r1 := f1(1, 2)
	fmt.Printf("r1: %v\n", r1)
	f2 = max
	r2 := f2(1, 2)
	fmt.Printf("r2: %v\n", r2)

	type myFuncType2 = func(int, int) string
	var f3, f4 myFuncType2
	f3 = add
	r3 := f3(3, 4)
	fmt.Printf("r3: %v\n", r3)
	f4 = max
	r4 := f4(3, 4)
	fmt.Printf("r4: %v\n", r4)

}

func add(a int, b int) string {
	return strconv.Itoa(a) + " + " + strconv.Itoa(b) + " = " + strconv.Itoa(a+b)
}

func max(a int, b int) string {
	max := 0
	if a < b {
		max = b
	} else {
		max = a
	}
	return "max(" + strconv.Itoa(a) + ", " + strconv.Itoa(b) + ") = " + strconv.Itoa(max)
}
```



高阶函数

在Go语言中，函数可以作为函数的参数，也可以作为函数的返回值。通过将函数作为参数传递或返回函数，可以大大提高代码的可重用性和可组合性。

下面是高阶函数的示例：

1. 函数作为参数

   ```
   package main
   
   import "fmt"
   
   func main() {
   	r := f1("zhangsan", f2)
   	fmt.Printf("%v\n", r)
   }
   
   func f1(name string, f func(string) string) string {
   	return f2(name)
   }
   
   func f2(name string) string {
   	return "hello, " + name
   }
   ```

   上边的示例中，``f1`` 函数接收一个 `string` 类型的参数 `name` 和一个函数类型的参数 `f` ，并返回 `f2` 函数调用的结果，类型为 string 。 

2. 函数作为返回值

   ```
   package main
   
   import "fmt"
   
   func main() {
   	f := f3()
   	r2 := f("lisi")
   	fmt.Printf("r2: %v\n", r2)
   }
   
   func f2(name string) string {
   	return "hello, " + name
   }
   
   func f3() func(string) string {
   	return f2
   }
   ```

   上边的示例中，`f3` 函数的返回值是函数 `f2` 。

匿名函数

在Go语言中，匿名函数是一种没有函数名的函数。匿名函数可以直接赋值给变量。下面是一个使用匿名函数的示例：

```
package main  
  
import "fmt"  
  
func main() {  
    // 声明一个匿名函数并将其赋值给变量add  
    add := func(a, b int) int {  
        return a + b  
    }  
  
    // 调用匿名函数  
    result := add(3, 4)  
    fmt.Printf("add: %v\n", add) // 输出 7  
}
```

匿名函数也可以自己调用自己，下面是一个示例：

```
package main

import "fmt"

func main() {
	add := func(a int, b int) int {
		return a + b
	}(3, 4)
	fmt.Printf("add: %v\n", add)
}
```

闭包

在Go语言中，闭包（closure）是定义在函数内部的函数，闭包可以引用自身之外的变量，闭包是连接函数内部和外部的桥梁。下面是一个示例：

```
package main

import "fmt"

func main() {
	f := say("hello")
	r := f("zhangsan")
	fmt.Printf("r: %v\n", r)
	f1 := say("goodbye")
	r = f1("lisi")
	fmt.Printf("r: %v\n", r)
}

func say(op string) func(string) string {
	return func(name string) string {
		return op + ", " + name
	}
}
```

在上边的示例中，函数 `say` 是一个闭包，它包含了一个内部函数，这个函数中引用了 `say` 函数中的变量。每次调用 `say` 函数时，都会创建一个新的闭包实例，每个闭包都有自己的 `op` 变量。

```
package main

import "fmt"

func main() {
	add, sub := cal(100)
	i := add(100) // i=200
	i = add(100)  // i=300
	fmt.Printf("i: %v\n", i)
	i = sub(50) // i=250
	fmt.Printf("i: %v\n", i)
}

func cal(base int) (func(int) int, func(int) int) {
	add := func(a int) int {
		base += a
		return base
	}

	sub := func(a int) int {
		base -= a
		return base
	}
	return add, sub
}
```

上边的示例中，函数 cal(100) 返回两个函数 add 和 sub，在 add 和 sub 的生命周期内，base 的值是一直保持的。