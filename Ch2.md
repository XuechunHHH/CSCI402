# **Chapter2: OMultithreaded Programming**

## **Introduction to Multithreading**  

### **Why Do We Need Concurrency?**  
Modern computer systems must **handle multiple tasks at the same time**:  
✅ **Multiple running programs** → OS must share CPU time between them.  
✅ **I/O operations** → Disk, network, keyboard events all need attention.  
✅ **Efficient resource usage** → Prevents CPU from sitting idle while waiting for I/O.  

Even with **a single processor**, the OS **rapidly switches between tasks**, creating the **illusion of parallel execution**.  

---

### **What Will This Chapter Cover?**  
📌 **Multithreaded Programming Basics** → Writing concurrent programs.  
📌 **POSIX Threads (Pthreads) & Windows Threads (Win32)** → Multithreading in C/C++.  
📌 **Thread Synchronization** → Preventing race conditions & ensuring correctness.  
📌 **Thread Cancellation** → Managing & terminating threads safely.  
📌 **Application-Level Perspective** → Learning how to use threads in user programs before diving into OS-level threading.  

Even though **operating system internals** handle threading differently, **the core concepts remain the same**.  

---

## **Why Use Threads?**  

### **Why Do We Need Threads Instead of Just Processes?**  
We already use **multiple processes** in an OS, so why introduce **threads** inside a single process?  

✅ **Natural Concurrency** → Threads provide an easy way to implement concurrency.  
✅ **Not Just for Multiprocessors** → Even **single-core** CPUs benefit from threads through better I/O handling.  
✅ **Simpler, More Efficient Code** → Multi-threaded solutions are often **easier to write, read, and debug** than complex single-threaded ones.  

---

### **Example 1: Remote Login (`rlogind`)**
**Problem** → `rlogind` handles **two input/output streams simultaneously**:  
1. Reads **keyboard input** from a remote client and sends it to the server.  
2. Reads **application output** from the server and sends it back to the client.  

**Single-Threaded Approach (Hard to Read)**  
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
❌ **Hard to Read**  
❌ **Complex Non-Blocking I/O Handling**  
❌ **Difficult Debugging**  

---

### **Using Threads: Cleaner Code**
Instead of managing both streams **in one function**, we **split them into two threads**:

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
✅ **Easier to Understand**  
✅ **Simpler I/O Handling**  
✅ **Better Debugging**  

---

### **Example 2: Multithreaded Database Server**  
A **single-threaded database** must handle **many clients** one by one.  
- **Problem:** Some requests take longer than others.  
- **Single-threaded solution:** Each request gets a time slice (not efficient).  

**Multithreaded Approach:**  
- Each **client request gets a separate thread**.  
- Threads handle **short and long requests concurrently**.  
- If running on a **multiprocessor**, requests can execute in **true parallelism**.  

---

### **Key Takeaways**  
✅ **Threads make concurrent programming simpler & more efficient.**  
✅ **Threads aren’t just for multiprocessors—single processors benefit from better I/O handling.**  
✅ **Multithreaded programs are easier to write, debug, and maintain.**  

---

## **Programming with Threads**  

### **The Need for Standard Thread APIs**  
Despite the **benefits of multithreading**, standard **APIs for threads were only developed recently**:  
- **POSIX Threads (Pthreads)** → Standardized in **1995** as **POSIX 1003.1c**.  
- **Windows Threads (Win32 Threads)** → Microsoft’s custom implementation for **Windows OS**.  

Both APIs **work well for multithreading** but have **significant differences**:  
✅ **Different interfaces** → Functions & syntax vary.  
✅ **Different capabilities** → Some features in one API **cannot be easily replicated** in the other.  

Despite these differences, **both APIs enable effective multithreaded programming**.  

---

## **Thread Creation and Termination in POSIX (Pthreads)**  

### **Creating Threads in POSIX (Pthreads)**  
To create a thread in **Unix systems**, we use the `pthread_create()` function. This function:  
✅ **Creates a new thread** that runs a given function.  
✅ **Passes arguments** to the thread function.  
✅ **Stores the thread ID** in a `pthread_t` variable.  

Here’s an example of how to create multiple server threads:  

