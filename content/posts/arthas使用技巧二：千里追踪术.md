+++
title = 'arthas使用技巧二：千里追踪术'
date = 2019-10-24T13:40:36+08:00
draft = false
comment = true
tags = ["Java", "Arthas"]
categories = "编程技术"
+++


## 一、追踪方法执行流程和耗时

使用trace命令可以列出目标方法执行时的调用链以及耗时。有如下两个作用：

可以排查执行流程，特别是当代码中有各种if/else的时候，可以协助看下到底是执行了哪些方法
可以排查执行耗时，找出消耗时间最多的方法，进行优化
比如：

> trace com.chengjf.snippet.spring.mvc.controller.IndexController index

结果如下：


## 二、获取方法执行时的参数和返回值或异常

如果方法缺少日志进行排查，可以使用watch命令可以实时看到这些。

> watch com.chengjf.snippet.spring.mvc.controller.IndexController hello "{params,returnObj,throwExp,isReturn,isThrow}"

结果如下：


当参数或返回值的对象层次较深导致无法查看到内部的数据时，可以加上-x参数指定深度：

> watch com.chengjf.snippet.spring.mvc.controller.IndexController hello "{params,returnObj,throwExp,isReturn,isThrow}" -x 10

结果如下：


如果要排查的方法被调用次数很多，无法准确排查或排查难度较大，可以进行过滤。

首先，可以指定你想要的参数：

> watch com.chengjf.snippet.spring.mvc.controller.IndexController hello "{params,returnObj,throwExp,isReturn,isThrow}" 'params[0]=="world"' -x 10

> watch com.chengjf.snippet.spring.mvc.controller.IndexController hello "{params,returnObj,throwExp,isReturn,isThrow}" 'params[1]==10' -x 10

可以组合使用：

> watch com.chengjf.snippet.spring.mvc.controller.IndexController hello "{params,returnObj,throwExp,isReturn,isThrow}" 'params[0]=="world" \&\& params[1]==10' -x 10

> watch com.chengjf.snippet.spring.mvc.controller.IndexController hello "{params,returnObj,throwExp,isReturn,isThrow}" 'params[0]=="world" || params[1]==10' -x 10

如果参数是对象，还可以直接使用field：

> watch com.chengjf.snippet.spring.mvc.controller.IndexController test "{params,returnObj,throwExp,isReturn,isThrow}" 'params[0].name=="world"' -x 10

对返回值也是一样的：

> watch com.chengjf.snippet.spring.mvc.controller.IndexController test "{params,returnObj,throwExp,isReturn,isThrow}" 'returnObj.age==99' -x 10

## 三、获取方法执行的统计信息

使用monitor可以获取方法的统计信息，包括执行次数，成功次数，失败次数，平均耗时以及失败率。

其中，-c参数可以指定监控周期，默认是60s。

> monitor com.chengjf.snippet.spring.mvc.controller.IndexController * -c 10

结果如下：