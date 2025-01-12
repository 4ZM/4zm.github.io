---
date: 2024-12-25
slug: a-simple-elf
categories:
  - Toolchains
---

# A Simple ELF

Let's write a simple program for Linux. How hard can it be? Well, simple is the opposite of complex, not of hard, and it is surprisingly hard to create something simple. What is left when we get rid of the complexity from the standard library, all the modern security features, debugging information, and error handling mechanisms?

<!-- more -->

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

Let's start with something complex:

```
#include <stdio.h>

int main() {
    printf("Hello Simplicity!\n");
}
```

Wait, what?! It doesn't look very complex, does it... Hmm, let's compile it and take a look:

```
$ gcc -o hello hello.c
$ ./hello
Hello Simplicity!
```

Still looks pretty simple, right? Wrong! While this might be familiar territory and *easy* to comprehend, the program is far from *simple*. Let's take a look behind the curtain.

```
$ objdump -t hello

hello:     file format elf64-x86-64

SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000              Scrt1.o
000000000000038c l     O .note.ABI-tag	0000000000000020              __abi_tag
0000000000000000 l    df *ABS*	0000000000000000              crtstuff.c
0000000000001090 l     F .text	0000000000000000              deregister_tm_clones
00000000000010c0 l     F .text	0000000000000000              register_tm_clones
0000000000001100 l     F .text	0000000000000000              __do_global_dtors_aux
0000000000004010 l     O .bss	0000000000000001              completed.0
0000000000003dc0 l     O .fini_array	0000000000000000              __do_global_dtors_aux_fini_array_entry
0000000000001140 l     F .text	0000000000000000              frame_dummy
0000000000003db8 l     O .init_array	0000000000000000              __frame_dummy_init_array_entry
0000000000000000 l    df *ABS*	0000000000000000              hello.c
0000000000000000 l    df *ABS*	0000000000000000              crtstuff.c
00000000000020f8 l     O .eh_frame	0000000000000000              __FRAME_END__
0000000000000000 l    df *ABS*	0000000000000000
0000000000003dc8 l     O .dynamic	0000000000000000              _DYNAMIC
0000000000002018 l       .eh_frame_hdr	0000000000000000              __GNU_EH_FRAME_HDR
0000000000003fb8 l     O .got	0000000000000000              _GLOBAL_OFFSET_TABLE_
0000000000000000       F *UND*	0000000000000000              __libc_start_main@GLIBC_2.34
0000000000000000  w      *UND*	0000000000000000              _ITM_deregisterTMCloneTable
0000000000004000  w      .data	0000000000000000              data_start
0000000000000000       F *UND*	0000000000000000              puts@GLIBC_2.2.5
0000000000004010 g       .data	0000000000000000              _edata
0000000000001168 g     F .fini	0000000000000000              .hidden _fini
0000000000004000 g       .data	0000000000000000              __data_start
0000000000000000  w      *UND*	0000000000000000              __gmon_start__
0000000000004008 g     O .data	0000000000000000              .hidden __dso_handle
0000000000002000 g     O .rodata	0000000000000004              _IO_stdin_used
0000000000004018 g       .bss	0000000000000000              _end
0000000000001060 g     F .text	0000000000000026              _start
0000000000004010 g       .bss	0000000000000000              __bss_start
0000000000001149 g     F .text	000000000000001e              main
0000000000004010 g     O .data	0000000000000000              .hidden __TMC_END__
0000000000000000  w      *UND*	0000000000000000              _ITM_registerTMCloneTable
0000000000000000  w    F *UND*	0000000000000000              __cxa_finalize@GLIBC_2.2.5
0000000000001000 g     F .init	0000000000000000              .hidden _init
```

That's a lot of symbols! Actually, as far as symbol tables go, this one is quite modest. Any non-trivial program will have many more symbols, but still, what are they all for? We're just printing a string!

We recognize our `main` function in the `.text` segment at address `0x1149`. But where is the `printf` function?