```c
#include <pthread.h>  
#include <stdio.h>  
#include <stdlib.h>  

#define NUM_THREADS 5  

void *server(void *arg) {
    int thread_id = *(int *)arg;
    printf("Server thread %d started.\n", thread_id);
    return NULL;
}

void start_servers() {
    pthread_t thread[NUM_THREADS];
    int i, args[NUM_THREADS];

    for (i = 0; i < NUM_THREADS; i++) {
        args[i] = i;
        if (pthread_create(&thread[i], NULL, server, &args[i]) != 0) {
            perror("pthread_create failed");
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

📌 **Explanation:**  
- `pthread_create(&thread[i], NULL, server, &args[i]);` → Creates a thread that runs `server()`.  
- `pthread_join(thread[i], NULL);` → Waits for the thread to finish.  
- Each thread prints its ID and exits.  

---

### **Handling Multiple Arguments in Threads**  
A challenge with `pthread_create()` is that it only allows **one argument** to be passed to the thread function.  

❌ **Wrong Approach: Passing a Local Variable's Address**  
```c
int arg = 5;
pthread_create(&thread, NULL, my_thread_function, &arg);
```
This might fail if `arg` **goes out of scope** before the thread uses it.  

✅ **Correct Approach: Using Dynamically Allocated Memory**  
```c
void *thread_func(void *arg) {
    int *val = (int *)arg;
    printf("Thread received: %d\n", *val);
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
Here, we use `malloc()` so that the memory remains valid until the thread is finished.  

---

### **Thread Termination in Pthreads**  
A thread can terminate in two ways:  
1. **Returning a value** → `return (void *) value;`  
2. **Calling `pthread_exit(value);`**  

Example:  
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

    printf("Thread returned: %ld\n", (long)ret_val);
    return 0;
}
```
📌 **Explanation:**  
- The thread returns `thread_id * 10`, which is retrieved by `pthread_join()`.  

---

### **Detached Threads (Avoiding Zombie Threads)**  
- Normally, a thread stays in a **"zombie state"** after finishing, until `pthread_join()` is called.  
- To **avoid zombies**, use `pthread_detach()`:  

```c
pthread_t thread;
pthread_create(&thread, NULL, thread_func, NULL);
pthread_detach(thread);
```
- Detached threads **free their resources automatically** when they finish.  

---

## **Threads and C++**  

### **Using C++ Objects for Multithreading**  
POSIX threads (`pthreads`) are implemented as a **subroutine library** rather than a built-in part of the C++ language. However, we can integrate thread management with **C++ object-oriented programming** to improve code structure and maintainability.  

### **Creating Threads in C++ Classes**  
A natural approach is to associate each thread with a C++ object, so that **creating an object automatically creates a thread**.  

Example:  
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
   static void *start(void *arg);  // Must be static!
   pthread_t thread_;
};

void *Server::start(void *arg) {
   Server* me = (Server*) arg;
   std::cout << "Server thread handling input: " << me->in_ << "\n";
   return NULL;
}

int main() {
   Server s1(1, 2);  // Automatically creates a thread
   pthread_exit(NULL);  // Allow the main thread to exit without terminating other threads
}
```
📌 **Explanation:**  
- The `Server` **constructor** creates a new thread that runs `start()`.  
- The `start()` function is **static** because POSIX requires a **standalone function pointer** (it doesn’t support C++ member functions directly).  
- We **pass `this`** to `start()` so the thread can access the object’s variables.  

---

### **Handling Thread Termination**  
To avoid memory leaks, we need to **delete the object when the thread terminates**.  

#### **Approach 1: Let the thread delete the object**  
```cpp
class Server {
public:
  Server(int in, int out): in_(in), out_(out) {
    pthread_create(&thread_, NULL, start, (void*)this);
    pthread_detach(thread_);  // No need for pthread_join()
  }
  ~Server() { }
private:
  int in_, out_;
  static void *start(void *arg);
  pthread_t thread_;
};

void *Server::start(void *arg) {
  Server* me = (Server*) arg;
  std::cout << "Processing on server\n";
  delete me;  // Delete object when thread finishes
  return NULL;
}
```
📌 **Pros & Cons:**  
✅ **No memory leaks** – object is deleted when the thread finishes.  
❌ **Cannot join the thread** – another thread cannot wait for its return value.  
❌ **Objects must be allocated dynamically (`new`)** – local objects would be deleted prematurely.  

---

#### **Approach 2: Destructor waits for thread termination**  
```cpp
class Server {
public:
   Server(int in, int out): in_(in), out_(out) {
      pthread_create(&thread_, NULL, start, (void*)this);
   }
   ~Server() {
      pthread_join(thread_, NULL);  // Wait for thread to terminate
   }
private:
   int in_, out_;
   static void *start(void *arg);
   pthread_t thread_;
};
```
📌 **Pros & Cons:**  
✅ **Ensures thread completes before object is destroyed**.  
❌ **Destructor cannot return a value** – thread return value is lost.  

---

#### **Approach 3: Separate Join Function**  
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
private:
   static void *start(void *arg);
   pthread_t thread_;
};
```
📌 **Pros & Cons:**  
✅ **Allows retrieving return value using `join()`**.  
❌ **Requires explicit `join()` call before deletion**.  

---

### **Creating a Generic Thread Base Class**  
A **base class for threads** allows easy subclassing for specific tasks.  

```cpp
class BaseThread {
public:
   BaseThread(void *(*start)(void *)) {
      pthread_create(&thread_, NULL, start, (void*)this);
   }
private:
   pthread_t thread_;
};

