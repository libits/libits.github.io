---
title: Goroutine
date: 2021-03-14 09:35:00 
categories: [Go]
tags: [Go并发]
pin: true
author: 小李学编程

toc: true
comments: true
typora-root-url: ../../../libits.github.io
math: false
mermaid: true  
---

##  goroutine

> Go程序会智能地将 goroutine 中的任务合理地分配给每个CPU
>
> main函数是一个goroutine

```go
package main

import (
    "fmt"
	"sync"
)
var wg sync.WaitGroup

func hello(i int) {
    defer wg.Done() // goroutine结束就登记-1
    fmt.Println("Hello Goroutine!", i)
}
func main() {

    for i := 0; i < 10; i++ {
        wg.Add(1) // 启动一个goroutine就登记+1
        go hello(i)
    }
    wg.Wait() // 等待所有登记的goroutine都结束
}
//每次打印的结果不一样
```

![image-20230630163118734](/assets/blog_res/2021-03-14-goroutine.assets/image-20230630163118734-16881138813531.png)

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // 合起来写
    go func() {
        i := 0
        for {
            i++
            fmt.Printf("new goroutine: i = %d\n", i)
            time.Sleep(time.Second)
        }
    }()
    i := 0
    for {
        i++
        fmt.Printf("main goroutine: i = %d\n", i)
        time.Sleep(time.Second)
        if i == 2 {
            break
        }
    }
}
```

![image-20230630163539207](/assets/blog_res/2021-03-14-goroutine.assets/image-20230630163539207-16881141407042.png)

