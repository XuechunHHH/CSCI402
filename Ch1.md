# **Chapter1: Operating System**

## **Understanding the Operating System**

### **What is an Operating System?**  
An operating system (OS) is the **core software** that sits between application programs and the computer hardware. It **manages resources** and provides essential **abstractions** so that users and programs can interact with the system efficiently.

A simple definition could be:  
> **An OS is software that most other programs depend on to interact with the hardware.**  

However, defining an OS precisely is difficult because its boundaries vary across different computing environments.

---

### **Functions of an Operating System**  
From a **programmer’s perspective**, an OS provides **abstractions** of the underlying hardware. Additionally, because multiple programs may use the same hardware resources at the same time, the OS **manages resource sharing** efficiently.

The OS abstracts several hardware components, including:
- **Processors** – Controls CPU (Core) execution for multiple processes.
   + **threads** of control
- **RAM (Memory)** – Manages primary storage allocation.
   + **process(program)**
- **Disks** – Handles secondary storage and file management.
   + **Files** - file system
- **Network Interface** – Provides network communication capabilities.
   + **Communication**
- **Display** – Manages graphical output and rendering.
   + **windows, graphic**
- **Keyboard & Mouse** – Handles user input.
   + **input & locator**

---

### **Key OS Abstractions**

1. **Process Abstraction**  
   - Programs believe they have exclusive access to the CPU and memory.
   - The OS **schedules processes** and **allocates memory dynamically**.
   - Some memory may actually be stored on **disk (virtual memory)**.

2. **File Abstraction**  
   - Instead of dealing with complex disk operations, programmers interact with **files**.
   - The OS allows creating, reading, writing, and deleting files.
   - Users do not need to manage **low-level disk sectors** manually.

3. **Networking Abstraction**  
   - The OS simplifies network communication through:
     - **Sockets** (for direct network programming).
     - **Remote Procedure Calls (RPC)** (to execute functions over a network).
     - **Web-based protocols** (for internet services).

4. **Display, Keyboard, and Mouse Abstraction**  
   - The OS provides **windowing and graphics systems** for GUI interactions.
   - Mouse clicks and keyboard inputs are **translated into system events**.
   - Applications do not need to process **low-level input signals** directly.

---

### **Why Use Abstractions?**  

1. **Simplifies Programming**  
   - Developers do not need to handle hardware details directly.
   - Example: Instead of manually accessing disk storage, they use **files**.

2. **Supports Resource Sharing**  
   - The OS **multiplexes** CPU time and memory across multiple processes.
   - Example: Multiple users can access a shared server without conflicts.

3. **Improves Efficiency**  
   - The OS optimizes resource utilization, ensuring **fair scheduling**.
   - Example: **Virtual memory** allows programs to run even if RAM is limited.

---

### **The Boundaries of an OS**  
Some abstractions are traditionally part of the **operating system**, while others belong to separate fields:

| OS Component           | Separate Field |
|------------------------|---------------|
| **File Systems**       | Databases |
| **Basic Graphics Handling** | Computer Graphics |
| **Process Management** | Distributed Systems |

Thus, while **file systems** are part of the OS, **databases** are not. Similarly, **low-level graphics handling** belongs to the OS, but **high-level windowing systems** are considered separate.

---

### **Conclusion**  
The OS plays a crucial role in computing by managing hardware and **providing structured abstractions**.  
Its key responsibilities include:
✅ **Process Management** – Ensuring smooth execution of programs.  
✅ **File System Management** – Organizing and handling disk storage.  
✅ **Memory Management** – Efficiently allocating and managing RAM.  
✅ **Networking** – Enabling communication between systems.  
✅ **User Interface** – Handling input/output from the display, keyboard, and mouse.  

The OS ensures that users and programs can **use computing resources efficiently** without worrying about hardware-level complexity.

---

## **Operating Systems as a Field of Study**

### **Why Study Operating Systems?**  
Operating systems (OS) have been a **core subject** in computer science since the field was established. One might argue that OS development is a **solved problem**, meaning that modern operating systems already provide efficient process management, robust file systems, and secure networking.  

However, **new challenges constantly emerge**, making OS research **an ongoing endeavor** rather than a closed subject.

---

### **Is the Operating System a Solved Problem?**  

✅ **What We Have Achieved So Far:**  
- Modern OSs **support multiple concurrent programs** that can either run independently or cooperate.  
- Users interact with a system **as if they have full control**, even though multiple processes run simultaneously.  
- **Bug isolation** ensures that application crashes **do not bring down the entire system**.  
- **File systems are highly optimized** for performance and storage efficiency.  
- **Fault tolerance mechanisms** allow systems to recover from hardware failures.  
- **Networking capabilities** enable reliable, high-speed data transfer over the internet.

⚠️ **But Challenges Still Exist:**  
- **Security attacks**: Systems are frequently attacked by hackers, and some attacks succeed.  
- **System crashes**: Even today, OSs sometimes crash due to application bugs or poor handling of errors.  
- **Denial-of-service (DoS) attacks**: An OS can be overwhelmed by excessive requests, making it **effectively unusable**.  
- **Data corruption**: Despite redundancy and error-checking, **data loss and corruption still happen**.

---

### **Modern Challenges in Operating Systems**  

1. **Isolation Between Programs (Process & Security Isolation)**  
   - OSs should **isolate processes** to prevent one program from affecting another.
   - This has been a **goal since the early days of OS design**, yet remains **imperfect**.
   - **Virtual machines (VMs) and containers** have improved isolation, but still face vulnerabilities.

2. **Security in the Age of Cyber Warfare**  
   - Traditional security focused on **preventing program bugs from crashing other programs**.
   - Early file systems ensured basic security, but **true security meant keeping computers offline**.
   - **Today’s OSs must defend against national intelligence agencies** and advanced cyberattacks.

