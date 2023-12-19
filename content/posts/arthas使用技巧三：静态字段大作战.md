+++
title = 'arthas使用技巧三：静态字段大作战'
date = 2019-11-01T13:40:36+08:00
draft = false
comment = true
tags = ["Java", "Arthas"]
categories = "编程技术"
+++

首先，有这么一个接口

```java
package com.chengjf.snippet.spring.mvc.service;

/**
 * @author jeff.cheng
 * @date 2019-10-30 09:31
 */
public interface HelloService {

    /**
     * say hello
     *
     * @param name
     * @return
     */
    String sayHello(String name);
}
```

然后实现如下，里面有两个static字段，分别是HELLO_PREFIX和HELLO_SUFFIX：

```java
package com.chengjf.snippet.spring.mvc.service.impl;

import com.chengjf.snippet.spring.mvc.service.HelloService;
import org.springframework.stereotype.Service;

/**
 * @author jeff.cheng
 * @date 2019-10-30 09:32
 */
@Service
public class HelloServiceImpl implements HelloService {

    private static String HELLO_PREFIX = "Hello, ";
    private static String HELLO_SUFFIX = "!";

    @Override
    public String sayHello(String name) {
        return HELLO_PREFIX + name + HELLO_SUFFIX;
    }
}
```

可以使用getstatic方法获取到static字段的值：

> getstatic com.chengjf.snippet.spring.mvc.service.impl.HelloServiceImpl HELLO_PREFIX

结果如下：


可以传入通配符获取该类的所有static字段

> getstatic com.chengjf.snippet.spring.mvc.service.impl.HelloServiceImpl *

结果如下：


arthas提供了ognl这个强大的工具，也可以用来获取static字段：

> ognl "@com.chengjf.snippet.spring.mvc.service.impl.HelloServiceImpl@HELLO_PREFIX"

结果如下：


但是ognl的功能远不止如此，还可以修改static字段的值：

> ognl "#c=@com.chengjf.snippet.spring.mvc.service.impl.HelloServiceImpl@class,#f=#c.getDeclaredField('HELLO_PREFIX'),#f.setAccessible(true),#f.set(#c,'123')"

结果如下：


查询一下，会发现值已经被修改了：


上面这段命令就将HELLO_PREFIX这个static字段修改成了123。下面来看下这个命令：

> \#c=@com.chengjf.snippet.spring.mvc.service.impl.HelloServiceImpl@class，获取HelloServiceImpl这个class
> \#f=\#c.getDeclaredField('HELLO_PREFIX')，获取Field

> \#f.setAccessible(true)，设置Field的修改性

> \#f.set(\#c,'123')，修改字段的值为123