class DerivedThread : public BaseThread {
   static void *start(void *arg) {
      DerivedThread* t = (DerivedThread*) arg;
      std::cout << "Derived thread: " << t->a << "\n";
      return NULL;
   }
public:
   DerivedThread(int z) : a(z), BaseThread((void*(*)(void*))start) { }
   int a;
};
```
📌 **Problem:**  
- In C++, **base-class constructors run before derived-class constructors**.  
- The new thread might access `a` before it is initialized in `DerivedThread`.  

❌ **No simple way to delay thread execution until object is fully initialized**.  

---

## **English Explanation: Synchronization in Multithreaded Programming**  

### **Why Synchronization?**  
In multithreaded programs, multiple threads **share data** and **execute concurrently**. If multiple threads access the same data structure **without proper coordination**, **race conditions** and **data corruption** can occur.  

We need **synchronization mechanisms** to ensure that:  
1. **Mutual Exclusion (Mutex)** → Only one thread modifies shared data at a time.  
2. **Ordering (Synchronization Primitives)** → Threads execute in the correct sequence.  

This section focuses on **POSIX synchronization** (not Win32).  

---

### **Mutual Exclusion (Mutex)**  
A **mutex (mutual exclusion lock)** ensures that only **one thread** can access a **critical section** (shared data) at a time.  

#### **Using `pthread_mutex_t` in POSIX**  
```cpp
#include <pthread.h>
#include <stdio.h>

pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
int shared_counter = 0;

void *increment(void *arg) {
    for (int i = 0; i < 1000000; i++) {
        pthread_mutex_lock(&lock);   // Lock before modifying shared data
        shared_counter++;
        pthread_mutex_unlock(&lock); // Unlock after modification
    }
    return NULL;
}

int main() {
    pthread_t t1, t2;
    pthread_create(&t1, NULL, increment, NULL);
    pthread_create(&t2, NULL, increment, NULL);

    pthread_join(t1, NULL);
    pthread_join(t2, NULL);

    printf("Final counter value: %d\n", shared_counter);
    return 0;
}
```
📌 **Explanation:**  
- `pthread_mutex_lock(&lock)` **prevents other threads** from entering the critical section.  
- `pthread_mutex_unlock(&lock)` **allows other threads** to enter after modification.  
- `PTHREAD_MUTEX_INITIALIZER` **initializes the mutex**.  
- **Without mutex**, race conditions would cause incorrect results.  

---

### **Deadlocks: The Risk of Improper Locking**  
A **deadlock** occurs when two or more threads are **waiting for each other to release a lock**, causing the program to freeze.  

#### **Example of Deadlock**
```cpp
pthread_mutex_t lock1, lock2;

void *task1(void *arg) {
    pthread_mutex_lock(&lock1);
    pthread_mutex_lock(&lock2);  // Waits for lock2, but task2 holds it!
    // Critical section
    pthread_mutex_unlock(&lock2);
    pthread_mutex_unlock(&lock1);
}

void *task2(void *arg) {
    pthread_mutex_lock(&lock2);
    pthread_mutex_lock(&lock1);  // Waits for lock1, but task1 holds it!
    // Critical section
    pthread_mutex_unlock(&lock1);
    pthread_mutex_unlock(&lock2);
}
```
📌 **How Deadlock Happens:**  
1. **Task1 locks `lock1`**, waits for `lock2`.  
2. **Task2 locks `lock2`**, waits for `lock1`.  
3. **Neither can proceed → Deadlock!**  

✅ **Solution:** Always lock resources in the same order.  

---

📌 **How It Works:**  
- `pthread_cond_wait(&cond, &lock)` makes the **consumer wait** for `ready`.  
- `pthread_cond_signal(&cond)` **wakes up the consumer** when `ready = 1`.  

---

## **Beyond Mutual Exclusion**  

### **1. The Limitations of Mutexes**  
Mutexes ensure that only **one thread** executes a critical section at a time. This is great for protecting shared data but **too restrictive** for some scenarios.  

- **Problem 1: Readers-Writers**  
  - If **multiple readers** only read data, we don’t need to block them.  
  - But if a **writer** wants to modify the data, it must have **exclusive access**.  
  - Mutexes don’t differentiate between readers and writers—they block all access.  

- **Problem 2: Producer-Consumer (Bounded Buffer)**  
  - A producer **adds data** to a buffer, while a consumer **removes data**.  
  - If the buffer is **full**, the producer must **wait** before adding more.  
  - If the buffer is **empty**, the consumer must **wait** until new data arrives.  

- **Problem 3: Event Synchronization**  
  - Sometimes, multiple threads must **wait** for a specific event (e.g., a **disk read completes**).  
  - When the event occurs, **all waiting threads should be notified**.  

👉 **To solve these problems, we introduce Semaphores and Condition Variables.**  

---

## **2. Semaphores: A Powerful Synchronization Tool**  
A **semaphore** is an integer that supports **two operations**:  

- **P (wait operation)**: If the semaphore value is greater than 0, decrement it. Otherwise, the thread waits.  
- **V (signal operation)**: Increments the semaphore value, allowing other threads to proceed.  

```cpp
when (semaphore > 0) [  
    semaphore = semaphore - 1;
]

[semaphore = semaphore + 1]
```

### **2.1 Implementing Mutex with Semaphores**
We can use a **binary semaphore (value 0 or 1)** to implement a mutex:

```cpp
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

sem_t semaphore;

void *task(void *arg) {
    sem_wait(&semaphore);  // P operation (decrease)
    printf("Thread %ld is running\n", (long)arg);
    sleep(1);
    sem_post(&semaphore);  // V operation (increase)
    return NULL;
}

