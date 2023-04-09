---
title: Golang中的泛型
date: 2023-04-09 20:33:11
categories: Programming Skills
tags: golang
excerpt: 泛型函数是一种能接受不同类型的参数的通用函数。
---

# Golang中的泛型

泛型函数是一种通用函数。它能接受不同类型的参数。

```go
func Foo[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```

对于函数`Foo`的参数`m map[K]V`：

* K 接受 comparable 泛型约束。comparable包括所有能进行比较的类型(`int8` ,`uint16`, `float32`, `string`...)。
* V 接受`int64 | float64`泛型约束。V可以是两者中的任意一个。

## 使用接口

### 为什么使用接口

* 实现对参数类型约束本身的代码复用。

```go
type Number interface {
    int64 | float64 // 可复用！
}

// 为什么K的约束为comparable?
func Foo[K comparable, V Number](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```
* 使用参数类型提供的方法。
```go
// From "github.com/2023-Melting-Backend"
type sth interface {
	db.User | db.Template | db.ProposalInfo | db.Tag | db.Question | db.Game
	TableName() string
	GetKey() (string, int)
}

func UpdateSth[T sth](value T) error {
	pk, id := value.GetKey()
	result := db.DB.Table(value.TableName()).Updates(value).Where(pk+" = ?", id)
	return result.Error
}
```

如果不在接口中规定方法，编译会报错（即使约束中所有类型都实现了相关方法）。

{% asset_img 1.png p1%}

## 参考

* [Tutorial: Getting started with generics](https://go.dev/doc/tutorial/generics)

  ​    
