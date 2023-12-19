+++
title = 'arthas使用技巧一：你的类咋的啦'
date = 2019-10-23T13:40:36+08:00
draft = false
comment = true
tags = ["Java", "Arthas"]
categories = "编程技术"
+++

本篇主要介绍如何查看所有相关你的类的东东。

## 一、查看当前的classloader
键入命令

> classloader

结果如下：


## 二、查看你的类是否被加载
应用启动或运行的时候，可能会抛出ClassNotFoundException这种和类加载相关的异常，这时候你需要确认classloader是否正确加载了你的类。

键入命令，传入类的全路径

> sc com.chengjf.snippet.spring.mvc.controller.IndexController

结果如下：


sc命令也支持通配符

> sc com.chengjf.*

结果如下


或者

> sc *IndexController*

结果如下


## 三、确认你的类代码是否最新

执行过程中，可能发现这个类方法执行的效果和自己当初设想的不对，比如方法入口的日志没有打印，方法返回的值明显不对，又或者该插入数据库或redis没有进行。

这个时候，你要首先确认，这个类确实是你写的那个类，而不是因为打包为题或部署问题导致类代码没有更新。

键入命令：

> jad com.chengjf.snippet.spring.mvc.controller.IndexController

结果如下
