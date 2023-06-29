---
title: select与channel
date: 2022-05-08 09:35:00 +0800
categories: [Go]
tags: [Go基础]
pin: true
author: 小李学编程

toc: true
comments: true
typora-root-url: ../../../libits.github.io
math: false
mermaid: true  
---

> select：监听 IO 操作，当 IO 操作发生时，触发相应的动作。 
>
> 所说的IO操作就是对channle的操作：向通道发送数据，或者从通道中读取数据。
>
> 在执行select语句的时候，运行时系统会自上而下地判断每个case中的发送或接收操作是否可以被**立即执行**。

```go
package main

import "fmt"

func main() {
   ch1 := make(chan int, 1)
   ch1 <- 2
   select {
   case v := <-ch1:
      fmt.Println("取到的数据：", v)
   case ch1 <- 1:
      fmt.Println("写入数据")
   }
}
```

- nil channel代表channel未初始化，向未初始化的channel读写数据会造成阻塞
- 关闭(close)未初始化的channel会引起panic。
- **从一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值，并不会引起panic。**

##  从已经关闭**并且没有值**的通道中取值

```go
package main

import "fmt"

//从关闭的通道中取值示例：
func main() {
   //声明实例化通道ch1
   ch1 := make(chan int, 1)
   //关闭通道
   close(ch1)
   select {
   //通通道ch1中取值
   case v := <-ch1:
      fmt.Printf("从ch1中取值：%d\n", v)
   default:
      fmt.Println("默认case")
   }
}

//运行结果：从ch1中取值：0
```

##  从已经关闭**并且有值**的通道中取值

```go
package main

import "fmt"

//从关闭的通道中取值示例：
func main() {
   //声明实例化通道ch1
   ch1 := make(chan int, 1)
   //向通道中赋值
   ch1 <- 1
   //关闭通道
   close(ch1)
   //关闭之后取值
   after_close_value := <-ch1
   fmt.Printf("关闭之后取值：%d\n", after_close_value) //打印结果：关闭之后取值：1
   select {
   //通通道ch1中取值
   case v := <-ch1:
      fmt.Printf("从ch1中取值：%d\n", v) //打印结果：从ch1中取值：0
   default:
      fmt.Println("默认case")
   }
}
//关闭之后取值：1
//从ch1中取值：0
```

- 通道关闭后，如果通道中仍然有值，还是可以正常取到通道中的值的。
- 通道关闭后，如果通道中已经没有值了，再从通道中取值，并不会引起panic，而是会取到对应类型的零值。

![图片](/assets/blog_res/2022-05-08-select与channel.assets/640.png)
