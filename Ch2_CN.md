# **第二章：多线程**

## **多线程介绍**  

### **为什么需要并发（Concurrency）？**  
现代计算机系统需要 **同时处理多个任务**：  
✅ **多个运行的程序** → 操作系统必须在程序间分配 CPU 时间。  
✅ **输入/输出操作** → 需要处理 **磁盘、网络、键盘等事件**。  
✅ **高效利用资源** → 避免 CPU 等待 I/O 而空闲。  

即使 **只有一个处理器**，操作系统也会 **快速切换任务**，制造出 **同时运行的假象**（**时间片轮转**）。  

---

### **本章将涵盖什么？**  
📌 **多线程编程基础** → 如何编写并发程序。  
📌 **POSIX 线程（Pthreads）和 Windows 线程（Win32）** → C/C++ 多线程编程标准。  
📌 **线程同步** → **防止竞争条件**，保证正确性。  
📌 **线程取消** → **安全管理 & 终止线程**。  
📌 **从应用程序视角学习** → 先学习 **用户态** 线程，再理解 **操作系统内核** 的线程管理。  

虽然 **OS 内部线程实现不同**，但 **核心概念是一样的**！ 🚀  

---

## **为什么使用线程？**  

### **为什么我们需要线程，而不仅仅是进程？**  
我们已经在操作系统中使用 **多个进程**，为什么还需要在**同一个进程内部使用线程**？  

✅ **线程能更自然地实现并发**。  
✅ **不仅仅适用于多核处理器**，**单核 CPU** 也能通过线程提高 I/O 处理效率。  
✅ **多线程代码通常比单线程代码更简单、更高效、更易于调试**。  

---

### **示例 1：远程登录 (`rlogind`)**
**问题：`rlogind` 需要同时处理两个输入/输出流**：  
1. 读取 **远程客户端的键盘输入**，发送到服务器。  
2. 读取 **服务器应用程序的输出**，发送到远程客户端。  

📌 **单线程版本（代码复杂，难以维护）**
```c
void rlogind(int r_in, int r_out, int l_in, int l_out) {
   fd_set in = 0, out;
   int want_l_write = 0, want_r_write = 0;
   int want_l_read = 1, want_r_read = 1;
   int eof = 0, tsize, fsize, wret;
   char fbuf[BSIZE], tbuf[BSIZE];

   fcntl(r_in, F_SETFL, O_NONBLOCK);
   fcntl(r_out, F_SETFL, O_NONBLOCK);
   fcntl(l_in, F_SETFL, O_NONBLOCK);
   fcntl(l_out, F_SETFL, O_NONBLOCK);

   while(!eof) {
      FD_ZERO(&in);
      FD_ZERO(&out);
      if (want_l_read) FD_SET(l_in, &in);
      if (want_r_read) FD_SET(r_in, &in);
      if (want_l_write) FD_SET(l_out, &out);
      if (want_r_write) FD_SET(r_out, &out);

      select(MAXFD, &in, &out, 0, 0);
      if (FD_ISSET(l_in, &in)) {
         if ((tsize = read(l_in, tbuf, BSIZE)) > 0) {
            want_l_read = 0;
            want_r_write = 1;
         } else eof = 1;
      }
      if (FD_ISSET(r_in, &in)) {
         if ((fsize = read(r_in, fbuf, BSIZE)) > 0) {
            want_r_read = 0;
            want_l_write = 1;
         } else eof = 1;
      }
      if (FD_ISSET(l_out, &out)) {
         if ((wret = write(l_out, fbuf, fsize)) == fsize) {
            want_r_read = 1;
            want_l_write = 0;
         } else if (wret >= 0) tsize -= wret;
         else eof = 1;
      }
      if (FD_ISSET(r_out, &out)) {
         if ((wret = write(r_out, tbuf, tsize)) == tsize) {
            want_l_read = 1;
            want_r_write = 0;
         } else if (wret >= 0) tsize -= wret;
         else eof = 1;
      }
   }
}
```
❌ **难以阅读**  
❌ **需要复杂的非阻塞 I/O**  
❌ **调试困难**  

📌 **使用线程的版本（更清晰的逻辑）**
```c
void incoming(int r_in, int l_out) {           
   int eof = 0;
   char buf[BSIZE];
   int size;

   while (!eof) {                                
      size = read(r_in, buf, BSIZE);
      if (size <= 0) eof = 1;  
      if (write(l_out, buf, size) <= 0) eof = 1;
   }                                             
}                                                

void outgoing(int l_in, int r_out) {           
   int eof = 0;
   char buf[BSIZE];
   int size;

   while (!eof) {                                
      size = read(l_in, buf, BSIZE);
      if (size <= 0) eof = 1;  
      if (write(r_out, buf, size) <= 0) eof = 1;
   }                                             
}
```
✅ **代码更简洁**  
✅ **I/O 逻辑更简单**  
✅ **更容易调试和维护**  

