+++
title = 'flutter桌面程序打包发布'
date = 2021-03-09T12:37:36+08:00
draft = false
comment = true
tags = ["flutter"]
categories = "编程技术"
+++

fluter 2.0 刚刚发布，其中对Desktop的应用程序支持更加完善，虽然现在仍然任务是“beta snapshot”。

我就早早下载了flutter 2.0的sdk，用Microsoft Visual Code编写，将官方的默认的例子跑了起来。

1、到官网下载sdk，注意使用国内的官网，速度会快很多，按照教程进行安装，官网链接：https://flutter.cn/docs/get-started/install。注意，国内下载的sdk里面引用的repo地址依然是国外的，想提高下载速度，需要设定：

> setx PUB_HOSTED_URL "https://pub.flutter-io.cn"

> setx FLUTTER_STORAGE_BASE_URL "https://storage.flutter-io.cn"


2、安装完sdk，配置下vs code，安装flutter的插件

3、按照官网的流程，创建项目，带有的默认例子是一个counting的例子

4、默认的flutter项目是不支持desktop的，需要设定，

> flutter config --enable-windows-desktop

5、这样在vs code中，就可以编译成windows的应用程序了。打包后包括一个exe文件、一个flutter_windows.dll，还有一个data，这些必须在同一个目录下。

6、程序分发的时候，当然可以将这些文件压缩打包，然后再解压缩执行。

7、如果想使用installer这种安装工具进行安装，支持window的HKEY、卸载等功能，那么就需要进行打包，推荐使用NSIS这个免费强大的工具，链接：https://nsis.sourceforge.io/Main_Page。

附上我写的打包脚本：

```powershell
; The name of the installer
Name "demo_install"

; The file to write
OutFile "demo_install.exe"

; Request application privileges for Windows Vista
RequestExecutionLevel user

; Build Unicode installer
Unicode True

; The default installation directory
InstallDir $PROGRAMFILES\demo_install

;--------------------------------

ShowInstDetails show

; Pages

Page directory
Page instfiles

UninstPage uninstConfirm
UninstPage instfiles

RequestExecutionLevel admin

;--------------------------------

; The stuff to install
Section "" ;No components page, name is not important

  ; Set output path to the installation directory.
  SetOutPath $INSTDIR
  
  ; Put file there
  ;RegDLL $INSTDIR/flutter_windows.dll
  File flutter_windows.dll
  File flutter_application_1.exe
  File /r data
  Rename $INSTDIR\flutter_application_1.exe demo_install.exe
  WriteUninstaller "$INSTDIR\uninstall.exe"
  
SectionEnd ; end the section

; Optional section (can be disabled by the user)
Section "Start Menu Shortcuts"

  CreateDirectory "$SMPROGRAMS\demo_install"
  CreateShortcut "$SMPROGRAMS\demo_install\Uninstall.lnk" "$INSTDIR\uninstall.exe"
  CreateShortcut "$SMPROGRAMS\demo_install\demo_install.lnk" "$INSTDIR\demo_install.exe"

SectionEnd

;--------------------------------

; Uninstaller

Section "Uninstall"
  
  ; Remove registry keys
  DeleteRegKey HKLM "Software\Microsoft\Windows\CurrentVersion\Uninstall\Example2"
  DeleteRegKey HKLM SOFTWARE\NSIS_Example2

  ; Remove files and uninstaller
  Delete $INSTDIR\example2.nsi
  Delete $INSTDIR\uninstall.exe
  Delete $INSTDIR\*
  ; Remove shortcuts, if any
  Delete "$SMPROGRAMS\demo_install\*.lnk"

  ; Remove directories
  RMDir /r "$SMPROGRAMS\demo_install"
  RMDir /r "$INSTDIR\data"
  RMDir /r "$INSTDIR"

SectionEnd
```