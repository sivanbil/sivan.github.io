# gdb 的调试步骤

在 Linux 下既然是使用命令行来调试，顾名思义就是手敲命令来调试程序，大体分为下面几个步骤，后面会详细介绍：

- 编译可以调试的程序
- 运行程序，打断点
- 单步调试，监控变量
- 可视化调试
- 其他调试技术

## 1. 编译可以调试的程序
我们平常使用 gcc 编译的程序如果不加 [-g] 选项：
<pre lang="shell">
g++ hello.cpp -o hello
</pre>
gdb 会提示该可执行文件没有调试符号，不能调试：
<pre lang="shell">
gdb hello
# gdb 提示信息
Reading symbols from a.out...(no debugging symbols found)...done.
</pre>

如果需要让程序可以调试，就必须在编译的时候加上 [-g] 参数：
<pre>
g++ hello.cpp -g -o hello
</pre>

## 2. 载入要调试的程序

方法一 - gdb 可执行文件
使用如下的命令来载入可执行文件 `hello` 到 gdb 中：
<pre lang="shell">
gdb hello
</pre>
载入成功，gdb 会打印一段提示信息，并且命令行前缀变为 (gdb)，下面是我的 Ubuntu 打印的信息：
<pre lang="shell">
NU gdb (Ubuntu 9.2-0ubuntu1~20.04) 9.2
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from hello...
(gdb) l
1	#include <iostream>
2	
3	using namespace std;
4	
5	int main()
6	{
7		cout << "Hello world" << endl;
8		return 0;
9	}
(gdb) 

(gdb) q

</pre>

注：按 q 退出 gdb

方法二 - 使用 gdb 提供的 file 命令
第二种方法是在 gdb 环境中使用 file 命令，我们需要先进入 gdb 环境下：
<pre>
gdb
</pre>

使用 file hello 载入待调试程序：
<pre>
...
(gdb) file hello
Reading symbols from hello...done.
(gdb) q

</pre>

## 3. 查看调试程序
在 gdb 下查看调试程序使用命令 list 或简写 l，「回车」列出后面程序：


## 4. 添加断点
在 gdb 下添加断点使用命令 break 或简写 b，有下面几个常见用法（这里统一用 b）：
    b function_name
    b row_num
    b file_name:row_num
    b row_num if condition

比如我们以第一个为例，在 main 函数上添加断点：
<pre>
(gdb) b main
Breakpoint 1 at 0x11a9: file hello.cpp, line 6.
</pre>
打印的信息告诉我们在 hello.c 文件的第 4 行，地址 0x6e8 处添加了一个断点，那如何查看断点呢？

## 5. 查看断点

在 gdb 下查看断点使用命令 info break 或简写 i b，比如查看刚才打的断点：
<pre>
(gdb) i b
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000000011a9 in main() at hello.cpp:6
</pre>
可以看到打印出刚才添加的 main 函数的断点信息：编号，类型，显示状态，是否启用，地址，其他信息，那又如何删除这个断点呢？

## 6. 禁用断点
在 gdb 下禁用断点使用命令 disable Num，比如禁用刚才打的断点：
<pre>
(gdb) disable 1
(gdb) i b
Num     Type           Disp Enb Address            What
1       breakpoint     keep n   0x00000000000006e8 in main at hello.c:4

</pre>
可以看到字段「Enb」已经变为 n，表示这个断点已经被禁用了。

## 7. 删除断点
在 gdb 下删除断点使用命令 delete 断点 Num 或简写 d Num，比如删除刚才的 Num = 1 的断点：
<pre>
(gdb) d 1
(gdb) i b
No breakpoints or watchpoints.

</pre>
删除后再次查看断点，提示当前没有断点，说明删除成功啦，下面来运行程序试试。

## 8. 运行程序
在 gdb 下使用命令 run 或简写 r 来运行当前载入的程序：
<pre>
(gdb) r
Starting program: /home/orange/Desktop/gdb/hello 
a = 1
b = 2
1 + 2 = 3
[Inferior 1 (process 10415) exited normally]

</pre>
我这次没有添加断点，程序全速运行，然后正常退出了。

## 9. 单步执行下一步
在 gdb 下使用命令 next 或简写 n 来单步执行下一步，假设我们在 main 打了断点：
<pre>
(gdb) b main
Breakpoint 1 at 0x6e8: file hello.c, line 4.
(gdb) r
Starting program: /home/orange/Desktop/gdb/hello 