---

### **示例 2：多线程数据库服务器**  
- **单线程数据库** 只能 **逐个** 处理客户端请求。  
- **问题**：有些请求很快完成，有些请求需要很长时间。  

📌 **单线程解决方案**  
- **轮流处理每个请求**（例如，每个请求 1 毫秒）。  
- 但 **长请求仍然会影响短请求的处理**。  

📌 **多线程解决方案**  
- **每个客户端请求都有一个独立线程**。  
- **长短请求可以同时执行，不互相影响**。  
- **在多核 CPU 上，线程可以真正并行运行**。  

---

### **总结**
✅ **多线程能让并发编程更简单、更高效**。  
✅ **线程不仅适用于多核 CPU，单核 CPU 也能通过更好的 I/O 处理获益**。  
✅ **相比于单线程代码，多线程程序更容易编写、调试和维护**。 🚀  

---

## **使用线程编程**  

### **为什么需要标准的线程 API？**  
尽管 **多线程有很多优势**，但 **标准 API 直到最近才被开发**：  
- **POSIX 线程（Pthreads）** → 在 **1995 年** 标准化，编号 **POSIX 1003.1c**。  
- **Windows 线程（Win32 Threads）** → 微软 **专门为 Windows 开发**。  

📌 **两者的主要区别**：  
✅ **接口不同** → **函数调用和语法** 需要适配不同 API。  
✅ **能力不同** → **某些功能** 在 **Pthreads 和 Win32 之间不兼容**。  

尽管 **Pthreads 和 Win32 线程有不同实现方式**，但 **两者都可以高效支持多线程编程**。  

---


## **POSIX（Pthreads）线程的创建和终止**  

### **在 POSIX（Pthreads）中创建线程**  
在 **Unix 系统** 中，我们使用 `pthread_create()` 来创建线程。  
✅ **创建新线程**，执行一个函数。  
✅ **向线程传递参数**。  
✅ **存储线程 ID** 在 `pthread_t` 变量中。  

### **POSIX 线程创建示例**  

```c
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  

#define NUM_THREADS 5  

void *server(void *arg) {
    int thread_id = *(int *)arg;
    printf("服务器线程 %d 启动。\n", thread_id);
    return NULL;
}

void start_servers() {
    pthread_t thread[NUM_THREADS];
    int i, args[NUM_THREADS];

    for (i = 0; i < NUM_THREADS; i++) {
        args[i] = i;
        if (pthread_create(&thread[i], NULL, server, &args[i]) != 0) {
            perror("pthread_create 失败");
            exit(1);
        }
    }

    for (i = 0; i < NUM_THREADS; i++) {
        pthread_join(thread[i], NULL);
    }
}

int main() {
    start_servers();
    return 0;
}
```

📌 **解释:**  
- `pthread_create(&thread[i], NULL, server, &args[i]);` → 创建线程，并执行 `server()`。  
- `pthread_join(thread[i], NULL);` → 等待线程完成。  
- 每个线程 **打印自己的 ID**，然后退出。  

---

### **如何向线程传递多个参数**  
`pthread_create()` 只能接受 **一个参数**，但很多时候我们需要传递多个参数。  

❌ **错误方法：直接传递局部变量地址**  
```c
int arg = 5;
pthread_create(&thread, NULL, my_thread_function, &arg);
```
这里 `arg` 可能在 **线程运行前被释放**，导致 **未定义行为**。  

✅ **正确方法：使用动态分配的内存**  
```c
void *thread_func(void *arg) {
    int *val = (int *)arg;
    printf("线程接收到：%d\n", *val);
    free(arg);
    return NULL;
}

void create_thread() {
    pthread_t thread;
    int *arg = malloc(sizeof(int));
    *arg = 5;
    pthread_create(&thread, NULL, thread_func, arg);
    pthread_join(thread, NULL);
}
```
📌 **解释：**  
- 使用 `malloc()` 分配参数，使其在 **线程运行期间不会被释放**。  
- 线程结束时，使用 `free(arg);` **释放内存**，防止内存泄漏。  

---

### **线程的终止方式**  
线程有两种终止方式：  
1. **返回一个值** → `return (void *) value;`  
2. **调用 `pthread_exit(value);`**  

示例代码：  
```c
void *thread_func(void *arg) {
    int thread_id = *(int *)arg;
    pthread_exit((void *)(thread_id * 10));
}

int main() {
    pthread_t thread;
    int arg = 5;
    void *ret_val;

    pthread_create(&thread, NULL, thread_func, &arg);
    pthread_join(thread, &ret_val);

    printf("线程返回值：%ld\n", (long)ret_val);
    return 0;
}
```
📌 **解释:**  
- 线程返回 `thread_id * 10`，由 `pthread_join()` **获取返回值**。  

---

