+++
title = 'typecho开启https后无法登录后台'
date = 2019-05-06T13:40:36+08:00
draft = false
comment = true
tags = ["typecho"]
categories = "typecho"
+++

我的blog使用了cloudflare的https，在typecho里也开始https后，初看一切正常，但是登录后台时，返回302，重定向到了登录页。

我在网上搜索了好久，记录下解决方案。

修改typecho下的这个文件：

config.inc.php

增加如下代码：

```php
define('__TYPECHO_SECURE__', true);
```