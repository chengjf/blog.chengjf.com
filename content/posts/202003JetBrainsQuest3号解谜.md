+++
title = '202003 JetBrains Quest#3解谜'
date = 2020-03-16T13:40:36+08:00
draft = false
comment = true
tags = ["JetBrains", "Quest"]
categories = "解密"
+++

这次是JetBrains的最后一个谜题了。

## No.0
在twitter上JetBrains发布了最后一个谜题的线索：

>   SGF2ZSB5b3Ugc2VlbiB0aGUgcG9zdCBvbiBvdXIgSW5zdGFncmFtIGFjY291bnQ/

这个大小写加数字加斜杠的组合，让人一下想到了base64，解一下，果然是：

>   Have you seen the post on our Instagram account?

## No.1
去到JetBrains的ins看下，最新一条图片的标题就是“JetBrains Quest”，他的内容是：

> Welcome to the final Quest! You should start on the Kotlin Playground: https://jb.gg/kotlin_quest

> P.S. If you don’t know about the #JetBrainsQuest, it’s not too late to find out.

打开提示的网址 https://jb.gg/kotlin_quest，有这么一段代码：

```javascript
fun main() {
   val s = "Zh#kdyh#ehhq#zrunlqj#552:#rq#wkh#ylghr#iru#wkh#iluvw#hslvrgh#ri#wkh#SksVwrup#HDS1#Li#zh#jdyh#|rx#d#foxh/#lw#zrxog#eh#hdv|#dv#sl1"

   val n: Int = TODO()
   for (c in s) {
       print(c - n)
   }
}
```

这个文本的样子，非常像第一个谜题中的caesar解法，然后就是这个shift要试下，最终这个shift是3，所以这个代码最终是下面这个样子：

```javascript
fun main() {
    val s = "Zh#kdyh#ehhq#zrunlqj#552:#rq#wkh#ylghr#iru#wkh#iluvw#hslvrgh#ri#wkh#SksVwrup#HDS1#Li#zh#jdyh#|rx#d#foxh/#lw#zrxog#eh#hdv|#dv#sl1"

    val n: Int = 3
    for (c in s) {
        print(c - n)
    }
}
```

执行结果是：

> We have been working 22/7 on the video for the first episode of the PhpStorm EAP. If we gave you a clue, it would be easy as pi.123

## No.2
在Google中进行搜索，关键字是“jetbrains episode of the PhpStorm EAP”，发现首页的视频就是PhpStorm 2020.1版本的三个episode，打开第一个episode，地址是 https://www.youtube.com/watch?v=OtQuAr3n87c。

pi就是这个视频的线索，在这个视频的3分14秒处，展示的是一个composer.json文件，里面的homepage字段是个url，地址是：https://jb.gg/31415926。

## No.3
打开上面提供的网址，进入到一个名为“JetBrains Quest Quiz”的网站，要在1分钟内回答5到和JetBrains有关的题目，地址： https://www.quiz-maker.com/Q2723CE。
我可是看了一篇2019年的报告（地址： https://www.jetbrains.com/company/annualreport/2019/）加Google才搞定。（这个网址是quchou-maker的，没有找到hack的方法）

全部答对5道题，可以看到下面的内容：

```
Almost there! The last challenge is in the Tips of the Day of a specific IntelliJ IDEA Community version from our latest build page in Confluence, but… there is a catch. You have to know which version to look for. To find the build number, you need sight beyond sight:

. Not Everything Today Does All You Could Ask. Lessons Learned From Other Relevant Solutions, Possibly Even Another Kind Emerge. Risking Sometimes Being Liberal Or Generous Proves Ordinary Simple Tests Infinitely More Annoying. Get Examining Hidden Initial Designated Early Symbols. They Have Everything Needed, Except Xerox, To Completely Level Up Everything.
```

## No.4
最后一个挑战，是从JetBrains的Confluence里面的最近build页面，找个特定的IDEA社区版本，提示就在每日提示里（这个提示经常嫌烦被关掉）。
给的线索下面这段话，一开始没有头绪，我还以为是圣经里面的一段，版本号是就是这段话在圣经中所处的章节小节等。
我Google了一下，圣经中并没有这段话。
接着，我发现这段话的首字母全是大写，会不会是藏头？我把每个大写字母找出来，发现是下面这个样子：

> .net day call for speaker's blog post image hides the next clue

## No.5
接着上面，我一开始，对这个“.net day call”是啥，完全没有概念，通过Google，才发现是JetBrains的活动，地址是： https://blog.jetbrains.com/dotnet/2020/02/13/jetbrains-net-day-online-2020-call-speakers/。

blog中，只有一张图片，使用Chrome+F12，锁定这张图片，发现这张图片的alt属性的内容如下：

you_are_looking_for_build_201-6303
这样，要找到的版本号就是201-6303。

## No.6
JetBrains的最近build页面是这个 https://confluence.jetbrains.com/display/IDEADEV/Latest+builds。

最近的build是2020.1，页面是这个 https://confluence.jetbrains.com/display/IDEADEV/IDEA+2020.1+latest+builds。

我在这个页面上，发现了“IntelliJ IDEA 201.6303”的字眼，点击这个版本的Release Notes，跳转到 https://confluence.jetbrains.com/display/IDEADEV/IDEA+2020.1+Custom+Build+Edition。

页面没有找到，但是提示我，这个页面可能被重命名为“IDEA 2020.1 Quest Build Edition”，所以我接着点击这个链接 https://confluence.jetbrains.com/display/IDEADEV/IDEA+2020.1+Quest+Build+Edition。

在这个页面，终于找到了熟悉的“201.6303”，我就下载了ideaIU-201.6303.exe到本地。

## No.7
安装并打开下载好的idea，打开每日提示（可以通过菜单Help->Tip of the day打开），浏览一遍，终于发现了一个tip有如下内容：

```
JETBRAINS QUEST: LAST PUZZLE
You have discovered our JetBrains Quest! If you don’t know what this is, you should start from the beginning.
This is it. The last puzzle. You are just one step away from glory!
Now you just need the Key to unlock the Quest page.
The Key is the first and last 4 digits of the 50 * 10^6 position of the Fibonacci sequence (F(50 Million)).
As you may know by now, not all that glitters is gold, and to solve this puzzle you should not go straight for the obvious answer. May you make a glorious choice.
Remember that you have until the 15th of March 12:00 CEST.
```

内容中的Quest page是一个链接，地址是 https://www.jetbrains.com/promo/quest/，这个页面就和前面两个解谜的最终结果一样，填写邮箱和code。

这次的code是要计算Fibonacci数列结果的前4位和后四位数字，乍一看还不复杂，但是要计算的这个数字实在是太大了-50000000。

我在网上找的计算程序，不是栈爆了，就是内存不足了，有几个能算出结果的，互相直接结果不一样，我填进去试了下也不正确。

最后的最后，过了12点，是这个网站拯救了我 https://www.wolframalpha.com/。
使用这个计算后四位： https://www.wolframalpha.com/input/?i=%28fibonnaci+50000000%29+mod+10000，结果是3125。
使用这个计算整个结果，取前四位： https://www.wolframalpha.com/input/?i=fibonnaci+50000000，结果是4602。

这样，最终的code就是46023125。

第三个谜题的奖励是8折全家桶优惠码。

写完这个，就1点了，睡了。