int main() {
    pthread_t threads[3];
    sem_init(&semaphore, 0, 1);  // Initialize semaphore to 1 (binary semaphore)

    for (long i = 0; i < 3; i++)
        pthread_create(&threads[i], NULL, task, (void *)i);

    for (int i = 0; i < 3; i++)
        pthread_join(threads[i], NULL);

    sem_destroy(&semaphore);
    return 0;
}
```
📌 **Explanation:**  
- **Only one thread runs at a time** because `sem_wait()` blocks all others until `sem_post()` is called.  
- This ensures **mutual exclusion** using a **semaphore instead of a mutex**.  

---

### **2.2 Producer-Consumer Problem with Semaphores**  
In the **producer-consumer problem**, we use:  
- **`empty` semaphore** → Tracks available slots in the buffer.  
- **`full` semaphore** → Tracks filled slots in the buffer.  
- **Mutex (`pthread_mutex_t`)** → Prevents multiple threads from modifying shared data at the same time.  

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
        sem_wait(&empty);    // Wait if buffer is full
        pthread_mutex_lock(&mutex);

        buffer[in] = i;      // Produce item
        printf("Produced: %d\n", i);
        in = (in + 1) % BUFFER_SIZE;

        pthread_mutex_unlock(&mutex);
        sem_post(&full);     // Signal that buffer has data
        sleep(1);
    }
    return NULL;
}

void *consumer(void *arg) {
    for (int i = 0; i < 10; i++) {
        sem_wait(&full);     // Wait if buffer is empty
        pthread_mutex_lock(&mutex);

        int item = buffer[out];  // Consume item
        printf("Consumed: %d\n", item);
        out = (out + 1) % BUFFER_SIZE;

        pthread_mutex_unlock(&mutex);
        sem_post(&empty);    // Signal that buffer has space
        sleep(2);
    }
    return NULL;
}

int main() {
    pthread_t prod, cons;
    sem_init(&empty, 0, BUFFER_SIZE); // Initially, all slots are empty
    sem_init(&full, 0, 0);            // No slots are full at the start
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
📌 **How It Works:**  
- **Producer waits (`sem_wait(empty)`) if the buffer is full.**  
- **Consumer waits (`sem_wait(full)`) if the buffer is empty.**  
- **Ensures correct execution order** between producer and consumer threads.  

---

## **3. Condition Variables: A More Efficient Synchronization Tool**  
Condition variables allow **threads to wait for a specific condition to become true**.  

### **3.1 Reader-Writer Problem with Condition Variables**
- **Multiple readers can read simultaneously.**  
- **A writer must have exclusive access.**  

```cpp
#include <pthread.h>
#include <stdio.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t readerQ = PTHREAD_COND_INITIALIZER;
pthread_cond_t writerQ = PTHREAD_COND_INITIALIZER;
int readers = 0, writers = 0;

void *reader(void *arg) {
    pthread_mutex_lock(&mutex);
    while (writers > 0)   // Wait if a writer is active
        pthread_cond_wait(&readerQ, &mutex);
    readers++;
    pthread_mutex_unlock(&mutex);

    printf("Reader %ld reading\n", (long)arg);
    sleep(1);

    pthread_mutex_lock(&mutex);
    readers--;
    if (readers == 0) 
        pthread_cond_signal(&writerQ); // Allow writers to proceed
    pthread_mutex_unlock(&mutex);
    return NULL;
}

void *writer(void *arg) {
    pthread_mutex_lock(&mutex);
    while (readers > 0 || writers > 0) // Wait if readers or writers exist
        pthread_cond_wait(&writerQ, &mutex);
    writers++;
    pthread_mutex_unlock(&mutex);

    printf("Writer %ld writing\n", (long)arg);
    sleep(2);

    pthread_mutex_lock(&mutex);
    writers--;
    pthread_cond_broadcast(&readerQ); // Allow readers to proceed
    pthread_cond_signal(&writerQ);    // Allow other writers
    pthread_mutex_unlock(&mutex);
    return NULL;
}

int main() {
    pthread_t r1, r2, w1;
    pthread_create(&r1, NULL, reader, (void *)1);
    pthread_create(&r2, NULL, reader, (void *)2);
    pthread_create(&w1, NULL, writer, (void *)1);

    pthread_join(r1, NULL);
    pthread_join(r2, NULL);
    pthread_join(w1, NULL);

    return 0;
}
```
📌 **How It Works:**  
- **`pthread_cond_wait(&readerQ, &mutex)`** → Readers **wait** if a writer is active.  
- **`pthread_cond_wait(&writerQ, &mutex)`** → Writers **wait** if readers are active.  
- **`pthread_cond_signal()`** wakes up a **single** waiting thread.  
- **`pthread_cond_broadcast()`** wakes up **all** waiting threads.  

---

## **Summary**  
✅ **Semaphores** → Used for producer-consumer problems.  
✅ **Condition Variables** → Used for reader-writer problems.  
✅ **Both help in solving complex synchronization challenges.**  

---

## **Guarded Command and POSIX Thread Synchronization Implementation**
---

In **multithreaded synchronization**, a **Guarded Command** is a logical structure that ensures a **thread executes a specific block of code only when a condition (guard) is met**. If the condition is not met, the thread must **wait**.

In **POSIX threads (`pthread`)**, **condition variables (Condition Variable)** provide a similar mechanism, allowing threads **to enter a sleep state while waiting for a condition to be met**, until another thread **modifies the condition and notifies the waiting threads**.

---

### **1. Concept of Guarded Command**
**Guarded Command Structure**
```cpp
when (guard) [  
    statement 1;  
    ...  
    statement n;  
]