### **分离线程（避免僵尸线程）**  
- 默认情况下，线程终止后会进入 **"僵尸状态"**，直到 `pthread_join()` **回收资源**。  
- 若不需要等待线程结束，可以使用 `pthread_detach()`：  

```c
pthread_t thread;
pthread_create(&thread, NULL, thread_func, NULL);
pthread_detach(thread);
```
- **分离的线程** 在结束时 **自动释放资源**，不需要 `pthread_join()`。  

---

## **Summary 总结**  
✅ `pthread_create()` → 创建线程。  
✅ `pthread_exit()` → 线程主动终止。  
✅ `pthread_join()` → 等待线程结束并获取返回值。  
✅ `pthread_detach()` → 使线程独立运行，无需回收资源。  

---

## **C++ 线程 (Pthreads)**  

### **将 C++ 对象与线程结合**  
POSIX 线程 (`pthreads`) 并不是 C++ 语言的内置功能，而是一个**库函数**。但我们可以使用 **C++ 的面向对象特性** 来组织多线程代码，使其更清晰、可维护。  

---

### **C++ 线程类的基本实现**  
我们可以定义一个类，每次 **创建对象时自动创建线程**。  

示例：  
```cpp
#include <pthread.h>  
#include <iostream>  

class Server {
public:
   Server(int in, int out) : in_(in), out_(out) {
      pthread_create(&thread_, NULL, start, (void*)this);
   }
private:
   int in_, out_;
   static void *start(void *arg);
   pthread_t thread_;
};

void *Server::start(void *arg) {
   Server* me = (Server*) arg;
   std::cout << "服务器线程处理输入：" << me->in_ << "\n";
   return NULL;
}

int main() {
   Server s1(1, 2);
   pthread_exit(NULL);
}
```
📌 **解释:**  
- `pthread_create()` 在 `Server` **构造函数** 中调用，自动启动线程。  
- `start()` **必须是静态函数**，否则 `pthread_create()` 不能正确调用它。  
- **使用 `this` 传递对象地址**，线程才能访问成员变量。  

---

### **线程的终止方法**  

#### **方法 1: 线程自行删除对象**  
```cpp
class Server {
public:
  Server(int in, int out): in_(in), out_(out) {
    pthread_create(&thread_, NULL, start, (void*)this);
    pthread_detach(thread_);
  }
  ~Server() { }
private:
  int in_, out_;
  static void *start(void *arg);
  pthread_t thread_;
};

void *Server::start(void *arg) {
  Server* me = (Server*) arg;
  std::cout << "服务器线程正在运行...\n";
  delete me;  // 线程结束时删除对象
  return NULL;
}
```
📌 **优缺点:**  
✅ **自动删除对象，防止内存泄漏**。  
❌ **不能使用 `pthread_join()`**，无法获取返回值。  

---

#### **方法 2: 在析构函数中等待线程结束**  
```cpp
class Server {
public:
   Server(int in, int out): in_(in), out_(out) {
      pthread_create(&thread_, NULL, start, (void*)this);
   }
   ~Server() {
      pthread_join(thread_, NULL);  // 确保线程结束
   }
};
```
📌 **优缺点:**  
✅ **确保线程在对象销毁前结束**。  
❌ **无法返回线程的结果**。  

---

#### **方法 3: 通过 `join()` 允许线程返回值**  
```cpp
class Server {
public:
   Server(int in, int out): in_(in), out_(out) {
      pthread_create(&thread_, NULL, start, (void*)this);
   }
   int join() {
      int ret;
      pthread_join(thread_, (void**)&ret);
      return ret;
   }
};
```
📌 **优缺点:**  
✅ **允许获取线程的返回值**。  
❌ **需要显式调用 `join()` 以防止内存泄漏**。  

---

## **总结 Summary**  
✅ `pthread_create()` → 创建线程。  
✅ `pthread_detach()` → 让线程自动回收资源。  
✅ `pthread_join()` → 等待线程结束并获取返回值。  
✅ **C++ 线程类** 需要 **特殊处理对象销毁与线程结束的关系**。  

---

## **多线程同步**  

### **为什么需要同步？**  
在多线程程序中，多个线程可能**同时访问共享数据**，如果没有正确的同步机制，就会出现 **数据竞争（Race Condition）**，导致 **数据损坏** 或 **未定义行为**。  

✅ **同步的作用：**  
1. **互斥（Mutex）** → 确保只有一个线程可以访问共享资源。  
2. **线程通信（条件变量）** → 允许线程相互通知执行顺序。  

---

### **互斥锁（Mutex）**  
**`pthread_mutex_t`** 确保同一时刻只有**一个线程**可以访问**关键区域（critical section）**。  

