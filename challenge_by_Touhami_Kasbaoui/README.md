We gonna take a crack at a meduim binary exploitation challenge by Touhami Kasbaoui.

This is the link for the binary:

http://tracker.0btemoslab.com/challenges/take-it-easy

![file](https://pouch.jumpshare.com/preview/zlejgyvYLyHjzNh9Ymf8nOJE5Y9XUyV811MVVYOBjty4jWyd9mBPAR97IEnHrzB9xQysGijYlM7l3EWEjMzh9TJprjKJn4snkOQIlHyX-Do)

As you can see, it's a stripped x86_64 ELF binary. Let's check the ELF header.

![readelf -h header](https://pouch.jumpshare.com/preview/BvNXIFjQp8f6YqGbVYaneC7wiItwdnqsmDoPnC9lQo2LDFrfY55bpSL4txKwahbLJmHTIUyC7XoQy9o-VssUUTJprjKJn4snkOQIlHyX-Do)

What we can learn from this is that this binary entry point is at **0x400144**.

Let's execute the binary.

![./take-it-easy](https://pouch.jumpshare.com/preview/7MlrxrrC7lC1fLIyllR8XvdoDB-u6fFgxxlA0ec3_ifr268PTCZ_oX1v4Q1NpCUdH_5BUel3Fq_kXqVIVHC5bxA1jQB_zmxltR5bgJ1-nls)

As you can see that the binary output input and read characters from the stdin, After that output checkser and read again. Let's use gdb to examine what's happening.

*NOTE: Remember that it is a stripped binary so no symbols are available, we need to break at the entry point address 0x400144*

![gdb take-it-easy](https://pouch.jumpshare.com/preview/bnDtseOR67PEK6j0LtLkudqvKTKEiLcTtURiqQ4-t0DN7SNmTUG5C9KRUe0AaiezR6DQnalSTy3EhLe3szKU4BA1jQB_zmxltR5bgJ1-nls)

You can use the gdb to go step by step with instructions and use ***i r*** to list the registers and examin they values.

[GDB-PEDA](https://github.com/longld/peda) is a great extention that will help you visualize the register and the stack more easily.

This is the instructions the binary contain:

```

   0x400144:	mov    rax,0x1
   0x40014b:	mov    rdi,0x1
   0x400152:	movabs rsi,0x6000000                ("input\n")
   0x40015c:	mov    rdx,0x6
   0x400163:	syscall                             write syscall  
   0x400165:	xor    rax,rax
   0x400168:	xor    rdi,rdi
   0x40016b:	movabs rbx,0x6000400
   0x400175:	mov    rsi,rbx
   0x400178:	mov    rdx,0x8                     reads 8Bytes
   0x40017f:	syscall                            read syscall
   0x400181:	rdtsc                              Read Time-Stamp Counter
   0x400183:	xor    rax,rdx                     XOR rax and rdx that contain the Time-Stamp
   0x400186:	mov    rdi,rbx
   0x400189:	mov    rcx,0x0
   0x400190:	mov    rdi,rbx                     --------------------------------------------
   0x400193:	add    rdi,rcx                       A loop that iterate on the buffer read 
   0x400196:	xor    BYTE PTR [rdi],al               before and perfome an xop operation
   0x400198:	inc    rcx                              on each byte and the al register
   0x40019b:	cmp    rcx,0x8
   0x40019f:	jl     0x400190                    --------------------------------------------
   0x4001a1:	mov    rax,QWORD PTR [rbx]
   0x4001a4:	push   rax                         Push the sanitizer buffer to the stack
   0x4001a5:	sub    rsp,0x20                    Subtract the rsp pointer
   0x4001a9:	mov    rax,0x1
   0x4001b0:	mov    rdi,0x1
   0x4001b7:	movabs rsi,0x6000007               (checkser\n)
   0x4001c1:	mov    rdx,0xa
   0x4001c8:	syscall                            write syscall
   0x4001ca:	xor    rax,rax
   0x4001cd:	xor    rdi,rdi
   0x4001d0:	mov    rsi,rsp                     store the buffer read directly on the stack
   0x4001d3:	mov    rdx,0x100                   reads 256Bytes
   0x4001da:	syscall                            read syscall
   0x4001dc:	add    rsp,0x20                    restore the stack to it's original address wish mean 32Byte of out input is ignored
   0x4001e0:	mov    rdi,0x1
   0x4001e7:	mov    rsi,0x3
   0x4001ee:	mov    rdx,0x3
   0x4001f5:	mov    rcx,0x7
   0x4001fc:	cmp    rax,0x20                    If the buffer size read is less than or equal to 32Byte then the program exit
   0x400200:	jle    0x400214
   0x400202:	pop    r11                         pop the buffer from the stack
   0x400204:	xor    r11,QWORD PTR [rbx]         check if the buffer was tampred with
   0x400207:	jne    0x400214
   0x400209:	mov    QWORD PTR [rsp+0x8],0x23    set 0x23 at the rsp+0x8 wish will set the code segment at the return far
   0x400212:	retfq                              return far wish pop the instruction address and code segment from the top of the stack
   0x400214:	mov    rax,0x3c
   0x40021b:	mov    rbx,0x0
   0x400222:	syscall                            exit syscall

```