3. **The Shift to Cloud Computing**  
   - More data and applications are moving to **the cloud**, but cloud systems still **rely on traditional OSs**.  
   - Cloud storage works similarly to file systems on personal computers, managed by conventional OSs.  
   - This means **OS research remains critical**, even though computing is shifting away from personal devices.

---

### **Conclusion: Why Operating Systems Still Matter**  
Although **many fundamental OS problems have been solved**, the field **continues to evolve** due to:  
✅ **New security threats** that demand better isolation and protection.  
✅ **The growing scale of computing** (cloud, IoT, distributed systems).  
✅ **The need for better system resilience** against crashes, failures, and attacks.  

**Operating systems remain a vital area of study, adapting to new challenges in computing security, scalability, and performance.**  

---

## **A Simple OS – Early Unix**  

This section explores the fundamental abstractions provided by a **relatively simple operating system** and how they are implemented. The operating system we discuss is **an early version of Unix from the early 1970s**.  

---

### **Why Study Early Unix?**  

Choosing an OS to analyze is **controversial**, but **early Unix** is a great candidate for several reasons:  
1. **Simplicity & Elegance** – The system was designed with a **clear, minimalistic philosophy**, making it easy to understand.  
2. **Major Influence** – Modern OSs (e.g., **Linux, MacOS, Windows**) were heavily inspired by **early Unix**.  
3. **Still in Use** – Unlike many early OSs that disappeared, **Unix and its derivatives continue to evolve**.  

Although **IBM’s OS/360** also has modern descendants, it is rarely accessible to students in operating systems courses, making Unix a **better learning tool**.

For clarity, we refer to this version as **Unix**, though its proper name is **Sixth-Edition Unix (Unix V6)**.  

---

### **What Makes Early Unix a Simple OS?**  

Early Unix introduced several key **operating system concepts** that became standard:  
✅ **Processes** – Each program runs in its own **isolated process**.  
✅ **File System** – A hierarchical file system for **storing and managing files**.  
✅ **System Calls** – A standardized way for programs to **interact with the OS**.  
✅ **Multitasking** – Simple **time-sharing** allowed multiple users to run programs concurrently.  

Although primitive by today’s standards, **Unix V6 laid the foundation for modern OS design**.  

---

### **What’s Next?**  

In the following sections, we will examine:  
🔹 **Unix OS Structure** – How the kernel is organized.  
🔹 **Processes & Memory Management** – How Unix handles running programs.  
🔹 **Files & System Calls** – How applications interact with the file system.  

By studying early Unix, we gain a **deeper understanding of modern operating system principles**.  

---

## **OS Structure**  

### **Early Operating System Design**  
Early operating systems were **incredibly small** due to **hardware limitations**. For example:  
- **Sixth-Edition Unix** had to fit into **64 KB of memory**.  
- This meant the entire OS was structured as **a single executable file**, loaded into memory when the computer **booted**.  
- This design is called a **monolithic operating system**.

### **How Does a Simple OS Work?**  
A simple OS interacts with applications and hardware using:  
1. **Traps** – Mechanism for applications to **request services from the OS** (e.g., file access, memory allocation).  
2. **Interrupts** – Signals from hardware (e.g., disks, clocks) that notify the OS **to handle events asynchronously**.

### **User Mode vs. Privileged Mode**  
Most modern CPUs support **at least two execution modes**:  
- **User Mode** 🟢 – **Application programs run here** with restricted privileges.  
- **Privileged Mode** 🔴 – **OS kernel runs here** to directly manage system resources.  

### **The Role of the Kernel**  
- The **kernel** is the part of the OS that runs in **privileged mode**.  
- In **simple OSs (e.g., Unix V6)**, **the entire OS runs in privileged mode**.  
- In **modern OSs (e.g., Windows, Linux)**, **some OS components run in user mode** to improve security and modularity.  

### **Key Takeaways**  
✅ **Early OSs were simple** – Everything ran as a single program in privileged mode.  
✅ **Modern OSs are more complex** – Some OS services now run in **user mode** to enhance security.  
✅ **Kernel is critical** – It manages **hardware access, memory, and processes**, ensuring the OS functions correctly.  

---

## **Traps in an Operating System**  

### **What Are Traps?**  
Traps are a **mechanism for switching from user mode to kernel mode**. They are **used to invoke the OS kernel** from a user program.  

Traps can be **intentional or unintentional**:  
1. **Unintentional Traps (Faults & Errors)** – Occur due to **programming errors**, such as:  
   - Accessing an **invalid memory address**.  
   - **Dividing by zero**.  
2. **Intentional Traps (System Calls)** – A controlled way for **user programs to request OS services** (explored in the next section).  

---

### **How the OS Handles Traps**  
When a trap occurs, the OS **must respond appropriately**:  
- **Page Faults** → If a program accesses a **missing memory page** (memory), the OS **fetches it from secondary storage** (disk).  
- **Programming Errors** → If a program causes an error:
  - If **no handler is set**, the OS **terminates the program**.  
  - If a **handler exists**, it can **attempt recovery or perform cleanup** before terminating.  

---

### **Traps and Signals in Unix**  
Unix handles errors using **signals**, which allow the OS to notify a user program when an event occurs.  
- The OS sends a **signal** to the program.  
- The program can **define a signal handler** to handle the error **gracefully**.  
- This mechanism is called an **upcall**, where the kernel **calls a function inside the user program**.  

---

### **Key Takeaways**  
✅ **Traps switch from user mode to kernel mode** for error handling or system calls.  
✅ **Page faults and programming errors trigger OS responses** to manage execution.  
✅ **Unix uses signals** (upcall) to handle errors **gracefully**, allowing user programs to recover or terminate safely.  

---

## **System Calls**  

### **What Are System Calls?**  
System calls are **the interface between user programs and the operating system kernel**.  
- Regular **function calls** in a program execute in **user mode**.  
- **System calls** allow user programs to request **privileged services** from the OS kernel.  