It turns out that for simple cases, where there is no formatting work required by `printf`, GCC optimizes the code and replaces it with the simpler `puts@GLIBC_2.2.5` from libc. The address is all zeros since the symbol is undefined (`*UND*`). It will be resolved when the program is loaded together with the dynamic libc.so library as we run it.

```
0000000000001149 g     F .text	000000000000001e              main
0000000000000000       F *UND*	0000000000000000              puts@GLIBC_2.2.5
```

Let's keep digging. What sections are there in the program? The only data we have is the hardcoded string and its length. Surely we only need a `.text` section? Let's see what we got:

```
$ objdump -h hello

hello:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .interp       0000001c  0000000000000318  0000000000000318  00000318  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .note.gnu.property 00000030  0000000000000338  0000000000000338  00000338  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .note.gnu.build-id 00000024  0000000000000368  0000000000000368  00000368  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .note.ABI-tag 00000020  000000000000038c  000000000000038c  0000038c  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .gnu.hash     00000024  00000000000003b0  00000000000003b0  000003b0  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  5 .dynsym       000000a8  00000000000003d8  00000000000003d8  000003d8  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  6 .dynstr       0000008d  0000000000000480  0000000000000480  00000480  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  7 .gnu.version  0000000e  000000000000050e  000000000000050e  0000050e  2**1
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  8 .gnu.version_r 00000030  0000000000000520  0000000000000520  00000520  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  9 .rela.dyn     000000c0  0000000000000550  0000000000000550  00000550  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 10 .rela.plt     00000018  0000000000000610  0000000000000610  00000610  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 11 .init         0000001b  0000000000001000  0000000000001000  00001000  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 12 .plt          00000020  0000000000001020  0000000000001020  00001020  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 13 .plt.got      00000010  0000000000001040  0000000000001040  00001040  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 14 .plt.sec      00000010  0000000000001050  0000000000001050  00001050  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 15 .text         00000107  0000000000001060  0000000000001060  00001060  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 16 .fini         0000000d  0000000000001168  0000000000001168  00001168  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 17 .rodata       00000011  0000000000002000  0000000000002000  00002000  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 18 .eh_frame_hdr 00000034  0000000000002014  0000000000002014  00002014  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 19 .eh_frame     000000ac  0000000000002048  0000000000002048  00002048  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 20 .init_array   00000008  0000000000003db8  0000000000003db8  00002db8  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 21 .fini_array   00000008  0000000000003dc0  0000000000003dc0  00002dc0  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 22 .dynamic      000001f0  0000000000003dc8  0000000000003dc8  00002dc8  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 23 .got          00000048  0000000000003fb8  0000000000003fb8  00002fb8  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 24 .data         00000010  0000000000004000  0000000000004000  00003000  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 25 .bss          00000008  0000000000004010  0000000000004010  00003010  2**0
                  ALLOC
 26 .comment      0000002b  0000000000000000  0000000000000000  00003010  2**0
                  CONTENTS, READONLY
```

Ok, so that's definitely complex. It's not just a simple `.text` section. There are a LOT of them.

This is too much to deal with right now. Where does the program even start? It starts with `main`, right? Wrong again!

```
$ objdump -f hello

hello:     file format elf64-x86-64
architecture: i386:x86-64, flags 0x00000150:
HAS_SYMS, DYNAMIC, D_PAGED
start address 0x0000000000001060
```

The "start address" (also known as the entry point), is `_start`, not `main`. This mystery function at `0x1060` must call our `main` somehow, but where does it come from!?

```
0000000000001060 g     F .text	0000000000000026              _start
```

Let's start simplifying the program. As we peel off complexity, we will get a chance to focus on understanding a few things at a time.

## Life without libc

A major source of complexity in our program comes from the standard libraries. They are used for printing the string and initializing the program. Let's get rid of them.

Easy enough, just compile with: `-nostdlib`.

Unfortunately, that means we no longer have access to the `printf` (or the `puts`) function. That's unfortunate since we still want to print "Hello Simplicity!".

