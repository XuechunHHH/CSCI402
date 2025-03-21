# **Chapter3: Basic Concepts**

## **Procedures and Stack Frames in x86 Architecture**

### **1. Introduction**
This section explains how **procedure calls and stack frames** work in the **Intel x86 architecture**. When a function is called in C, a **stack frame** is created to store:
- **Function arguments**.
- **Return address** (instruction pointer).
- **Saved registers**.
- **Local variables**.

Understanding stack frames is essential for grasping **function calls, recursion, and system calls**.

---

## **2. Procedure Calls in C**
### **2.1 Example C Code**
Consider the following C program:

```c
int main() {
    int i;
    int a;
    
    // Some operations...

    i = sub(a, 1);  // Call subroutine 'sub'
    
    return 0;
}

int sub(int x, int y) {
    int i;
    int result = 1;
    for (i = 0; i < y; i++)
        result *= x;
    return result;
}
```
#### **Purpose**
- `sub(x, y)` computes **x^y** (x raised to the power of y).
- The function `sub` has **local variables** `i` and `result`, along with arguments `x` and `y`.
- When `sub` is called, a new **stack frame** is created.

---

## **3. Stack Frames in Intel x86**
### **3.1 Stack Organization**
In **Intel x86 architecture**, the stack is structured as follows:
1. **Arguments of the function**.
2. **Return address** (instruction pointer of the caller).
3. **Saved frame pointer (`ebp`)**.
4. **Local variables** and **saved registers**.

**Key Registers:**
- **`ebp` (Base Pointer)**: Points to the base of the current stack frame.
- **`esp` (Stack Pointer)**: Points to the top of the stack.
- **`eax` (Accumulator Register)**: Stores function return values.

### **3.2 Stack Frame Layout**
When `sub` is called:
```
[ Higher Memory ]
-------------------------
| Caller’s Stack Frame |
-------------------------
|  Return Address (EIP) | <- Stored when calling sub()
|  Old EBP              | <- Previous stack frame pointer
|  Local Variables      | <- Local variables of sub()
|  Saved Registers      | <- Preserved registers (if modified)
-------------------------
[ Lower Memory ]
```
---

## **4. Assembly Code for Procedure Calls**
### **4.1 Main Function Assembly Code**
```assembly
main:
    pushl  %ebp            # Save old base pointer
    movl   %esp, %ebp      # Set new base pointer
    subl   $8, %esp        # Allocate space for local variables

    pushl  $1              # Push argument y onto stack
    movl   -4(%ebp), %eax  # Load argument x from local variable
    pushl  %eax            # Push x onto stack
    call   sub             # Call subroutine 'sub'
    addl   $8, %esp        # Clean up stack (remove arguments)
    movl   %eax, -8(%ebp)  # Store return value in local variable

    movl   $0, %eax        # Set return value to 0
    popl   %ebp            # Restore base pointer
    ret                    # Return to caller
```

### **4.2 Subroutine Assembly Code**
```assembly
sub:
    pushl  %ebp            # Save caller's base pointer
    movl   %esp, %ebp      # Set new base pointer
    subl   $8, %esp        # Allocate space for local variables

    movl   $1, -4(%ebp)    # Initialize result = 1
    movl   $0, -8(%ebp)    # Initialize i = 0
beginloop:
    cmpl   12(%ebp), %eax  # Compare i with y
    jge    endloop         # If i >= y, exit loop
    imull  8(%ebp), %ecx   # result *= x
    addl   $1, %eax        # i++
    jmp    beginloop       # Loop again
endloop:
    movl   %ecx, -4(%ebp)  # Store result in stack
    movl   -4(%ebp), %eax  # Load result into return register
    movl   %ebp, %esp      # Restore stack pointer
    popl   %ebp            # Restore base pointer
    ret                    # Return to caller
```

### **4.3 Explanation**
1. **Prologue (`sub` function)**
   - Save the previous **stack frame** (`ebp`).
   - Allocate space for **local variables**.

2. **Function Execution**
   - Loop computes `result = x^y`.
   - Uses `imull` for multiplication.

3. **Epilogue**
   - Restore **stack pointer (`esp`)**.
   - Return using `ret` (pops return address into `eip`).

---

## **5. Summary**
- **Stack frames** manage function calls in x86.
- **Registers (`ebp`, `esp`, `eax`)** help organize stack memory.
- **Assembly language** shows function call mechanics.
- **Efficient stack usage** prevents corruption in recursive calls.

---