#### **POSIX 互斥锁示例**
```cpp
#include <pthread.h>
#include <stdio.h>

pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
int shared_counter = 0;

void *increment(void *arg) {
    for (int i = 0; i < 1000000; i++) {
        pthread_mutex_lock(&lock);
        shared_counter++;
        pthread_mutex_unlock(&lock);
    }
    return NULL;
}
```
📌 **关键点：**  
- `pthread_mutex_lock()` 加锁，保证 **同一时间只有一个线程能修改数据**。  
- `pthread_mutex_unlock()` 释放锁，让其他线程进入关键区域。  

---

### **死锁（Deadlock）**
**死锁** 发生在两个线程**相互等待对方释放资源**，导致程序卡死。  

```cpp
pthread_mutex_t lock1, lock2;
void *task1(void *arg) {
    pthread_mutex_lock(&lock1);
    pthread_mutex_lock(&lock2);  // 死锁等待
    pthread_mutex_unlock(&lock2);
    pthread_mutex_unlock(&lock1);
}
```
📌 **避免死锁的方法：**  
✅ **保证所有线程按相同顺序加锁**（比如 `lock1` → `lock2`）。  

---

## **总结 Summary**  
✅ `pthread_mutex_t` → 互斥锁，保证线程安全访问数据。  
✅ `pthread_cond_t` → 线程通信，通知某个线程执行。  

---

## **超越互斥锁：更复杂的线程同步问题**

### **1. 互斥锁的局限性**
互斥锁（mutex）能确保**一次只有一个线程**访问共享资源，但它过于严格，无法解决所有同步问题。例如：

- **问题 1：读者-写者问题（Readers-Writers Problem）**  
  - **多个读者（Readers）**同时读取数据时，它们并不会影响数据的完整性，因此应该允许多个读者**同时**访问资源。  
  - 但**写者（Writer）**修改数据时，需要**独占访问权限**，不能让读者或其他写者同时操作数据。  
  - 互斥锁不能区分**读**和**写**，会让所有访问变得串行，效率低下。

- **问题 2：生产者-消费者问题（Producer-Consumer Problem）**  
  - **生产者**将数据放入缓冲区，**消费者**从缓冲区取出数据。  
  - 如果**缓冲区已满**，生产者必须**等待**消费者取走数据。  
  - 如果**缓冲区为空**，消费者必须**等待**生产者放入新数据。  
  - 互斥锁不能解决这种**等待**和**通知**的问题。

- **问题 3：事件同步问题（Event Synchronization）**  
  - 线程需要等待某个**事件（Event）**发生，例如等待**磁盘读取完成**，然后所有等待的线程都可以继续执行。  
  - 互斥锁只能控制**访问顺序**，但无法**通知多个线程**，无法处理这种同步需求。

👉 **为了处理这些问题，我们需要**信号量（Semaphores）和**条件变量（Condition Variables）**。

---

## **2. 信号量（Semaphore）**
信号量是一个**非负整数**，它支持两种操作：
- **P 操作（等待，Wait）：**  
  - 如果信号量的值大于 0，执行 `semaphore--`，然后继续执行。  
  - 如果信号量的值是 0，线程必须**等待**，直到其他线程增加信号量的值。  
- **V 操作（信号，Signal）：**  
  - 线程执行 `semaphore++`，通知其他等待的线程它可以继续执行。

---

### **2.1 用信号量实现互斥锁**
我们可以用**二元信号量**（值为 0 或 1）来实现互斥锁：

```cpp
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

sem_t semaphore; // 定义信号量

void *task(void *arg) {
    sem_wait(&semaphore);  // P 操作（锁定资源）
    printf("线程 %ld 正在执行\n", (long)arg);
    sleep(1);
    sem_post(&semaphore);  // V 操作（释放资源）
    return NULL;
}

int main() {
    pthread_t threads[3];
    sem_init(&semaphore, 0, 1);  // 初始化信号量为 1（类似互斥锁）

    for (long i = 0; i < 3; i++)
        pthread_create(&threads[i], NULL, task, (void *)i);

    for (int i = 0; i < 3; i++)
        pthread_join(threads[i], NULL);

    sem_destroy(&semaphore);
    return 0;
}
```
📌 **解释：**  
- `sem_wait(&semaphore)` **锁定信号量**，让其他线程必须等待。  
- `sem_post(&semaphore)` **释放信号量**，让下一个线程执行。  

---

### **2.2 生产者-消费者问题**
在**生产者-消费者问题**中，我们需要：
- **`empty` 信号量** → 记录缓冲区中可用的空位。  
- **`full` 信号量** → 记录缓冲区中已经填满的数据。  
- **互斥锁 `pthread_mutex_t`** → 确保同时只有一个线程修改缓冲区。  