This ensures that the OS can **control access to system resources** and enforce security.

---

### **Why Use System Calls?**  
- The **kernel has full control over the system**, so direct access to hardware is restricted.  
- Instead, programs must request **services like file access, memory allocation, and process management** via system calls.  
- **System calls act as controlled entry points** where the OS can **validate requests** before executing them.

---

### **Example: The `write` System Call**  
A typical system call in Unix/Linux is `write()`, which sends data to a file.

#### **Example Code:**
```c
if (write(FileDescriptor, BufferAddress, BufferLength) == -1) {
    /* An error has occurred: handle it */
    printf("error: %d\n", errno);  /* Print error message */
}
```

#### **How It Works:**
1. The user program **calls `write()`** to request data to be written to a file.
2. The OS:
   - **Checks if the request is valid** (e.g., does the file exist? Do we have permission?).
   - **Performs the write operation**.
   - **Returns the number of bytes written** (or `-1` if there was an error).
   - If an error occurs, an error code is stored in **`errno`**.

---

### **How System Calls Work Internally**  
1. **User calls a system function** (e.g., `write()`).  
2. **A special instruction triggers a trap** (switching to kernel mode).  
3. **Control is transferred to the OS kernel**.  
4. **The OS verifies and executes the request**.  
5. **The result is returned to the user program**.  

This method ensures that the OS **regulates access to critical resources** while providing necessary services.

---

### **Key Takeaways**  
✅ **System calls allow user programs to interact with the OS kernel securely**.  
✅ **They provide controlled access to critical system resources** (files, memory, processes).  
✅ **A trap instruction transfers control from user mode to kernel mode**.  
✅ **The OS checks and processes the request before returning the result**.  

---

## **Interrupts in an Operating System**  

### **What is an Interrupt?**  
An **interrupt** is a **signal from an external device** that requests attention from the **processor**.  
- Interrupts occur **independently of the currently running program**.  
- They allow the OS to **handle asynchronous events**, such as:  
  - **Keyboard input** (a user presses a key).  
  - **Disk operations** (data is ready to be read/written).  
  - **Network activity** (a packet arrives).  

---

### **Interrupts vs. Traps**  
Interrupts and traps are both mechanisms that transfer control to the OS, but they differ in **origin and handling**:

| Feature       | **Interrupt** | **Trap** |
|--------------|-------------|----------|
| **Source**   | **External** (e.g., hardware devices) | **Internal** (e.g., errors, system calls) |
| **Trigger**  | Asynchronous (occurs anytime) | Synchronous (caused by program execution) |
| **Effect**   | Independent of running program | Directly affects running program |
| **Examples** | Disk I/O completion, Network packet arrival | Division by zero, Page fault, System call |

---

### **Example: Handling an Interrupt**  
#### **Scenario: Disk Read Completion**
1. **A program requests data from the disk**.  
2. **The CPU continues executing other instructions** (disk reads are slow).  
3. **When the disk read completes, the disk controller sends an interrupt**.  
4. **The CPU pauses execution, switches to kernel mode, and processes the interrupt**.  
5. **The OS notifies the requesting program that the data is ready**.  

---

### **Key Takeaways**  
✅ **Interrupts allow the OS to handle external events asynchronously**.  
✅ **They differ from traps**, which are caused by program execution.  
✅ **Interrupt handling ensures efficient use of the CPU**, preventing it from waiting idly for slow hardware operations.  

---

Got it! I will provide explanations in **both English and Chinese separately**, ensuring that you can **directly paste the content into your notes**. I will **include all example code in full** as well.

---

## **Processes, Address Spaces, and Threads**  

### **What is a Process?**  
A **process** is one of the most important abstractions in an operating system. From a programmer's perspective, a process consists of:  
1. **An address space** – Represents the memory the process can access.  
2. **Threads** – Execution units that run code within a process.  

The **address space** defines all memory addresses a program can generate and their associated storage. In modern OSs, **each process has a disjoint address space**, meaning processes cannot directly access each other's memory. This is enforced by **address-translation hardware** and OS-level security policies.

---

### **Address Space Layout in Unix**  
When a program runs, the OS **creates a process and loads the program into memory**. The process’s **address space** is structured as follows:

| **Memory Region** | **Description** |
|-------------------|----------------|
| **Text (Code Segment)** | Contains the executable instructions (read-only) |
| **Data Segment** | Stores global and static initialized variables |
| **BSS (Block Started by Symbol)** | Stores uninitialized global/static variables (automatically initialized to zero) |
| **Heap (Dynamic Memory)** | Used for dynamically allocated memory (`malloc()` in C, `new` in C++) |
| **Stack** | Stores local variables and function call information (grows upward) |

🔹 **Global variables** are stored in the **data/BSS segments** and exist throughout the process’s lifetime.  
🔹 **Local variables** are stored in the **stack** and only exist during function execution.  
🔹 **Dynamically allocated memory** is stored in the **heap** and must be manually managed.

#### **Memory Layout in Unix**
```plaintext
+----------------------+
|       text          |  <-- Executable code (low memory address)
+----------------------+
|       data          |  <-- Initialized global/static variables
+----------------------+
|       bss           |  <-- Uninitialized global/static variables
+----------------------+
|     heap (dynamic)  |  <-- Dynamically allocated memory (grows downward)
|        .            |
|        .            |
|        .            |
+----------------------+
|                    |  
|                    |  
|       stack        |  <-- Function call stack (grows upward)
+----------------------+
```

---

