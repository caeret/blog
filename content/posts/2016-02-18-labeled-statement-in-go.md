---
title: "Go 中的标记语句"
date: 2016-02-18T22:15:51+08:00
categories: ["go"]
tags: ["go"]
---
在`Go`中，标记语句（Labeled statements）可以和`goto`、`break`以及`continue`一起使用来控制程序的执行位置。

```sh
LabeledStmt = Label ":" Statement .
```

以下代码使用`label`标记整个`for`循环语句，将会无限循环输出`0123`：

```go
package main

import (
	"fmt"
)

func main() {
	label:
	for i := 0; i < 10; i++ {
		fmt.Println(i)
		if i >= 3 {
			goto label
		}
	}
}
```

可以使用`break`终结标记语句的整个循环：

```go
func main() {
	label:
	for i := 0; i < 3; i++ {
		for j := 0; j < 3; j++ {
			fmt.Println(i, j)
			if j == 2 {
				fmt.Println("break")
				break label
			}
		}
	}
}
```

输出：

```go
0 0
0 1
0 2
break
```

另外，也可以使用`continue`结束最外层当前循环，如以下代码：

```go
func main() {
	label:
	for i := 0; i < 3; i++ {
		for j := 0; j < 3; j++ {
			fmt.Println(i, j)
			if j == 2 {
    			fmt.Println("continue")
				continue label
			}
		}
	}
}
```

将会输出：

```sh
0 0
0 1
0 2
continue
1 0
1 1
1 2
continue
2 0
2 1
2 2
continue
```