```cpp
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

#define BUFFER_SIZE 5
int buffer[BUFFER_SIZE];
int in = 0, out = 0;

sem_t empty, full;
pthread_mutex_t mutex;

void *producer(void *arg) {
    for (int i = 0; i < 10; i++) {
        sem_wait(&empty);    // 等待有空位
        pthread_mutex_lock(&mutex);

        buffer[in] = i;      // 生产数据
        printf("生产：%d\n", i);
        in = (in + 1) % BUFFER_SIZE;

        pthread_mutex_unlock(&mutex);
        sem_post(&full);     // 增加已填充的数量
        sleep(1);
    }
    return NULL;
}

void *consumer(void *arg) {
    for (int i = 0; i < 10; i++) {
        sem_wait(&full);     // 等待数据可用
        pthread_mutex_lock(&mutex);

        int item = buffer[out];  // 消费数据
        printf("消费：%d\n", item);
        out = (out + 1) % BUFFER_SIZE;

        pthread_mutex_unlock(&mutex);
        sem_post(&empty);    // 增加空位
        sleep(2);
    }
    return NULL;
}

int main() {
    pthread_t prod, cons;
    sem_init(&empty, 0, BUFFER_SIZE);
    sem_init(&full, 0, 0);
    pthread_mutex_init(&mutex, NULL);

    pthread_create(&prod, NULL, producer, NULL);
    pthread_create(&cons, NULL, consumer, NULL);

    pthread_join(prod, NULL);
    pthread_join(cons, NULL);

    sem_destroy(&empty);
    sem_destroy(&full);
    pthread_mutex_destroy(&mutex);
    return 0;
}
```
📌 **解释：**  
- **生产者等待缓冲区有空位，消费者等待有数据可用。**  
- 通过 `sem_wait()` 和 `sem_post()` **保证正确的执行顺序。**  

---

## **3. 条件变量（Condition Variables）**
条件变量允许**线程等待某个特定的条件变为真**。

### **3.1 读者-写者问题**
- **多个读者可以同时读数据。**  
- **写者必须独占访问权限。**  

```cpp
#include <pthread.h>
#include <stdio.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t readerQ = PTHREAD_COND_INITIALIZER;
pthread_cond_t writerQ = PTHREAD_COND_INITIALIZER;
int readers = 0, writers = 0;

void *reader(void *arg) {
    pthread_mutex_lock(&mutex);
    while (writers > 0)   // 等待写者完成
        pthread_cond_wait(&readerQ, &mutex);
    readers++;
    pthread_mutex_unlock(&mutex);

    printf("读者 %ld 读取数据\n", (long)arg);
    sleep(1);

    pthread_mutex_lock(&mutex);
    readers--;
    if (readers == 0) 
        pthread_cond_signal(&writerQ); // 允许写者执行
    pthread_mutex_unlock(&mutex);
    return NULL;
}

void *writer(void *arg) {
    pthread_mutex_lock(&mutex);
    while (readers > 0 || writers > 0) // 等待所有读者和写者完成
        pthread_cond_wait(&writerQ, &mutex);
    writers++;
    pthread_mutex_unlock(&mutex);

    printf("写者 %ld 修改数据\n", (long)arg);
    sleep(2);

    pthread_mutex_lock(&mutex);
    writers--;
    pthread_cond_broadcast(&readerQ); // 唤醒所有读者
    pthread_cond_signal(&writerQ);    // 允许下一个写者执行
    pthread_mutex_unlock(&mutex);
    return NULL;
}
```
📌 **解释：**  
- `pthread_cond_wait(&readerQ, &mutex)` **等待写者完成**。  
- `pthread_cond_broadcast(&readerQ)` **唤醒所有等待的读者**。  
- `pthread_cond_signal(&writerQ)` **唤醒下一个等待的写者**。  

---

## **总结**
✅ **信号量（Semaphore）** → 适用于生产者-消费者问题。  
✅ **条件变量（Condition Variables）** → 适用于读者-写者问题。  
✅ **两者都能解决更复杂的线程同步问题！** 🚀

---

## **受保护命令（Guarded Command）与 POSIX 线程同步实现**
---

在**多线程同步**中，**受保护命令（Guarded Command）**是一种逻辑结构，它确保**某个条件（Guard）满足时，线程才能执行特定代码块**。如果条件不满足，线程必须**等待**。

POSIX 线程（`pthread`）的**条件变量（Condition Variable）**提供了类似的机制，允许线程**在等待某个条件成立时进入休眠**，直到其他线程**修改条件并通知等待的线程**。

---

### **1. 受保护命令的概念**
**受保护命令的结构**
```cpp
when (guard) [  
    statement 1;  
    ...  
    statement n;  
]

[...]
```
📌 **解释：**  
- `guard`（**保护条件**）：线程只有在 `guard` 为真时才可以执行 `{ statement 1 ... statement n }`。  
- 如果 `guard` **不满足**，线程必须**等待**，直到 `guard` 变为 `true`。

---

### **2. POSIX 条件变量实现（与受保护命令对比）**
在 POSIX 线程库中，我们使用 **互斥锁（Mutex）** 和 **条件变量（Condition Variable）** 实现受保护命令。