### **Example: Simple Process Execution**  
#### **Prime Number Calculation using the Sieve of Eratosthenes**  
```c
const int nprimes = 100;
int prime[nprimes];
int main() {
   int i;
   int current = 2;
   prime[0] = current;
   for (i = 1; i < nprimes; i++) {
      int j;
   NewCandidate:
      current++;
      for (j = 0; prime[j] * prime[j] <= current; j++) {
         if (current % prime[j] == 0)
            goto NewCandidate;
      }
      prime[i] = current;
   }
   return(0);
}
```
#### **What Happens in Memory?**  
- The OS **compiles and links the program**, storing it in the file system.  
- When executed, **a process is created**, and the program is loaded into memory.  
- **Memory allocations:**
  - **Global variables (`nprimes`, `prime[]`)** → Stored in the **data segment**.  
  - **Local variables (`i, j, current`)** → Stored in the **stack**.  

---

### **Dynamic Memory Allocation**  
Now, let's modify the program so that the number of primes is determined at runtime.

```c
int nprimes;
int *prime;
int main(int argc, char *argv[]) {
   int i;
   int current = 2;
   nprimes = atoi(argv[1]);
   prime = (int *)malloc(nprimes * sizeof(int));
   prime[0] = current;
   for (i = 1; i < nprimes; i++) {
      int j;
   NewCandidate:
      current++;
      for (j = 0; prime[j] * prime[j] <= current; j++) {
         if (current % prime[j] == 0)
            goto NewCandidate;
      }
      prime[i] = current;
   }
   return(0);
}
```

#### **What Changes in Memory?**  
- `malloc()` allocates memory in the **heap (dynamic memory)** instead of using a fixed-size array.
- The heap grows **dynamically** using the system call `sbrk()`.

---

#### **Key Takeaways**  
✅ **Processes consist of an address space and one or more threads.**  
✅ **Each process has a structured address space (text, data, BSS, heap, stack).**  
✅ **Heap memory is allocated dynamically and must be managed manually.**  
✅ **Stack memory is used for function calls and local variables.**  

---

## **Managing Processes**  

### **Creating a Process: The `fork()` System Call**  
In Unix, **the only way to create a process** is by using the `fork()` system call.  
- `fork()` **creates a new process**, which is a **copy of the calling process**.  
- The new process (child) and the original process (parent) **execute separately but share the same code initially**.  

#### **How `fork()` Works**  
- The child process gets **its own address space**, which contains:  
  - A copy of the **text (code), BSS, data, dynamic region, and stack** of the parent.  
  - **Shared text section** (since code is read-only, it doesn’t need to be copied).  
- `fork()` **returns twice**:  
  - **In the child process, it returns `0`**.  
  - **In the parent process, it returns the child’s process ID (PID)**.  

#### **Example: Using `fork()`**
```c
if (fork() == 0) {
   /* This is executed by the child process */
} else {
   /* This is executed by the parent process */
}
```
- This allows the parent and child **to execute different logic**.

---

### **Process Control Blocks (PCB)**  
Every process has a **process control block (PCB)**, a **data structure maintained by the OS**.  
- **Contains:**  
  - **Process ID (PID)**  
  - **Parent’s PID**  
  - **Process state** (running, sleeping, zombie, etc.)  
  - **Return code** after process termination  

📌 **Figure 1.4 (Conceptual Representation of PCBs)**  
```
+------------+      +------------+      +------------+
|  Parent    | ---> |  Child 1   | ---> |  Child 2   |
|  Process   |      |  (Zombie)  |      |  (Zombie)  |
+------------+      +------------+      +------------+
```

---

### **Process Termination: The `exit()` System Call**  
A process **terminates** by calling `exit()`, passing a **return code**:  
```c
exit(n);
```
- `n = 0` means **successful execution**.  
- `n > 0` usually **indicates an error**.  
- If a program **returns from `main()`**, it automatically calls `exit()`.  

---

### **Waiting for a Process: The `wait()` System Call**  
A parent process can **wait for a child process to terminate** using `wait()`:
```c
short pid;
if ((pid = fork()) == 0) {
   /* Child process executes some code */
   exit(n);
} else {
   int ReturnCode;
   while (pid != wait(&ReturnCode));
   /* Parent receives child's return code */
}
```
🔹 **Key Points:**  
✅ `wait()` ensures the parent **receives the child’s return code**.  
✅ `wait()` returns **the PID of the terminated child**.  
✅ A process **can only wait for its own children**, not arbitrary processes.

---

### **Zombie Processes & Reclaiming PIDs**  
- When a process calls `exit()`, it **does not immediately disappear**.  
- It enters a **zombie state**, where:  
  - Its **address space is freed**, but  
  - **Its PID and return code are stored in the PCB**.  
- The parent must call `wait()` to **remove the zombie process and reclaim resources**.  

**What if the parent dies first?**  
- The orphaned child process is **adopted by process ID 1 (`init`)**, which periodically calls `wait()` to clean up zombie processes.

---

### **Key Takeaways**  
✅ `fork()` creates a new process, which starts execution from the same code.  
✅ `fork()` returns `0` in the child and **PID of the child in the parent**.  
✅ `exit()` terminates a process and returns a status code.  
✅ `wait()` ensures a parent retrieves the return code of its child.  
✅ **Zombie processes** store return codes until the parent collects them.  

---

## **Loading Programs into Processes (`exec()`)**  

### **Replacing a Process's Address Space**  
We previously discussed how an OS **sets up a process's address space**. Now, we examine how **a process can load a new program into its address space**.  

🔹 The **`exec`** system call **replaces the current process's memory with a new program**.  
🔹 **Common use case**:  
   - A **parent process** creates a **child process** using `fork()`.  
   - The **child process replaces its memory with a new program** using `exec()`.  
   - The **parent waits** for the child process to finish execution.

---

### **How `exec()` Works**  
`exec()` **replaces everything in the process's memory**:  
✅ **Text section** → Replaced with the new program's executable code.  
✅ **Data & BSS sections** → Replaced with the new program’s variables.  
✅ **Heap (dynamic memory)** → Cleared.  
✅ **Stack** → Replaced with new command-line arguments.  
✅ **Execution starts from `main()` in the new program**.  