[...]
```
📌 **Explanation:**  
- `guard` (**the condition**): The thread can execute `{ statement 1 ... statement n }` **only when `guard` is true**.  
- If `guard` **is not met**, the thread must **wait** until `guard` becomes `true`.

---

### **2. POSIX Condition Variable Implementation (Comparison with Guarded Command)**
In the POSIX thread library, we use **Mutex (Mutual Exclusion Lock)** and **Condition Variables** to implement the guarded command.

#### **2.1 Thread Waiting for Condition**
```cpp
pthread_mutex_lock(&mutex);
while (!guard)   // Wait while condition is false
    pthread_cond_wait(&cond_var, &mutex);  

statement 1;  
...  
statement n;  

pthread_mutex_unlock(&mutex);
```
📌 **Explanation:**  
1. **Locks the mutex `mutex`**, preventing other threads from modifying `guard` concurrently.  
2. **If `guard` is false**, calling `pthread_cond_wait(&cond_var, &mutex);` makes the thread **enter a waiting state** until `guard` is modified.  
3. **Once `guard` becomes true**, the thread resumes execution and executes `{ statement 1 ... statement n }`.  
4. **Releases the mutex `mutex`**, allowing other threads to access resources.  

---

#### **2.2 Modifying `guard` and Waking Up Waiting Threads**
```cpp
pthread_mutex_lock(&mutex);  

// Modify guard
...

pthread_cond_broadcast(&cond_var); // Wake up all waiting threads  
pthread_mutex_unlock(&mutex);
```
📌 **Explanation:**  
1. The thread **locks `mutex` first**, ensuring that modifying `guard` is an atomic operation.  
2. Modifies the value of `guard`, making it valid for waiting threads.  
3. Calls `pthread_cond_broadcast(&cond_var);` **to wake up all waiting threads**, allowing them to recheck the value of `guard`.  
4. **Unlocks `mutex`**, allowing other threads to continue execution.  

---

### **3. Code Example: Producer-Consumer Problem**
A typical application of the guarded command is the **producer-consumer problem**, where **threads must wait when the buffer is full/empty**.

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
        while (count == BUFFER_SIZE)  // Wait if the buffer is full
            pthread_cond_wait(&not_full, &mutex);

        buffer[count++] = i;  // Produce an item
        printf("Produced: %d\n", i);

        pthread_cond_signal(&not_empty);  // Wake up the consumer
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void *consumer(void *arg) {
    for (int i = 0; i < 10; i++) {
        pthread_mutex_lock(&mutex);
        while (count == 0)  // Wait if the buffer is empty
            pthread_cond_wait(&not_empty, &mutex);

        int item = buffer[--count];  // Consume an item
        printf("Consumed: %d\n", item);

        pthread_cond_signal(&not_full);  // Wake up the producer
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
📌 **Explanation:**
- **`while (count == BUFFER_SIZE)`**: The producer waits if the buffer is full.  
- **`pthread_cond_wait(&not_full, &mutex);`**: Makes the producer wait until the consumer removes an item.  
- **`pthread_cond_signal(&not_empty);`**: The producer notifies the consumer after producing an item.  
- **`pthread_cond_broadcast(&cond_var);`**: Ensures all waiting threads are awakened when necessary.  

---

### **4. Key Summary**
| **Functionality**   | **Guarded Command** | **POSIX Thread Implementation** |
|--------------------|------------------|----------------------------|
| **Checking Condition** | `when (guard) {}` | `while (!guard) pthread_cond_wait();` |
| **Waiting for Condition** | **Automatically blocks the thread** | `pthread_cond_wait(&cond_var, &mutex);` |
| **Modifying Condition & Notification** | **No additional operation required** | `pthread_cond_broadcast(&cond_var);` |
| **Mutual Exclusion** | **Implicitly supported** | `pthread_mutex_lock(&mutex);` |

---

### **5. Why Use `while (!guard)` Instead of `if (!guard)`?**
#### **Issue: Why Use `while (!guard)` Instead of `if (!guard)`?**
- `pthread_cond_wait(&cond_var, &mutex);` **may experience spurious wakeups (unexpected wakeups).**  
- Other threads might also modify `guard`, **causing it to become false again**.  
- `while` ensures that **the thread rechecks `guard` before proceeding** after being awakened.

📌 **Incorrect `if (!guard)` Code**
```cpp
pthread_mutex_lock(&mutex);
if (!guard)
    pthread_cond_wait(&cond_var, &mutex);  
// Here, guard might become false again, but the thread still proceeds, causing errors
```
📌 **Correct `while (!guard)` Code**
```cpp
pthread_mutex_lock(&mutex);
while (!guard)
    pthread_cond_wait(&cond_var, &mutex);  