#### **2.1 线程等待条件成立**
```cpp
pthread_mutex_lock(&mutex);
while (!guard)   // 只要条件不满足，就等待
    pthread_cond_wait(&cond_var, &mutex);  

statement 1;  
...  
statement n;  

pthread_mutex_unlock(&mutex);
```
📌 **解释：**  
1. **锁住互斥量 `mutex`**，防止其他线程并发修改 `guard`。  
2. **如果 `guard` 为假**，调用 `pthread_cond_wait(&cond_var, &mutex);` 让线程**进入等待状态**，直到 `guard` 被修改。  
3. **一旦 `guard` 为真**，线程恢复执行，执行 `{ statement 1 ... statement n }`。  
4. **释放互斥量 `mutex`**，允许其他线程访问资源。  

---

#### **2.2 代码修改 `guard` 并唤醒等待的线程**
```cpp
pthread_mutex_lock(&mutex);  

// 修改 guard
...

pthread_cond_broadcast(&cond_var); // 唤醒所有等待的线程  
pthread_mutex_unlock(&mutex);
```
📌 **解释：**  
1. 线程先**锁定 `mutex`**，确保对 `guard` 的修改是原子操作。  
2. 修改 `guard` 的值，使其满足等待线程的条件。  
3. 调用 `pthread_cond_broadcast(&cond_var);` **唤醒所有等待的线程**，让它们重新检查 `guard` 的值。  
4. **解锁 `mutex`**，让其他线程可以继续执行。  

---

### **3. 代码示例：生产者-消费者问题**
受保护命令的典型应用是**生产者-消费者问题**，它需要**在缓冲区满/空时让线程等待**。

```cpp
#include <stdio.h>
#include <pthread.h>

#define BUFFER_SIZE 5
int buffer[BUFFER_SIZE], count = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t not_full = PTHREAD_COND_INITIALIZER;
pthread_cond_t not_empty = PTHREAD_COND_INITIALIZER;

void *producer(void *arg) {
    for (int i = 0; i < 10; i++) {
        pthread_mutex_lock(&mutex);
        while (count == BUFFER_SIZE)  // 当缓冲区满时等待
            pthread_cond_wait(&not_full, &mutex);

        buffer[count++] = i;  // 生产数据
        printf("生产：%d\n", i);

        pthread_cond_signal(&not_empty);  // 唤醒消费者
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void *consumer(void *arg) {
    for (int i = 0; i < 10; i++) {
        pthread_mutex_lock(&mutex);
        while (count == 0)  // 当缓冲区为空时等待
            pthread_cond_wait(&not_empty, &mutex);

        int item = buffer[--count];  // 消费数据
        printf("消费：%d\n", item);

        pthread_cond_signal(&not_full);  // 唤醒生产者
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    pthread_t prod, cons;
    pthread_create(&prod, NULL, producer, NULL);
    pthread_create(&cons, NULL, consumer, NULL);
    
    pthread_join(prod, NULL);
    pthread_join(cons, NULL);
    
    return 0;
}
```
📌 **解释：**
- **`while (count == BUFFER_SIZE)`**：如果缓冲区已满，生产者等待。  
- **`pthread_cond_wait(&not_full, &mutex);`**：让生产者进入等待，直到消费者取出数据。  
- **`pthread_cond_signal(&not_empty);`**：当生产者放入数据后，唤醒消费者。  
- **`pthread_cond_broadcast(&cond_var);`**：确保所有等待的线程都能被唤醒。  

---

### **4. 关键点总结**
| **功能**            | **受保护命令** | **POSIX 线程实现** |
|--------------------|-------------|----------------|
| **保护条件检查**    | `when (guard) {}` | `while (!guard) pthread_cond_wait();` |
| **等待条件满足**    | **自动阻塞线程** | `pthread_cond_wait(&cond_var, &mutex);` |
| **修改条件并通知**  | **无须额外操作** | `pthread_cond_broadcast(&cond_var);` |
| **互斥保护**        | **默认支持** | `pthread_mutex_lock(&mutex);` |

---

### **5. 为什么 `pthread_cond_wait` 需要 `while` 而不是 `if`？**
#### **问题：为什么 `while (!guard)` 而不是 `if (!guard)`？**
- `pthread_cond_wait(&cond_var, &mutex);` **可能会被意外唤醒（spurious wakeup）**。  
- 其他线程也可能在修改 `guard` 后，**导致 `guard` 再次变为 `false`**。  
- `while` 确保**线程在被唤醒后必须重新检查 `guard` 是否真的为真**。  