Breakpoint 1, main () at hello.c:4
4       int a = 1;
(gdb) n
5       printf("a = %d\n", a);

</pre>
可以看到当前停在 int a = 1; 这一行，按 n 执行了下一句代码 printf("a = %d\n", a);

## 10. 跳入，跳出函数
在 gdb 下使用命令 step 或简写 s 来跳入一个函数，使用 finish 来跳出一个函数，我们在第 14 行 int c = add(a, b); 添加一个断点：
<pre>
(gdb) b 14
Breakpoint 1 at 0x6f6: file hello.c, line 14.
(gdb) r
Starting program: /home/orange/Desktop/gdb/hello 
a = 1
b = 2

Breakpoint 1, main () at hello.c:14
14      int c = add(a, b);
(gdb) s
add (x=1, y=2) at hello.c:4
4       return x + y;
(gdb) finish
Run till exit from #0  add (x=1, y=2) at hello.c:4
0x0000555555554705 in main () at hello.c:14
14      int c = add(a, b);
Value returned is $1 = 3
(gdb) n
15      printf("%d + %d = %d\n", a, b, c);

</pre>

这个过程是这样的：

    在 14 行 int c = add(a, b); 添加断点
    程序运行并停到 int c = add(a, b); 这一行
    s 跳入 add 函数
    finish 跳出 add 函数，并输出一些函数返回的信息
## 11. 打印变量

在 gdb 中使用命令 print var 或简写 p var 来打印一个变量或者函数的返回值，我们在第 10 行 int b = 2; 添加一个断点：
<pre>
(gdb) b 10
Breakpoint 1 at 0x6c3: file hello.c, line 10.
(gdb) r
Starting program: /home/orange/Desktop/gdb/hello 

Breakpoint 1, main () at hello.c:10
10      int b = 2;
(gdb) p a
$1 = 1

</pre>
我们打印出变量 a 的值为 1，在调试中比较频繁的操作是「监视变量」，在 gdb 中如何做呢？

## 12. 监控变量

在 gdb 中使用命令 watch var 来监控一个变量，使用 info watch 来查看监控的变量，我们这里来监控变量 c：

<pre>
(gdb) b 14
Breakpoint 1 at 0x6f6: file hello.c, line 14.
(gdb) r
Starting program: /home/orange/Desktop/gdb/hello 
a = 1
b = 2

Breakpoint 1, main () at hello.c:14
14      int c = add(a, b);
(gdb) watch c
Hardware watchpoint 2: c
(gdb) info watch
Num     Type           Disp Enb Address            What
2       hw watchpoint  keep y                      c

</pre>
注意：程序必须要先运行才能监控。

## 13. 查看变量类型

在 gdb 下使用命令 whatis 查看一个变量的类型：
<pre>
(gdb) b 10
Breakpoint 1 at 0x6c3: file hello.c, line 10.
(gdb) r
Starting program: /home/orange/Desktop/gdb/hello 

Breakpoint 1, main () at hello.c:10
10      int b = 2;
(gdb) whatis b
type = int

</pre>
这里变量 b 是 int 类型。

## 14. 在 gdb 中进入 shell

在 gdb 下使用命令 shell 启动 shell ：
<pre>
(gdb) shell
orange@ubuntu:~/Desktop/gdb$ exit
exit
(gdb) 

</pre>
使用 exit 会再次退回到 gdb 中。

## 15. 在 gdb 中实现可视化调试
谁说 gdb 只能在命令行调试呢？gdb 也支持「图形界面」，不过这里的图形界面都是用字符显示的，当然不如 VS 那种好看，不过使用可视化相比直接看命令行更加直观了。

在 gdb 下使用 wi 启动可视化调试：
<pre>
(gdb) wi
效果如下图所示，上面是代码效果，下面是命令界面：
![image](https://user-images.githubusercontent.com/3192587/119617344-5646f600-be34-11eb-8a7a-ea0e3aa85ddd.png)

</pre>
有了图形界面，就再对照着图形界面将前面的命令再练习练习，看看自己手敲的命令背后到底做了些什么，加深下影响。

- 作者：登龙zZ
- 链接：https://www.jianshu.com/p/08de5cef2de9
- 来源：简书
- 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
