---
title: golang程序优雅退出
slug: golang_exit
date: 2020-04-02
categories:
- tech
tags:
- golang
autoThumbnailImage: false
thumbnailImagePosition: left
thumbnailImage: https://gitee.com/jayos/imgs/raw/master/20200412/202004121818232.jpg
coverImage: https://gitee.com/jayos/imgs/raw/master/20200412/202004121818232.jpg
metaAlignment: center
---

如何优雅退出golang服务器程序？
<!--more-->
```
// Notify方法将signal发送到channel，
func Notify(c chan<- os.Signal, sig ...os.Signal)
// 初始化一个接受os.Signal的通道
c := make(chan os.Signal)
// 调用Notify方法，绑定signal到channel，一旦有信号到达，signal会发送到channel中
signal.Notify(c, os.Interrupt)
//用法:
sig := make(chan os.Signal)  // 创建信号channel
signal.Notify(sig, os.Kill, syscall.SIGINT)  //捕获信号
<-sig  //信号channel阻塞
```

### 例子
```go
package main

import (
	"log"
	"os"
	"os/signal"
	"sync"
	"syscall"
	"time"
)

func main() {
	wait := &sync.WaitGroup{}
	wait.Add(1)
	go func() {
		sig := make(chan os.Signal)
		signal.Notify(sig, os.Kill, os.Interrupt, syscall.SIGINT)
		t := time.Tick(time.Second)
		for {
			select {
			case <-sig:
				log.Printf("Goroutine has exit !")
				wait.Done()
				return

			case <-t:
				log.Printf("Goroutine are running ...")
			}
		}
	}()
	wait.Wait()
}

```
输出
```
2020/03/10 21:19:16 Goroutine are running ...
2020/03/10 21:19:17 Goroutine are running ...
2020/03/10 21:19:18 Goroutine are running ...
2020/03/10 21:19:19 Goroutine are running ...
2020/03/10 21:19:20 Goroutine are running ...
2020/03/10 21:19:21 Goroutine are running ...
2020/03/10 21:19:22 Goroutine are running ...
2020/03/10 21:19:23 Goroutine are running ...
2020/03/10 21:19:24 Goroutine are running ...
^C2020/03/10 21:19:24 Goroutine has exit !
```