📌 **示例：错误的 `if (!guard)` 代码**
```cpp
pthread_mutex_lock(&mutex);
if (!guard)
    pthread_cond_wait(&cond_var, &mutex);  
// 这里 guard 可能变成 false，但线程仍然继续执行，可能导致错误
```
📌 **正确的 `while (!guard)` 代码**
```cpp
pthread_mutex_lock(&mutex);
while (!guard)
    pthread_cond_wait(&cond_var, &mutex);  
// 这里确保 guard 真的为真后才执行后续代码
```

---

### **6. 结论**
1. **受保护命令** 是一种概念模型，POSIX 线程库用 **条件变量（Condition Variables）+ 互斥锁（Mutex）** 实现它。  
2. **`pthread_cond_wait()`** 让线程等待，直到某个条件成立。  
3. **`pthread_cond_broadcast()`** 唤醒所有等待的线程，**`pthread_cond_signal()`** 仅唤醒一个线程。  
4. **使用 `while` 而不是 `if`**，以防止**意外唤醒（spurious wakeup）**导致的错误执行。  

🚀 **POSIX 线程同步是一种强大的工具，它提供了灵活的方式来实现复杂的多线程同步逻辑！**

---

## **线程安全（Thread Safety）**
---

### **1. 引言**
许多 Unix 库函数是在 **多线程编程普及之前** 设计的，因此它们使用的一些编程模式 **与多线程程序不兼容**。

其中一个典型问题是**全局变量 `errno`** 的使用，它用于存储系统调用的错误码。但在多线程程序中，**所有线程共享同一个 `errno` 变量**，当多个线程同时发生错误时，`errno` 的值会被覆盖，导致错误处理混乱。

---

### **2. 多线程中的 `errno` 问题**
#### **2.1 问题：全局 `errno` 变量**
```cpp
int IOfunc() {
   extern int errno;
   ...
   if (write(fd, buffer, size) == -1) {  
       if (errno == EIO)  // 检查错误码  
         fprintf(stderr, "IO problems ...\n");
       ...
       return (0);
   }
   ...
}
```
📌 **问题：**  
- `errno` 是 **全局变量**，所有线程共享它。
- 如果**两个线程同时发生错误**，它们会**覆盖彼此的 `errno` 值**，导致错误处理错误。

#### **2.2 解决方案：使用线程局部存储（TSD）**
POSIX 提供了**线程特定数据（Thread-Specific Data，TSD）**，用于在每个线程中**存储私有变量**。

```cpp
#define errno pthread_getspecific(errno_key)
```
📌 **解释：**
- `pthread_getspecific(errno_key)` 让每个线程访问它**自己**的 `errno`。
- `pthread_setspecific(errno_key, err);` **存储错误码** 到线程局部存储中。

---

### **3. POSIX 线程特定数据（TSD）**
| **函数** | **描述** |
|----------|---------|
| `pthread_key_create(pthread_key_t *key, void (*destructor)(void *))` | 创建线程特定存储的键。 |
| `pthread_setspecific(pthread_key_t key, const void *value)` | 设置当前线程的值。 |
| `pthread_getspecific(pthread_key_t key)` | 获取当前线程存储的值。 |
| `pthread_key_delete(pthread_key_t key)` | 删除键（不影响线程本地数据）。 |

---

### **4. 示例：让 `errno` 线程安全**
```cpp
pthread_key_t errno_key;

void startup() {
   pthread_key_create(&errno_key, NULL);
}

int write(...) {
  int err = syscallTrap(WRITE, ...);
  if (err)
    pthread_setspecific(errno_key, (void *)(long)err);
}

#define errno ((int)(long)pthread_getspecific(errno_key))
```

---

### **5. 示例：线程安全的凭据存储**
```cpp
pthread_key_create(&cred_key, free_cred);
```
📌 **保证线程退出时自动释放存储的凭据！**

---

### **6. 总结**
🚀 **POSIX 线程特定数据是编写线程安全代码的强大工具！**

---

## **线程执行的偏差（Deviations）**
---

### **1. 引言：为什么需要控制线程的执行偏差**
通常情况下，线程的执行只会受到**同步机制**的影响（如互斥锁、条件变量）。但在某些情况下，我们希望有更强的控制能力，例如**强制终止一个线程**或让它改变执行流程。

- **POSIX 方案**：提供**线程取消机制**，允许一个线程请求另一个线程干净地终止。
- **Windows 方案**：**没有内置线程取消机制**，但可以通过其他方法实现类似功能。

---

### **2. Unix 信号：传统的中断机制**
Unix 通过**信号（Signal）**机制来处理系统中断，例如：

#### **何时触发信号？**
- **用户输入**：按 **Ctrl-C** 发送 `SIGINT` 信号。
- **定时器**：时间到达后触发信号。
- **进程间通信**：使用 `kill()` 发送信号。
- **异常**：除零错误或访问非法内存。

