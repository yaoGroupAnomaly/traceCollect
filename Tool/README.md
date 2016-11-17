#System call traces collection

## 1) Using strace 

To obtain the PC value from which a system call was invoked, we need to go back through the call stacks until finding a 
valid PC along with each system call in a program. Therefore, we compiled strace with -libunwind support, which enables stack unwinding,
so as to print call stacks (libunwind-based backtraces) on every system call. 

For example:

we use strace-4.13 to trace system calls

strace -k  -i  -tt -T -o ./strace.log ./test

we get the system call traces with call stacks like the following:


18:21:17.042053 [00007ffff7b0987a] mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7ffff7ff5000 <0.000028>

 > /lib/x86_64-linux-gnu/libc-2.19.so(mmap64+0xa) [0xf487a]

 > /lib/x86_64-linux-gnu/libc-2.19.so(_IO_file_doallocate+0x7c) [0x6d82c]

 > /lib/x86_64-linux-gnu/libc-2.19.so(_IO_doallocbuf+0x35) [0x7b4a5]

 > /lib/x86_64-linux-gnu/libc-2.19.so(_IO_file_overflow+0x198) [0x7a858]

 > /lib/x86_64-linux-gnu/libc-2.19.so(_IO_file_xsputn+0xb1) [0x795a1]

 > /lib/x86_64-linux-gnu/libc-2.19.so(_IO_vfprintf+0x1cc5) [0x4b905]

 > /lib/x86_64-linux-gnu/libc-2.19.so(_IO_printf+0x99) [0x543d9]

 > /home/SIR/objects/gzip/source/gzip.exe(help+0x33) [0x8b2e]

 > /home/SIR/objects/gzip/source/gzip.exe(main+0x307) [0x8f93]
 
 
## 2) Using SystemTap
 
Below is the Stap script of using SystemTap for system call tracing:

process/strace.stp - Trace system calls

https://sourceware.org/systemtap/examples/process/strace.stp

The script loosely emulates strace, when applied to individual processes or hierarchies (via -c/-x), or the entire system (without -c/-x). A few output configuration parameters may be set with -G.

This is the output of the above example:

xx@desktop:~/systemstap$ sudo stap strace.stp -w -c "echo hello world" 
hello world

Wed Oct 26 16:03:40 2016.981683 rt_sigsuspend([EMPTY], 8) = -514

Wed Oct 26 16:03:40 2016.981800 rt_sigreturn() = -4 (EINTR)

Wed Oct 26 16:03:40 2016.981816 rt_sigaction(SIGUSR1, {SIG_DFL}, 0x0, 8) = 0

Wed Oct 26 16:03:40 2016.981821 rt_sigprocmask(SIG_SETMASK, [EMPTY], 0x0, 8) = 0

Wed Oct 26 16:03:40 2016.981838 execve("/usr/local/sbin/echo", ["echo", "hello", "world"], [/* 19 vars */]) = -2 (ENOENT)

Wed Oct 26 16:03:40 2016.981851 execve("/usr/local/bin/echo", ["echo", "hello", "world"], [/* 19 vars */]) = -2 (ENOENT)

Wed Oct 26 16:03:40 2016.981856 execve("/usr/sbin/echo", ["echo", "hello", "world"], [/* 19 vars */]) = -2 (ENOENT)

Wed Oct 26 16:03:40 2016.981861 execve("/usr/bin/echo", ["echo", "hello", "world"], [/* 19 vars */]) = -2 (ENOENT)

Wed Oct 26 16:03:40 2016.981866 execve("/sbin/echo", ["echo", "hello", "world"], [/* 19 vars */]) = -2 (ENOENT)

Wed Oct 26 16:03:40 2016.981872 execve("/bin/echo", ["echo", "hello", "world"], [/* 19 vars */]) = 0

Wed Oct 26 16:03:40 2016.982018 brk(0x0) = 6324224

Wed Oct 26 16:03:40 2016.982030 access("/etc/ld.so.nohwcap", F_OK) = -2 (ENOENT)

Wed Oct 26 16:03:40 2016.982038 mmap2(0x0, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7ffff7ff6000

Wed Oct 26 16:03:40 2016.982048 access("/etc/ld.so.preload", R_OK) = -2 (ENOENT)

Wed Oct 26 16:03:40 2016.982056 open("/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3

Wed Oct 26 16:03:40 2016.982063 fstat(3, 0x7fffffffdcd0) = 0

Wed Oct 26 16:03:40 2016.982067 mmap2(0x0, 212337, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7ffff7fc2000

Wed Oct 26 16:03:40 2016.982071 close(3) = 0

Wed Oct 26 16:03:40 2016.982077 access("/etc/ld.so.nohwcap", F_OK) = -2 (ENOENT)

Wed Oct 26 16:03:40 2016.982089 open("/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3

Wed Oct 26 16:03:40 2016.982093 read(3, 0x7fffffffde88, 832) = 832

Wed Oct 26 16:03:40 2016.982097 fstat(3, 0x7fffffffdd20) = 0

Wed Oct 26 16:03:40 2016.982101 mmap2(0x0, 3967488, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7ffff7a0e000

Wed Oct 26 16:03:40 2016.982107 mprotect(0x7ffff7bce000, 2093056, PROT_NONE) = 0

Wed Oct 26 16:03:40 2016.982117 mmap2(0x7ffff7dcd000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 1830912) = 0x7ffff7dcd000

Wed Oct 26 16:03:40 2016.982127 mmap2(0x7ffff7dd3000, 14848, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7ffff7dd3000

Wed Oct 26 16:03:40 2016.982133 close(3) = 0

Wed Oct 26 16:03:40 2016.982145 mmap2(0x0, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7ffff7fc1000

Wed Oct 26 16:03:40 2016.982149 mmap2(0x0, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7ffff7fc0000

Wed Oct 26 16:03:40 2016.982153 mmap2(0x0, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7ffff7fbf000

Wed Oct 26 16:03:40 2016.982158 arch_prctl(ARCH_SET_FS, 140737353877248) = 0

Wed Oct 26 16:03:40 2016.982198 mprotect(0x7ffff7dcd000, 16384, PROT_READ) = 0

Wed Oct 26 16:03:40 2016.982204 mprotect(0x606000, 4096, PROT_READ) = 0

Wed Oct 26 16:03:40 2016.982209 mprotect(0x7ffff7ffc000, 4096, PROT_READ) = 0

Wed Oct 26 16:03:40 2016.982213 munmap(0x7ffff7fc2000, 212337) = 0

Wed Oct 26 16:03:40 2016.982275 brk(0x0) = 6324224

Wed Oct 26 16:03:40 2016.982278 brk(0x629000) = 6459392

Wed Oct 26 16:03:40 2016.982285 open(00007ffff7ba0f30, O_RDONLY|O_CLOEXEC) = 3

Wed Oct 26 16:03:40 2016.982294 fstat(3, 0x7ffff7dd2980) = 0

Wed Oct 26 16:03:40 2016.982297 mmap2(0x0, 4763056, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7ffff7583000

Wed Oct 26 16:03:40 2016.982304 close(3) = 0

Wed Oct 26 16:03:40 2016.982332 fstat(1, 0x7fffffffe440) = 0

Wed Oct 26 16:03:40 2016.982337 write(1, "hello world\n", 12) = 12

Wed Oct 26 16:03:40 2016.982349 close(1) = 0

Wed Oct 26 16:03:40 2016.982351 close(2) = 0

Wed Oct 26 16:03:40 2016.982355 exit_group(0) = 