It also means we will lose the `_start` function. It is provided by the C runtime library (CRT) to perform some initialization (like clearing the `.bss` segment) and call our `main` function. Since we still need our `main` to be executed, we will have to do something about that.

Fortunately, we can provide our own entry point with `-Wl,-e,<function_name>`. We *could* specify `main` as our entry point directly, but that would mean treating it as `void main()` instead of `int main()`. The entry point doesn't return anything. I feel that changing the signature of `main` is one bridge too far; let's instead create our own `void startup()` function that calls `main`.

For writing to `stdout`, we resort to the `syscall` assembly instruction. This instruction is how we ask the Linux kernel to do things for us. In this particular case, we would like to execute the `write` syscall to write a string to `stdout` (file descriptor = 1). Later on, we also want to call `exit` to terminate the process.

When calling the `syscall` instruction, we pass the syscall number in the `rax` register and the arguments in registers `rdi`, `rsi`, and `rdx`. The `write` syscall has number `0x01` and the `exit` syscall has number `0x3c`.

These are their C signatures:

```
ssize_t write(int fildes, const void *buf, size_t nbyte);
void exit(int status);
```

and this is our new program `hello-syscall.c`:

```
int main() {

  volatile const char message[] = "Hello Simplicity!\n";
  volatile const unsigned long length = sizeof(message) - 1;

  // write(1, message, length)
  asm volatile("mov $1, %%rax\n"                // write syscall number (0x01)
               "mov $1, %%rdi\n"                // Stdout file descriptor (0x01)
               "mov %0, %%rsi\n"                // Message buffer
               "mov %1, %%rdx\n"                // Buffer length
               "syscall"                        // Make the syscall
               :                                // No output operands
               : "r"(message), "r"(length)      // Input operands
               : "%rax", "%rdi", "%rsi", "%rdx" // Clobbered registers
  );

  return 0;
}

void startup() {

  volatile unsigned long status = main();

  // exit(status)
  asm volatile("mov $0x3c, %%rax\n" // exit syscall number (0x3c)
               "mov %0, %%rdi\n"    // exit status
               "syscall"            // Make the syscall
               :                    // No output operands
               : "r"(status)        // Input operands
               : "%rax", "%rdi"     // Clobbered registers
  );
}
```

In case you are wondering, the `volatile` keyword is required to prevent GCC from optimizing away the variables. And `unsigned long` is used instead of `int` to match the size of the `r__` 64-bit registers.

We build it like so:

```
gcc -Wl,-entry=startup -nostdlib -o hello-nostd hello-syscall.c
```

Is this really simpler than before? Well, yes!

It might not be *easier* to understand unless you are accustomed to assembly language, syscalls, and custom entry points. But simple is not synonymous with easy. Simple is the opposite of complex. Complex things are intrinsically hard to understand, no matter how much you know. Simple things are only hard to understand until you have acquired the appropriate skills. Rich Hickey explains this eloquently in his 2011 talk "[Simple Made Easy](https://youtu.be/SxdOUGdseq4?si=hsXwuad7doytJeJc)".

Still not convinced that we have actually made the program simpler? Let's take a look at the symbols and sections:

