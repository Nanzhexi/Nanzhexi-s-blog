---
title: CS144 lab0编写webget
categories: 计网
tags: tcp/ip
date: 2023-04-27
---
**编写webget**
现在是实现<font color=red>webget</font>的时候了，这是一个利用操作系统的TCP支持和流套接字抽象在互联网上获取网页的程序——就像你在本实验室早些时候手动完成的那样。

在<font color=red>build</font>目录中，用文本编辑器或IDE打开<font color=red>../apps/webget.cc</font>。

在<font color=red>get_URL</font>函数中，删去以”<font color=red>// Your code here</font>“为开头的注释。

按照本章所述实现简单的Web客户端，使用你之前使用的HTTP（Web）请求的格式。使用<font color=red>TCPSocket</font>和<font color=red>Address</font>类。

提示：

	请注意，在HTTP中，每一行都必须以”\r\n”结束（只使用”\n”或<font color=red>endl</font>是不够的）。
	当你完成了向套接字写请求时，通过结束你的出站字节流（从你的套接字到服务器的套接字的字节流）告诉服务器你已经完成了请求。你可以通过调用一个参数为<font color=red>SHUT_WR</font>的<font color=red>TCPSocket</font>的<font color=red>shutdown</font>方法来做到这一点。作为回应，服务器将向你发送一个回复，然后将结束它自己的出站字节流（从服务器的套接字到你的套接字的字节流）。你会发现传入字节流已经结束了，因为当你读完来自服务器的整个字节流后，你的套接字将到达”EOF”（结束）。(如果你不关闭你的出站字节流，服务器将等待一段时间，让你发送更多的请求，并且也不会结束它的出站字节流）。
	确保从服务器上读取并打印所有的输出，直到套接字到达”EOF”（结束）——单次调用<font color=red>read</font>是不够的。
	我们预计你需要写十行左右的代码。
**这是我的代码**
![i96wAE.png](https://i.328888.xyz/2023/04/27/i96wAE.png)
运行<font color=red>make</font>来编译我的程序
	![i9FtVb.png](https://i.328888.xyz/2023/04/27/i9FtVb.png)
通过运行<font color=red>./apps/webget cs144.keithw.org /hello</font>来测试你的程序。这与你在网络浏览器中访问<font color=red>http://cs144.keithw.org/hello</font>时看到的情况相比，有什么不同？它与第2.1节的结果相比如何？请用你喜欢的任何http URL进行实验或测试吧!
	第二个测试的链接是我的博客地址，我试了一下，因为我开了代理加速这个网站只有显示cloudflare的信息正常
	如下图
	![i9FlCa.png](https://i.328888.xyz/2023/04/27/i9FlCa.png)
当它看起来工作正常时，运行<font color=red>make check webget</font>来运行自动测试。在实现<font color=red>get_URL</font>功能之前，你应该能看到以下情况：
	![i9FrFy.png](https://i.328888.xyz/2023/04/27/i9FrFy.png)
	<font color=red>测试通过</font>
