---
title: Golang中的异常处理
date: 2023-05-03 00:14:09
categories: Programming Skills
tags: golang
excerpt: Defer, Panic, Recover
---

# 异常处理

{% asset_img 1683112281963.jpeg %}

你或许受够了丑陋的 `if  err != nil {...}`，那么来聊一聊不太常见的

## Defer

### 实现

下面是`_defer`结构的源代码。不细说，因为我也不大懂。

```go
// A _defer holds an entry on the list of deferred calls.
// If you add a field here, add code to clear it in freedefer and deferProcStack
// This struct must match the code in cmd/compile/internal/gc/reflect.go:deferstruct
// and cmd/compile/internal/gc/ssa.go:(*state).call.
// Some defers will be allocated on the stack and some on the heap.
// All defers are logically part of the stack, so write barriers to
// initialize them are not required. All defers must be manually scanned,
// and for heap defers, marked.
type _defer struct {
   siz     int32 // includes both arguments and results
   started bool
   heap    bool
   // openDefer indicates that this _defer is for a frame with open-coded
   // defers. We have only one defer record for the entire frame (which may
   // currently have 0, 1, or more defers active).
   openDefer bool
   sp        uintptr  // sp at time of defer
   pc        uintptr  // pc at time of defer
   fn        *funcval // can be nil for open-coded defers
   _panic    *_panic  // panic that is running defer
   link      *_defer

   // If openDefer is true, the fields below record values about the stack
   // frame and associated function that has the open-coded defer(s). sp
   // above will be the sp for the frame, and pc will be address of the
   // deferreturn call in the function.
   fd   unsafe.Pointer // funcdata for the function associated with the frame
   varp uintptr        // value of varp for the stack frame
   // framepc is the current pc associated with the stack frame. Together,
   // with sp above (which is the sp associated with the stack frame),
   // framepc/sp can be used as pc/sp pair to continue a stack trace via
   // gentraceback().
   framepc uintptr
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
    return
}

func main() {
    f()
}

// runtime: goroutine stack exceeds 1000000000-byte limit
// runtime: sp=0xc0200f9380 stack=[0xc0200f8000, 0xc0400f8000]
// fatal error: stack overflow
```

### 注意

1. Defer 语句中函数的参数是在 defer 语句声明时确定的。
2. 当函数返回后，defer 声明的函数按照后进先出 (Last In First Out) 的顺序执行。
3. 如果函数返回的是命名返回值（named return values），defer 声明的函数调用可以读写这个返回值。（见小练习！`foo3()`）



### defer 与 return

Go语言的函数中`return`语句在底层并不是原子操作，它分为给`返回值赋值`和`RET指令`两步。而`defer`语句执行的时机就**在返回值赋值操作后，RET指令执行前**。



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

> 提示：你可能需要了解命名返回值函数。

### defer的用途

* 释放资源（数据库连接，文件句柄等）
* 释放**读写锁**
* 视情况修改返回值
* 执行`recover()`

## Panic

`panic()`会立即造成一个运行时错误。如果这个错误没有被捕获，程序会中止。十分显然的，defer语句会在程序中止前运行。

产生**运行时**错误意味着某个 `goroutine` 可以直接中止整个程序，所以你应该慎用。

> This is only an example but real library functions should avoid `panic`. If the problem can be masked or worked around, it's always better to let things continue to run rather than taking down the whole program. One possible counterexample is during initialization: if the library truly cannot set itself up, it might be reasonable to panic, so to speak. 		—— Effective Go

有时你的程序会遇到一些致命错误（比如，你连不上数据库），你可以继续运行，但没什么意义。这时，直接`panic()`或许比较合理。

## Recover

解决一切`panic`并让正常控制流夺回控制权，无论`panic`是内部的（数组越界之类的）或是你调用 `panic()`造成的。

正常情况下，调用 recover 函数将返回 nil 且没有任何效果。只有当前 goroutine 为 panicking 状态，调用 recover 函数将捕获 panic 的值并且回到正常控制流。

特别的：

* 对 `Goroutine Dead Lock`无效。实际上，所有的 `Fatal Error` 都无效。（`runtime` 都炸了，谁还能给你`recover`呢）
* 对子协程的`panic`无效。 `recover`只作用于当前`goroutine`的`_panic`链表
* 对 `os.Exit()`无效。（那是正常退出，无论程序返回值是多少 —— 那不是`runtime`该操心的事情）

### 原理

当 panic 被调用后，程序将立刻终止当前函数的执行，并开始自底向上的回溯 goroutine 的栈，运行defer函数。 若回溯到达 goroutine 栈的顶端，程序就会终止。调用 recover 将停止回溯过程，并返回传入 panic 的实参。 由于在回溯时只有defer函数能够运行，因此 recover 只能在defer中才有效。

### 注意

* `defer`一定要在可能引发`panic`的**语句之前定义**。

```go
// goroutine 源代码节选
type g struct {
	// Stack parameters.
	// stack describes the actual stack memory: [stack.lo, stack.hi).
	// stackguard0 is the stack pointer compared in the Go stack growth prologue.
	// It is stack.lo+StackGuard normally, but can be StackPreempt to trigger a preemption.
	// stackguard1 is the stack pointer compared in the C stack growth prologue.
	// It is stack.lo+StackGuard on g0 and gsignal stacks.
	// It is ~0 on other goroutine stacks, to trigger a call to morestackc (and crash).
	stack       stack   // offset known to runtime/cgo
	stackguard0 uintptr // offset known to liblink
	stackguard1 uintptr // offset known to liblink

	_panic    *_panic // innermost panic - offset known to liblink
	_defer    *_defer // innermost defer
    // ...
}
```

# 参考

* [Effective Go: Panic](https://golang.google.cn/doc/effective_go#panic)

* [the Go Blog: Defer, Panic, and Recover](https://golang.google.cn/blog/defer-panic-and-recover)

* [go/runtime2.go](https://github.com/golang/go/blob/master/src/runtime/runtime2.go#L414)