```
$ objdump -h -t hello-nostd

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .interp       0000001c  0000000000000318  0000000000000318  00000318  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .note.gnu.property 00000020  0000000000000338  0000000000000338  00000338  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .note.gnu.build-id 00000024  0000000000000358  0000000000000358  00000358  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .gnu.hash     0000001c  0000000000000380  0000000000000380  00000380  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .dynsym       00000018  00000000000003a0  00000000000003a0  000003a0  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  5 .dynstr       00000001  00000000000003b8  00000000000003b8  000003b8  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  6 .text         0000007f  0000000000001000  0000000000001000  00001000  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  7 .eh_frame_hdr 0000001c  0000000000002000  0000000000002000  00002000  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  8 .eh_frame     00000058  0000000000002020  0000000000002020  00002020  2**3
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  9 .dynamic      000000e0  0000000000003f20  0000000000003f20  00002f20  2**3
                  CONTENTS, ALLOC, LOAD, DATA
 10 .comment      0000002b  0000000000000000  0000000000000000  00003000  2**0
                  CONTENTS, READONLY

SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000 hello-syscall.c
0000000000000000 l    df *ABS*	0000000000000000
0000000000003f20 l     O .dynamic	0000000000000000 _DYNAMIC
0000000000002000 l       .eh_frame_hdr	0000000000000000 __GNU_EH_FRAME_HDR
0000000000001050 g     F .text	000000000000002f startup
0000000000004000 g       .dynamic	0000000000000000 __bss_start
0000000000001000 g     F .text	0000000000000050 main
0000000000004000 g       .dynamic	0000000000000000 _edata
0000000000004000 g       .dynamic	0000000000000000 _end
```

There's still a lot going on here, but at least it now fits on one screen. As expected, `objdump -f` gives us a new start address: `0x1050`. It's our `startup` function!

Let's continue simplifying!

## Life without PIE

For the last 20 years, your programs have been loaded into memory at random addresses as a security mitigation. ASLR (Address Space Layout Randomization) makes it harder to write exploits since the shellcode can't jump to hardcoded destinations. It also means jumps in your regular programs can't be hardcoded.

By default, programs on modern systems are built as Position Independent Executables (PIE). Addresses are resolved when the program is loaded into memory. It's great for security, but it adds complexity. Let's get rid of it with: `-no-pie`.

To further unclutter our assembly code, we turn off some more safety features with `-fcf-protection=none` and `-fno-stack-protector`. We also get rid of some metadata generation with `-Wl,--build-id=none` and some debugger-friendly stack unwinding info with `-fno-unwind-tables` and `-fno-asynchronous-unwind-tables`.

```
gcc -no-pie \
    -nostdlib \
    -Wl,-e,startup \
    -Wl,--build-id=none \
    -fcf-protection=none \
    -fno-stack-protector \
    -fno-asynchronous-unwind-tables \
    -fno-unwind-tables \
    -o hello-nostd-nopie hello.c
```

We are now down to this:

```
$ objdump -h -t hello-nostd-nopie

hello-nostd-nopie:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 .text         00000077  0000000000401000  0000000000401000  00001000  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .comment      0000002b  0000000000000000  0000000000000000  00001077  2**0
                  CONTENTS, READONLY
SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000 hello-syscall.c
000000000040104c g     F .text	000000000000002b startup
0000000000402000 g       .text	0000000000000000 __bss_start
0000000000401000 g     F .text	000000000000004c main
0000000000402000 g       .text	0000000000000000 _edata
0000000000402000 g       .text	0000000000000000 _end
```

Did you notice how the symbol addresses changed with `-no-pie`? Before, they were relative, waiting for some offset to be added at load time. Now, they are absolute, and `main` will really be at `0x00401000`.

```
$ gdb hi
(gdb) break main
Breakpoint 1 at 0x401004
(gdb) run
Breakpoint 1, 0x0000000000401004 in main ()
```

Phew! We are finally approaching something simple-ish. Now, our entire program even fits on one screen:

