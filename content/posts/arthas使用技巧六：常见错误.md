+++
title = 'arthas使用技巧六：常见错误'
date = 2020-04-28T13:40:36+08:00
draft = false
comment = true
tags = ["Java", "Arthas"]
categories = "编程技术"
+++

### 0、无法连接jvm

错误信息如下：

```
com.sun.tools.attach.AttachNotSupportedException: Unable to open socket file: target process not responding or HotSpot VM not loaded
    at sun.tools.attach.LinuxVirtualMachine.<init>(LinuxVirtualMachine.java:106)
    at sun.tools.attach.LinuxAttachProvider.attachVirtualMachine(LinuxAttachProvider.java:78)
    at com.sun.tools.attach.VirtualMachine.attach(VirtualMachine.java:250)
    at com.taobao.arthas.core.Arthas.attachAgent(Arthas.java:85)
    at com.taobao.arthas.core.Arthas.<init>(Arthas.java:28)
    at com.taobao.arthas.core.Arthas.main(Arthas.java:123)
[ERROR] attach fail, targetPid: 6958
```


排查思路如下：

目标pid是否正确，可以通过jps或ps进行确认
观察pid的执行用户是否和执行arthas的用户一致，比如常见的跑jvm的是tomcat用户，但是登录用户并不是

### 1、arthas端口被占用
错误信息如下：

```
[INFO] arthas-boot version: 3.1.1
[INFO] Process 30482 already using port 3658
[INFO] Process 30482 already using port 8563
[ERROR] Target process 24255 is not the process using port 3658, you will connect to an unexpected process.
[ERROR] 1. Try to restart arthas-boot, select process 30482, shutdown it first.
[ERROR] 2. Or try to use different telnet port, for example: java -jar arthas-boot.jar --telnet-port 9998 --http-port -1
```

原因是arthas需要本地启用一个端口去进行操作，默认端口是3658。
如果端口占用，那么要么结束掉占用这个端口的arthas，要么另外指定一个arthas使用的端口。

根据提示，需要如下操作：

重启arthas，链接旧的那个pid，比如上面就是30482，然后执行shutdown命令，顺利结束
获取换个端口，按照提示执行命令:
>   java -jar arthas-boot.jar --telnet-port 9998 --http-port -1

### 2、arthas端口不对
错误信息如下：

```
onnect to telnet server error: 127.0.0.1 3658
java.net.ConnectException: Connection refused
    at java.net.PlainSocketImpl.socketConnect(Native Method)
    at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
    at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
    at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
    at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
    at java.net.Socket.connect(Socket.java:589)
    at org.apache.commons.net.SocketClient.connect(SocketClient.java:188)
    at org.apache.commons.net.SocketClient.connect(SocketClient.java:209)
    at com.taobao.arthas.client.TelnetConsole.main(TelnetConsole.java:249)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at com.taobao.arthas.boot.Bootstrap.main(Bootstrap.java:503)
```

原因基本同第一点差不多。因为启动arthas时指定了端口，但是arthas没有shutdown进行正常关闭。导致再次连接的时候，端口不对，这个时候就需要指定端口进行连接才可以。

至于怎么获取这个端口，如果你正的记不住当时用的哪个端口的话，可以通过

>   netstat -anpt | grep LISTEN | grep 24255

获取到目标pid所有监听的端口，arthas监听的ip是127.0.0.1，而不是0.0.0.0，试下这几个端口即可。