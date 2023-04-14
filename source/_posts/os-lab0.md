---
title: OS-lab0
categories: 操作系统
date: 2023-4-14
---
<font size=5>**pingpong**</font>
编写一个使用UNIX系统调用的程序来在两个进程之间“ping-pong”一个字节，请使用两个管道，每个方向一个。父进程应该向子进程发送一个字节;子进程应该打印“<pid>: received ping”，其中<pid>是进程ID，并在管道中写入字节发送给父进程，然后退出;父级应该从读取从子进程而来的字节，打印“<pid>: received pong”，然后退出。您的解决方案应该在文件user/pingpong.c中。
提示：

	使用pipe来创造管道

	使用fork创建子进程

	使用read从管道中读取数据，并且使用write向管道中写入数据

	使用getpid获取调用进程的pid

	将程序加入到Makefile的UPROGS

	xv6上的用户程序有一组有限的可用库函数。您可以在user/user.h中看到可调用的程序列表；源代码（系统调用除外）位于user/ulib.c、user/printf.c和user/umalloc.c中。
```
$ make qemu
...
init: starting sh
$ pingpong
4: received ping
3: received pong
$
```
**如果您的程序在两个进程之间交换一个字节并产生如上所示的输出，那么您的解决方案是正确的。**
这是我的代码
```
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main()
{
	char buf='P';

	int fd1[2];//子进程->父进程
	int fd2[2];//父进程->子进程
      	pipe(fd1);
	pipe(fd2);

	int pid=fork();
	if(pid<0)
	{
		printf("error\n");
		exit(1);
	}
	else if(pid == 0)
	{
		close(fd1[1]);
		close(fd2[0]);
		if(read(fd1[0],&buf,sizeof(char))!=sizeof(char))
		{
			printf("child read() error!\n");
			exit(1);
		}
		else
		{
			printf("%d: received ping\n",getpid());
		}
		if(write(fd2[1],&buf,sizeof(char))!=sizeof(char))
		{
			printf("child wirte() error!\n");
			exit(1);
		}
		close(fd1[1]);
		close(fd2[0]);
		exit(0);
	}
	else
	{
		close(fd1[0]);
		close(fd2[1]);
		if(write(fd1[1],&buf,sizeof(char))!=sizeof(char))
		{
			printf("parent write() error!\n");
			exit(1);
		}
		if(read(fd2[0],&buf,sizeof(char))!=sizeof(char))
		{
			printf("parent write() error!\n");
			exit(1);
		}
		else
		{
			printf("%d: received pong\n",getpid());
		}
		close(fd1[1]);
		close(fd2[0]);
		wait(0);
		exit(0);
	}
}
```
**辅助图**
![](https://ask.qcloudimg.com/http-save/yehe-8223537/c8d80c6174fa9ef860b91147c16ec240.png?imageView2/2/w/2560/h/7000)
**执行效果**
![[](https://ask.qcloudimg.com/http-save/yehe-8223537/da36c953478ff1c3ba85d33e883331a0.png?imageView2/2/w/2560/h/7000)
