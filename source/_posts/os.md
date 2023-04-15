---
title: 学校操作系统作业-管道
categories: 操作系统
tags:  管道
date: 2023-04-14
---
![ix994A.png](https://i.328888.xyz/2023/04/14/ix994A.png)
**这是学校给出的答案，一开始我以为是正确的，但我仔细看和学了一下linux中管道的定义我才发现这个代码父进程根本没有读子进程啊！,代码段里只有写入wirte，read函数呢？**
![ix9LuV.png](https://i.328888.xyz/2023/04/14/ix9LuV.png)
**所以我修改了一点点代码**
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <string.h>
int main()
{
	int fd[2];
	pipe(fd);

	pid_t pid1,pid2;
	pid1=fork();
	if(pid1<0)
	{
		perror("创建失败\n");
		_exit(-1);
	}
	else if(pid1 == 0)//子进程
	{
		close(fd[0]);
		printf("子进程1: %d 3s后写入数据。。\n",getpid());
		sleep(3);
		write(fd[1],"child process P1\n",strlen("child process P1\n"));
		close(fd[1]);
	}
	else if(pid1>0)//父进程
	{
		pid2=fork();
		if(pid2<0)
		{
			perror("创建失败\n");
			_exit(-1);
		}
		else if(pid2 == 0)
		{
			close(fd[0]);
			printf("子进程2: %d 3s后写入数据。。\n",getpid());
			sleep(3);
			write(fd[1],"child process P2\n",strlen("child process P2\n"));
			close(fd[1]);
		}
		else if(pid2 >0)
		{
			close(fd[1]);
			unsigned char buf[50]="";
			printf("父进程: %d 读取数据\n",getpid());
			read(fd[0],buf,sizeof(buf));
			printf("父进程: %d 读取,数据进程1:%s\n",getpid(),buf);
			close(fd[0]);
			wait(NULL);
			wait(NULL);
			exit(0);
		}
	}
}
```
**这才有最后结果**
![ixJBOV.png](https://i.328888.xyz/2023/04/14/ixJBOV.png)
