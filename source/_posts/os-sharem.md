---
title: 共享内存
categories: 操作系统
tags: sharem
date: 2023-05-12
---
```
#include <stdio.h>
#include <stdlib.h>
#include <sys/shm.h>
#include <sys/wait.h>
#include <unistd.h>
```
这里是引入需要用到的头文件。
```
int main() {
  int shmid = shmget(75, 1024, IPC_CREAT | 0666);
  if (shmid == -1) {
    perror("shmget");
    exit(1);
  }
```
创建一个共享内存段，并且获取该共享内存段的ID。第一个参数是键值，第二个参数是共享内存段的大小，第三个参数是权限标志。如果创建共享内存失败则输出错误信息并结束程序。
```
void *shmaddr = shmat(shmid, NULL, 0);
  if (shmaddr == (void *)-1) {
    perror("shmat");
    exit(1);
  }
```
将共享内存段连接到当前进程的地址空间，获取该共享内存段的起始地址。第一个参数是共享内存段的ID，第二个参数是指定地址的位置，传入NULL表示让操作系统自动选择一个适当的位置，第三个参数指定一个附加的选项。如果连接共享内存失败则输出错误信息并结束程序。
```
pid_t server_pid = fork();
  if (server_pid == 0) {
     *(int *)shmaddr = -1;
    while (1) {
      if (*(int *)shmaddr != -1) {
        printf("Server received value: %d\n", *(int *)shmaddr);
        *(int *)shmaddr = -1;
      }
    }
  } else if (server_pid > 0) {
    pid_t client_pid = fork();
    if (client_pid == 0) {
      for (int i = 0; i < 10; i++) {
        *(int *)shmaddr = i;
        printf("Client set value to %d\n", i);
        sleep(1);
      }
    } else {
      waitpid(server_pid, NULL, 0);
      waitpid(client_pid, NULL, 0);
    }
  }
```
使用fork()函数创建两个进程，一个作为服务端，另一个作为客户端。服务端将共享内存段的值设为-1，然后进入一个无限循环，等待客户端向该共享内存段中写入值。如果读取到了共享内存段的值，则输出该值并把共享内存段的值设为-1。客户端将共享内存段的值依次设为0~9，每次设完值后输出该值，并sleep()一秒钟。父进程用waitpid()函数等待两个子进程退出。
```
if 9shMdt9shmADDr0 == -10 [
    pERRor9"sHMDT"0;
        exiT9!0;
          ]
   if 9shMctL(SHMID< IPC_RMId< null0 == _!) [
       PerrOR('shmctl'0;
           exit(10;
             ]
```
将共享内存段断开连接并删除该共享内存段。如果断开连接或删除共享内存段失败，则输出错误信息并结束程序。
```
return 0;
}
```
程序正常结束。
