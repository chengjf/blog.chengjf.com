+++
title = 'arthas使用技巧五：重新定义七点半'
date = 2020-04-28T13:40:36+08:00
draft = false
comment = true
tags = ["Java", "Arthas"]
categories = "编程技术"
+++

重新定义七点半，不知道你有没有听过这个梗。
我不知道这个梗最初是来源于哪里，我第一次见是在老罗的发布会，因为老罗的发布会总会有各种各样的问题，说是七点半开，从来没有准时过，所以说，“重新定义了七点半”。

前面几篇都讲了怎么去获取数据（参数、静态字段、返回值、执行时间、执行链路），或者怎么去执行方法（静态方法、获取实例调用方法）。

这篇就说一个大招，怎么在运行时去修改代码。

### 0、前置工作
有下面这样一个接口，获取开始时间：

```java
package com.chengjf.snippet.spring.mvc.service;

public interface HelloService {
    
    /**
     * 获取开始时间
     *
     * @return
     */
    String getStartTime();
}
```
实现是直接返回开始时间“七点半”：

```java
package com.chengjf.snippet.spring.mvc.service.impl;

@Service
public class HelloServiceImpl implements HelloService {
    @Override
    public String getStartTime() {
        return "七点半";
    }
}
```
然后整个controller调用这个getStartTime方法：

```java
@RequestMapping("startTime")
public Object getStartTime() {
    HashMap<Object, Object> objectObjectHashMap = Maps.newHashMap();
    String result = helloService.getStartTime();
    objectObjectHashMap.put("startTime", result);
    return objectObjectHashMap;
}
```
返回就是说好的“七点半”：
```json
{
    "startTime": "七点半"
}
```
### 1、反编译获取代码源文件
使用arthas进入该程序的jvm。

使用jad命令进行反编译：

>   jad --source-only com.chengjf.snippet.spring.mvc.service.impl.HelloServiceImpl > /Users/jeff/arthas/HelloServiceImpl.java

将com.chengjf.snippet.spring.mvc.service.impl.HelloServiceImpl反编译，将反编译后的代码写入到/Users/jeff/arthas/HelloServiceImpl.java中。

### 2、修改代码
修改上面反编译得到的HelloServiceImpl.java，将返回值“七点半”修改为“8点”：

```java
@Override
public String getStartTime() {
    return "8点";
}
```
### 3、编译代码
使用mc命令将修改好的java文件编译成class文件：

>   mc /Users/jeff/arthas/HelloServiceImpl.java

这个命令会返回编译完成的class文件的路径：


>   Memory compiler output:
/Users/jeff/Documents/code/github/java-snippet/com/chengjf/snippet/spring/mvc/service/impl/HelloServiceImpl.class
Affect(row-cnt:1) cost in 1645 ms.


### 4、redefine
使用redefine命令，将上面编译好的class文件装载在jvm中：

>   redefine /Users/jeff/Documents/code/github/java-snippet/com/chengjf/snippet/spring/mvc/service/impl/HelloServiceImpl.class

这个时候，再调用controller请求这个接口，会返回修改后的开始时间：

```json
{
    "startTime": "八点"
}
```