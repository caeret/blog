---
title: "Go 中的 slice 与 nil"
date: 2016-01-23T22:18:17+08:00
categories: ["go"]
tags: ["go"]
---

[`nil`](https://golang.org/pkg/builtin/#nil "godoc")在`go`中是`pointer`、`channel`、`func`、`interface`、`map`、`slice`类型的零值。
<!--more-->

我们先看下面一段代码：

```go
package main

import (
	"fmt"
)

func display(name string, slice []string) {
	var isNilText string
	if slice == nil {
		isNilText = "slice is nil"
	} else {
		isNilText = "slice is not nil"
	}
	fmt.Printf("%s: %T, %s, len=%d, cap=%d\n", name, slice, isNilText, len(slice), cap(slice))
}

func main() {
	var sliceA []string
	var sliceB []string = make([]string, 0)
	var sliceC []string = []string{}
	display("sliceA", sliceA)
	display("sliceB", sliceB)
	display("sliceC", sliceC)
}
```

输出：

```sh
sliceA: [], slice is nil, len=0, cap=0
sliceB: [], slice is not nil, len=0, cap=0
sliceC: [], slice is not nil, len=0, cap=0
```
## slice 初始化与 apend

### slice 初始化

```go
var slice []int // slice is nil
slice = append(slice, 1) // slice is now {1}

// or

slice := []int{} // slice is not nil

// or

slice := make([]int, 0) // slice is not nil
slice = append(slice, 1) // slice is now {1}

// or

slice := make([]int, 1) // slice is not nil
slice = append(slice, 1) // slice is now {0, 1}
```

### 关于 append

```go
package main

import (
	"fmt"
)

func display(name string, slice []int) {
	var isNilText string
	if slice == nil {
		isNilText = "slice is nil"
	} else {
		isNilText = "slice is not nil"
	}
	fmt.Printf("%s: %v, %s, len=%d, cap=%d\n", name, slice, isNilText, len(slice), cap(slice))
}

func main() {
	var ints []int

	display("init", ints)

	for i := 0; i < 10; i++ {
		ints = append(ints, i)
		display(fmt.Sprintf("%d", i), ints)
	}
}
```

输出：

```sh
init: [], slice is nil, len=0, cap=0
0: [0], slice is not nil, len=1, cap=1
1: [0 1], slice is not nil, len=2, cap=2
2: [0 1 2], slice is not nil, len=3, cap=4
3: [0 1 2 3], slice is not nil, len=4, cap=4
4: [0 1 2 3 4], slice is not nil, len=5, cap=8
5: [0 1 2 3 4 5], slice is not nil, len=6, cap=8
6: [0 1 2 3 4 5 6], slice is not nil, len=7, cap=8
7: [0 1 2 3 4 5 6 7], slice is not nil, len=8, cap=8
8: [0 1 2 3 4 5 6 7 8], slice is not nil, len=9, cap=16
9: [0 1 2 3 4 5 6 7 8 9], slice is not nil, len=10, cap=16
```

可以看出来，如果`slice`是`nil`，那么`append`会先初始化`slice`，length为增加N个元素后length，capacity会按一定规律增大。