```
$ objdump -d -M intel hello-nostd-nopie

Disassembly of section .text:

0000000000401000 <main>:
  401000:	55                   	push   rbp
  401001:	48 89 e5             	mov    rbp,rsp
  401004:	48 b8 48 65 6c 6c 6f 	movabs rax,0x6953206f6c6c6548
  40100b:	20 53 69
  40100e:	48 ba 6d 70 6c 69 63 	movabs rdx,0x79746963696c706d
  401015:	69 74 79
  401018:	48 89 45 e0          	mov    QWORD PTR [rbp-0x20],rax
  40101c:	48 89 55 e8          	mov    QWORD PTR [rbp-0x18],rdx
  401020:	66 c7 45 f0 21 0a    	mov    WORD PTR [rbp-0x10],0xa21
  401026:	c6 45 f2 00          	mov    BYTE PTR [rbp-0xe],0x0
  40102a:	48 c7 45 d8 12 00 00 	mov    QWORD PTR [rbp-0x28],0x12
  401031:	00
  401032:	4c 8b 45 d8          	mov    r8,QWORD PTR [rbp-0x28]
  401036:	48 8d 4d e0          	lea    rcx,[rbp-0x20]
  40103a:	48 c7 c0 01 00 00 00 	mov    rax,0x1
  401041:	48 c7 c7 01 00 00 00 	mov    rdi,0x1
  401048:	48 89 ce             	mov    rsi,rcx
  40104b:	4c 89 c2             	mov    rdx,r8
  40104e:	0f 05                	syscall
  401050:	b8 00 00 00 00       	mov    eax,0x0
  401055:	5d                   	pop    rbp
  401056:	c3                   	ret

0000000000401057 <startup>:
  401057:	55                   	push   rbp
  401058:	48 89 e5             	mov    rbp,rsp
  40105b:	48 83 ec 10          	sub    rsp,0x10
  40105f:	b8 00 00 00 00       	mov    eax,0x0
  401064:	e8 97 ff ff ff       	call   401000 <main>
  401069:	48 98                	cdqe
  40106b:	48 89 45 f8          	mov    QWORD PTR [rbp-0x8],rax
  40106f:	48 8b 55 f8          	mov    rdx,QWORD PTR [rbp-0x8]
  401073:	48 c7 c0 3c 00 00 00 	mov    rax,0x3c
  40107a:	48 89 d7             	mov    rdi,rdx
  40107d:	0f 05                	syscall
  40107f:	90                   	nop
  401080:	c9                   	leave
  401081:	c3                   	ret
```

You can see the `startup` function calling `main`, the two syscalls, and the "Hello Simplicity!" string hardcoded as a large number of ASCII values (being loaded onto the stack, relative to the stack base pointer `rbp`).

There's not a lot of complexity left, at least not at this level. Our ELF is actually quite simple! But wait, there is more!

## Linker Scripts

Where do the strange symbols (like `__bss_start`) come from? And who decides that our `startup` function should be loaded into memory at `0x0040104c`? What if we want our code to live in the cool `0xc0d30000` address range?

These things are specified in the linker script. Until now, we have been using the default one, which you can see with `ld -verbose`. It's very complex. Let's get rid of it.

Our simple hello world application doesn't use any global variables. If it had, they would fall into three categories:

* `.rodata`: Constants with values provided at compile time, like our hardcoded string.
* `.data`: Non-const variables with values provided at compile time.
* `.bss`: Uninitialized global variables.

Let's complicate our program a tiny bit by introducing a symbol for each category. This will provide a more interesting linker script example. Here is the new program `hello-data.c`:

```
const char message[] = "Hello Simplicity!\n";   // .rodata
unsigned long length = sizeof(message) - 1;     // .data
unsigned long status;                           // .bss

int main() {
  // write(1, message, length)
  asm volatile("mov $1, %%rax\n"                // write syscall number (0x01)
               "mov $1, %%rdi\n"                // Stdout file descriptor (0x01)
               "mov %0, %%rsi\n"                // Message buffer
               "mov %1, %%rdx\n"                // Buffer length
               "syscall"                        // Make the syscall
               :                                // No output operands
               : "r"(message), "r"(length)      // Input operands
               : "%rax", "%rdi", "%rsi", "%rdx" // Clobbered registers
  );

  return 0;
}

void startup() {
  status = main();

  // exit(status)
  asm volatile("mov $0x3c, %%rax\n" // exit syscall number (0x3c)
               "mov %0, %%rdi\n"    // exit status
               "syscall"            // Make the syscall
               :                    // No output operands
               : "r"(status)        // Input operands
               : "%rax", "%rdi"     // Clobbered registers
  );
}
```

Looking at the symbol table again, without using a custom linker script, we can see the globals in `.data`, `.rodata` and `.bss` respectively:

