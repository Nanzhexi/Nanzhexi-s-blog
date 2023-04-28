---
title: 进程通信
categories: 操作系统
tags: msg
date: 2023-04-28
---
**这是我的代码**
```
#include<unistd.h>
#include<string.h>
#include<stdlib.h>
#include<stdio.h>
#include<sys/wait.h>
#include<sys/types.h>
#include<sys/msg.h>
#define MSGKEY 77 
struct msgmess{
    long type;
    char text[1024];
};

int main()
{
    int id;
    struct msgmess msg;
    pid_t pid;
    
    id = msgget(MSGKEY,0666|IPC_CREAT);
    pid = fork();
    
    if(id == -1){
        perror("获取 IPC_ID 失败");
        return -1;
    }
    
    if(-1==pid)
    {
        printf("fork error!\n");
    }

    else if(0==pid)
    {
        msg.type = 1;
        char in[1024];
        char mess[1024];
        while(1){
          sprintf(in,"被子进程%d成功发送了！",getpid());
          printf("请输入信息:");
          scanf("%s",mess);  //scanf("%s",mess);
          strcat(mess,in);
          strcpy(msg.text,mess);
          msgsnd(id,&msg,sizeof(msg),0);
          if(strncmp(mess,"end",3) == 0){
            break;
          }
        }
        exit(0);
    }
    
    else
    {
        
            wait(NULL);
            char in[1024];
            char mess[1024];
            while(1){
              msgrcv(id,&msg,sizeof(msg),1,0); 
              if (strncmp(msg.text,"end",3) == 0){
                break;
              }
              printf("父进程%d",getpid());
              printf("获得消息:%s\n",msg.text); 
            }
            if(msgctl(id,IPC_RMID,0) == -1){
              printf("队列删除失败!\n");
            }
        
        
    }
    msgctl(id,IPC_RMID,0);
    
}
```
**我将逐行解释，以便加强我的理解**
```
#include<unistd.h>
#include<string.h>
#include<stdlib.h>
#include<stdio.h>
#include<sys/wait.h>
#include<sys/types.h>
#include<sys/msg.h>
```
	**这些为头文件，<font color=red>include<unistd.h></font>调用了<font color=red>fork()</font>;，<font color=red>include<string.h></font>调用了我输入信息将使用的函数<font color=red>strcat()</font>,<font color=red>strcopy()</font>还有函数比较<font color=red>strcmp()</font>来判断循环跳出; <font color=red>include<stdlib.h>#include<stdio.h></font>c语言在linux上的标准库没啥好说,<font color=red>include<sys/wait.h></font>>包括了<font color=red>wait()</font>函数，在这块代码中我使用了<font color=red>wait(NULL)</font>等待子进程的退出,<font color=red>NULL</font>的意思为退出时不被关注, <font color=red>include<sys/types.h></font>是<font color=red>Unix/Linux</font>系统的基本系统数据类型的头文件，含有<font color=red>size_t，time_t，pid_t</font>等类型。<font color=red>#include<sys/msg.h></font>包含了<font color=red>msgget(), msgsnd(), msgrcv(), msgctl()</font>这些要用的函数**
```
#define MSGKEY 77 
```
	**定义了一个常量MSGKEY 为77，之后的msgget需要用到**
```
struct msgmess{
    long type;
    char text[1024];
};
```
	**我们定义了一个结构体msgmess 信息用来 定义c语言中我需要的数据信息 long type 一个数据类型为长数据的type, char text[1024]数组**
