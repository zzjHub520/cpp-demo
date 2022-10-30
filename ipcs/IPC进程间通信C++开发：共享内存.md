# IPC进程间通信C++开发：共享内存

不同进程之间通信，通常可以用共享内存/消息队列/信号量/管道等方法，在Linux系统下提供了相关的库函数来方便使用。

# A. 共享内存

共享内存就相当于开辟了一块物理内存空间，不同的进程通过虚拟地址的映射都访问到同一块物理内存，这样就能直接在内存读写数据。下面说一下具体用到的函数：

```bash
# 查看当前系统的共享内存状态。
ipcs -m
```

## 1. shmget创建共享内存

函数原型：`int shmget(key_t key , size_t size , int shmflag)`

key：共享内存的标识符，一般通过ftok函数得到。

size：申请的内存大小，一般为4k的倍数。

shmflag：共享内存的权限标志，一般有3种情况：

1）IPC_CREATE：如果存在与key值对应的共享内存，则直接返回共享内存ID；否则先创建共享内存，再返回共享内存ID。

2）IPC_CREATE | IPC_EXCL：如果存在与key值对应的共享内存，则返回-1；否则先创建共享内存，再返回共享内存ID。

3）0。如果存在与key值对应的共享内存，则返回共享内存ID；否则返回-1。

## 2. shmat挂载共享内存

函数原型：`void *shmat(int shmid, const void *shmaddr, int shmflg)`

shmid：就是shmget成功调用后的返回值。

shmaddr：指定进程空间映射的虚拟地址，一般直接设为NULL。

shmflag：一般可以直接设为0；设为SHM_RDONLY表示只读模式。

## 3. shmdt卸载共享内存

意思就是说断开与共享内存的连接，禁止本进程访问该共享内存。

函数原型：`int shmdt(const void *shmaddr)`

shmaddr：就是shmat返回的虚拟地址。

成功则返回0；否则返回-1。

## 4. shmctl管理共享内存

用于控制该共享内存，包括获取、改变内存状态，销毁内存等。

函数原型：`int shmctl(int shmid, int cmd, struct shmid_ds *buf)`

shmid：共享内存ID，就是shmget调用成功后的返回值。

cmd：控制指令，一般包括以下三种：

1）IPC_STAT：得到共享内存的状态，把共享内存的shmid_ds结构复制到buf中。

2）IPC_SET：改变共享内存的状态，把buf所指的shmid_ds结构中的uid、gid、mode复制到共享内存的shmid_ds结构内。

3）IPC_RMID：删除这片共享内存

buf：buf是一个结构指针，它指向共享内存模式和访问权限的结构。

成功调用则返回0；否则返回-1。

## 5. 共享内存C++范例

### server：读内存

```cpp
#include <sys/shm.h>
#include <sys/ipc.h>
#include <iostream>
#include <unistd.h>
int main()
{
    key_t key = ftok(".", 1);
    int shmId = shmget(key, 4096, IPC_CREAT);
    char *addr = (char*)shmat(shmId, NULL, SHM_RDONLY);
    int input = 20;
    while (input--) {
        sleep(1);
        std::cout << addr << std::endl;
    }
    if (shmdt(addr) == -1) {
        perror("shmdt fail");
        return -1;
    }

    if (shmctl(shmId, IPC_RMID, NULL) == -1) {
        perror("shmctl fail");
        return -2;
    }
    return 0;
}
```

### client：写内存

```cpp
#include <sys/shm.h>
#include <sys/ipc.h>
#include <iostream>
#include <unistd.h>
int main()
{
    key_t key = ftok(".", 1);
    int shmId = shmget(key, 4096, 0);
    if (shmId == -1) {
        perror("shmget fail");
        return -1;
    }
    char *addr = (char*)shmat(shmId, NULL, 0);
    int input = 20;
    int i = 0;
    while (input--) {
        sleep(1);
        addr[i++] = 'A';
        std::cout << "在第" << i << "个位置写入了A." << std::endl;
    }
    if (shmdt(addr) == -1) {
        perror("shmdt fail");
        return -1;
    }

    if (shmctl(shmId, IPC_RMID, NULL) == -1) {
        perror("shmctl fail");
        return -2;
    }
    return 0;
}
```

# B. 消息队列

消息队列可以理解为是一个消息的链表，有写权限的进程可以向消息队列中添加消息，有读权限的进程可以从消息队列中读走消息，本质就是个数据结构。

## msgget创建消息队列

函数原型：`extern int msgget(key_t key, int _msgflg);`

第一个参数key是由ftok创建的key值。

第二个参数_msgflg的低位用来确定消息队列的访问权限，如0770为文件的访问类型；此外还可以附加以下参数值(通过或的方式与权限一起使用)：

IPC_CREAT：如果key不存在，则创建；存在，则直接返回。

IPC_EXCL：如果key存在，返回失败。

IPC_NOWAIT：如果需要等待，则直接返回错误。

## msgctl控制消息队列

函数原型：`extern int msgctl(int _msqid, int _cmd, struct msqid_ds *_buf);`

待续~

# C. 管道

管道本质上是内核的一块缓存，主要分无名管道和有名管道。无名管道只能用在具有亲缘关系的进程，有名管道可以用在两个不相干的进程。

待续~

# D. 信号量

信号量的本质是数据操作锁，它本身不具有数据交换的功能，而是通过控制其他的通信资源（文件，外部设备）来实现进程间通信，它本身只是一种外部资源的标识。信号量在此过程中负责数据操作的互斥、同步等功能。