📌 **Key point**:  
- **`exec()` does not return** if successful because the old program **no longer exists** in memory.  
- If `exec()` **fails**, it returns an error code.

---

### **Example: Using `execl()`**  
```c
int pid;
if ((pid = fork()) == 0) {
     /* Child process replaces itself with the new program */
     execl("/home/twd/bin/primes", "primes", "300", 0);
     exit(1);  // If exec fails, exit with an error
}

/* Parent process continues here */
while(pid != wait(0)) /* Ignore the return code */
   ;
```

#### **What Happens?**  
1. **The parent process calls `fork()`**, creating a child process.  
2. **The child process replaces itself** with the `primes` program using `execl()`.  
3. **The parent process waits** for the child process to terminate.  

📌 **Command Equivalent in a Shell:**  
```
% primes 300
```
This works exactly like a **shell command execution**:  
- The shell **creates a process** (`fork()`),  
- **Loads the command** (`exec()`),  
- **Waits for it to finish** (`wait()`).  

---

### **Key Takeaways**  
✅ **`exec()` replaces the current process’s memory with a new program**.  
✅ **Successful `exec()` calls never return** (since the original process is erased).  
✅ **If `exec()` fails, the process must handle the error (e.g., call `exit(1)`)**.  
✅ **Shells use `fork()` + `exec()` to execute user commands**.  

---

## **Files in Unix**  

### **Why Do We Need Files?**  
In our previous examples, the **primes program only runs in memory**, meaning its results disappear when the process terminates.  

🔹 **Files solve this issue** by providing **persistent storage** for data that:  
1. **Survives after the process exits**.  
2. **Can be accessed by other programs or users**.  

---

### **Unix Files: A Universal Abstraction**  
Unix treats **everything as a file**, including:  
- **Regular files** (text files, binary files, logs, etc.).  
- **Devices** (keyboards, displays, disks).  
- **Inter-process communication (IPC)** (pipes, sockets).  

📌 **Key Idea**:  
- Files allow programs to **store, retrieve, and share data** in a structured way.  
- A process can **read/write to a file**, just like it interacts with other processes or hardware.  

---

### **How Unix Uses Files**
Unix files serve two main purposes:  
1. **Persistent Storage** – Storing data on disk (text, logs, databases).  
2. **Interfacing with External Data Sources** – Communicating with other processes or devices.

Example:  
- A program **writes data to a file** → another program **reads it later**.  
- A process **reads input from a keyboard** → another process **writes output to the display**.  

---

### **Key Takeaways**  
✅ **Files provide a way to store data persistently outside a process**.  
✅ **Unix treats everything as a file (including devices and IPC mechanisms)**.  
✅ **Programs use files to read, write, and share data efficiently**.  

---

## **Naming Files**  

### **How Do We Refer to Files?**  
Since **files exist outside a process's address space**, we need a **naming system** to locate them.  
- **Most operating systems use a hierarchical (tree-structured) directory system**.  
- **File paths represent a sequence of directories leading to the file**.  

📌 **Path Naming Conventions:**  
| **OS** | **Path Separator** | **Example** |
|--------|------------------|-------------|
| **Unix/Linux** | `/` (Forward Slash) | `/home/user/file.txt` |
| **Windows** | `\` (Backslash) | `C:\Users\User\file.txt` |

---

### **How Does the OS Access Files?**  
1. **A process requests a file by its path (`/home/twd/file`)**.  
2. **The OS verifies the file’s existence and checks access permissions**.  
3. **If access is allowed, the OS returns a file handle (file descriptor in Unix)**.  
4. **Subsequent file operations (read/write) use the file descriptor instead of the path**.  

📌 **Why Use File Handles?**  
- **Efficiency**: Searching the directory tree every time would be slow.  
- **Security**: The OS enforces access permissions.  
- **Convenience**: Once a file is opened, multiple reads/writes can be performed without rechecking its path.

---

### **Example: Opening & Reading a File in Unix**
```c
int fd;
char buffer[1024];
int count;

if ((fd = open("/home/twd/file", O_RDWR)) == -1) {
   /* The file couldn't be opened */
   perror("/home/twd/file");
   exit(1);
}

if ((count = read(fd, buffer, 1024)) == -1) {
   /* The read failed */
   perror("read");
   exit(1);
}
/* buffer now contains count bytes read from the file */
```
📌 **Explanation:**  
- `open()` attempts to open `/home/twd/file` with **read/write (`O_RDWR`) permissions**.  
- If `open()` fails, `fd == -1`, and `perror()` prints an error message.  
- `read()` reads **up to 1024 bytes** from the file into `buffer`.  
- If `read()` fails, `perror("read")` prints the error message.  

---

### **Key Takeaways**  
✅ **Files are identified by hierarchical paths**.  
✅ **The OS provides file handles (file descriptors) for efficient access**.  
✅ **Once opened, file operations (read/write) use the handle instead of the file path**.  

---

## **Using File Descriptors in Unix**  

### **What Are File Descriptors?**  
A **file descriptor** is a **small integer** that the OS assigns when a process opens a file.  
- File descriptors allow a process to access files **like an extension of its address space**.  
- File descriptors **persist after an `exec`**, meaning open files remain open after a new program is loaded.  

📌 **Standard File Descriptors in Unix**  
| **File Descriptor** | **Purpose** | **Default Association** |
|---------------------|------------|------------------------|
| `0` | **Standard Input** | Keyboard (`stdin`) |
| `1` | **Standard Output** | Display (`stdout`) |
| `2` | **Standard Error** | Display (`stderr`) |

🔹 **Redirecting output**: A program can **change which file is associated** with these descriptors.

---

### **Example: Redirecting Output to a File Before `exec`**
```c
if (fork() == 0) { // Child process
   close(1); // Close standard output
   if (open("/home/twd/Output", O_WRONLY) == -1) {
      perror("/home/twd/Output");
      exit(1);
   }
   execl("/home/twd/bin/primes", "primes", "300", 0);
   exit(1);
}