// Ensures the guard is truly valid before proceeding
```

---

### **6. Conclusion**
1. **Guarded Command** is a conceptual model implemented in POSIX threads using **Condition Variables + Mutexes**.  
2. **`pthread_cond_wait()`** allows a thread to wait until a certain condition is met.  
3. **`pthread_cond_broadcast()`** wakes up all waiting threads, while **`pthread_cond_signal()`** wakes up only one thread.  
4. **Use `while` instead of `if`** to avoid errors due to **spurious wakeups**.  

🚀 **POSIX thread synchronization is a powerful tool, providing a flexible way to implement complex multithreaded synchronization logic!**

---

## **Thread Safety in Multithreaded Programming**
---

### **English Explanation**
#### **1. Introduction**
Many Unix library functions were designed **before** multithreading became common. As a result, certain programming practices in these libraries **conflict with multithreaded programming**.

A classic example is the use of the global variable `errno` to store error codes from system calls. This approach is problematic in multithreaded programs because **all threads share the same `errno` variable**, leading to conflicts when multiple threads fail system calls simultaneously.

---

### **2. The `errno` Problem in Multithreading**
#### **2.1 Problem: Global `errno` Variable**
```cpp
int IOfunc() {
   extern int errno;
   ...
   if (write(fd, buffer, size) == -1) {  
       if (errno == EIO)  // Check error code  
         fprintf(stderr, "IO problems ...\n");
       ...
       return (0);
   }
   ...
}
```
📌 **Issue:**  
- `errno` is a **global variable**, shared by all threads.
- If two threads fail system calls at the same time, they will **overwrite each other's `errno` value**, causing incorrect error handling.

#### **2.2 Solution: Using Thread-Specific Data**
A better approach is to make `errno` **thread-local**, so each thread has its own instance of `errno`.  
POSIX provides **thread-specific data** for this purpose.

```cpp
#define errno pthread_getspecific(errno_key)
```
📌 **Explanation:**
- `pthread_getspecific(errno_key)` retrieves a **thread-local** version of `errno`, avoiding conflicts.
- `pthread_setspecific(errno_key, err);` stores error codes in **thread-specific storage** instead of a global variable.

---

### **3. POSIX Thread-Specific Data (TSD)**
POSIX provides the following functions to manage **thread-local storage**:

| **Function**        | **Description** |
|---------------------|----------------|
| `pthread_key_create(pthread_key_t *key, void (*destructor)(void *))` | Creates a unique key for thread-specific storage. |
| `pthread_setspecific(pthread_key_t key, const void *value)` | Sets the value for the calling thread. |
| `pthread_getspecific(pthread_key_t key)` | Retrieves the value stored for the calling thread. |
| `pthread_key_delete(pthread_key_t key)` | Deletes the key (not thread-local data itself). |

---

### **4. Example: Making `errno` Thread-Specific**
```cpp
pthread_key_t errno_key;  // Global key

void startup() {
   // This should be executed by only one thread at startup
   pthread_key_create(&errno_key, NULL);  
}

int write(...) {  // Wrapper in the system-call library
  int err = syscallTrap(WRITE, ...); // Actual syscall
  if (err)
    pthread_setspecific(errno_key, (void *)(long)err);
  ...
}

#define errno ((int)(long)pthread_getspecific(errno_key))

void IOfunc() {
  if (write(fd, buffer, size) == -1) {  
    if (errno == EIO)  
       fprintf(stderr, "IO problems ...\n");
    ...
    return (0);
  }
  ...
}
```
📌 **How It Works:**
1. `pthread_key_create(&errno_key, NULL);` **creates a key** that allows each thread to store its own `errno`.
2. Each thread **stores its error code** using `pthread_setspecific(errno_key, err);`
3. When a thread accesses `errno`, it actually calls `pthread_getspecific(errno_key)`, retrieving its **own instance** of `errno`.

---

### **5. Example: Using Thread-Specific Data for Credentials**
#### **5.1 Problem: Storing Request Credentials in a Multithreaded Server**
A multithreaded server creates a **new thread** for each client request.  
Each thread has **credentials** associated with the request, which must be safely stored and cleaned up.

#### **5.2 Solution: Store Credentials Using Thread-Specific Data**
```cpp
#define cred pthread_getspecific(cred_key)
pthread_key_t cred_key;

int main() {
  pthread_key_create(&cred_key, free_cred);  // Create key with cleanup function

  while (more_requests) {
     pthread_t thread;
     pthread_create(&thread, 0, server_start, request);
  }
}

void *server_start(void *req) {
  cred_t *credp = (cred_t *)malloc(sizeof(cred_t));  // Allocate storage
  pthread_setspecific(cred_key, credp);  // Store thread-specific data

  handle_request(req);
  return NULL;
}

void handle_request(req_t req) {
  if (credentials_valid(req, cred))  
    perform(req);
}

