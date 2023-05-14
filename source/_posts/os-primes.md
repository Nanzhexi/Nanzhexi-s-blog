---
title: prime sieve(筛选素数)
categories: 操作系统
tags: 管道
date: 2023-05-14
---
请看以下逐行解释代码：
```
#include "kernel/types.h"  // 包含系统定义类型的头文件
#include "user/user.h"     // 包含用户调用系统函数的头文件
 #define RD 0  // 读取端管道符的索引
#define WR 1  // 写入端管道符的索引
 const uint INT_LEN = sizeof(int);  // int类型的长度
 /**
 * @brief 读取左邻居的第一个数据
 * @param lpipe 左邻居的管道符
 * @param pfirst 用于存储第一个数据的地址
 * @return 如果没有数据返回-1,有数据返回0
 */
int lpipe_first_data(int lpipe[2], int *dst)
{
  if (read(lpipe[RD], dst, sizeof(int)) == sizeof(int)) {
    printf("prime %d\n", *dst);  // 打印素数
    return 0;
  }
  return -1;
}
```
该函数用于读取左邻居的第一个数据，它接收一个左邻居的管道符和一个用于存储数据地址的指针作为参数。函数首先从左邻居的管道符中读取一个整数数据，如果读取成功，则将数据存储至指针变量*dst指向的内存地址中，并打印这个数（即素数）。如果没有数据，则返回-1，否则返回0.
```
* @brief 读取左邻居的数据，将不能被first整除的写入右邻居
 * @param lpipe 左邻居的管道符
 * @param rpipe 右邻居的管道符
 * @param first 左邻居的第一个数据
 */
void transmit_data(int lpipe[2], int rpipe[2], int first)
{
  int data;
  // 从左管道读取数据
  while (read(lpipe[RD], &data, sizeof(int)) == sizeof(int)) {
    // 将无法整除的数据传递入右管道
    if (data % first)
      write(rpipe[WR], &data, sizeof(int));
  }
  close(lpipe[RD]);
  close(rpipe[WR]);
}
```
该函数用于从左邻居的管道符lpipe中读取数据，将不能被first整除的数据写入右邻居的管道符rpipe中。函数接受三个参数：左邻居的管道符、右邻居的管道符和左邻居的第一个素数。函数使用while循环从左邻居中读取数据，如果读取成功，就检查该数据是否能被左邻居的第一个数据（即first）整除，如果不能，就将该数据写入右邻居的管道符中。最后，关闭左邻居的读取端管道符lpipe[RD]和右邻居的写入端管道符rpipe[WR]。
```
* @brief 寻找素数
 * @param lpipe 左邻居管道
 */
void primes(int lpipe[2])
{
  close(lpipe[WR]);  // 关闭写端管道符
  int first;
  if (lpipe_first_data(lpipe, &first) == 0) {  // 读取左邻居的第一个数据
    int p[2];
    pipe(p);  // 当前进程的管道
     transmit_data(lpipe, p, first);  // 将左邻居数据传递到右邻居
     if (fork() == 0) {  // 子进程
      primes(p);  // 递归调用primes函数
    } else {  // 父进程
      close(p[RD]);  // 关闭当前进程的读端管道符
      wait(0);  // 等待子进程结束
    }
  }
  exit(0);  // 结束进程
}
```
该函数用于找出素数。它接收一个左邻居的管道符lpipe作为参数。首先，关闭该管道符的写入端（这里关闭写入端是为了让程序在读取完所有数据后自然退出）。然后，调用lpipe_first_data函数尝试从左邻居读取数据。如果成功读取到了数据，则创建一个新的管道符p。然后，使用transmit_data函数将数据从左邻居传递到右邻居。接着，创建子进程并在子进程中递归调用primes函数。在父进程中，关闭p[RD]（也就是当前进程的读取端管道符），并等待子进程结束。最后，退出进程。
```
int main(int argc, char const *argv[])
{
  int p[2];
  pipe(p);  // 创建管道符
   for (int i = 2; i <= 35; ++i)  // 写入初始数据
    write(p[WR], &i, INT_LEN);
   if (fork() == 0) {  // 子进程
    primes(p);  // 调用primes函数
  } else {  // 父进程
    close(p[WR]);  // 关闭写端管道符
    close(p[RD]);  // 关闭读端管道符
    wait(0);  // 等待子进程结束
  }
   exit(0);  // 结束进程
}
```
main函数是程序的入口。它首先创建一个管道符p，并在该管道符中写入2到35之间的整数。然后，使用fork函数创建一个子进程，并在子进程中调用primes函数。在父进程中，关闭p的写入端和读取端管道符，然后等待子进程结束。最后，程序退出。  
 
总结： 该程序使用管道和进程实现了寻找素数的功能。它使用管道符p存储整数数据，并创建子进程在该数据中查找素数。每个子进程递归调用primes函数，将左邻居的数据传递给右邻居。当数据传递完毕后，程序自然退出。
![VZrOJZ.png](https://i.328888.xyz/2023/05/14/VZrOJZ.png)