/* Parent process continues here */
while (pid != wait(0)); // Wait for child process to finish
```
📌 **Explanation:**  
1. **The child process closes file descriptor `1` (stdout)**.  
2. **A file (`/home/twd/Output`) is opened**, and since file descriptors are allocated lowest first, it becomes **descriptor `1`**.  
3. **After `exec()`, `primes` writes to file descriptor `1`**, which now points to `/home/twd/Output`.  

---

### **Example: Writing Primes to a File in Binary**
```c
int nprimes;
int *prime;
int main(int argc, char *argv[]) {
   int i;
   int current = 2;
   nprimes = atoi(argv[1]);
   prime = (int *)malloc(nprimes * sizeof(int));
   prime[0] = current;

   for (i = 1; i < nprimes; i++) {
      int j;
   NewCandidate:
      current++;
      for (j = 0; prime[j] * prime[j] <= current; j++) {
         if (current % prime[j] == 0)
            goto NewCandidate;
      }
      prime[i] = current;
   }
   if (write(1, prime, nprimes * sizeof(int)) == -1) {
     perror("primes output");
     exit(1);
   }
   return(0);
}
```
📌 **Key Points:**  
✅ **Writes binary data to file descriptor `1`**.  
✅ **If redirected, the output file stores raw numbers instead of readable text**.  

---

### **Example: Writing Primes as Human-Readable Text**
```c
int nprimes;
int *prime;
int main(int argc, char *argv[]) {
   int i;
   int current = 2;
   nprimes = atoi(argv[1]);
   prime = (int *)malloc(nprimes * sizeof(int));
   prime[0] = current;

   for (i = 1; i < nprimes; i++) {
      int j;
   NewCandidate:
      current++;
      for (j = 0; prime[j] * prime[j] <= current; j++) {
         if (current % prime[j] == 0)
            goto NewCandidate;
      }
      prime[i] = current;
   }
   for (i = 0; i < nprimes; i++) {
      printf("%d\n", prime[i]);
   }
   return(0);
}
```
📌 **How It Works:**  
✅ **Uses `printf("%d\n", prime[i])` to format numbers as readable text**.  
✅ **If redirected (`primes 300 > /home/twd/Output`), output is saved as text**.  

---

### **File Descriptor Redirection & Duplication**
📌 **Redirecting Normal Output (`1`) and Errors (`2`) to the Same File**
```c
if (fork() == 0) {
   close(1);
   close(2);
   if (open("/home/twd/Output", O_WRONLY) == -1) {
      exit(1);
   }
   dup(1); // Duplicate file descriptor 1 to 2
   execl("/home/twd/bin/program", "program", 0);
   exit(1);
}
```
📌 **Alternative in a Shell (`csh` syntax):**
```
% program >& /home/twd/Output
```

---

### **Handling File Descriptors in `fork()`**
```c
int logfile = open("log", O_WRONLY);
if (fork() == 0) {
   write(logfile, LogEntry, strlen(LogEntry)); // Child writes to log
   exit(0);
}
write(logfile, LogEntry, strlen(LogEntry)); // Parent writes to log
```
📌 **Key Insight:**  
✅ **The file context (cursor position) is shared between parent & child**.  
✅ **Ensures both processes append to the log instead of overwriting it**.  

---

### **Key Takeaways**  
✅ **File descriptors extend a process's address space**.  
✅ **File descriptors persist across `exec()` calls**.  
✅ **Redirection (`close() + open()`) allows programs to modify stdin/stdout**.  
✅ **Use `dup()` to share the same write context across multiple descriptors**.  

---

## **Random Access in Files**  

### **Sequential vs. Random Access**  
- By default, **file reading and writing in Unix is sequential**.  
- **Successive `read()` and `write()` operations progress through the file automatically**.  
- **Random access** allows reading/writing **at arbitrary positions** instead of sequentially.  

📌 **Key Mechanism:**  
- **Each open file has a "file-location" field** that tracks the **current position**.  
- **`lseek()` updates this position** to allow random access.

---

### **Using `lseek()` for Random File Access**
```c
fd = open("textfile", O_RDONLY);
/* Go to the last character in the file */
fptr = lseek(fd, (off_t)-1, SEEK_END);
while (fptr != -1) {
   read(fd, buf, 1);
   write(1, buf, 1);
   fptr = lseek(fd, (off_t)-2, SEEK_CUR);
}
```
📌 **Explanation:**  
1. **`open("textfile", O_RDONLY)`** → Opens the file in read-only mode.  
2. **`lseek(fd, -1, SEEK_END)`** → Moves file position to **the last character**.  
3. **Read & write 1 byte** → Displays it on the screen.  
4. **Move file position backward (`-2` bytes) using `lseek()`**.  
5. **Repeat until reaching the start of the file**.  

---

### **How `lseek()` Works?**  
```c
off_t lseek(int fd, off_t offset, int whence);
```
| **Parameter** | **Description** |
|--------------|----------------|
| `fd` | File descriptor of the opened file |
| `offset` | Number of bytes to move (can be positive or negative) |
| `whence` | Reference point for movement (`SEEK_SET`, `SEEK_CUR`, `SEEK_END`) |

📌 **`whence` Options**  
| **Mode** | **Meaning** |
|---------|------------|
| `SEEK_SET` | Offset measured **from the beginning** of the file |
| `SEEK_CUR` | Offset measured **from the current position** |
| `SEEK_END` | Offset measured **from the end** of the file |

✅ **`lseek()` does not read/write data**, it **only updates the file location**.

---

### **Key Takeaways**  
✅ **By default, file access is sequential**.  
✅ **`lseek()` allows positioning the read/write cursor anywhere in the file**.  
✅ **Random access is useful for reverse reading, modifying specific sections, etc.**  

---

## **Pipes in Unix**  

### **What Is a Pipe?**  
- A **pipe** allows **two processes to communicate** like a file.  
- **The sender writes to the pipe**, just like writing to a file.  
- **The receiver reads from the pipe**, just like reading from a file.  

📌 **Key Properties of a Pipe**:  
✅ **Unidirectional** → One process writes, another reads.  
✅ **Exists only in memory** → Not stored as a named file.  
✅ **Shared only by related processes** → The pipe **must be created before `fork()`**, so the child inherits it.  

---

### **Creating a Pipe: The `pipe()` System Call**
```c
int p[2];   /* Array to hold pipe's file descriptors */
pipe(p);    /* Create the pipe; assume no errors */
```
📌 **How It Works?**  
- **`p[0]` → Read end of the pipe**.  
- **`p[1]` → Write end of the pipe**.  

After calling `pipe(p)`, the kernel creates a **pipe object** and returns two file descriptors:  
| **Descriptor** | **Usage** |
|--------------|---------|
| `p[0]` | Read from the pipe |
| `p[1]` | Write to the pipe |

---

### **Example: Parent-to-Child Communication**
```c
int p[2];    /* Pipe file descriptors */
pipe(p);     /* Create a pipe */
if (fork() == 0) {   // Child process
   char buf[80];
   close(p[1]);  /* Close write end (not needed by child) */
   while (read(p[0], buf, 80) > 0) {
      /* Process data from parent */
      ...
   }
} else {  // Parent process
   char buf[80];
   close(p[0]);  /* Close read end (not needed by parent) */
   for (;;) {
      /* Prepare data for child */
      ...
      write(p[1], buf, 80);
   }
}
```
📌 **Explanation:**  
1. **Parent creates a pipe before `fork()`** → Both processes inherit it.  
2. **Child closes `p[1]`** (it only reads from the pipe).  
3. **Parent closes `p[0]`** (it only writes to the pipe).  
4. **Parent writes data to `p[1]`**, and **child reads from `p[0]`**.  

✅ **This allows parent and child to communicate!**  

---

### **How Pipes Work in the Shell**
A pipe (`|`) in the shell works the same way:
```
ls | grep "txt"
```
📌 **How It Works Internally?**  
- The shell **creates a pipe**.  
- `ls` **writes** to the pipe (`p[1]`).  
- `grep "txt"` **reads** from the pipe (`p[0]`).  

---

### **Key Takeaways**  
✅ **Pipes enable inter-process communication (IPC)**.  
✅ **The pipe must be created before `fork()` so that both processes can use it**.  
✅ **Only processes with inherited file descriptors can access the pipe**.  
✅ **A pipe is unidirectional: one side writes, the other reads**.  

---

## **Directories in Unix**  

### **What Is a Directory?**  
A **directory** is a **special type of file** that contains **references (links) to other files**.  
- **It stores file names and their corresponding inode numbers**.  
- **Each inode uniquely identifies a file** in the file system.  

📌 **Key Features of a Directory**  
✅ **Contains file references (name + inode number)**.  
✅ **Supports hierarchical organization of files**.  
✅ **Includes two special entries: `.` (current directory) and `..` (parent directory)**.  

---

### **Understanding Inodes & Hard Links**
- **An inode is a data structure that represents a file**.  
- **Hard links** allow multiple directory entries to reference the **same inode**.  
- **A file is deleted only when all links to it are removed**.

📌 **Example: Hard Link Behavior**
```
$ ln /unix /etc/image
```
Now both `/unix` and `/etc/image` refer to **the same file** (same inode).  
- If `/unix` is deleted (`rm /unix`), the file **still exists** because `/etc/image` is another reference.  
- The file is **only removed when all links are deleted**.  

✅ **The kernel tracks how many directory entries reference a file (link count).**  
✅ **A file is deleted only when both the link count and reference count (open file count) reach zero.**  

---

### **Symbolic (Soft) Links**
A **symbolic link** is a **special file that contains a path to another file**.  
- Unlike hard links, **a symbolic link is a separate file**.  
- The system follows the path stored in the symbolic link to find the actual file.  

📌 **Example: Creating a Symbolic Link**
```
$ ln -s /unix /home/twd/mylink
```
Now, `/home/twd/mylink` **points to** `/unix`.  
- If `/unix` is **deleted**, `/home/twd/mylink` **becomes broken** (it still exists, but points to nothing).  

✅ **Symbolic links allow linking across different file systems.**  
✅ **They can reference directories, unlike hard links.**  

---

### **Working Directory & Path Resolution**
- Every **process has a working directory** (where relative paths start from).  
- **Changing the working directory**:
  ```
  $ cd /home/user
  ```
  - Now, a file called `file.txt` is accessed as `./file.txt` (instead of `/home/user/file.txt`).  

📌 **Finding the Current Working Directory**
```
$ pwd
```
- The `pwd` command retrieves **the absolute path** of the working directory.  

---

### **Key Takeaways**
✅ **A directory stores file references (names + inode numbers).**  
✅ **Hard links reference the same inode; files exist until all links are removed.**  
✅ **Symbolic links store paths and can link across file systems.**  
✅ **Each process has a working directory, used for relative paths.**  

---

## **Access Protection in Unix**  

### **Why Is Access Control Needed?**  
An operating system must **prevent unauthorized access** to files and resources.  
- **Each file has access permissions that define who can read, write, or execute it**.  
- **Permissions are associated with security principals**, which can be:  
  1. **A user (file owner)**.  
  2. **A group of users**.  
  3. **Everyone else (other users on the system)**.  

---

### **Understanding File Permissions**
Each file has **three sets of permissions**:  
1. **User (Owner) Permissions** → Applies to the file's owner.  
2. **Group Permissions** → Applies to users in the same group as the owner.  
3. **Others Permissions** → Applies to all other users.  

📌 **Permission Types**  
| **Symbol** | **Meaning for Files** | **Meaning for Directories** |
|----------|----------------------|----------------------------|
| `r` (Read) | Can read the file | Can list directory contents |
| `w` (Write) | Can modify the file | Can create/delete files in directory |
| `x` (Execute) | Can run the file as a program | Can traverse the directory |

📌 **Example: Listing File Permissions**  
```
$ ls -lR
.:
total 2
drwxr-x--x  2 tom    adm     1024 Dec 17 13:34 A
drwxr-----  2 tom    adm     1024 Dec 17 13:34 B