```
int main()
{
    int id;
    struct msgmess msg;
    pid_t pid;
    
    id = msgget(MSGKEY,0666|IPC_CREAT);
    pid = fork();
    
    if(id == -1){
        perror("获取 IPC_ID 失败");
        return -1;
    }

```
	**Int main ()函数没啥说的，c语言的主函数一切的开头其实我觉得应该为 int main(void)这样更标准.int id;这个id为msgget() 函数中信息队列的id，如果小于0的话信息队列创建失败struct msgmess msg;我们利用结构体创建了一个名为msg 的类pid_t pid; pid_t类型的 pid 用来记录进程的id是什么。 id = msgget(MSGKEY,0666|IPC_CREAT);msgget 创建了一个信息队列，然后将它的地址给到id来记录这个信息队列 msgget(MSGKEY,0666|IPC_CREAT) 中的MSGKEY你把它看成msgget(77,0666|IPC_CREAT)所以id 应该为77 id=77 原型：int msgget(key_t key, int msgflg) key 意味着使用同样的key可以获得同一个队列的 ID 值，设置后使用一样的即可；msgflg详见参考（里面的0777、0600就是设置的权限.**
	**pid = fork();创建子进程不多说了.    if(id == -1){perror("获取 IPC_ID 失败");return -1;}用来判断信息队列有没有创建成功**
```
 if(-1==pid)
    {
        printf("fork error!\n");
    }
```
	**来判断pid 的值如果是负数的 话就说明子进程没有创建成功,来判断pid 的值如果是负数的 话就说明子进程没有创建成功 因为父进程大于0而子进程为0 然后fork会进行两次返回真正的pid 的值，这块如果不懂看我之前的文章fork()这里不赘述了**
```
 else if(0==pid)
    {
        msg.type = 1;
        char in[1024];
        char mess[1024];
        while(1){
          sprintf(in,"被子进程%d成功发送了！",getpid());
          printf("请输入信息:");
          scanf("%s",mess);  //scanf("%s",mess);
          strcat(mess,in);
          strcpy(msg.text,mess);
          msgsnd(id,&msg,sizeof(msg),0);
          if(strncmp(mess,"end",3) == 0){
            break;
          }
        }
        exit(0);
    }
```
	**这里就很经典是一个子进程的代码块让我们慢慢看,if(0==pid)这里说明如果pid 为0就进入了子进程,char in[1024]; char mess[1024];这里我们定义了in，mess这些字符数组用来输出我们需要的字符,while(1){ }然后进入while死循环，让他一直循环,当然根据题目意思发送10次我们需要添加跳出循环的条件,if(strncmp(mess,"end",3) == 0){break;}这里我们定义了如果你想结束就输入end这个字符就行了**
	**sprintf(in,"被子进程%d成功发送了！",getpid());提示语句并且显示进程pid，printf("请输入信息:");提示输入,scanf("%s",mess);输入strcat(mess,in);拼接起来mess和in 的字符strcpy(msg.text,mess); msg这个数据类中的text输入为mess也就是我们输入的字符msgsnd(id,&msg,sizeof(msg),0); 发送信息，id 就是创建的信息队列的id，它不为负的时候可以正常发送信息，&msg就是发送这个数据的信息，sizeof(msg)这个msg 的信息数多大，0表示为正常进行,exit(0);正常退出**
```
  	    wait(NULL);
            char in[1024];
            char mess[1024];
            while(1){
              msgrcv(id,&msg,sizeof(msg),1,0); 
              if (strncmp(msg.text,"end",3) == 0){
                break;
              }
              printf("父进程%d",getpid());
              printf("获得消息:%s\n",msg.text); 
            }
            if(msgctl(id,IPC_RMID,0) == -1){
              printf("队列删除失败!\n");
            }
```
	**wait(NULL);等待子程序退出Char in mess while不赘述,msgrcv(id,&msg,sizeof(msg),1,0);接受信息函数中的其他参数与发送类似,if(msgctl(id,IPC_RMID,0) == -1){ printf("队列删除失败!\n");}是对消息队列的处理，如IPC_RMID是从系统内核中删除队列消息、IPC_STAT获取消息队列的详细信息。IPC_SET设置消息队列的信息。**
```
msgctl(id,IPC_RMID,0);
```
	**之前创建了信息队列，现在删除这个信息队列**
