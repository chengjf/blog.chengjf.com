+++
title = 'arthas使用技巧四：ognl魔法变变变'
date = 2019-11-04T13:40:36+08:00
draft = false
comment = true
tags = ["Java", "Arthas"]
categories = "编程技术"
+++

本篇让我来体验一下ognl的强大魔法。

有这么一个场景，你的接口中有这么一个方法，清除所有的东东，让数据走上正轨。但是这个方法，你没有在后台管理系统调用，导致你现在无法调用这个方法来清除数据。

你现在怒火攻心，心急如焚，像热锅上的蚂蚁--团团转。

这个时候，ognl就派上大用场了。

但是ognl需要一个关键的对象，就是你的接口实现的对象实例，有了一个对象实例，才可以去调用这个实例的方法。那么怎么获取这个对象实例呢？

有两个方法，我来一一介绍。

### 一、watch命令

使用watch命令获取，watch命令里可以获取到当前观测的类实例，使用参数target就可以获取到这个类实例。

但是想被watch抓到需要其他方法触发，选择一个有http接口调用的方法作触发即可。比如下面使用sayHello方法。

> watch com.chengjf.snippet.spring.mvc.service.HelloService sayHello "{target}" -x 10

结果如下：


获取到这个target，就可以调用这个target的方法了。

> watch com.chengjf.snippet.spring.mvc.service.HelloService sayHello "{target.clear()}" -x 10

结果如下：


上线这个命令有个问题，就是sayHello这个方法执行多少次，那么这个clear方法就要执行多少次，这个明显不是我想要的。当然你可以在这个命令执行一次后，就马上CTRL+C结束掉。但是这个watch有个-n参数指定执行次数明显更方便一点。

> watch com.chengjf.snippet.spring.mvc.service.HelloService sayHello "{target.clear()}" -x 10 -n 1

结果如下：


### 二、tt命令+ognl命令

tt（TimeTunnel）命令的官方说明：方法执行数据的时空隧道，记录下指定方法每次调用的入参和返回信息，并能对这些不同的时间下调用进行观测。

这个命令最厉害的是可以记录方法执行的时的现场，这样我们就可以获取到执行的对象实例了。

> tt -t com.chengjf.snippet.spring.mvc.service.HelloService sayHello

调用sayHello的http接口触发，结果如下：


前面这个index，就是这个方法执行现场的编号，可以使用这个编号就行时空回溯：

> tt -i 1000 -w "{target.clear()}"

结果如下：


使用-l参数可以获取到所有记录的现场：

> tt -l

结果如下：