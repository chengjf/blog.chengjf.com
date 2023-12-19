+++
title = 'arthas使用技巧七：ognl增强'
date = 2020-07-16T13:40:36+08:00
draft = false
comment = true
tags = ["Java", "Arthas"]
categories = "编程技术"
+++

在官方github中的issue-482中介绍了使用tt命令获取执行现场，然后用ognl获取spring的ApplicationContext从而为所欲为的示例。

在spring的生态中，如果可以获取到ApplicationContext，无疑真的可以为所欲为了。

但是在实际使用的，我发现这个获取途径有如下两个问题：

1. 需要触发才能获取到执行现场，从而进行后续的步骤。这意味着你的应用必须提供一个类似于http接口的形式。
2. 在多classloader的应用中，使用tt命令执行ognl表达式的时候，会出现ClassNotFound的错误，是因为classloader不对的问题，但是tt命令无法指定classloader

为了解决上述两个问题，只能使用一个类静态变量保存这个ApplicationContext，然后直接使用ongl命令去操作，ognl命令是可以使用-c参数指定classloader的。当然这个方法的缺陷就是侵入了应用。

下面附一个我用的保存ApplicationContext的示例：

```java
package com.chengjf.example.context;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.stereotype.Component;

@Component("contextHolder")
public class ContextHolder {

    public static ApplicationContext context;

    @Autowired
    public void setApplicationContext(ApplicationContext applicationContext) {
        context = applicationContext;
    }

}
```

这样，就可以直接使用ognl表达式为所欲为了。

第一步，获取合适的classloader，使用sc命令：
>   sc -d com.chengjf.example.context.ContextHolder

结果中可能有多个classloader，根据class-loader选择合适的classloader，记住classLoaderHash
第二部，直接使用ognl表达式，获取ApplicationContext，然后获取你想要的bean，为所欲为：

>   ognl -c 1fe72c5c '#c=@com.chengjf.example.context.ContextHolder@context,#o=#c.getBean("roomService"),#o.queryById(0L)'