./A:
total 1
-rw-rw-rw-  1 tom    adm      593 Dec 17 13:34 x

./B:
total 2
-r--rw-rw-  1 tom    adm      446 Dec 17 13:34 x
-rw----rw-  1 trina  adm      446 Dec 17 13:45 y
```
📌 **How to Read This?**  
```
drwxr-x--x  2 tom  adm   1024 Dec 17 13:34 A
```
- `d` → **Directory**.  
- `rwx` → **Owner (`tom`) can read, write, and execute**.  
- `r-x` → **Group (`adm`) can read and execute**.  
- `--x` → **Others can only execute (traverse the directory, but not list its contents)**.  

---

### **Checking Access for Different Users**
#### **Scenario: Can Andy List Directory A?**
❌ **No**  
- Andy belongs to "others" (`--x`).  
- He can **traverse** the directory (`x`), but **not list its contents** (missing `r`).  

#### **Scenario: Can Andy Read `A/x`?**
✅ **Yes**  
- Andy can **follow the path through A** (has `x` permission).  
- The file `x` has **world-readable (`rw-rw-rw-`) permissions**.  

#### **Scenario: Can Trina List Directory B?**
✅ **Yes**  
- Trina belongs to the `adm` group, and B allows **group read (`r-----`)**.  

#### **Scenario: Can Trina Modify `B/y`?**
❌ **No**  
- **B restricts `x` permission for the group**, so Trina **cannot traverse it** to reach `y`.  

#### **Scenario: Can Tom Modify `B/x`?**
❌ **No**  
- Tom is the **owner of `B/x`**, but only has **read (`r--`) permission**.  

#### **Scenario: Can Tom Read `B/y`?**
❌ **No**  
- Tom is in the **`adm` group**, and the group has **no permissions (`------`)** on `y`.  

---

### **Modifying File Permissions**
📌 **Using `chmod` to Change Permissions**
```
$ chmod 755 filename
```
- `755` sets permissions as **rwxr-xr-x**:  
  - **Owner: read, write, execute**.  
  - **Group: read, execute**.  
  - **Others: read, execute**.  

📌 **Using `chown` to Change File Owner**
```
$ chown tom:adm filename
```
- Makes `tom` the owner and `adm` the group.  

📌 **Using `umask` to Set Default Permissions**
```
$ umask 022
```
- New files **default to `rw-r--r--`** (only owner can write).  

---

### **Key Takeaways**
✅ **File permissions control who can read, write, or execute a file**.  
✅ **Permissions apply to owner, group, and others**.  
✅ **Directories require execute (`x`) permission for traversal**.  
✅ **Use `chmod`, `chown`, and `umask` to manage permissions**.  

---

## **Creating Files in Unix**  

### **How to Create a File?**  
Unix provides two ways to create a file:  
1. **`creat()` system call** (legacy method).  
2. **`open()` system call with `O_CREAT` flag** (modern method).  

---

### **Using `creat()` to Create a File**  
```c
int fd = creat("newfile.txt", 0666);
```
📌 **Explanation:**  
- Creates a file named `newfile.txt` **if it does not already exist**.  
- `0666` → Sets the default permissions (**read & write for owner, group, others**).  
- If the file **already exists, `creat()` truncates (empties) it**.  

❗ **Problem with `creat()`**:  
- It **does not allow opening an existing file without truncation**.  
- Modern Unix systems prefer using `open()` instead.  

---

### **Using `open()` to Create a File**
```c
int fd = open("newfile.txt", O_CREAT | O_EXCL, 0666);
```
📌 **Explanation:**  
- **`O_CREAT`** → Creates the file **if it does not exist**.  
- **`O_EXCL`** → Ensures **the file is not overwritten if it already exists**.  
- If `O_EXCL` is not set and the file exists, `open()` just **opens** it.  

📌 **Truncating (Emptying) a File**  
```c
int fd = open("existingfile.txt", O_CREAT | O_TRUNC, 0666);
```
- **If the file exists, `O_TRUNC` erases its contents**.  

---

### **Understanding `umask`: Controlling Default File Permissions**
When a file is created, **its permissions are set based on two factors**:  
1. **Mode argument in `open()` or `creat()` (e.g., `0666`)**.  
2. **The process's `umask` (a system setting that removes permissions)**.  

📌 **How `umask` Works?**  
The actual file permissions are:  
```c
final_permissions = mode & ~umask;
```
✅ **Example**:  
- You **create a file with `0666` (read & write for everyone)**.  
- Your **`umask` is `0022` (removes write permission for group & others)**.  
- **Final permissions**: `0666 & ~0022 = 0644` (read & write for owner, read-only for others).  

📌 **Setting `umask` in a Shell**
```sh
$ umask 0027
```
- Ensures new files **cannot be accessed by others**.  

📌 **Setting `umask` in a C Program**
```c
umask(0027);
```

---

### **Key Takeaways**
✅ **`creat()` creates a new file but always truncates existing ones**.  
✅ **`open()` with `O_CREAT` is preferred for file creation**.  
✅ **`O_EXCL` prevents overwriting an existing file**.  
✅ **`umask` determines which permissions are removed from new files**.  

---