```
000000000040102f g     F .text	000000000000002d startup
0000000000403010 g     O .data	0000000000000008 length
0000000000402000 g     O .rodata	000000000000000e message
0000000000401000 g     F .text	000000000000002f main
0000000000403018 g     O .bss	0000000000000008 status
```

Now, let's create a simple and fun linker script (`hello.ld`) with a cool memory map and emojis in the section names:

```
MEMORY {
  IRAM (rx) : ORIGIN = 0xC0DE0000, LENGTH = 0x1000
  RAM  (rw) : ORIGIN = 0xFEED0000, LENGTH = 0x1000
  ROM  (r)  : ORIGIN = 0xDEAD0000, LENGTH = 0x1000
}

SECTIONS
{
  "📜 .text" : {
    *(.text*)
  } > IRAM

  "📦 .data" : {
    *(.data*)
  } > RAM

  "📁 .bss" : {
    *(.bss*)
  } > RAM

  "🧊 .rodata" : {
    *(.rodata*)
  }  > ROM

  /DISCARD/ : { *(.comment) }
}

ENTRY(startup)
```

We use the same build options as before but add `-T hello.ld` to start using our linker script.

This is the simple program in its final form:

```
$ objdump -t -h hello-data

hello-data:     file format elf64-x86-64

Sections:
Idx Name          Size      VMA               LMA               File off  Algn
  0 📜 .text    0000005c  00000000c0de0000  00000000c0de0000  00001000  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 📦 .data    00000008  00000000feed0000  00000000feed0000  00003000  2**3
                  CONTENTS, ALLOC, LOAD, DATA
  2 📁 .bss     00000008  00000000feed0008  00000000feed0008  00003008  2**3
                  ALLOC
  3 🧊 .rodata  00000013  00000000dead0000  00000000dead0000  00002000  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
SYMBOL TABLE:
0000000000000000 l    df *ABS*	0000000000000000 hello-data.c
00000000c0de002f g     F 📜 .text	000000000000002d startup
00000000feed0000 g     O 📦 .data	0000000000000008 length
00000000dead0000 g     O 🧊 .rodata	0000000000000013 message
00000000c0de0000 g     F 📜 .text	000000000000002f main
00000000feed0008 g     O 📁 .bss	0000000000000008 status
```

Isn't that absolutely adorable?!

I've put some sample code over at [github.com/4ZM/elf-shenanigans](https://github.com/4ZM/elf-shenanigans) to reproduce the examples in this article.

If you want to learn more about linker scripts (and why wouldn't you?!) this is an outstanding technical documentation: "[c_Using_LD](https://users.informatik.haw-hamburg.de/~krabat/FH-Labor/gnupro/5_GNUPro_Utilities/c_Using_LD/ldLinker_scripts.html)".

If you want to explore more ridiculous things to do with section names, check out my other article: "[ELF Shenanigans](../2024-12-25-elf-shenanigans/elf-shenanigans.md)".

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

*Update 2024-12-28:* This article made it to the top of [Hacker News](https://news.ycombinator.com/item?id=42516697)! There is a lot of interesting comments and links to similar content in the comments.

*Update 2025-01-12:* There are a few issues with the inline assembly in my examples. Here is a better way to write it:

```
// write(1, message, length)
asm volatile("syscall"        // Make the syscall
             :                // No output operands
             : "a"(1),        // rax (s#)
               "D"(1),        // rdi (fd)
               "S"(message),  // rsi
               "d"(length)    // rdi
             : "rcx", "r11",  // Clobbered registers
               "cc", "memory" // Clobbered flags and memory
    );
```

This will make sure the right values are in the right registers, some times without actually having to do anything. It also lists the `r11` as clobbered. But, there is also another issue in the examples. It's related to stack alignment. If you try compiling with `-O2` you will likely get a segfault since the stack is not 16 byte aligned. Fixing this would sidetrack and obscure the narrative of the article. Just be warned, this is not how you write production code, this is blogware.