void free_cred(cred_t *credp) {
  free(credp);  // Free storage when the thread exits
}
```
📌 **How It Works:**
1. **Each thread gets its own instance of `cred`**, avoiding conflicts.
2. **When a thread exits**, `free_cred()` **automatically cleans up memory**.
3. The **POSIX thread library ensures that `free_cred()` is called** when a thread terminates.

---

### **6. Summary**
| **Problem** | **Solution** |
|------------|-------------|
| **Global `errno` conflicts** | Use `pthread_getspecific()` and `pthread_setspecific()` to store a thread-specific version of `errno`. |
| **Each thread needs private storage** | Use POSIX **thread-specific data (TSD)**. |
| **Threads must clean up allocated data** | Register a **destructor function** with `pthread_key_create()`. |

🚀 **Thread-specific data is a powerful tool for writing thread-safe code in POSIX!**

---

## **Deviations in Thread Execution**
---

### **1. Introduction: Why Deviations Matter**
By default, a thread’s execution can only be influenced by others **through synchronization mechanisms** such as mutexes or condition variables. However, in certain cases, it is beneficial to have the ability to **force a thread to terminate or change its execution flow**.  

- **POSIX Solution**: **Cancellation Mechanism** allows one thread to request the clean termination of another.
- **Windows Solution**: **No built-in cancellation mechanism**, but it provides workarounds.

---

### **2. Unix Signals: Traditional Interruption Mechanism**
Unix introduced **signals** as a way for the operating system to interrupt a process when specific events occur.  

#### **Examples of when signals occur**:
- **User input**: Pressing **Ctrl-C** sends `SIGINT`.
- **Timers**: A timeout triggers a signal.
- **Explicit signals**: A process can send a signal to another process using `kill()`.
- **Exceptions**: Division by zero or invalid memory access.

#### **Example: Handling SIGINT to Perform Cleanup**
```c
#include <signal.h>
#include <stdlib.h>
#include <stdio.h>

void handler(int sig) {
   printf("Cleaning up before termination...\n");
   exit(1);
}

int main() {
   signal(SIGINT, handler);  // Register SIGINT handler
   while (1) {
      // Some long-running code
   }
   return 0;
}
```
📌 **Explanation**:
- If the user presses **Ctrl-C**, `SIGINT` is triggered.
- The signal handler **performs cleanup** and exits the program.
- Without a handler, the OS would terminate the process **immediately**.

---

### **3. Problems with Signals in Multi-Threaded Programs**
In a **single-threaded** program, signals work well. But in **multi-threaded** applications, problems arise:
1. **Which thread receives the signal?**  
   - **Signals are sent to the process**, not a specific thread.  
   - POSIX rules: The signal is delivered to **one random thread** that does not have the signal blocked.
  
2. **Unsafe operations inside signal handlers**  
   - If the handler modifies shared data, it can cause **race conditions**.
   - Mutexes **cannot be used** inside a signal handler because the thread might already hold the lock, leading to a **deadlock**.

---

### **4. Ensuring Safe Signal Handling**
To prevent race conditions, we need to make our signal handlers **async-signal safe**.

#### **Solution 1: Temporarily Blocking Signals**
We can **block signals** while modifying shared data using `sigprocmask()`:

```c
#include <signal.h>

sigset_t set;

void handler(int sig) {
   printf("Signal received\n");
}

int main() {
   sigemptyset(&set);
   sigaddset(&set, SIGINT);
   signal(SIGINT, handler);

   sigprocmask(SIG_BLOCK, &set, NULL); // Block SIGINT

   // Critical section - update shared data
   sigprocmask(SIG_UNBLOCK, &set, NULL); // Unblock SIGINT

   while (1) { }
   return 0;
}
```
📌 **Explanation**:
- `sigprocmask(SIG_BLOCK, &set, NULL);` **prevents** SIGINT from interrupting critical operations.
- Once the operation is finished, `SIGINT` is **unblocked**.

⚠ **Drawback**: Using `sigprocmask()` frequently **reduces performance** because it involves a system call.

---

### **5. Better Solution: Dedicated Signal Handling Thread**
Instead of handling signals in random threads, we can create a **dedicated signal-handling thread**:

```c
#include <pthread.h>
#include <signal.h>
#include <stdio.h>

sigset_t set;
pthread_mutex_t mut = PTHREAD_MUTEX_INITIALIZER;

void *monitor(void *arg) {
    int sig;
    while (1) {
        sigwait(&set, &sig);  // Wait for signal
        pthread_mutex_lock(&mut);
        printf("Handled signal: %d\n", sig);
        pthread_mutex_unlock(&mut);
    }
    return NULL;
}

