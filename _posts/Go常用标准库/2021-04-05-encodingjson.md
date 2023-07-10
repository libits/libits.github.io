---
title: encoding/json
date: 2021-04-05
categories: [Go常用标准库]
tags: [encoding/json]
pin: true
author: 小李学编程

toc: true
comments: true
typora-root-url: ../../../libits.github.io
math: false
mermaid: true
---

##  序列化和反序列化

```go
type ColorGroup struct {
    ID     int
    Name   string
    Colors []string
}
group := ColorGroup{
    ID:     1,
    Name:   "Reds",
    Colors: []string{"Crimson", "Red", "Ruby", "Maroon"},
}
b, err := json.Marshal(group)
if err != nil {
    fmt.Println("error:", err)
}
os.Stdout.Write(b)
```