#### **示例：处理 SIGINT 信号**
```c
#include <signal.h>
#include <stdlib.h>
#include <stdio.h>

void handler(int sig) {
   printf("程序终止前清理资源...\n");
   exit(1);
}

int main() {
   signal(SIGINT, handler);  // 绑定信号处理程序
   while (1) {
      // 运行长时间代码
   }
   return 0;
}
```
📌 **说明**：
- 用户按 **Ctrl-C** 触发 `SIGINT`。
- `handler()` **执行清理任务**，然后退出程序。
- 如果没有 `handler()`，OS 直接终止进程。

---

### **3. 多线程信号处理的问题**
在**单线程程序**中，信号处理通常没问题。但在**多线程**程序中，会产生以下问题：
1. **哪个线程接收信号？**  
   - **信号是发送给进程的**，但**线程是进程的一部分**。
   - POSIX 规则：信号会传递给**未屏蔽该信号的随机线程**。

2. **信号处理函数可能修改共享数据**  
   - 多个线程可能同时修改同一个变量，导致**数据竞争**。
   - **不能使用互斥锁**，否则可能导致死锁。

---

### **4. 解决方案**
#### **方案 1：阻塞信号**
使用 `sigprocmask()` **在关键代码中屏蔽信号**：
```c
sigset_t set;
sigemptyset(&set);
sigaddset(&set, SIGINT);
sigprocmask(SIG_BLOCK, &set, NULL);  // 阻塞 SIGINT
// 关键操作
sigprocmask(SIG_UNBLOCK, &set, NULL);  // 解除屏蔽
```
⚠ **缺点**：
- 需要频繁调用 **系统调用**，可能影响性能。

---

#### **方案 2：使用专用的信号处理线程**
```c
void *monitor(void *arg) {
    int sig;
    while (1) {
        sigwait(&set, &sig);
        pthread_mutex_lock(&mut);
        printf("处理信号: %d\n", sig);
        pthread_mutex_unlock(&mut);
    }
    return NULL;
}
```
📌 **优势**：
✅ **不影响主线程执行**。  
✅ **安全访问共享数据**（使用互斥锁）。  
✅ **避免信号处理引发的竞态条件**。  

---
🚀 **结论：使用专用线程处理信号，更加安全和高效！**

---

## **POSIX 线程取消机制：安全终止线程**

### **1. 引言**
在许多程序中，**无法中途取消长时间运行的任务**，尤其是在 **多线程程序** 中。我们可能希望终止某个线程：
- **用户请求退出**（例如按下“取消”按钮）。
- **任务已完成，不再需要该线程**。

#### **线程终止的核心问题**
- **强制终止是危险的**：可能导致**互斥锁未释放**或**数据结构损坏**。
- **应确保资源正确释放**：如已分配的内存应被释放。
- **终止应发生在安全点**，避免破坏共享数据。

---

### **2. 为什么不能随意终止线程？**
考虑以下 **双向链表插入函数**：
```c
void insert(list_item_t *item) {
  pthread_mutex_lock(&list_mutex);
  item->backward_link = list_head;
  item->forward_link = list_head.forward_link;
  if (list_head.forward_link != 0)
    list_head.forward_link->backward_link = item;
  list_head.forward_link = item;
  pthread_mutex_unlock(&list_mutex);
}
```
📌 **问题**：
- 如果在 **执行中途终止** 线程：
  - **互斥锁（mutex）未释放**，其他线程会**永远阻塞**。
  - **链表变得不完整**，导致数据损坏。

✅ **解决方案**：
- 线程 **只能在预定义的“安全点”终止**。

---

### **3. POSIX 线程取消机制**
POSIX 允许 **一个线程请求另一个线程终止**：
```c
pthread_cancel(thread_id);
```
- **不会立即终止**线程，而是**标记**该线程需要取消。
- 线程是否终止取决于其 **取消状态（cancel state）** 和 **取消类型（cancel type）**。

#### **取消状态**
```c
pthread_setcancelstate(PTHREAD_CANCEL_ENABLE, NULL);  // 允许取消
pthread_setcancelstate(PTHREAD_CANCEL_DISABLE, NULL); // 禁止取消
```
- **启用取消（ENABLE）**：线程可以被终止。
- **禁用取消（DISABLE）**：线程忽略取消请求。

#### **取消类型**
```c
pthread_setcanceltype(PTHREAD_CANCEL_ASYNCHRONOUS, NULL); // 立即终止
pthread_setcanceltype(PTHREAD_CANCEL_DEFERRED, NULL);     // 延迟终止
```
- **异步（ASYNCHRONOUS）**：线程**随时可能终止**（不安全）。
- **延迟（DEFERRED）**：线程**仅在特定点终止**（安全）。

---

### **4. 结论**
✅ **POSIX 取消机制确保线程安全终止**。  
✅ **必须在安全点进行取消**，避免资源损坏。  
✅ **应使用清理处理程序（cleanup handlers）释放资源**。  
✅ **C++ 线程取消需要特别注意对象析构**。