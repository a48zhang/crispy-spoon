---
title: Golang中的异常处理
date: 2023-05-03 00:14:09
categories: Programming Skills
tags: golang
excerpt: Defer, Panic, Recover
---

# 异常处理

{% asset_img 1683112281963.jpeg %}

你或许受够了丑陋的 `if  err != nil {...}`，那么来聊一聊不太常见的`defer`,`panic`,`recover()`

## Defer

在函数结束之前执行指定的函数。

### 实现

下面是`_defer`结构的源代码。不细说，因为我也不大懂。

```go
// A _defer holds an entry on the list of deferred calls.
// If you add a field here, add code to clear it in deferProcStack.
// This struct must match the code in cmd/compile/internal/ssagen/ssa.go:deferstruct
// and cmd/compile/internal/ssagen/ssa.go:(*state).call.
// Some defers will be allocated on the stack and some on the heap.
// All defers are logically part of the stack, so write barriers to
// initialize them are not required. All defers must be manually scanned,
// and for heap defers, marked.
type _defer struct {
	// ...
    
	sp        uintptr // sp at time of defer
	pc        uintptr // pc at time of defer
	fn        func()  // can be nil for open-coded defers
	_panic    *_panic // panic that is running defer
	link      *_defer // next defer on G; can point to either heap or stack!

	// ...
}
```

defer chain 是一条单向链表（看上面的 `link`）。很多文章说`defer`这里是一个**栈**，其实我们在聊同一件事。

```mermaid
graph LR
A[goroutine]
B[defer func 3]
C[defer func 2]
D[defer func 1]
A --> B --> C --> D
```

```go
package main

func f() {
    defer f()
}

func main() {
    f()
}

// runtime: goroutine stack exceeds 1000000000-byte limit
// runtime: sp=0xc0200f9380 stack=[0xc0200f8000, 0xc0400f8000]
// fatal error: stack overflow
```

### 注意

1. `defer` 语句中函数的参数是在 `defer` 语句声明时确定的。
2. 当函数返回后，defer 声明的函数按照后进先出 (Last In First Out) 的顺序执行。
3. 如果函数返回的是命名返回值（named return values），`defer` 声明的函数调用可以读写这个返回值。（见小练习！`foo3()`）
4. `os.Exit()`会造成程序立即终止，此时`defer`不会被执行。

### defer 与 return

Go语言的函数中`return`语句在底层并不是原子操作，它分为**给返回值赋值**和**RET**两步。而`defer`语句执行的时机就**在返回值赋值操作后，RET指令执行前**。

```mermaid
graph LR
A[返回值 = x]
B[执行defer]
C[ret]
A --> B --> C 
```


> 可容纳 64 位（包括 **`__m64`** 类型）的标量返回值是通过 RAX 返回的。  非标量类型（包括浮点类型、双精度类型和向量类型，例如 [`__m128`](https://learn.microsoft.com/zh-cn/cpp/cpp/m128?view=msvc-170)、[`__m128i`](https://learn.microsoft.com/zh-cn/cpp/cpp/m128i?view=msvc-170)、[`__m128d`](https://learn.microsoft.com/zh-cn/cpp/cpp/m128d?view=msvc-170)）以 XMM0 的形式返回。 
>
>  [Microsoft Docs]([x64 调用约定 | Microsoft Learn](https://learn.microsoft.com/zh-cn/cpp/build/x64-calling-convention?view=msvc-170#return-values))

### 小练习！

```go
package main

import "fmt"

func foo1(i int) int {
	func() { i++ }()
	fmt.Println("foo1:", i)
	defer func() { i++ }()
	return i
}

func foo2(i int) int {
	defer func(i *int) { *i++ }(&i)
	fmt.Println("foo2:", i)
	return i
}

func foo3(i int) (x int) {
	defer func() { x++ }()
	fmt.Println("foo3:", x)
	return i
}

func main() {
	i := 1
	fmt.Printf("main: %d\n", foo1(i))
	fmt.Printf("main: %d\n", foo2(i))
	fmt.Printf("main: %d\n", foo3(i))

	return
}

// Output:
// foo1: 2
// main: 2
// foo2: 1
// main: 1
// foo3: 0
// main: 2

// Why?

```

> 提示：你可能需要了解**命名返回值函数**。

### `defer`的用途

* 释放资源（数据库连接，文件句柄等）
* 释放**读写锁**
* 视情况修改返回值
* 执行`recover()`

## Panic

`panic()`会立即造成一个运行时错误。如果这个错误没有被捕获，程序会中止。十分显然的，defer语句会在程序中止前运行。

产生**运行时**错误意味着某个 `goroutine` 可以直接中止整个程序，所以你应该慎用。

> This is only an example but real library functions should avoid `panic`. If the problem can be masked or worked around, it's always better to let things continue to run rather than taking down the whole program. One possible counterexample is during initialization: if the library truly cannot set itself up, it might be reasonable to panic, so to speak. 		—— Effective Go

有时你的程序会遇到一些致命错误（比如，你连不上数据库），你可以继续运行，但没什么意义。这时，直接`panic()`或许比较合理。

### 实现

```go
// runtime/runtime2.go

// A _panic holds information about an active panic.
//
// A _panic value must only ever live on the stack.
//
// The argp and link fields are stack pointers, but don't need special
// handling during stack growth: because they are pointer-typed and
// _panic values only live on the stack, regular stack pointer
// adjustment takes care of them.
type _panic struct {
	argp      unsafe.Pointer // pointer to arguments of deferred call run during panic; cannot move - known to liblink
	arg       any            // argument to panic
	link      *_panic        // link to earlier panic
	pc        uintptr        // where to return to in runtime if this panic is bypassed
	sp        unsafe.Pointer // where to return to in runtime if this panic is bypassed
	recovered bool           // whether this panic is over
	aborted   bool           // the panic was aborted
	goexit    bool
}
```

## Recover

解决眼下的`panic`并让正常控制流夺回控制权，无论`panic`是内部的（数组越界之类的）或是你蓄意 `panic()`造成的。

正常情况下，调用 recover 函数将返回 nil 且没有任何效果。只有当前 `goroutine` 为 `panicking` 状态，调用 `recover` 函数将捕获 `panic` 的值并且回到正常控制流。

特别的：

* 对 `Goroutine Dead Lock`无效。实际上，所有的 `Fatal Error` 都无效。（`runtime` 都炸了，谁还能给你`recover`呢）
* 对子协程的`panic`无效。 `recover`只作用于当前`goroutine`的`_panic`链表。
* 对 `os.Exit()`无效。

### 原理 

// 这部分好像有问题，待改动

当 `panic` 被调用后，程序将立刻终止当前函数的执行，并开始自底向上的回溯 `goroutine` 的栈，运行`defer`函数。 若回溯到达 `goroutine` 栈的顶端，程序就会终止。调用 `recover` 将停止回溯过程，并返回传入 `panic` 的实参。由于在回溯时只有`defer`函数能够运行，因此 `recover` 只能在`defer`中才有效。

### 注意

* `defer`一定要在可能引发`panic`的**语句之前定义**。

## `goroutine` 源代码节选

```go
type g struct {
	// ...

	_panic    *_panic // innermost panic - offset known to liblink
	_defer    *_defer // innermost defer
    // ...
}
```

# 参考

* [Effective Go: Panic](https://golang.google.cn/doc/effective_go#panic)

* [the Go Blog: Defer, Panic, and Recover](https://golang.google.cn/blog/defer-panic-and-recover)

* [go/runtime2.go](https://github.com/golang/go/blob/master/src/runtime/runtime2.go#L414)
