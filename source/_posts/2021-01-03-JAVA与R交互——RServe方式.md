---
title: JAVA与R交互——RServe方式
date: 2021-01-03 01:37:30
tags:
- R
- RServe
categories:
- tools
---

如何用Java调用R
<!-- more -->

R语言适合做数据统计与分析，java适合开发应用系统，本文介绍如何以Rserve的方式在java中调用R

Rserve：Rserve是一个基于TCP/IP协议的，允许R语言与其他语言通信的C/S结构的程序，支持C/C++,Java,PHP,Python,Ruby,Nodejs等

环境准备： Centos7，R



**Rserve安装**

`R #进入R交互环境`

```R
> install.packages("Rserve")  #安装Rserve包


installing via 'install.libs.R' to /usr/local/lib/R/site-library/Rserve
** R
** inst
** preparing package for lazy loading
** help
*** installing help indices
** building package indices
** testing if installed package can be loaded
* DONE (Rserve)

> q()  #安装完成后退出

```

**启动Rserve**

`R CMD Rserve --RS-enable-remote  # 加上RS-enable-remote参数后才可以远程调用`

**JAVA调用**

引入依赖包

```xml
<dependency>
    <groupId>org.rosuda.REngine</groupId>
    <artifactId>REngine</artifactId>
    <version>2.1.0</version>
</dependency>

<dependency>
    <groupId>org.rosuda.REngine</groupId>
    <artifactId>Rserve</artifactId>
    <version>1.8.1</version>
</dependency>
```

示例代码

```java
RConnection rc = new RConnection("10.16.3.230",6311);	//建立连接；Rserve默认启动端口6311


//取最大值
int[] arr = {145,202,53};
rc.assign("arr",arr);				//声明变量
REXP r1 = rc.eval("max(arr)");	//执行语句
System.out.println(r1.asInteger());	//获取结果


// 打印 R 的版本
REXP r2 = c.eval("R.version.string");
System.out.println(r2.asString());


/**
 * 调用 R 源文件 add.R 中的自定义函数:
 * myAdd <- function(x, y) {
 *         sum <- x + y
 *         return (sum)
 * }
 */
rc.eval("source('/home/R/add.R')");			//因为R的实际执行是在服务端，所以在读取文件等操作时读的是服务端的文件
int sum = rc.eval("myAdd(1, 2)").asInteger();
System.out.println(sum);
```