int main() {
    pthread_t thread;
    sigemptyset(&set);
    sigaddset(&set, SIGINT);
    pthread_sigmask(SIG_BLOCK, &set, NULL);  // Block SIGINT in all threads

    pthread_create(&thread, NULL, monitor, NULL);  // Dedicated thread to handle SIGINT

    while (1) { }  // Main work
    return 0;
}
```
📌 **Explanation**:
- **Main thread blocks SIGINT** so it doesn't receive the signal.
- A separate **monitor thread waits for the signal** using `sigwait()`.
- The monitor thread **processes the signal synchronously**, avoiding unsafe interruptions.

✅ **Advantages**:
- No unexpected **interruptions**.
- **Safe access** to shared resources using mutexes.
- Easier debugging and maintenance.

---

# **POSIX Thread Cancellation: Managing Thread Termination Safely**

## **English Version**

### **1. Introduction**
A common annoyance in many programs is the inability to cancel a long-running operation. This issue is even more challenging in **multi-threaded programs**, where we may need to terminate a thread cleanly without corrupting shared resources.

#### **Key Issues with Thread Termination**
- **Arbitrary termination is dangerous**: If a thread is forcefully stopped, it could leave **mutexes locked** or **data structures in an inconsistent state**.
- **Resources should be cleaned up**: If a thread allocated memory before being terminated, that memory should be freed.
- **Cancellation should happen at safe points**: If a thread is terminated while modifying shared data, other threads may encounter corrupted data.

---

### **2. Why Arbitrary Termination is Unsafe**
Consider the following **linked list insertion function**:
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
📌 **Problem**: If the thread executing `insert()` is terminated **midway**, it may leave:
- **The mutex locked**, preventing other threads from modifying the list.
- **The list in an inconsistent state**, causing future operations to fail.

**Solution**: Thread termination should **only** occur at predefined **safe points** where cleanup is possible.

---

### **3. POSIX Thread Cancellation Mechanism**
POSIX provides **thread cancellation**, which allows **one thread to request another thread’s termination safely**. It behaves like an exception that a thread can handle at **safe points**.

#### **How to Request Thread Cancellation**
```c
pthread_cancel(thread_id);
```
- The target thread **is not immediately terminated**.
- Instead, it is marked as "pending cancellation."
- Its termination depends on **cancellation state and type**.

#### **Cancellation States**
A thread can **enable or disable** cancellation:
```c
pthread_setcancelstate(PTHREAD_CANCEL_ENABLE, NULL);  // Allow cancellation
pthread_setcancelstate(PTHREAD_CANCEL_DISABLE, NULL); // Ignore cancellation
```
- **`PTHREAD_CANCEL_ENABLE`**: The thread can be canceled.
- **`PTHREAD_CANCEL_DISABLE`**: The thread ignores cancellation requests.

#### **Cancellation Types**
Defines **when** the thread responds to a cancellation request:
```c
pthread_setcanceltype(PTHREAD_CANCEL_ASYNCHRONOUS, NULL); // Immediate termination
pthread_setcanceltype(PTHREAD_CANCEL_DEFERRED, NULL);     // Safe point termination
```
- **`PTHREAD_CANCEL_ASYNCHRONOUS`**: The thread can be canceled **at any moment** (unsafe).
- **`PTHREAD_CANCEL_DEFERRED`**: The thread is canceled only **at specific points** (safe).

---

### **4. Safe Points: Cancellation Points**
POSIX defines **cancellation points**, which are system calls where a thread can be safely terminated:
- **I/O operations** (`read()`, `write()`, `close()`, `open()`)
- **Thread synchronization** (`pthread_join()`, `pthread_cond_wait()`)
- **Sleeping operations** (`sleep()`, `nanosleep()`)
- **Others** (`system()`, `waitpid()`)

#### **Example: Cancellation in a Thread**
```c
void *thread_func(void *arg) {
    while (1) {
        pthread_testcancel();  // Check if cancel request exists
        printf("Running...\n");
        sleep(1);
    }
    return NULL;
}
```
📌 **Explanation**:
- `pthread_testcancel()` acts as a **manual cancellation point**.
- The thread checks for cancellation **before continuing** execution.

---

### **5. Cleanup Handlers: Ensuring Resource Release**
If a thread **allocated memory** or **locked a mutex**, it must clean up before termination. POSIX provides **cleanup handlers** using:
```c
pthread_cleanup_push(handler_function, argument);
pthread_cleanup_pop(execute_flag);
```
- `pthread_cleanup_push()` **registers a cleanup function**.
- `pthread_cleanup_pop(1)` **executes the cleanup function** if the flag is `1`.

#### **Example: Ensuring Memory Cleanup**
```c
void cleanup(void *ptr) {
    free(ptr);  // Free allocated memory
}

void *thread_func(void *arg) {
    char *buffer = malloc(256);
    pthread_cleanup_push(cleanup, buffer);

    while (1) {
        pthread_testcancel();
        printf("Processing...\n");
        sleep(1);
    }

    pthread_cleanup_pop(1);  // Execute cleanup
    return NULL;
}
```
📌 **Explanation**:
- If the thread is canceled, `cleanup()` ensures that `buffer` is freed.
- Using `pthread_cleanup_pop(1)`, cleanup **always executes** when exiting.

---

### **6. Thread Cancellation in C++**
Cancellation in **C++ is tricky** due to destructors. If a thread is canceled **before** an object’s destructor runs, **resource leaks may occur**.

#### **Example: Potential Resource Leak**
```cpp
void thread_func() {
    A obj;  // Will destructor run if thread is canceled?
    pthread_cleanup_push(handler, NULL);
    work();
    pthread_cleanup_pop(0);
}
```
📌 **Issue**:
- If the thread is canceled inside `work()`, **the destructor of `obj` may never run**.
- Some C++ compilers (e.g., older GCC versions) **don't guarantee destructor execution**.

✅ **Solution**:
- Place objects **inside a cleanup handler**.
- Manually call destructors if needed.

---

### **7. Conclusion**
✅ **POSIX cancellation allows controlled termination** of threads.  
✅ **Safe points (cancellation points) prevent resource corruption**.  
✅ **Cleanup handlers ensure resource deallocation** before termination.  
✅ **C++ threads need special care due to destructors**.  
