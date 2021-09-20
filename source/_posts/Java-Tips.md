---
title: Java Tips
date: 2021-01-17 14:38:09
tags: [Java]
categories: [Java]
description: <center>粗略学习Java时小知识点的记录</center>
copyright: true
photos: "http://r.photo.store.qq.com/psc?/V13BnVsC23MbK8/TmEUgtj9EK6.7V8ajmQrEFTiGnOFh8Rb0*LAV799BMK9DkAchub9bhNkvTOHWOCKNtBXuc8eFzYorSeuvj47rh8Y5syl3BvROxMjizbysOQ!/r"
---

---

# 大数值
- BigInteger和BigDecimal

---

# 字符串
- String类对象为不可变字符串，不能直接使用下标修改字符串的某个位置，而是需要提取、拼接来修改
- char：一般不使用char
- ==：不使用`==`进行字符串间的比较，比较字符串是否相等使用`equls()`或者`compareTo()`
- StringBuilder Class：适用于使用多个小段字符串构建一个字符串。使用`append()`方法添加，在使用`toString()`方法构建出字符串

---

# I/O
- Scanner Class：通过与标准输入流`System.in`关联，通过控制台输入(输入是可见的)
```Java
Scanner in = new Scanner(System.in);
//输入一行
in.nextLine();
//输入一个不含空格的字符串
in.next();
//输入整数
in.nextInt();
```
- Console Class：适用于在控制台输入密码（不显示明文）
```Java
Console cons = System.console();
char[] passwd = cons.readPassword("Password: ");
```
为了安全起见，在对密码进行处理之后，应该马上用一个填充值覆盖数组元素
- 文件读写：使用例如`Scanner in = new Scanner(Paths.get("myfile.txt"), "UTF-8");`创建一个读取文件的对象（读取的文件必须存在）；使用例如`PrintWriter out = new PrintWriter("myfile.txt", "UTF-8");`创建一个写文件的对象（若文件不存在则创建，文件名必须是可被创建的）

---

# 控制流程
- 块作用域：不能在内层命名空间（作用域）重复定义、声明外层代码块的同名变量
- 带标签break：设置标签，跳出语句块，跳转到带标签的语句块末尾
```Java
label:
{
	...
	if(condition) break label; //exit block
	...
}
//jumps here when the break statement executes
```
- for each：依次处理数组中的每一个元素（或实现了Iterable接口的类对象）
```Java
for (variable : collection) statement;
```
若要遍历二维数组，则需要两层for each：
```Java
for (double[] row : a) {
	for (double value : row) {
		do something with value;
	}
}
```

---

# 数组
- 数组拷贝：将一个数组变量拷贝给另一个数组变量，这两个变量引用同一个数组；若是希望拷贝一个数组到新的数组，则需要使用Arrays类的copyOf方法。
- 命令行参数args：在Java应用程序的main方法中，程序名并没有存储在args数组中，例如
```bash
java Mesage -h world
```
args[0]是“-h”，而不是java或Message

---

# 类与对象
- 更改器方法：能修改对象的方法
- 访问器方法：只访问对象而不修改对象的方法
- Java文件名必须与public类的名字相匹配。在源文件中，只能有一个公有类，但可以有任意数目的非公有类
- 构造器总是伴随new操作符一起使用：`new Employee("James", ...)`
- Java中，所有方法必须在类的内部定义
- 不要返回引用可变对象的访问器方法，如果需要返回一个可变对象的引用，应该首先对其进行克隆
- 类的方法可以访问类的人一个对象的私有域
- Java可以直接初始化类的实例域
- （this）调用另一个构造器：构造器的第一个语句形如`this(...)`，这个构造器将调用同一类的另一个构造器
- 初始化块：在一个类的声明中，可以包含多个代码块。只要构造类的对象，这些块就会被执行
- Java不支持析构器
- instanceof：双目运算符，用来测试一个对象是否为一个类的实例，eg：`boolean result = obj instanceof Class`

---

# 继承
- Java中所有继承都是公有继承
- Java使用关键字super调用超类方法，也可以实现对超类构造器的调用。
- 阻止继承：使用final类和方法
- Object class：默认为所有类的超类（根类）
- Object.equals(a, b)：防备私有数据成员可能为为NULL的情况，若两个参数都是NULL则返回true；其中一个参数为NULL则返回false；两个都不为NULL则调用`ａ.equals(b)`
- ArrayList：类似于C++的vector，但不能使用`[]`去访问数组列表中的元素，`add`新增、`set`修改、`get`访问
- 可变参数数量：使用`...`

---

# 接口、lambda表达式、内部类
- 接口没有实例

---

# 日后看情况补充

---