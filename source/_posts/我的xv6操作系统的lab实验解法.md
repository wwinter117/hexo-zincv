---
title: 我的xv6操作系统的lab实验解法
tags:
  - 操作系统
  - xv6
  - 汇编
  - riscv
date: 2023-6-26 22:46:49
---
![](https://zincv.oss-cn-hangzhou.aliyuncs.com/images/xv6-Pasted%20image%2020241002142130.png)


- 课程官网：[6.S081 Fall 2020](https://pdos.csail.mit.edu/6.828/2020/xv6.html) 或者 [6.S081 Fall 2023](https://pdos.csail.mit.edu/6.S081/2023/)
- 课程视频：[6.S081--bilibili](https://www.bilibili.com/video/BV19k4y1C7kA?from=search&seid=5542820295808098475)
+ 我的私人仓库：https://github.com/wwinter117/xv6-labs.git
# Lab1: Xv6 and Unix utilities

## 实验任务

### 启动xv6(难度：Easy)

获取实验室的xv6源代码并切换到util分支

```shell
$ git clone git://g.csail.mit.edu/xv6-labs-2020
Cloning into 'xv6-labs-2020'...
...
$ cd xv6-labs-2020
$ git checkout util
Branch 'util' set up to track remote branch 'util' from 'origin'.
Switched to a new branch 'util'
```

`xv6-labs-2020` 存储库与本书的 `xv6-riscv` 稍有不同;它主要添加一些文件。如果你好奇的话，可以执行 `git log`:

```shell
$ git log
```

您将需要使用Git版本控制系统管理和提交文件以及后续的实验室作业。接下来，切换到一个分支(执行`git checkout util`)，其中包含针对该实验室定制的xv6版本。要了解关于Git的更多信息，请查看Git用户手册。Git允许您跟踪对代码所做的更改。例如，如果你完成了其中一个练习，并且想检查你的进度，你可以通过运行以下命令来提交你的变化:

```shell
$ git commit -am 'my solution for util lab exercise 1'
Created commit 60d2135: my solution for util lab exercise 1
 1 files changed, 1 insertions(+), 0 deletions(-)
$
```

您可以使用`git diff`命令跟踪您的更改。运行`git diff`将显示自上次提交以来对代码的更改，`git diff origin/util`将显示相对于初始xv6-labs-2020代码的更改。这里，_**origin/xv6-labs-2020**_ 是git分支的名称，它是包含您下载的初始代码分支。

- **构建并运行xv6**

```shell
$ make qemu
riscv64-unknown-elf-gcc    -c -o kernel/entry.o kernel/entry.S
riscv64-unknown-elf-gcc -Wall -Werror -O -fno-omit-frame-pointer -ggdb -DSOL_UTIL -MD -mcmodel=medany -ffreestanding -fno-common -nostdlib -mno-relax -I. -fno-stack-protector -fno-pie -no-pie   -c -o kernel/start.o kernel/start.c
...  
riscv64-unknown-elf-ld -z max-page-size=4096 -N -e main -Ttext 0 -o user/_zombie user/zombie.o user/ulib.o user/usys.o user/printf.o user/umalloc.o
riscv64-unknown-elf-objdump -S user/_zombie &gt; user/zombie.asm
riscv64-unknown-elf-objdump -t user/_zombie | sed '1,/SYMBOL TABLE/d; s/ .* / /; /^$/d' &gt; user/zombie.sym
mkfs/mkfs fs.img README  user/xargstest.sh user/_cat user/_echo user/_forktest user/_grep user/_init user/_kill user/_ln user/_ls user/_mkdir user/_rm user/_sh user/_stressfs user/_usertests user/_grind user/_wc user/_zombie 
nmeta 46 (boot, super, log blocks 30 inode blocks 13, bitmap blocks 1) blocks 954 total 1000
balloc: first 591 blocks have been allocated
balloc: write bitmap block at sector 45
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$ 
```

如果你在提示符下输入 `ls`，你会看到类似如下的输出:

```shell
$ ls
.              1 1 1024
..             1 1 1024
README         2 2 2059
xargstest.sh   2 3 93
cat            2 4 24256
echo           2 5 23080
forktest       2 6 13272
grep           2 7 27560
init           2 8 23816
kill           2 9 23024
ln             2 10 22880
ls             2 11 26448
mkdir          2 12 23176
rm             2 13 23160
sh             2 14 41976
stressfs       2 15 24016
usertests      2 16 148456
grind          2 17 38144
wc             2 18 25344
zombie         2 19 22408
console        3 20 0
```

这些是`mkfs`在初始文件系统中包含的文件；大多数是可以运行的程序。你刚刚跑了其中一个：`ls`。

xv6没有`ps`命令，但是如果您键入Ctrl-p，内核将打印每个进程的信息。如果现在尝试，您将看到两行：一行用于init，另一行用于sh。

退出 qemu : `Ctrl-a x`。

### sleep(难度：Easy)

> [!TIP|label:YOUR JOB] **实现xv6的UNIX程序**`sleep`**：您的**`sleep`**应该暂停到用户指定的计时数。一个滴答(tick)是由xv6内核定义的时间概念，即来自定时器芯片的两个中断之间的时间。您的解决方案应该在文件_user/sleep.c_中**

**提示：**

- 在你开始编码之前，请阅读《book-riscv-rev1》的第一章
    
- 看看其他的一些程序（如***/user/echo.c, /user/grep.c, /user/rm.c***）查看如何获取传递给程序的命令行参数
    
- 如果用户忘记传递参数，`sleep`应该打印一条错误信息
    
- 命令行参数作为字符串传递; 您可以使用`atoi`将其转换为数字（详见***/user/ulib.c***）
    
- 使用系统调用`sleep`
    
- 请参阅_**kernel/sysproc.c**_以获取实现`sleep`系统调用的xv6内核代码（查找`sys_sleep`），_**user/user.h**_提供了`sleep`的声明以便其他程序调用，用汇编程序编写的_**user/usys.S**_可以帮助`sleep`从用户区跳转到内核区。
    
- 确保`main`函数调用`exit()`以退出程序。
    
- 将你的`sleep`程序添加到_**Makefile**_中的`UPROGS`中；完成之后，`make qemu`将编译您的程序，并且您可以从xv6的shell运行它。
    
- 看看Kernighan和Ritchie编著的《C程序设计语言》（第二版）来了解C语言。
    

从xv6 shell运行程序：

```shell
$ make qemu
...
init: starting sh
$ sleep 10
(nothing happens for a little while)
$
```

如果程序在如上所示运行时暂停，则解决方案是正确的。运行`make grade`看看你是否真的通过了睡眠测试。

请注意，`make grade`运行所有测试，包括下面作业的测试。如果要对一项作业运行成绩测试，请键入（不要启动XV6，在外部终端下使用）：

```shell
$ ./grade-lab-util sleep
```

这将运行与`sleep`匹配的成绩测试。或者，您可以键入：

```shell
$ make GRADEFLAGS=sleep grade
```

效果是一样的。

### pingpong（难度：Easy）

> [!TIP|label:YOUR JOB] **编写一个使用UNIX系统调用的程序来在两个进程之间“ping-pong”一个字节，请使用两个管道，每个方向一个。父进程应该向子进程发送一个字节;子进程应该打印“`<pid>: received ping`”，其中`<pid>`是进程ID，并在管道中写入字节发送给父进程，然后退出;父级应该从读取从子进程而来的字节，打印“`<pid>: received pong`”，然后退出。您的解决方案应该在文件_user/pingpong.c_中。**

**提示：**

- 使用`pipe`来创造管道
    
- 使用`fork`创建子进程
    
- 使用`read`从管道中读取数据，并且使用`write`向管道中写入数据
    
- 使用`getpid`获取调用进程的pid
    
- 将程序加入到_**Makefile**_的`UPROGS`
    
- xv6上的用户程序有一组有限的可用库函数。您可以在_**user/user.h**_中看到可调用的程序列表；源代码（系统调用除外）位于_**user/ulib.c**_、_**user/printf.c**_和_**user/umalloc.c**_中。

运行程序应得到下面的输出

```shell
$ make qemu
...
init: starting sh
$ pingpong
4: received ping
3: received pong
$
```

如果您的程序在两个进程之间交换一个字节并产生如上所示的输出，那么您的解决方案是正确的。

### Primes(素数，难度：Moderate/Hard)

> [!TIP|label:YOUR JOB] **使用管道编写**`prime sieve`**(筛选素数)的并发版本。这个想法是由Unix管道的发明者Doug McIlroy提出的。请查看[这个网站](http://swtch.com/~rsc/thread/)(翻译在下面)，该网页中间的图片和周围的文字解释了如何做到这一点。您的解决方案应该在_user/primes.c_文件中。**

您的目标是使用`pipe`和`fork`来设置管道。第一个进程将数字2到35输入管道。对于每个素数，您将安排创建一个进程，该进程通过一个管道从其左邻居读取数据，并通过另一个管道向其右邻居写入数据。由于xv6的文件描述符和进程数量有限，因此第一个进程可以在35处停止。

**提示：**

- 请仔细关闭进程不需要的文件描述符，否则您的程序将在第一个进程达到35之前就会导致xv6系统资源不足。
    
- 一旦第一个进程达到35，它应该使用`wait`等待整个管道终止，包括所有子孙进程等等。因此，主`primes`进程应该只在打印完所有输出之后，并且在所有其他`primes`进程退出之后退出。
    
- 提示：当管道的`write`端关闭时，`read`返回零。
    
- 最简单的方法是直接将32位（4字节）int写入管道，而不是使用格式化的ASCII I/O。
    
- 您应该仅在需要时在管线中创建进程。
    
- 将程序添加到_**Makefile**_中的`UPROGS`
    

如果您的解决方案实现了基于管道的筛选并产生以下输出，则是正确的：

```shell
$ make qemu
...
init: starting sh
$ primes
prime 2
prime 3
prime 5
prime 7
prime 11
prime 13
prime 17
prime 19
prime 23
prime 29
prime 31
$
```

**参考资料翻译：**

> 考虑所有小于1000的素数的生成。Eratosthenes的筛选法可以通过执行以下伪代码的进程管线来模拟：

```c
p = get a number from left neighbor
print p
loop:
    n = get a number from left neighbor
    if (p does not divide n)
        send n to right neighbor
p = 从左邻居中获取一个数
print p
loop:
    n = 从左邻居中获取一个数
    if (n不能被p整除)
        将n发送给右邻居
```

[![img](https://github.com/duguosheng/6.S081-All-in-one/raw/main/labs/requirements/images/p1.png)](https://github.com/duguosheng/6.S081-All-in-one/blob/main/labs/requirements/images/p1.png)

> 生成进程可以将数字2、3、4、…、1000输入管道的左端：行中的第一个进程消除2的倍数，第二个进程消除3的倍数，第三个进程消除5的倍数，依此类推。

### find（难度：Moderate）

> [!TIP|label:YOUR JOB] **写一个简化版本的UNIX的`find`程序：查找目录树中具有特定名称的所有文件，你的解决方案应该放在_user/find.c_**

提示：

- 查看_**user/ls.c**_文件学习如何读取目录
- 使用递归允许`find`下降到子目录中
- 不要在“`.`”和“`..`”目录中递归
- 对文件系统的更改会在qemu的运行过程中一直保持；要获得一个干净的文件系统，请运行`make clean`，然后`make qemu`
- 你将会使用到C语言的字符串，要学习它请看《C程序设计语言》（K&R）,例如第5.5节
- 注意在C语言中不能像python一样使用“`==`”对字符串进行比较，而应当使用`strcmp()`
- 将程序加入到Makefile的`UPROGS`

如果你的程序输出下面的内容，那么它是正确的（当文件系统中包含文件_**b**_和_**a/b**_的时候）

```shell
$ make qemu
...
init: starting sh
$ echo > b
$ mkdir a
$ echo > a/b
$ find . b
./b
./a/b
$ 
```

### xargs（难度：Moderate）

> [!TIP|label:YOUR JOB] 编写一个简化版UNIX的`xargs`程序：它从标准输入中按行读取，并且为每一行执行一个命令，将行作为参数提供给命令。你的解决方案应该在_**user/xargs.c**_

下面的例子解释了`xargs`的行为

```shell
$ echo hello too | xargs echo bye
bye hello too
$
```

注意，这里的命令是`echo bye`，额外的参数是`hello too`，这样就组成了命令`echo bye hello too`，此命令输出`bye hello too`

请注意，UNIX上的`xargs`进行了优化，一次可以向该命令提供更多的参数。 我们不需要您进行此优化。 要使UNIX上的`xargs`表现出本实验所实现的方式，请将`-n`选项设置为1。例如

```shell
$ echo "1\n2" | xargs -n 1 echo line
line 1
line 2
$
```

**提示：**

- 使用`fork`和`exec`对每行输入调用命令，在父进程中使用`wait`等待子进程完成命令。
- 要读取单个输入行，请一次读取一个字符，直到出现换行符（'\n'）。
- _**kernel/param.h**_声明`MAXARG`，如果需要声明`argv`数组，这可能很有用。
- 将程序添加到_**Makefile**_中的`UPROGS`。
- 对文件系统的更改会在qemu的运行过程中保持不变；要获得一个干净的文件系统，请运行`make clean`，然后`make qemu`

`xargs`、`find`和`grep`结合得很好

```shell
$ find . b | xargs grep hello
```

将对“`.`”下面的目录中名为_**b**_的每个文件运行`grep hello`。

要测试您的`xargs`方案是否正确，请运行shell脚本_**xargstest.sh**_。如果您的解决方案产生以下输出，则是正确的：

```shell
$ make qemu
...
init: starting sh
$ sh < xargstest.sh
$ $ $ $ $ $ hello
hello
hello
$ $   
```

你可能不得不回去修复你的`find`程序中的bug。输出有许多`$`，因为xv6 shell没有意识到它正在处理来自文件而不是控制台的命令，并为文件中的每个命令打印`$`。

## 提交实验

**这就完成了实验**。确保你通过了所有的成绩测试。如果这个实验有问题，别忘了把你的答案写在_**answers-lab-name.txt**_中***。_**提交你的更改（包括**_answers-lab-name.txt***），然后在实验目录中键入`make handin`以提交实验。

**花费的时间**

创建一个命名为_**time.txt**_的新文件，并在其中输入一个整数，即您在实验室花费的小时数。不要忘记`git add`和`git commit`文件。

**提交**

你将使用 **[提交网站](https://6828.scripts.mit.edu/2020/handin.py/)** 提交作业。您需要从提交网站请求一次API密钥，然后才能提交任何作业或实验。

将最终更改提交到实验后，键入`make handin`以提交实验。

```shell
$ git commit -am "ready to submit my lab"
[util c2e3c8b] ready to submit my lab
 2 files changed, 18 insertions(+), 2 deletions(-)

$ make handin
tar: Removing leading `/' from member names
Get an API key for yourself by visiting https://6828.scripts.mit.edu/2020/handin.py/
Please enter your API key: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 79258  100   239  100 79019    853   275k --:--:-- --:--:-- --:--:--  276k
$
```

`make handin`将把你的API密钥存储在_**myapi.key**_中。如果需要更改API密钥，只需删除此文件并让`make handin`再次生成它(_**myapi.key**_不得包含换行符）。

如果你运行了`make handin`，并且你有未提交的更改或未跟踪的文件，则会看到类似于以下内容的输出：

```
 M hello.c
?? bar.c
?? foo.pyc
Untracked files will not be handed in.  Continue? [y/N]
```

检查上述行，确保跟踪了您的实验解决方案所需的所有文件，即以??开头的行中所显示的文件。您可以使用`git add filename`命令使git追踪创建的新文件。

如果`make handin`无法正常工作，请尝试使用curl或Git命令修复该问题。或者你可以运行`make tarball`。这将为您制作一个tar文件，然后您可以通过我们的web界面上传。

- 请运行“`make grade`”以确保您的代码通过所有测试
- 在运行“`make handin`”之前提交任何修改过的源代码`
- 您可以检查提交的状态，并在以下位置下载提交的代码：[https://6828.scripts.mit.edu/2020/handin.py/](https://6828.scripts.mit.edu/2020/handin.py/)

## 可选的挑战练习

- 编写一个`uptime`程序，使用`uptime`系统调用以滴答为单位打印计算机正常运行时间。（easy）
    
- 在`find`程序的名称匹配中支持正则表达式。_**grep.c**_对正则表达式有一些基本的支持。（easy）
    
- xv6 shell（_**user/sh.c**_）只是另一个用户程序，您可以对其进行改进。它是一个最小的shell，缺少建立在真实shell中的许多特性。例如，
    
    - 在处理**文件中的**shell命令时，将shell修改为不打印$（moderate）
    - 将shell修改为支持`wait`（easy）
    - 将shell修改为支持用“`;`”分隔的命令列表（moderate）
    - 通过实现左括号“`(`” 以及右括号“`)`”来修改shell以支持子shell（moderate）
    - 将shell修改为支持`tab`键补全（easy）
    - 修改shell使其支持命令历史记录（moderate）
    - 或者您希望shell执行的任何其他操作。
- 如果您非常雄心勃勃，可能需要修改内核以支持所需的内核特性；xv6支持的并不多。

# Lab2: system calls

在上一个实验室中，您使用系统调用编写了一些实用程序。在本实验室中，您将向xv6添加一些新的系统调用，这将帮助您了解它们是如何工作的，并使您了解xv6内核的一些内部结构。您将在以后的实验室中添加更多系统调用。

> [!WARNING|label:Attention] 在你开始写代码之前，请阅读xv6手册《book-riscv-rev1》的第2章、第4章的第4.3节和第4.4节以及相关源代码文件：
> 
> - 系统调用的用户空间代码在_**user/user.h**_和_**user/usys.pl**_中。
> - 内核空间代码是_**kernel/syscall.h**_、_**kernel/syscall.c**_。
> - 与进程相关的代码是_**kernel/proc.h**_和_**kernel/proc.c**_。

要开始本章实验，请将代码切换到**syscall**分支：

```shell
$ git fetch
$ git checkout syscall
$ make clean
```

如果运行`make grade`，您将看到测试分数的脚本无法执行`trace`和`sysinfotest`。您的工作是添加必要的系统调用和存根（stubs）以使它们工作。

## 实验任务
### System call tracing（moderate）

> [!TIP|label:YOUR JOB] 在本作业中，您将添加一个系统调用跟踪功能，该功能可能会在以后调试实验时对您有所帮助。您将创建一个新的`trace`系统调用来控制跟踪。它应该有一个参数，这个参数是一个整数“掩码”（mask），它的比特位指定要跟踪的系统调用。例如，要跟踪`fork`系统调用，程序调用`trace(1 << SYS_fork)`，其中`SYS_fork`是_**kernel/syscall.h**_中的系统调用编号。如果在掩码中设置了系统调用的编号，则必须修改xv6内核，以便在每个系统调用即将返回时打印出一行。该行应该包含进程id、系统调用的名称和返回值；您不需要打印系统调用参数。`trace`系统调用应启用对调用它的进程及其随后派生的任何子进程的跟踪，但不应影响其他进程。

我们提供了一个用户级程序版本的`trace`，它运行另一个启用了跟踪的程序（参见_**user/trace.c**_）。完成后，您应该看到如下输出：

```shell
$ trace 32 grep hello README
3: syscall read -> 1023
3: syscall read -> 966
3: syscall read -> 70
3: syscall read -> 0
$
$ trace 2147483647 grep hello README
4: syscall trace -> 0
4: syscall exec -> 3
4: syscall open -> 3
4: syscall read -> 1023
4: syscall read -> 966
4: syscall read -> 70
4: syscall read -> 0
4: syscall close -> 0
$
$ grep hello README
$
$ trace 2 usertests forkforkfork
usertests starting
test forkforkfork: 407: syscall fork -> 408
408: syscall fork -> 409
409: syscall fork -> 410
410: syscall fork -> 411
409: syscall fork -> 412
410: syscall fork -> 413
409: syscall fork -> 414
411: syscall fork -> 415
...
$   
```

在上面的第一个例子中，`trace`调用`grep`，仅跟踪了`read`系统调用。`32`是`1<<SYS_read`。在第二个示例中，`trace`在运行`grep`时跟踪所有系统调用；`2147483647`将所有31个低位置为1。在第三个示例中，程序没有被跟踪，因此没有打印跟踪输出。在第四个示例中，在`usertests`中测试的`forkforkfork`中所有子孙进程的`fork`系统调用都被追踪。如果程序的行为如上所示，则解决方案是正确的（尽管进程ID可能不同）

**提示：**

- 在_**Makefile**_的**UPROGS**中添加`$U/_trace`
- 运行`make qemu`，您将看到编译器无法编译_**user/trace.c**_，因为系统调用的用户空间存根还不存在：将系统调用的原型添加到_**user/user.h**_，存根添加到_**user/usys.pl**_，以及将系统调用编号添加到_**kernel/syscall.h**_，_**Makefile**_调用perl脚本_**user/usys.pl**_，它生成实际的系统调用存根_**user/usys.S**_，这个文件中的汇编代码使用RISC-V的`ecall`指令转换到内核。一旦修复了编译问题（_注：如果编译还未通过，尝试先`make clean`，再执行`make qemu`_），就运行`trace 32 grep hello README`；但由于您还没有在内核中实现系统调用，执行将失败。
- 在_**kernel/sysproc.c**_中添加一个`sys_trace()`函数，它通过将参数保存到`proc`结构体（请参见_**kernel/proc.h**_）里的一个新变量中来实现新的系统调用。从用户空间检索系统调用参数的函数在_**kernel/syscall.c**_中，您可以在_**kernel/sysproc.c**_中看到它们的使用示例。
- 修改`fork()`（请参阅_**kernel/proc.c**_）将跟踪掩码从父进程复制到子进程。
- 修改_**kernel/syscall.c**_中的`syscall()`函数以打印跟踪输出。您将需要添加一个系统调用名称数组以建立索引。

### Sysinfo（moderate）

> [!TIP|label:YOUR JOB] 在这个作业中，您将添加一个系统调用`sysinfo`，它收集有关正在运行的系统的信息。系统调用采用一个参数：一个指向`struct sysinfo`的指针（参见_**kernel/sysinfo.h**_）。内核应该填写这个结构的字段：`freemem`字段应该设置为空闲内存的字节数，`nproc`字段应该设置为`state`字段不为`UNUSED`的进程数。我们提供了一个测试程序`sysinfotest`；如果输出“**sysinfotest: OK**”则通过。

**提示：**

- 在_**Makefile**_的**UPROGS**中添加`$U/_sysinfotest`
- 当运行`make qemu`时，_**user/sysinfotest.c**_将会编译失败，遵循和上一个作业一样的步骤添加`sysinfo`系统调用。要在_**user/user.h**_中声明`sysinfo()`的原型，需要预先声明`struct sysinfo`的存在：

```c
struct sysinfo;
int sysinfo(struct sysinfo *);
```

一旦修复了编译问题，就运行`sysinfotest`；但由于您还没有在内核中实现系统调用，执行将失败。

- `sysinfo`需要将一个`struct sysinfo`复制回用户空间；请参阅`sys_fstat()`(_**kernel/sysfile.c**_)和`filestat()`(_**kernel/file.c**_)以获取如何使用`copyout()`执行此操作的示例。
- 要获取空闲内存量，请在_**kernel/kalloc.c**_中添加一个函数
- 要获取进程数，请在_**kernel/proc.c**_中添加一个函数

## 可选的挑战

- 打印所跟踪的系统调用的参数（easy）。
- 计算平均负载并通过`sysinfo`导出（moderate）。

# Lab3: page tables

在本实验室中，您将探索页表并对其进行修改，以简化将数据从用户空间复制到内核空间的函数。

> [!WARNING|label:Attention] 开始编码之前，请阅读xv6手册的第3章和相关文件：
> 
> - _**kernel/memlayout.h**_，它捕获了内存的布局。
> - _**kernel/vm.c**_，其中包含大多数虚拟内存（VM）代码。
> - _**kernel/kalloc.c**_，它包含分配和释放物理内存的代码。

要启动实验，请切换到pgtbl分支：

```shell
$ git fetch
$ git checkout pgtbl
$ make clean
```

### Print a page table (easy)

为了帮助您了解RISC-V页表，也许为了帮助将来的调试，您的第一个任务是编写一个打印页表内容的函数。

> [!TIP|label:YOUR JOB] 定义一个名为`vmprint()`的函数。它应当接收一个`pagetable_t`作为参数，并以下面描述的格式打印该页表。在`exec.c`中的`return argc`之前插入`if(p->pid==1) vmprint(p->pagetable)`，以打印第一个进程的页表。如果你通过了`pte printout`测试的`make grade`，你将获得此作业的满分。

现在，当您启动xv6时，它应该像这样打印输出来描述第一个进程刚刚完成`exec()`ing`init`时的页表：

```
page table 0x0000000087f6e000
..0: pte 0x0000000021fda801 pa 0x0000000087f6a000
.. ..0: pte 0x0000000021fda401 pa 0x0000000087f69000
.. .. ..0: pte 0x0000000021fdac1f pa 0x0000000087f6b000
.. .. ..1: pte 0x0000000021fda00f pa 0x0000000087f68000
.. .. ..2: pte 0x0000000021fd9c1f pa 0x0000000087f67000
..255: pte 0x0000000021fdb401 pa 0x0000000087f6d000
.. ..511: pte 0x0000000021fdb001 pa 0x0000000087f6c000
.. .. ..510: pte 0x0000000021fdd807 pa 0x0000000087f76000
.. .. ..511: pte 0x0000000020001c0b pa 0x0000000080007000
```

第一行显示`vmprint`的参数。之后的每行对应一个PTE，包含树中指向页表页的PTE。每个PTE行都有一些“`..`”的缩进表明它在树中的深度。每个PTE行显示其在页表页中的PTE索引、PTE比特位以及从PTE提取的物理地址。不要打印无效的PTE。在上面的示例中，顶级页表页具有条目0和255的映射。条目0的下一级只映射了索引0，该索引0的下一级映射了条目0、1和2。

您的代码可能会发出与上面显示的不同的物理地址。条目数和虚拟地址应相同。

**一些提示：**

- 你可以将`vmprint()`放在_**kernel/vm.c**_中
- 使用定义在_**kernel/riscv.h**_末尾处的宏
- 函数`freewalk`可能会对你有所启发
- 将`vmprint`的原型定义在_**kernel/defs.h**_中，这样你就可以在`exec.c`中调用它了
- 在你的`printf`调用中使用`%p`来打印像上面示例中的完成的64比特的十六进制PTE和地址

> [!NOTE|label:QUESTION] 根据文本中的图3-4解释`vmprint`的输出。page 0包含什么？page 2中是什么？在用户模式下运行时，进程是否可以读取/写入page 1映射的内存？

### A kernel page table per process (hard)

Xv6有一个单独的用于在内核中执行程序时的内核页表。内核页表直接映射（恒等映射）到物理地址，也就是说内核虚拟地址`x`映射到物理地址仍然是`x`。Xv6还为每个进程的用户地址空间提供了一个单独的页表，只包含该进程用户内存的映射，从虚拟地址0开始。因为内核页表不包含这些映射，所以用户地址在内核中无效。因此，当内核需要使用在系统调用中传递的用户指针（例如，传递给`write()`的缓冲区指针）时，内核必须首先将指针转换为物理地址。本节和下一节的目标是允许内核直接解引用用户指针。

> [!TIP|label:YOUR JOB] 你的第一项工作是修改内核来让每一个进程在内核中执行时使用它自己的内核页表的副本。修改`struct proc`来为每一个进程维护一个内核页表，修改调度程序使得切换进程时也切换内核页表。对于这个步骤，每个进程的内核页表都应当与现有的的全局内核页表完全一致。如果你的`usertests`程序正确运行了，那么你就通过了这个实验。

阅读本作业开头提到的章节和代码；了解虚拟内存代码的工作原理后，正确修改虚拟内存代码将更容易。页表设置中的错误可能会由于缺少映射而导致陷阱，可能会导致加载和存储影响到意料之外的物理页存页面，并且可能会导致执行来自错误内存页的指令。

**提示：**

- 在`struct proc`中为进程的内核页表增加一个字段
- 为一个新进程生成一个内核页表的合理方案是实现一个修改版的`kvminit`，这个版本中应当创造一个新的页表而不是修改`kernel_pagetable`。你将会考虑在`allocproc`中调用这个函数
- 确保每一个进程的内核页表都关于该进程的内核栈有一个映射。在未修改的XV6中，所有的内核栈都在`procinit`中设置。你将要把这个功能部分或全部的迁移到`allocproc`中
- 修改`scheduler()`来加载进程的内核页表到核心的`satp`寄存器(参阅`kvminithart`来获取启发)。不要忘记在调用完`w_satp()`后调用`sfence_vma()`
- 没有进程运行时`scheduler()`应当使用`kernel_pagetable`
- 在`freeproc`中释放一个进程的内核页表
- 你需要一种方法来释放页表，而不必释放叶子物理内存页面。
- 调式页表时，也许`vmprint`能派上用场
- 修改XV6本来的函数或新增函数都是允许的；你或许至少需要在_**kernel/vm.c**_和_**kernel/proc.c**_中这样做（但不要修改_**kernel/vmcopyin.c**_, _**kernel/stats.c**_, _**user/usertests.c**_, 和_**user/stats.c**_）
- 页表映射丢失很可能导致内核遭遇页面错误。这将导致打印一段包含`sepc=0x00000000XXXXXXXX`的错误提示。你可以在_**kernel/kernel.asm**_通过查询`XXXXXXXX`来定位错误。

### Simplify `copyin`/`copyinstr`（hard）

内核的`copyin`函数读取用户指针指向的内存。它通过将用户指针转换为内核可以直接解引用的物理地址来实现这一点。这个转换是通过在软件中遍历进程页表来执行的。在本部分的实验中，您的工作是将用户空间的映射添加到每个进程的内核页表（上一节中创建），以允许`copyin`（和相关的字符串函数`copyinstr`）直接解引用用户指针。

> [!TIP|label:YOUR JOB] 将定义在_**kernel/vm.c**_中的`copyin`的主题内容替换为对`copyin_new`的调用（在_**kernel/vmcopyin.c**_中定义）；对`copyinstr`和`copyinstr_new`执行相同的操作。为每个进程的内核页表添加用户地址映射，以便`copyin_new`和`copyinstr_new`工作。如果`usertests`正确运行并且所有`make grade`测试都通过，那么你就完成了此项作业。

此方案依赖于用户的虚拟地址范围不与内核用于自身指令和数据的虚拟地址范围重叠。Xv6使用从零开始的虚拟地址作为用户地址空间，幸运的是内核的内存从更高的地址开始。然而，这个方案将用户进程的最大大小限制为小于内核的最低虚拟地址。内核启动后，在XV6中该地址是`0xC000000`，即PLIC寄存器的地址；请参见_**kernel/vm.c**_中的`kvminit()`、_**kernel/memlayout.h**_和文中的图3-4。您需要修改xv6，以防止用户进程增长到超过PLIC的地址。

**一些提示：**

- 先用对`copyin_new`的调用替换`copyin()`，确保正常工作后再去修改`copyinstr`
- 在内核更改进程的用户映射的每一处，都以相同的方式更改进程的内核页表。包括`fork()`, `exec()`, 和`sbrk()`.
- 不要忘记在`userinit`的内核页表中包含第一个进程的用户页表
- 用户地址的PTE在进程的内核页表中需要什么权限？(在内核模式下，无法访问设置了`PTE_U`的页面）
- 别忘了上面提到的PLIC限制

Linux使用的技术与您已经实现的技术类似。直到几年前，许多内核在用户和内核空间中都为当前进程使用相同的自身进程页表，并为用户和内核地址进行映射以避免在用户和内核空间之间切换时必须切换页表。然而，这种设置允许边信道攻击，如Meltdown和Spectre。

> [!NOTE|label:QUESTION] 解释为什么在`copyin_new()`中需要第三个测试`srcva + len < srcva`：给出`srcva`和`len`值的例子，这样的值将使前两个测试为假（即它们不会导致返回-1），但是第三个测试为真 （导致返回-1）。

## 可选的挑战练习

- 使用超级页来减少页表中PTE的数量
- 扩展您的解决方案以支持尽可能大的用户程序；也就是说，消除用户程序小于PLIC的限制
- 取消映射用户进程的第一页，以便使对空指针的解引用将导致错误。用户文本段必须从非0处开始，例如4096

---
# Lab4: traps

本实验探索如何使用陷阱实现系统调用。您将首先使用栈做一个热身练习，然后实现一个用户级陷阱处理的示例。

> [!WARNING|label:Attention] 开始编码之前，请阅读xv6手册的第4章和相关源文件：
> 
> - _**kernel/trampoline.S**_：涉及从用户空间到内核空间再到内核空间的转换的程序集
> - _**kernel/trap.c**_：处理所有中断的代码

要启动实验，请切换到`traps`分支：

```shell
$ git fetch
$ git checkout traps
$ make clean
```

### RISC-V assembly (easy)

理解一点RISC-V汇编是很重要的，你应该在6.004中接触过。xv6仓库中有一个文件_**user/call.c**_。执行`make fs.img`编译它，并在_**user/call.asm**_中生成可读的汇编版本。

阅读_**call.asm**_中函数`g`、`f`和`main`的代码。RISC-V的使用手册在[参考页](https://pdos.csail.mit.edu/6.828/2020/reference.html)上。以下是您应该回答的一些问题（将答案存储在_**answers-traps.txt**_文件中）：

1. 哪些寄存器保存函数的参数？例如，在`main`对`printf`的调用中，哪个寄存器保存13？
2. `main`的汇编代码中对函数`f`的调用在哪里？对`g`的调用在哪里(提示：编译器可能会将函数内联）
3. `printf`函数位于哪个地址？
4. 在`main`中`printf`的`jalr`之后的寄存器`ra`中有什么值？
5. 运行以下代码。

```c
unsigned int i = 0x00646c72;
printf("H%x Wo%s", 57616, &i);
```

程序的输出是什么？这是将字节映射到字符的[ASCII码表](http://web.cs.mun.ca/~michael/c/ascii-table.html)。

输出取决于RISC-V小端存储的事实。如果RISC-V是大端存储，为了得到相同的输出，你会把`i`设置成什么？是否需要将`57616`更改为其他值？

[这里有一个小端和大端存储的描述](http://www.webopedia.com/TERM/b/big_endian.html)和一个[更异想天开的描述](http://www.networksorcery.com/enp/ien/ien137.txt)。

6. 在下面的代码中，“`y=`”之后将打印什么(注：答案不是一个特定的值）？为什么会发生这种情况？

```c
printf("x=%d y=%d", 3);
```

### Backtrace(moderate)

回溯(Backtrace)通常对于调试很有用：它是一个存放于栈上用于指示错误发生位置的函数调用列表。

在_**kernel/printf.c**_中实现名为`backtrace()`的函数。在`sys_sleep`中插入一个对此函数的调用，然后运行`bttest`，它将会调用`sys_sleep`。你的输出应该如下所示：

```shell
backtrace:
0x0000000080002cda
0x0000000080002bb6
0x0000000080002898
```

​ 在`bttest`退出qemu后。在你的终端：地址或许会稍有不同，但如果你运行`addr2line -e kernel/kernel`（或`riscv64-unknown-elf-addr2line -e kernel/kernel`），并将上面的地址剪切粘贴如下：

```shell
$ addr2line -e kernel/kernel
0x0000000080002de2
0x0000000080002f4a
0x0000000080002bfc
Ctrl-D
```

​ 你应该看到类似下面的输出：

```
kernel/sysproc.c:74
kernel/syscall.c:224
kernel/trap.c:85
```

​ 编译器向每一个栈帧中放置一个帧指针（frame pointer）保存调用者帧指针的地址。你的`backtrace`应当使用这些帧指针来遍历栈，并在每个栈帧中打印保存的返回地址。

**提示：**

- 在_**kernel/defs.h**_中添加`backtrace`的原型，那样你就能在`sys_sleep`中引用`backtrace`
    
- GCC编译器将当前正在执行的函数的帧指针保存在`s0`寄存器，将下面的函数添加到_**kernel/riscv.h**_
    

```c
static inline uint64
r_fp()
{
  uint64 x;
  asm volatile("mv %0, s0" : "=r" (x) );
  return x;
}
```

​ 并在`backtrace`中调用此函数来读取当前的帧指针。这个函数使用[内联汇编](https://gcc.gnu.org/onlinedocs/gcc/Using-Assembly-Language-with-C.html)来读取`s0`

- 这个[课堂笔记](https://pdos.csail.mit.edu/6.828/2020/lec/l-riscv-slides.pdf)中有张栈帧布局图。注意返回地址位于栈帧帧指针的固定偏移(-8)位置，并且保存的帧指针位于帧指针的固定偏移(-16)位置

[![img](https://github.com/duguosheng/6.S081-All-in-one/raw/main/labs/requirements/images/p2.png)](https://github.com/duguosheng/6.S081-All-in-one/blob/main/labs/requirements/images/p2.png)

- XV6在内核中以页面对齐的地址为每个栈分配一个页面。你可以通过`PGROUNDDOWN(fp)`和`PGROUNDUP(fp)`（参见_**kernel/riscv.h**_）来计算栈页面的顶部和底部地址。这些数字对于`backtrace`终止循环是有帮助的。

一旦你的`backtrace`能够运行，就在_**kernel/printf.c**_的`panic`中调用它，那样你就可以在`panic`发生时看到内核的`backtrace`。

### Alarm(Hard)

> [!TIP|label:YOUR JOB] 在这个练习中你将向XV6添加一个特性，在进程使用CPU的时间内，XV6定期向进程发出警报。这对于那些希望限制CPU时间消耗的受计算限制的进程，或者对于那些计算的同时执行某些周期性操作的进程可能很有用。更普遍的来说，你将实现用户级中断/故障处理程序的一种初级形式。例如，你可以在应用程序中使用类似的一些东西处理页面故障。如果你的解决方案通过了`alarmtest`和`usertests`就是正确的。

你应当添加一个新的`sigalarm(interval, handler)`系统调用，如果一个程序调用了`sigalarm(n, fn)`，那么每当程序消耗了CPU时间达到n个“滴答”，内核应当使应用程序函数`fn`被调用。当`fn`返回时，应用应当在它离开的地方恢复执行。在XV6中，一个滴答是一段相当任意的时间单元，取决于硬件计时器生成中断的频率。如果一个程序调用了`sigalarm(0, 0)`，系统应当停止生成周期性的报警调用。

你将在XV6的存储库中找到名为_**user/alarmtest.c**_的文件。将其添加到_**Makefile**_。注意：你必须添加了`sigalarm`和`sigreturn`系统调用后才能正确编译（往下看）。

`alarmtest`在`test0`中调用了`sigalarm(2, periodic)`来要求内核每隔两个滴答强制调用`periodic()`，然后旋转一段时间。你可以在_**user/alarmtest.asm**_中看到`alarmtest`的汇编代码，这或许会便于调试。当`alarmtest`产生如下输出并且`usertests`也能正常运行时，你的方案就是正确的：

```shell
$ alarmtest
test0 start
........alarm!
test0 passed
test1 start
...alarm!
..alarm!
...alarm!
..alarm!
...alarm!
..alarm!
...alarm!
..alarm!
...alarm!
..alarm!
test1 passed
test2 start
................alarm!
test2 passed
$ usertests
...
ALL TESTS PASSED
$
```

​当你完成后，你的方案也许仅有几行代码，但如何正确运行是一个棘手的问题。我们将使用原始存储库中的_**alarmtest.c**_版本测试您的代码。你可以修改_**alarmtest.c**_来帮助调试，但是要确保原来的`alarmtest`显示所有的测试都通过了。

### test0: invoke handler(调用处理程序)

首先修改内核以跳转到用户空间中的报警处理程序，这将导致`test0`打印“alarm!”。不用担心输出“alarm!”之后会发生什么；如果您的程序在打印“alarm！”后崩溃，对于目前来说也是正常的。以下是一些**提示**：

- 您需要修改_**Makefile**_以使_**alarmtest.c**_被编译为xv6用户程序。
    
- 放入_**user/user.h**_的正确声明是：
    
```c
int sigalarm(int ticks, void (*handler)());
int sigreturn(void);
```

- 更新_**user/usys.pl**_（此文件生成_**user/usys.S**_）、_**kernel/syscall.h**_和_**kernel/syscall.c**_以允许`alarmtest`调用`sigalarm`和`sigreturn`系统调用。
    
- 目前来说，你的`sys_sigreturn`系统调用返回应该是零。
    
- 你的`sys_sigalarm()`应该将报警间隔和指向处理程序函数的指针存储在`struct proc`的新字段中（位于_**kernel/proc.h**_）。
    
- 你也需要在`struct proc`新增一个新字段。用于跟踪自上一次调用（或直到下一次调用）到进程的报警处理程序间经历了多少滴答；您可以在_**proc.c**_的`allocproc()`中初始化`proc`字段。
    
- 每一个滴答声，硬件时钟就会强制一个中断，这个中断在_**kernel/trap.c**_中的`usertrap()`中处理。
    
- 如果产生了计时器中断，您只想操纵进程的报警滴答；你需要写类似下面的代码
    

```c
if(which_dev == 2) ...
```

- 仅当进程有未完成的计时器时才调用报警函数。请注意，用户报警函数的地址可能是0（例如，在_**user/alarmtest.asm**_中，`periodic`位于地址0）。
    
- 您需要修改`usertrap()`，以便当进程的报警间隔期满时，用户进程执行处理程序函数。当RISC-V上的陷阱返回到用户空间时，什么决定了用户空间代码恢复执行的指令地址？
    
- 如果您告诉qemu只使用一个CPU，那么使用gdb查看陷阱会更容易，这可以通过运行

```shell
make CPUS=1 qemu-gdb
```

- 如果`alarmtest`打印“alarm!”，则您已成功。

### test1/test2(): resume interrupted code(恢复被中断的代码)

`alarmtest`打印“alarm!”后，很可能会在`test0`或`test1`中崩溃，或者`alarmtest`（最后）打印“test1 failed”，或者`alarmtest`未打印“test1 passed”就退出。要解决此问题，必须确保完成报警处理程序后返回到用户程序最初被计时器中断的指令执行。必须确保寄存器内容恢复到中断时的值，以便用户程序在报警后可以不受干扰地继续运行。最后，您应该在每次报警计数器关闭后“重新配置”它，以便周期性地调用处理程序。

作为一个起始点，我们为您做了一个设计决策：用户报警处理程序需要在完成后调用`sigreturn`系统调用。请查看_**alarmtest.c**_中的`periodic`作为示例。这意味着您可以将代码添加到`usertrap`和`sys_sigreturn`中，这两个代码协同工作，以使用户进程在处理完警报后正确恢复。

**提示：**

- 您的解决方案将要求您保存和恢复寄存器——您需要保存和恢复哪些寄存器才能正确恢复中断的代码？(提示：会有很多）
    
- 当计时器关闭时，让`usertrap`在`struct proc`中保存足够的状态，以使`sigreturn`可以正确返回中断的用户代码。
    
- 防止对处理程序的重复调用——如果处理程序还没有返回，内核就不应该再次调用它。`test2`测试这个。
    
- 一旦通过`test0`、`test1`和`test2`，就运行`usertests`以确保没有破坏内核的任何其他部分。

## 可选的挑战练习

- 在`backtrace()`中打印函数的名称和行号，而不仅仅是数字化的地址。(hard)

---
# Lab5: xv6 lazy page allocation

操作系统可以使用页表硬件的技巧之一是延迟分配用户空间堆内存（lazy allocation of user-space heap memory）。Xv6应用程序使用`sbrk()`系统调用向内核请求堆内存。在我们给出的内核中，`sbrk()`分配物理内存并将其映射到进程的虚拟地址空间。内核为一个大请求分配和映射内存可能需要很长时间。例如，考虑由262144个4096字节的页组成的千兆字节；即使单独一个页面的分配开销很低，但合起来如此大的分配数量将不可忽视。此外，有些程序申请分配的内存比实际使用的要多（例如，实现稀疏数组），或者为了以后的不时之需而分配内存。为了让`sbrk()`在这些情况下更快地完成，复杂的内核会延迟分配用户内存。也就是说，`sbrk()`不分配物理内存，只是记住分配了哪些用户地址，并在用户页表中将这些地址标记为无效。当进程第一次尝试使用延迟分配中给定的页面时，CPU生成一个页面错误（page fault），内核通过分配物理内存、置零并添加映射来处理该错误。您将在这个实验室中向xv6添加这个延迟分配特性。

> [!WARNING|label:Attention] 在开始编码之前，请阅读xv6手册的第4章（特别是4.6），以及可能要修改的相关文件：
> 
> - _**kernel/trap.c**_
> - _**kernel/vm.c**_
> - _**kernel/sysproc.c**_

要启动实验，请切换到`lazy`分支：

```shell
$ git fetch
$ git checkout lazy
$ make clean
```

### Eliminate allocation from sbrk() (easy)

> [!TIP|label:YOUR JOB] 你的首项任务是删除`sbrk(n)`系统调用中的页面分配代码（位于_**sysproc.c**_中的函数`sys_sbrk()`）。`sbrk(n)`系统调用将进程的内存大小增加n个字节，然后返回新分配区域的开始部分（即旧的大小）。新的`sbrk(n)`应该只将进程的大小（`myproc()->sz`）增加n，然后返回旧的大小。它不应该分配内存——因此您应该删除对`growproc()`的调用（但是您仍然需要增加进程的大小！）。

试着猜猜这个修改的结果是什么：将会破坏什么？

进行此修改，启动xv6，并在shell中键入`echo hi`。你应该看到这样的输出：

```shell
init: starting sh
$ echo hi
usertrap(): unexpected scause 0x000000000000000f pid=3
            sepc=0x0000000000001258 stval=0x0000000000004008
va=0x0000000000004000 pte=0x0000000000000000
panic: uvmunmap: not mapped
```

“`usertrap(): …`”这条消息来自_**trap.c**_中的用户陷阱处理程序；它捕获了一个不知道如何处理的异常。请确保您了解发生此页面错误的原因。“`stval=0x0..04008`”表示导致页面错误的虚拟地址是`0x4008`。

### Lazy allocation (moderate)

> [!TIP|label:YOUR JOB] 修改_**trap.c**_中的代码以响应来自用户空间的页面错误，方法是新分配一个物理页面并映射到发生错误的地址，然后返回到用户空间，让进程继续执行。您应该在生成“`usertrap(): …`”消息的`printf`调用之前添加代码。你可以修改任何其他xv6内核代码，以使`echo hi`正常工作。

**提示：**

- 你可以在`usertrap()`中查看`r_scause()`的返回值是否为13或15来判断该错误是否为页面错误
- `stval`寄存器中保存了造成页面错误的虚拟地址，你可以通过`r_stval()`读取
- 参考_**vm.c**_中的`uvmalloc()`中的代码，那是一个`sbrk()`通过`growproc()`调用的函数。你将需要对`kalloc()`和`mappages()`进行调用
- 使用`PGROUNDDOWN(va)`将出错的虚拟地址向下舍入到页面边界
- 当前`uvmunmap()`会导致系统`panic`崩溃；请修改程序保证正常运行
- 如果内核崩溃，请在_**kernel/kernel.asm**_中查看`sepc`
- 使用pgtbl lab的`vmprint`函数打印页表的内容
- 如果您看到错误“incomplete type proc”，请include“spinlock.h”然后是“proc.h”。

如果一切正常，你的lazy allocation应该使`echo hi`正常运行。您应该至少有一个页面错误（因为延迟分配），也许有两个。

### Lazytests and Usertests (moderate)

我们为您提供了`lazytests`，这是一个xv6用户程序，它测试一些可能会给您的惰性内存分配器带来压力的特定情况。修改内核代码，使所有`lazytests`和`usertests`都通过。

- 处理`sbrk()`参数为负的情况。
- 如果某个进程在高于`sbrk()`分配的任何虚拟内存地址上出现页错误，则终止该进程。
- 在`fork()`中正确处理父到子内存拷贝。
- 处理这种情形：进程从`sbrk()`向系统调用（如`read`或`write`）传递有效地址，但尚未分配该地址的内存。
- 正确处理内存不足：如果在页面错误处理程序中执行`kalloc()`失败，则终止当前进程。
- 处理用户栈下面的无效页面上发生的错误。

如果内核通过`lazytests`和`usertests`，那么您的解决方案是可以接受的：

```shell
$ lazytests
lazytests starting
running test lazy alloc
test lazy alloc: OK
running test lazy unmap...
usertrap(): ...
test lazy unmap: OK
running test out of memory
usertrap(): ...
test out of memory: OK
ALL TESTS PASSED
$ usertests
...
ALL TESTS PASSED
$
```

## 可选的挑战练习

- 让延时分配协同上一个实验中简化版的`copyin`一起工作。

---
# Lab6: Copy-on-Write Fork for xv6

虚拟内存提供了一定程度的间接寻址：内核可以通过将PTE标记为无效或只读来拦截内存引用，从而导致页面错误，还可以通过修改PTE来更改地址的含义。在计算机系统中有一种说法，任何系统问题都可以用某种程度的抽象方法来解决。Lazy allocation实验中提供了一个例子。这个实验探索了另一个例子：写时复制分支（copy-on write fork）。

在开始本实验前，将仓库切换到cow分支

```shell
$ git fetch
$ git checkout cow
$ make clean
```

**问题**

xv6中的`fork()`系统调用将父进程的所有用户空间内存复制到子进程中。如果父进程较大，则复制可能需要很长时间。更糟糕的是，这项工作经常造成大量浪费；例如，子进程中的`fork()`后跟`exec()`将导致子进程丢弃复制的内存，而其中的大部分可能都从未使用过。另一方面，如果父子进程都使用一个页面，并且其中一个或两个对该页面有写操作，则确实需要复制。

**解决方案**

copy-on-write (COW) fork()的目标是推迟到子进程实际需要物理内存拷贝时再进行分配和复制物理内存页面。

COW fork()只为子进程创建一个页表，用户内存的PTE指向父进程的物理页。COW fork()将父进程和子进程中的所有用户PTE标记为不可写。当任一进程试图写入其中一个COW页时，CPU将强制产生页面错误。内核页面错误处理程序检测到这种情况将为出错进程分配一页物理内存，将原始页复制到新页中，并修改出错进程中的相关PTE指向新的页面，将PTE标记为可写。当页面错误处理程序返回时，用户进程将能够写入其页面副本。

COW fork()将使得释放用户内存的物理页面变得更加棘手。给定的物理页可能会被多个进程的页表引用，并且只有在最后一个引用消失时才应该被释放。

### Implement copy-on write (hard)

> [!TIP|label:YOUR JOB] 您的任务是在xv6内核中实现copy-on-write fork。如果修改后的内核同时成功执行`cowtest`和`usertests`程序就完成了。

为了帮助测试你的实现方案，我们提供了一个名为`cowtest`的xv6程序（源代码位于_**user/cowtest.c**_）。`cowtest`运行各种测试，但在未修改的xv6上，即使是第一个测试也会失败。因此，最初您将看到：

```shell
$ cowtest
simple: fork() failed
$ 
```

“simple”测试分配超过一半的可用物理内存，然后执行一系列的`fork()`。`fork`失败的原因是没有足够的可用物理内存来为子进程提供父进程内存的完整副本。

完成本实验后，内核应该通过`cowtest`和`usertests`中的所有测试。即：

```shell
$ cowtest
simple: ok
simple: ok
three: zombie!
ok
three: zombie!
ok
three: zombie!
ok
file: ok
ALL COW TESTS PASSED
$ usertests
...
ALL TESTS PASSED
$
```

**这是一个合理的攻克计划：**

1. 修改`uvmcopy()`将父进程的物理页映射到子进程，而不是分配新页。在子进程和父进程的PTE中清除`PTE_W`标志。
2. 修改`usertrap()`以识别页面错误。当COW页面出现页面错误时，使用`kalloc()`分配一个新页面，并将旧页面复制到新页面，然后将新页面添加到PTE中并设置`PTE_W`。
3. 确保每个物理页在最后一个PTE对它的引用撤销时被释放——而不是在此之前。这样做的一个好方法是为每个物理页保留引用该页面的用户页表数的“引用计数”。当`kalloc()`分配页时，将页的引用计数设置为1。当`fork`导致子进程共享页面时，增加页的引用计数；每当任何进程从其页表中删除页面时，减少页的引用计数。`kfree()`只应在引用计数为零时将页面放回空闲列表。可以将这些计数保存在一个固定大小的整型数组中。你必须制定一个如何索引数组以及如何选择数组大小的方案。例如，您可以用页的物理地址除以4096对数组进行索引，并为数组提供等同于_**kalloc.c**_中`kinit()`在空闲列表中放置的所有页面的最高物理地址的元素数。
4. 修改`copyout()`在遇到COW页面时使用与页面错误相同的方案。

**提示：**

- lazy page allocation实验可能已经让您熟悉了许多与copy-on-write相关的xv6内核代码。但是，您不应该将这个实验室建立在您的lazy allocation解决方案的基础上；相反，请按照上面的说明从一个新的xv6开始。
- 有一种可能很有用的方法来记录每个PTE是否是COW映射。您可以使用RISC-V PTE中的RSW（reserved for software，即为软件保留的）位来实现此目的。
- `usertests`检查`cowtest`不测试的场景，所以别忘两个测试都需要完全通过。
- _**kernel/riscv.h**_的末尾有一些有用的宏和页表标志位的定义。
- 如果出现COW页面错误并且没有可用内存，则应终止进程。

## 可选的挑战练习

- 修改xv6以同时支持lazy allocation和COW。
- 测量您的COW实现减少了多少xv6拷贝的字节数以及分配的物理页数。寻找并利用机会进一步减少这些数字。

---
# Lab7: Multithreading

本实验将使您熟悉多线程。您将在用户级线程包中实现线程之间的切换，使用多个线程来加速程序，并实现一个屏障。

> [!WARNING|label:Attention] 在编写代码之前，您应该确保已经阅读了xv6手册中的“第7章: 调度”，并研究了相应的代码。

要启动实验，请切换到thread分支：

```shell
$ git fetch
$ git checkout thread
$ make clean
```

### Uthread: switching between threads (moderate)

在本练习中，您将为用户级线程系统设计上下文切换机制，然后实现它。为了让您开始，您的xv6有两个文件：_**user/uthread.c**_和_**user/uthread_switch.S**_，以及一个规则：运行在_**Makefile**_中以构建`uthread`程序。_**uthread.c**_包含大多数用户级线程包，以及三个简单测试线程的代码。线程包缺少一些用于创建线程和在线程之间切换的代码。

> [!TIP|label:YOUR JOB] 您的工作是提出一个创建线程和保存/恢复寄存器以在线程之间切换的计划，并实现该计划。完成后，`make grade`应该表明您的解决方案通过了`uthread`测试。

完成后，在xv6上运行`uthread`时应该会看到以下输出（三个线程可能以不同的顺序启动）：

```shell
$ make qemu
...
$ uthread
thread_a started
thread_b started
thread_c started
thread_c 0
thread_a 0
thread_b 0
thread_c 1
thread_a 1
thread_b 1
...
thread_c 99
thread_a 99
thread_b 99
thread_c: exit after 100
thread_a: exit after 100
thread_b: exit after 100
thread_schedule: no runnable threads
$
```

该输出来自三个测试线程，每个线程都有一个循环，该循环打印一行，然后将CPU让出给其他线程。

然而在此时还没有上下文切换的代码，您将看不到任何输出。

您需要将代码添加到_**user/uthread.c**_中的`thread_create()`和`thread_schedule()`，以及_**user/uthread_switch.S**_中的`thread_switch`。一个目标是确保当`thread_schedule()`第一次运行给定线程时，该线程在自己的栈上执行传递给`thread_create()`的函数。另一个目标是确保`thread_switch`保存被切换线程的寄存器，恢复切换到线程的寄存器，并返回到后一个线程指令中最后停止的点。您必须决定保存/恢复寄存器的位置；修改`struct thread`以保存寄存器是一个很好的计划。您需要在`thread_schedule`中添加对`thread_switch`的调用；您可以将需要的任何参数传递给`thread_switch`，但目的是将线程从`t`切换到`next_thread`。

**提示：**

- `thread_switch`只需要保存/还原被调用方保存的寄存器（callee-save register，参见LEC5使用的文档《Calling Convention》）。为什么？
- 您可以在_**user/uthread.asm**_中看到`uthread`的汇编代码，这对于调试可能很方便。
- 这可能对于测试你的代码很有用，使用`riscv64-linux-gnu-gdb`的单步调试通过你的`thread_switch`，你可以按这种方法开始：

```shell
(gdb) file user/_uthread
Reading symbols from user/_uthread...
(gdb) b uthread.c:60
```

这将在_**uthread.c**_的第60行设置断点。断点可能会（也可能不会）在运行`uthread`之前触发。为什么会出现这种情况？

一旦您的xv6 shell运行，键入“`uthread`”，gdb将在第60行停止。现在您可以键入如下命令来检查`uthread`的状态：

`(gdb) p/x *next_thread`

使用“`x`”，您可以检查内存位置的内容：

`(gdb) x/x next_thread->stack`

您可以跳到`thread_switch` 的开头，如下：

`(gdb) b thread_switch`

`(gdb) c`

您可以使用以下方法单步执行汇编指令：

`(gdb) si`

gdb的在线文档在[这里](https://sourceware.org/gdb/current/onlinedocs/gdb/)。

### Using threads (moderate)

在本作业中，您将探索使用哈希表的线程和锁的并行编程。您应该在具有多个内核的真实Linux或MacOS计算机（不是xv6，不是qemu）上执行此任务。最新的笔记本电脑都有多核处理器。

这个作业使用UNIX的pthread线程库。您可以使用`man pthreads`在手册页面上找到关于它的信息，您可以在web上查看，例如[这里](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_lock.html)、[这里](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_mutex_init.html)和[这里](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_create.html)。

文件_**notxv6/ph.c**_包含一个简单的哈希表，如果单个线程使用，该哈希表是正确的，但是多个线程使用时，该哈希表是不正确的。在您的xv6主目录（可能是`~/xv6-labs-2020`）中，键入以下内容：

```shell
$ make ph
$ ./ph 1
```

请注意，要构建`ph`，_**Makefile**_使用操作系统的gcc，而不是6.S081的工具。`ph`的参数指定在哈希表上执行`put`和`get`操作的线程数。运行一段时间后，`ph 1`将产生与以下类似的输出：

```
100000 puts, 3.991 seconds, 25056 puts/second
0: 0 keys missing
100000 gets, 3.981 seconds, 25118 gets/second
```

您看到的数字可能与此示例输出的数字相差两倍或更多，这取决于您计算机的速度、是否有多个核心以及是否正在忙于做其他事情。

`ph`运行两个基准程序。首先，它通过调用`put()`将许多键添加到哈希表中，并以每秒为单位打印puts的接收速率。之后它使用`get()`从哈希表中获取键。它打印由于puts而应该在哈希表中但丢失的键的数量（在本例中为0），并以每秒为单位打印gets的接收数量。

通过给`ph`一个大于1的参数，可以告诉它同时从多个线程使用其哈希表。试试`ph 2`：

```shell
$ ./ph 2
100000 puts, 1.885 seconds, 53044 puts/second
1: 16579 keys missing
0: 16579 keys missing
200000 gets, 4.322 seconds, 46274 gets/second
```

这个`ph 2`输出的第一行表明，当两个线程同时向哈希表添加条目时，它们达到每秒53044次插入的总速率。这大约是运行`ph 1`的单线程速度的两倍。这是一个优秀的“并行加速”，大约达到了人们希望的2倍（即两倍数量的核心每单位时间产出两倍的工作）。

然而，声明`16579 keys missing`的两行表示散列表中本应存在的大量键不存在。也就是说，puts应该将这些键添加到哈希表中，但出现了一些问题。请看一下_**notxv6/ph.c**_，特别是`put()`和`insert()`。

> [!TIP|label:YOUR JOB] 为什么两个线程都丢失了键，而不是一个线程？确定可能导致键丢失的具有2个线程的事件序列。在_**answers-thread.txt**_中提交您的序列和简短解释。

Tip

为了避免这种事件序列，请在_**notxv6/ph.c**_中的`put`和`get`中插入`lock`和`unlock`语句，以便在两个线程中丢失的键数始终为0。相关的pthread调用包括：

- `pthread_mutex_t lock; // declare a lock`
- `pthread_mutex_init(&lock, NULL); // initialize the lock`
- `pthread_mutex_lock(&lock); // acquire lock`
- `pthread_mutex_unlock(&lock); // release lock`

当`make grade`说您的代码通过`ph_safe`测试时，您就完成了，该测试需要两个线程的键缺失数为0。在此时，`ph_fast`测试失败是正常的。

不要忘记调用`pthread_mutex_init()`。首先用1个线程测试代码，然后用2个线程测试代码。您主要需要测试：程序运行是否正确呢（即，您是否消除了丢失的键？）？与单线程版本相比，双线程版本是否实现了并行加速（即单位时间内的工作量更多）？

在某些情况下，并发`put()`在哈希表中读取或写入的内存中没有重叠，因此不需要锁来相互保护。您能否更改_**ph.c**_以利用这种情况为某些`put()`获得并行加速？提示：每个散列桶加一个锁怎么样？

> [!TIP|label:YOUR JOB] 修改代码，使某些`put`操作在保持正确性的同时并行运行。当`make grade`说你的代码通过了`ph_safe`和`ph_fast`测试时，你就完成了。`ph_fast`测试要求两个线程每秒产生的`put`数至少是一个线程的1.25倍。

### Barrier(moderate)

在本作业中，您将实现一个[屏障](http://en.wikipedia.org/wiki/Barrier_(computer_science))（Barrier）：应用程序中的一个点，所有参与的线程在此点上必须等待，直到所有其他参与线程也达到该点。您将使用pthread条件变量，这是一种序列协调技术，类似于xv6的`sleep`和`wakeup`。

您应该在真正的计算机（不是xv6，不是qemu）上完成此任务。

文件_**notxv6/barrier.c**_包含一个残缺的屏障实现。

```shell
$ make barrier
$ ./barrier 2
barrier: notxv6/barrier.c:42: thread: Assertion `i == t' failed.
```

2指定在屏障上同步的线程数（_**barrier.c**_中的`nthread`）。每个线程执行一个循环。在每次循环迭代中，线程都会调用`barrier()`，然后以随机微秒数休眠。如果一个线程在另一个线程到达屏障之前离开屏障将触发断言（assert）。期望的行为是每个线程在`barrier()`中阻塞，直到`nthreads`的所有线程都调用了`barrier()`。

> [!TIP|label:YOUR JOB] 您的目标是实现期望的屏障行为。除了在`ph`作业中看到的lock原语外，还需要以下新的pthread原语；详情请看[这里](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_cond_wait.html)和[这里](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread_cond_broadcast.html)。
> 
> - `// 在cond上进入睡眠，释放锁mutex，在醒来时重新获取`
> - `pthread_cond_wait(&cond, &mutex);`
> - `// 唤醒睡在cond的所有线程`
> - `pthread_cond_broadcast(&cond);`

确保您的方案通过`make grade`的`barrier`测试。

`pthread_cond_wait`在调用时释放`mutex`，并在返回前重新获取`mutex`。

我们已经为您提供了`barrier_init()`。您的工作是实现`barrier()`，这样panic就不会发生。我们为您定义了`struct barrier`；它的字段供您使用。

**有两个问题使您的任务变得复杂：**

- 你必须处理一系列的`barrier`调用，我们称每一连串的调用为一轮（round）。`bstate.round`记录当前轮数。每次当所有线程都到达屏障时，都应增加`bstate.round`。
- 您必须处理这样的情况：一个线程在其他线程退出`barrier`之前进入了下一轮循环。特别是，您在前后两轮中重复使用`bstate.nthread`变量。确保在前一轮仍在使用`bstate.nthread`时，离开`barrier`并循环运行的线程不会增加`bstate.nthread`。

使用一个、两个和两个以上的线程测试代码。

---
# Lab8: locks

在本实验中，您将获得重新设计代码以提高并行性的经验。多核机器上并行性差的一个常见症状是频繁的锁争用。提高并行性通常涉及更改数据结构和锁定策略以减少争用。您将对xv6内存分配器和块缓存执行此操作。

> [!WARNING|label:Attention] 在编写代码之前，请确保阅读xv6手册中的以下部分：
> 
> - 第6章：《锁》和相应的代码。
> - 第3.5节：《代码：物理内存分配》
> - 第8.1节至第8.3节：《概述》、《Buffer cache层》和《代码：Buffer cache》

要开始本实验，请将代码切换到`lock`分支

```shell
$ git fetch
$ git checkout lock
$ make clean
```

### Memory allocator(moderate)

程序_**user/kalloctest.c**_强调了xv6的内存分配器：三个进程增长和缩小地址空间，导致对`kalloc`和`kfree`的多次调用。`kalloc`和`kfree`获得`kmem.lock`。`kalloctest`打印（作为“#fetch-and-add”）在`acquire`中由于尝试获取另一个内核已经持有的锁而进行的循环迭代次数，如`kmem`锁和一些其他锁。`acquire`中的循环迭代次数是锁争用的粗略度量。完成实验前，`kalloctest`的输出与此类似：

```shell
$ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem: #fetch-and-add 83375 #acquire() 433015
lock: bcache: #fetch-and-add 0 #acquire() 1260
--- top 5 contended locks:
lock: kmem: #fetch-and-add 83375 #acquire() 433015
lock: proc: #fetch-and-add 23737 #acquire() 130718
lock: virtio_disk: #fetch-and-add 11159 #acquire() 114
lock: proc: #fetch-and-add 5937 #acquire() 130786
lock: proc: #fetch-and-add 4080 #acquire() 130786
tot= 83375
test1 FAIL
```

`acquire`为每个锁维护要获取该锁的`acquire`调用计数，以及`acquire`中循环尝试但未能设置锁的次数。`kalloctest`调用一个系统调用，使内核打印`kmem`和`bcache`锁（这是本实验的重点）以及5个最有具竞争的锁的计数。如果存在锁争用，则`acquire`循环迭代的次数将很大。系统调用返回`kmem`和`bcache`锁的循环迭代次数之和。

对于本实验，您必须使用具有多个内核的专用空载机器。如果你使用一台正在做其他事情的机器，`kalloctest`打印的计数将毫无意义。你可以使用专用的Athena 工作站或你自己的笔记本电脑，但不要使用拨号机。

`kalloctest`中锁争用的根本原因是`kalloc()`有一个空闲列表，由一个锁保护。要消除锁争用，您必须重新设计内存分配器，以避免使用单个锁和列表。基本思想是为每个CPU维护一个空闲列表，每个列表都有自己的锁。因为每个CPU将在不同的列表上运行，不同CPU上的分配和释放可以并行运行。主要的挑战将是处理一个CPU的空闲列表为空，而另一个CPU的列表有空闲内存的情况；在这种情况下，一个CPU必须“窃取”另一个CPU空闲列表的一部分。窃取可能会引入锁争用，但这种情况希望不会经常发生。

> [!TIP|label:YOUR JOB] 您的工作是实现每个CPU的空闲列表，并在CPU的空闲列表为空时进行窃取。所有锁的命名必须以“`kmem`”开头。也就是说，您应该为每个锁调用`initlock`，并传递一个以“`kmem`”开头的名称。运行`kalloctest`以查看您的实现是否减少了锁争用。要检查它是否仍然可以分配所有内存，请运行`usertests sbrkmuch`。您的输出将与下面所示的类似，在`kmem`锁上的争用总数将大大减少，尽管具体的数字会有所不同。确保`usertests`中的所有测试都通过。评分应该表明考试通过。

```shell
 $ kalloctest
start test1
test1 results:
--- lock kmem/bcache stats
lock: kmem: #fetch-and-add 0 #acquire() 42843
lock: kmem: #fetch-and-add 0 #acquire() 198674
lock: kmem: #fetch-and-add 0 #acquire() 191534
lock: bcache: #fetch-and-add 0 #acquire() 1242
--- top 5 contended locks:
lock: proc: #fetch-and-add 43861 #acquire() 117281
lock: virtio_disk: #fetch-and-add 5347 #acquire() 114
lock: proc: #fetch-and-add 4856 #acquire() 117312
lock: proc: #fetch-and-add 4168 #acquire() 117316
lock: proc: #fetch-and-add 2797 #acquire() 117266
tot= 0
test1 OK
start test2
total free number of pages: 32499 (out of 32768)
.....
test2 OK
$ usertests sbrkmuch
usertests starting
test sbrkmuch: OK
ALL TESTS PASSED
$ usertests
...
ALL TESTS PASSED
$ 
```

**提示：**

- 您可以使用_**kernel/param.h**_中的常量`NCPU`
- 让`freerange`将所有可用内存分配给运行`freerange`的CPU。
- 函数`cpuid`返回当前的核心编号，但只有在中断关闭时调用它并使用其结果才是安全的。您应该使用`push_off()`和`pop_off()`来关闭和打开中断。
- 看看_**kernel/sprintf.c**_中的`snprintf`函数，了解字符串如何进行格式化。尽管可以将所有锁命名为“`kmem`”。

### Buffer cache(hard)

这一半作业独立于前一半；不管你是否完成了前半部分，你都可以完成这半部分（并通过测试）。

如果多个进程密集地使用文件系统，它们可能会争夺`bcache.lock`，它保护_**kernel/bio.c**_中的磁盘块缓存。`bcachetest`创建多个进程，这些进程重复读取不同的文件，以便在`bcache.lock`上生成争用；（在完成本实验之前）其输出如下所示：

```shell
$ bcachetest
start test0
test0 results:
--- lock kmem/bcache stats
lock: kmem: #fetch-and-add 0 #acquire() 33035
lock: bcache: #fetch-and-add 16142 #acquire() 65978
--- top 5 contended locks:
lock: virtio_disk: #fetch-and-add 162870 #acquire() 1188
lock: proc: #fetch-and-add 51936 #acquire() 73732
lock: bcache: #fetch-and-add 16142 #acquire() 65978
lock: uart: #fetch-and-add 7505 #acquire() 117
lock: proc: #fetch-and-add 6937 #acquire() 73420
tot= 16142
test0: FAIL
start test1
test1 OK
```

您可能会看到不同的输出，但`bcache`锁的`acquire`循环迭代次数将很高。如果查看_**kernel/bio.c**_中的代码，您将看到`bcache.lock`保护已缓存的块缓冲区的列表、每个块缓冲区中的引用计数（`b->refcnt`）以及缓存块的标识（`b->dev`和`b->blockno`）。

> [!TIP|label:YOUR JOB] 修改块缓存，以便在运行`bcachetest`时，bcache（buffer cache的缩写）中所有锁的`acquire`循环迭代次数接近于零。理想情况下，块缓存中涉及的所有锁的计数总和应为零，但只要总和小于500就可以。修改`bget`和`brelse`，以便bcache中不同块的并发查找和释放不太可能在锁上发生冲突（例如，不必全部等待`bcache.lock`）。你必须保护每个块最多缓存一个副本的不变量。完成后，您的输出应该与下面显示的类似（尽管不完全相同）。确保`usertests`仍然通过。完成后，`make grade`应该通过所有测试。

```shell
$ bcachetest
start test0
test0 results:
--- lock kmem/bcache stats
lock: kmem: #fetch-and-add 0 #acquire() 32954
lock: kmem: #fetch-and-add 0 #acquire() 75
lock: kmem: #fetch-and-add 0 #acquire() 73
lock: bcache: #fetch-and-add 0 #acquire() 85
lock: bcache.bucket: #fetch-and-add 0 #acquire() 4159
lock: bcache.bucket: #fetch-and-add 0 #acquire() 2118
lock: bcache.bucket: #fetch-and-add 0 #acquire() 4274
lock: bcache.bucket: #fetch-and-add 0 #acquire() 4326
lock: bcache.bucket: #fetch-and-add 0 #acquire() 6334
lock: bcache.bucket: #fetch-and-add 0 #acquire() 6321
lock: bcache.bucket: #fetch-and-add 0 #acquire() 6704
lock: bcache.bucket: #fetch-and-add 0 #acquire() 6696
lock: bcache.bucket: #fetch-and-add 0 #acquire() 7757
lock: bcache.bucket: #fetch-and-add 0 #acquire() 6199
lock: bcache.bucket: #fetch-and-add 0 #acquire() 4136
lock: bcache.bucket: #fetch-and-add 0 #acquire() 4136
lock: bcache.bucket: #fetch-and-add 0 #acquire() 2123
--- top 5 contended locks:
lock: virtio_disk: #fetch-and-add 158235 #acquire() 1193
lock: proc: #fetch-and-add 117563 #acquire() 3708493
lock: proc: #fetch-and-add 65921 #acquire() 3710254
lock: proc: #fetch-and-add 44090 #acquire() 3708607
lock: proc: #fetch-and-add 43252 #acquire() 3708521
tot= 128
test0: OK
start test1
test1 OK
$ usertests
  ...
ALL TESTS PASSED
$
```

请将你所有的锁以“`bcache`”开头进行命名。也就是说，您应该为每个锁调用`initlock`，并传递一个以“`bcache`”开头的名称。

减少块缓存中的争用比`kalloc`更复杂，因为bcache缓冲区真正的在进程（以及CPU）之间共享。对于`kalloc`，可以通过给每个CPU设置自己的分配器来消除大部分争用；这对块缓存不起作用。我们建议您使用每个哈希桶都有一个锁的哈希表在缓存中查找块号。

在您的解决方案中，以下是一些存在锁冲突但可以接受的情形：

- 当两个进程同时使用相同的块号时。`bcachetest test0`始终不会这样做。
- 当两个进程同时在cache中未命中时，需要找到一个未使用的块进行替换。`bcachetest test0`始终不会这样做。
- 在你用来划分块和锁的方案中某些块可能会发生冲突，当两个进程同时使用冲突的块时。例如，如果两个进程使用的块，其块号散列到哈希表中相同的槽。`bcachetest test0`可能会执行此操作，具体取决于您的设计，但您应该尝试调整方案的细节以避免冲突（例如，更改哈希表的大小）。

`bcachetest`的`test1`使用的块比缓冲区更多，并且执行大量文件系统代码路径。

**提示：**

- 请阅读xv6手册中对块缓存的描述（第8.1-8.3节）。
- 可以使用固定数量的散列桶，而不动态调整哈希表的大小。使用素数个存储桶（例如13）来降低散列冲突的可能性。
- 在哈希表中搜索缓冲区并在找不到缓冲区时为该缓冲区分配条目必须是原子的。
- 删除保存了所有缓冲区的列表（`bcache.head`等），改为标记上次使用时间的时间戳缓冲区（即使用_**kernel/trap.c**_中的`ticks`）。通过此更改，`brelse`不需要获取bcache锁，并且`bget`可以根据时间戳选择最近使用最少的块。
- 可以在`bget`中串行化回收（即`bget`中的一部分：当缓存中的查找未命中时，它选择要复用的缓冲区）。
- 在某些情况下，您的解决方案可能需要持有两个锁；例如，在回收过程中，您可能需要持有bcache锁和每个bucket（散列桶）一个锁。确保避免死锁。
- 替换块时，您可能会将`struct buf`从一个bucket移动到另一个bucket，因为新块散列到不同的bucket。您可能会遇到一个棘手的情况：新块可能会散列到与旧块相同的bucket中。在这种情况下，请确保避免死锁。
- 一些调试技巧：实现bucket锁，但将全局`bcache.lock`的`acquire`/`release`保留在`bget`的开头/结尾，以串行化代码。一旦您确定它在没有竞争条件的情况下是正确的，请移除全局锁并处理并发性问题。您还可以运行`make CPUS=1 qemu`以使用一个内核进行测试。

## 可选的挑战练习

在buffer cache中进行无锁查找。提示：使用gcc的`__sync_*`函数。您如何证明自己的实现是正确的？

---
# Lab9: file system

在本实验室中，您将向xv6文件系统添加大型文件和符号链接。

> [!WARNING|label:Attention] 在编写代码之前，您应该阅读《xv6手册》中的《第八章：文件系统》，并学习相应的代码。

获取实验室的xv6源代码并切换到fs分支：

```shell
$ git fetch
$ git checkout fs
$ make clean
```

### Large files(moderate)

在本作业中，您将增加xv6文件的最大大小。目前，xv6文件限制为268个块或`268*BSIZE`字节（在xv6中`BSIZE`为1024）。此限制来自以下事实：一个xv6 inode包含12个“直接”块号和一个“间接”块号，“一级间接”块指一个最多可容纳256个块号的块，总共12+256=268个块。

`bigfile`命令可以创建最长的文件，并报告其大小：

```shell
$ bigfile
..
wrote 268 blocks
bigfile: file is too small
$
```

测试失败，因为`bigfile`希望能够创建一个包含65803个块的文件，但未修改的xv6将文件限制为268个块。

您将更改xv6文件系统代码，以支持每个inode中可包含256个一级间接块地址的“二级间接”块，每个一级间接块最多可以包含256个数据块地址。结果将是一个文件将能够包含多达65803个块，或256*256+256+11个块（11而不是12，因为我们将为二级间接块牺牲一个直接块号）。

#### 预备

`mkfs`程序创建xv6文件系统磁盘映像，并确定文件系统的总块数；此大小由_**kernel/param.h**_中的`FSSIZE`控制。您将看到，该实验室存储库中的`FSSIZE`设置为200000个块。您应该在`make`输出中看到来自`mkfs/mkfs`的以下输出：

```
nmeta 70 (boot, super, log blocks 30 inode blocks 13, bitmap blocks 25) blocks 199930 total 200000
```

这一行描述了`mkfs/mkfs`构建的文件系统：它有70个元数据块（用于描述文件系统的块）和199930个数据块，总计200000个块。

如果在实验期间的任何时候，您发现自己必须从头开始重建文件系统，您可以运行`make clean`，强制`make`重建_**fs.img**_。

#### 看什么

磁盘索引节点的格式由_**fs.h**_中的`struct dinode`定义。您应当尤其对`NDIRECT`、`NINDIRECT`、`MAXFILE`和`struct dinode`的`addrs[]`元素感兴趣。查看《XV6手册》中的图8.3，了解标准xv6索引结点的示意图。

在磁盘上查找文件数据的代码位于_**fs.c**_的`bmap()`中。看看它，确保你明白它在做什么。在读取和写入文件时都会调用`bmap()`。写入时，`bmap()`会根据需要分配新块以保存文件内容，如果需要，还会分配间接块以保存块地址。

`bmap()`处理两种类型的块编号。`bn`参数是一个“逻辑块号”——文件中相对于文件开头的块号。`ip->addrs[]`中的块号和`bread()`的参数都是磁盘块号。您可以将`bmap()`视为将文件的逻辑块号映射到磁盘块号。

#### 你的工作

修改`bmap()`，以便除了直接块和一级间接块之外，它还实现二级间接块。你只需要有11个直接块，而不是12个，为你的新的二级间接块腾出空间；不允许更改磁盘inode的大小。`ip->addrs[]`的前11个元素应该是直接块；第12个应该是一个一级间接块（与当前的一样）；13号应该是你的新二级间接块。当`bigfile`写入65803个块并成功运行`usertests`时，此练习完成：

```
$ bigfile
..................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
wrote 65803 blocks
done; ok
$ usertests
...
ALL TESTS PASSED
$ 
```

运行`bigfile`至少需要一分钟半的时间。

**提示：**

- 确保您理解`bmap()`。写出`ip->addrs[]`、间接块、二级间接块和它所指向的一级间接块以及数据块之间的关系图。确保您理解为什么添加二级间接块会将最大文件大小增加256*256个块（实际上要-1，因为您必须将直接块的数量减少一个）。
- 考虑如何使用逻辑块号索引二级间接块及其指向的间接块。
- 如果更改`NDIRECT`的定义，则可能必须更改_**file.h**_文件中`struct inode`中`addrs[]`的声明。确保`struct inode`和`struct dinode`在其`addrs[]`数组中具有相同数量的元素。
- 如果更改`NDIRECT`的定义，请确保创建一个新的_**fs.img**_，因为`mkfs`使用`NDIRECT`构建文件系统。
- 如果您的文件系统进入坏状态，可能是由于崩溃，请删除_**fs.img**_（从Unix而不是xv6执行此操作）。`make`将为您构建一个新的干净文件系统映像。
- 别忘了把你`bread()`的每一个块都`brelse()`。
- 您应该仅根据需要分配间接块和二级间接块，就像原始的`bmap()`。
- 确保`itrunc`释放文件的所有块，包括二级间接块。

### Symbolic links(moderate)

在本练习中，您将向xv6添加符号链接。符号链接（或软链接）是指按路径名链接的文件；当一个符号链接打开时，内核跟随该链接指向引用的文件。符号链接类似于硬链接，但硬链接仅限于指向同一磁盘上的文件，而符号链接可以跨磁盘设备。尽管xv6不支持多个设备，但实现此系统调用是了解路径名查找工作原理的一个很好的练习。

#### 你的工作

> [!TIP|label:YOUR JOB] 您将实现`symlink(char *target, char *path)`系统调用，该调用在引用由`target`命名的文件的路径处创建一个新的符号链接。有关更多信息，请参阅`symlink`手册页（注：执行`man symlink`）。要进行测试，请将`symlinktest`添加到_**Makefile**_并运行它。当测试产生以下输出（包括`usertests`运行成功）时，您就完成本作业了。

```shell
$ symlinktest
Start: test symlinks
test symlinks: ok
Start: test concurrent symlinks
test concurrent symlinks: ok
$ usertests
...
ALL TESTS PASSED
$ 
```

**提示：**

- 首先，为`symlink`创建一个新的系统调用号，在_**user/usys.pl**_、_**user/user.h**_中添加一个条目，并在_**kernel/sysfile.c**_中实现一个空的`sys_symlink`。
- 向_**kernel/stat.h**_添加新的文件类型（`T_SYMLINK`）以表示符号链接。
- 在k_**ernel/fcntl.h**_中添加一个新标志（`O_NOFOLLOW`），该标志可用于`open`系统调用。请注意，传递给`open`的标志使用按位或运算符组合，因此新标志不应与任何现有标志重叠。一旦将_**user/symlinktest.c**_添加到_**Makefile**_中，您就可以编译它。
- 实现`symlink(target, path)`系统调用，以在`path`处创建一个新的指向`target`的符号链接。请注意，系统调用的成功不需要`target`已经存在。您需要选择存储符号链接目标路径的位置，例如在inode的数据块中。`symlink`应返回一个表示成功（0）或失败（-1）的整数，类似于`link`和`unlink`。
- 修改`open`系统调用以处理路径指向符号链接的情况。如果文件不存在，则打开必须失败。当进程向`open`传递`O_NOFOLLOW`标志时，`open`应打开符号链接（而不是跟随符号链接）。
- 如果链接文件也是符号链接，则必须递归地跟随它，直到到达非链接文件为止。如果链接形成循环，则必须返回错误代码。你可以通过以下方式估算存在循环：通过在链接深度达到某个阈值（例如10）时返回错误代码。
- 其他系统调用（如`link`和`unlink`）不得跟随符号链接；这些系统调用对符号链接本身进行操作。
- 您不必处理指向此实验的目录的符号链接。

## 可选的挑战练习

实现三级间接块

---
# Lab10: mmap

### mmap(hard)

`mmap`和`munmap`系统调用允许UNIX程序对其地址空间进行详细控制。它们可用于在进程之间共享内存，将文件映射到进程地址空间，并作为用户级页面错误方案的一部分，如本课程中讨论的垃圾收集算法。在本实验室中，您将把`mmap`和`munmap`添加到xv6中，重点关注内存映射文件（memory-mapped files）。

获取实验室的xv6源代码并切换到`mmap`分支：

```shell
$ git fetch
$ git checkout mmap
$ make clean
```

手册页面（运行`man 2 mmap`）显示了`mmap`的以下声明：

```c
void *mmap(void *addr, size_t length, int prot, int flags,
           int fd, off_t offset);
```

可以通过多种方式调用`mmap`，但本实验只需要与内存映射文件相关的功能子集。您可以假设`addr`始终为零，这意味着内核应该决定映射文件的虚拟地址。`mmap`返回该地址，如果失败则返回`0xffffffffffffffff`。`length`是要映射的字节数；它可能与文件的长度不同。`prot`指示内存是否应映射为可读、可写，以及/或者可执行的；您可以认为`prot`是`PROT_READ`或`PROT_WRITE`或两者兼有。`flags`要么是`MAP_SHARED`（映射内存的修改应写回文件），要么是`MAP_PRIVATE`（映射内存的修改不应写回文件）。您不必在`flags`中实现任何其他位。`fd`是要映射的文件的打开文件描述符。可以假定`offset`为零（它是要映射的文件的起点）。

允许进程映射同一个`MAP_SHARED`文件而不共享物理页面。

`munmap(addr, length)`应删除指定地址范围内的`mmap`映射。如果进程修改了内存并将其映射为`MAP_SHARED`，则应首先将修改写入文件。`munmap`调用可能只覆盖`mmap`区域的一部分，但您可以认为它取消映射的位置要么在区域起始位置，要么在区域结束位置，要么就是整个区域(但不会在区域中间“打洞”)。

> [!TIP|label:YOUR JOB] 您应该实现足够的`mmap`和`munmap`功能，以使`mmaptest`测试程序正常工作。如果`mmaptest`不会用到某个`mmap`的特性，则不需要实现该特性。

完成后，您应该会看到以下输出：

```shell
$ mmaptest
mmap_test starting
test mmap f
test mmap f: OK
test mmap private
test mmap private: OK
test mmap read-only
test mmap read-only: OK
test mmap read/write
test mmap read/write: OK
test mmap dirty
test mmap dirty: OK
test not-mapped unmap
test not-mapped unmap: OK
test mmap two files
test mmap two files: OK
mmap_test: ALL OK
fork_test starting
fork_test OK
mmaptest: all tests succeeded
$ usertests
usertests starting
...
ALL TESTS PASSED
$ 
```

**提示：**

- 首先，向`UPROGS`添加`_mmaptest`，以及`mmap`和`munmap`系统调用，以便让_**user/mmaptest.c**_进行编译。现在，只需从`mmap`和`munmap`返回错误。我们在_**kernel/fcntl.h**_中为您定义了`PROT_READ`等。运行`mmaptest`，它将在第一次`mmap`调用时失败。
- 惰性地填写页表，以响应页错误。也就是说，`mmap`不应该分配物理内存或读取文件。相反，在`usertrap`中（或由`usertrap`调用）的页面错误处理代码中执行此操作，就像在lazy page allocation实验中一样。惰性分配的原因是确保大文件的`mmap`是快速的，并且比物理内存大的文件的`mmap`是可能的。
- 跟踪`mmap`为每个进程映射的内容。定义与第15课中描述的VMA（虚拟内存区域）对应的结构体，记录`mmap`创建的虚拟内存范围的地址、长度、权限、文件等。由于xv6内核中没有内存分配器，因此可以声明一个固定大小的VMA数组，并根据需要从该数组进行分配。大小为16应该就足够了。
- 实现`mmap`：在进程的地址空间中找到一个未使用的区域来映射文件，并将VMA添加到进程的映射区域表中。VMA应该包含指向映射文件对应`struct file`的指针；`mmap`应该增加文件的引用计数，以便在文件关闭时结构体不会消失（提示：请参阅`filedup`）。运行`mmaptest`：第一次`mmap`应该成功，但是第一次访问被`mmap`的内存将导致页面错误并终止`mmaptest`。
- 添加代码以导致在`mmap`的区域中产生页面错误，从而分配一页物理内存，将4096字节的相关文件读入该页面，并将其映射到用户地址空间。使用`readi`读取文件，它接受一个偏移量参数，在该偏移处读取文件（但必须lock/unlock传递给`readi`的索引结点）。不要忘记在页面上正确设置权限。运行`mmaptest`；它应该到达第一个`munmap`。
- 实现`munmap`：找到地址范围的VMA并取消映射指定页面（提示：使用`uvmunmap`）。如果`munmap`删除了先前`mmap`的所有页面，它应该减少相应`struct file`的引用计数。如果未映射的页面已被修改，并且文件已映射到`MAP_SHARED`，请将页面写回该文件。查看`filewrite`以获得灵感。
- 理想情况下，您的实现将只写回程序实际修改的`MAP_SHARED`页面。RISC-V PTE中的脏位（`D`）表示是否已写入页面。但是，`mmaptest`不检查非脏页是否没有回写；因此，您可以不用看`D`位就写回页面。
- 修改`exit`将进程的已映射区域取消映射，就像调用了`munmap`一样。运行`mmaptest`；`mmap_test`应该通过，但可能不会通过`fork_test`。
- 修改`fork`以确保子对象具有与父对象相同的映射区域。不要忘记增加VMA的`struct file`的引用计数。在子进程的页面错误处理程序中，可以分配新的物理页面，而不是与父级共享页面。后者会更酷，但需要更多的实施工作。运行`mmaptest`；它应该通过`mmap_test`和`fork_test`。

运行`usertests`以确保一切正常。

---
# Lab11: Network

在本实验室中，您将为网络接口卡（NIC）编写一个xv6设备驱动程序。

获取xv6实验的源代码并切换到`net`分支：

```shell
$ git fetch
$ git checkout net
$ make clean
```

#### 背景

Tip

在编写代码之前，您可能会发现阅读xv6手册中的《第5章：中断和设备驱动》很有帮助。

您将使用名为E1000的网络设备来处理网络通信。对于xv6（以及您编写的驱动程序），E1000看起来像是连接到真正以太网局域网（LAN）的真正硬件。事实上，用于与您的驱动程序对话的E1000是qemu提供的模拟，连接到的LAN也由qemu模拟。在这个模拟LAN上，xv6（“来宾”）的IP地址为10.0.2.15。Qemu还安排运行Qemu的计算机出现在IP地址为10.0.2.2的LAN上。当xv6使用E1000将数据包发送到10.0.2.2时，qemu会将数据包发送到运行qemu的（真实）计算机上的相应应用程序（“主机”）。

您将使用QEMU的“用户模式网络栈（user-mode network stack）”。[QEMU的文档](https://www.qemu.org/docs/master/system/net.html#using-the-user-mode-network-stack)中有更多关于用户模式栈的内容。我们已经更新了_**Makefile**_以启用QEMU的用户模式网络栈和E1000网卡。

_**Makefile**_将QEMU配置为将所有传入和传出数据包记录到实验目录中的_**packets.pcap**_文件中。查看这些记录可能有助于确认xv6正在发送和接收您期望的数据包。要显示记录的数据包，请执行以下操作：

```
tcpdump -XXnr packets.pcap
```

我们已将一些文件添加到本实验的xv6存储库中。_**kernel/e1000.c**_文件包含E1000的初始化代码以及用于发送和接收数据包的空函数，您将填写这些函数。_**kernel/e1000_dev.h**_包含E1000定义的寄存器和标志位的定义，并在[《英特尔E1000软件开发人员手册》](https://pdos.csail.mit.edu/6.828/2020/readings/8254x_GBe_SDM.pdf)中进行了描述。_**kernel/net.c**_和_**kernel/net.h**_包含一个实现IP、UDP和ARP协议的简单网络栈。这些文件还包含用于保存数据包的灵活数据结构（称为`mbuf`）的代码。最后，_**kernel/pci.c**_包含在xv6引导时在PCI总线上搜索E1000卡的代码。

#### 你的工作(hard)

> [!TIP|label:YOUR JOB] 您的工作是在_**kernel/e1000.c**_中完成`e1000_transmit()`和`e1000_recv()`，以便驱动程序可以发送和接收数据包。当`make grade`表示您的解决方案通过了所有测试时，您就完成了。

Tip

在编写代码时，您会发现自己参考了[《E1000软件开发人员手册》](https://pdos.csail.mit.edu/6.828/2020/readings/8254x_GBe_SDM.pdf)。以下部分可能特别有用：

- Section 2是必不可少的，它概述了整个设备。
- Section 3.2概述了数据包接收。
- Section 3.3与Section 3.4一起概述了数据包传输。
- Section 13概述了E1000使用的寄存器。
- Section 14可能会帮助您理解我们提供的init代码。

浏览[《E1000软件开发人员手册》](https://pdos.csail.mit.edu/6.828/2020/readings/8254x_GBe_SDM.pdf)。本手册涵盖了几个密切相关的以太网控制器。QEMU模拟82540EM。现在浏览第2章，了解该设备。要编写驱动程序，您需要熟悉第3章和第14章以及第4.1节（虽然不包括4.1的子节）。你还需要参考第13章。其他章节主要介绍你的驱动程序不必与之交互的E1000组件。一开始不要担心细节；只需了解文档的结构，就可以在以后找到内容。E1000具有许多高级功能，其中大部分您可以忽略。完成这个实验只需要一小部分基本功能。

我们在_**e1000.c**_中提供的`e1000_init()`函数将E1000配置为读取要从RAM传输的数据包，并将接收到的数据包写入RAM。这种技术称为DMA，用于直接内存访问，指的是E1000硬件直接向RAM写入和读取数据包。

由于数据包突发到达的速度可能快于驱动程序处理数据包的速度，因此`e1000_init()`为E1000提供了多个缓冲区，E1000可以将数据包写入其中。E1000要求这些缓冲区由RAM中的“描述符”数组描述；每个描述符在RAM中都包含一个地址，E1000可以在其中写入接收到的数据包。`struct rx_desc`描述描述符格式。描述符数组称为接收环或接收队列。它是一个圆环，在这个意义上，当网卡或驱动程序到达队列的末尾时，它会绕回到数组的开头。`e1000_init()`使用`mbufalloc()`为要进行DMA的E1000分配`mbuf`数据包缓冲区。此外还有一个传输环，驱动程序将需要E1000发送的数据包放入其中。`e1000_init()`将两个环的大小配置为`RX_RING_SIZE`和`TX_RING_SIZE`。

当_**net.c**_中的网络栈需要发送数据包时，它会调用`e1000_transmit()`，并使用一个保存要发送的数据包的`mbuf`作为参数。传输代码必须在TX（传输）环的描述符中放置指向数据包数据的指针。`struct tx_desc`描述了描述符的格式。您需要确保每个`mbuf`最终被释放，但只能在E1000完成数据包传输之后（E1000在描述符中设置`E1000_TXD_STAT_DD`位以指示此情况）。

当当E1000从以太网接收到每个包时，它首先将包DMA到下一个RX(接收)环描述符指向的`mbuf`，然后产生一个中断。`e1000_recv()`代码必须扫描RX环，并通过调用`net_rx()`将每个新数据包的`mbuf`发送到网络栈（在_**net.c**_中）。然后，您需要分配一个新的`mbuf`并将其放入描述符中，以便当E1000再次到达RX环中的该点时，它会找到一个新的缓冲区，以便DMA新数据包。

除了在RAM中读取和写入描述符环外，您的驱动程序还需要通过其内存映射控制寄存器与E1000交互，以检测接收到数据包何时可用，并通知E1000驱动程序已经用要发送的数据包填充了一些TX描述符。全局变量`regs`包含指向E1000第一个控制寄存器的指针；您的驱动程序可以通过将`regs`索引为数组来获取其他寄存器。您需要特别使用索引`E1000_RDT`和`E1000_TDT`。

要测试驱动程序，请在一个窗口中运行`make server`，在另一个窗口中运行`make qemu`，然后在xv6中运行`nettests`。`nettests`中的第一个测试尝试将UDP数据包发送到主机操作系统，地址是`make server`运行的程序。如果您还没有完成实验，E1000驱动程序实际上不会发送数据包，也不会发生什么事情。

完成实验后，E1000驱动程序将发送数据包，qemu将其发送到主机，`make server`将看到它并发送响应数据包，然后E1000驱动程序和`nettests`将看到响应数据包。但是，在主机发送应答之前，它会向xv6发送一个“ARP”请求包，以找出其48位以太网地址，并期望xv6以ARP应答进行响应。一旦您完成了对E1000驱动程序的工作，_**kernel/net.c**_就会处理这个问题。如果一切顺利，`nettests`将打印`testing ping: OK`，`make server`将打印`a message from xv6!`。

`tcpdump -XXnr packets.pcap`应该生成这样的输出:

```
reading from file packets.pcap, link-type EN10MB (Ethernet)
15:27:40.861988 IP 10.0.2.15.2000 > 10.0.2.2.25603: UDP, length 19
        0x0000:  ffff ffff ffff 5254 0012 3456 0800 4500  ......RT..4V..E.
        0x0010:  002f 0000 0000 6411 3eae 0a00 020f 0a00  ./....d.>.......
        0x0020:  0202 07d0 6403 001b 0000 6120 6d65 7373  ....d.....a.mess
        0x0030:  6167 6520 6672 6f6d 2078 7636 21         age.from.xv6!
15:27:40.862370 ARP, Request who-has 10.0.2.15 tell 10.0.2.2, length 28
        0x0000:  ffff ffff ffff 5255 0a00 0202 0806 0001  ......RU........
        0x0010:  0800 0604 0001 5255 0a00 0202 0a00 0202  ......RU........
        0x0020:  0000 0000 0000 0a00 020f                 ..........
15:27:40.862844 ARP, Reply 10.0.2.15 is-at 52:54:00:12:34:56, length 28
        0x0000:  ffff ffff ffff 5254 0012 3456 0806 0001  ......RT..4V....
        0x0010:  0800 0604 0002 5254 0012 3456 0a00 020f  ......RT..4V....
        0x0020:  5255 0a00 0202 0a00 0202                 RU........
15:27:40.863036 IP 10.0.2.2.25603 > 10.0.2.15.2000: UDP, length 17
        0x0000:  5254 0012 3456 5255 0a00 0202 0800 4500  RT..4VRU......E.
        0x0010:  002d 0000 0000 4011 62b0 0a00 0202 0a00  .-....@.b.......
        0x0020:  020f 6403 07d0 0019 3406 7468 6973 2069  ..d.....4.this.i
        0x0030:  7320 7468 6520 686f 7374 21              s.the.host!
```

您的输出看起来会有些不同，但它应该包含字符串“ARP, Request”，“ARP, Reply”，“UDP”，“a.message.from.xv6”和“this.is.the.host”。

`nettests`执行一些其他测试，最终通过（真实的）互联网将DNS请求发送到谷歌的一个名称服务器。您应该确保您的代码通过所有这些测试，然后您应该看到以下输出：

```shell
$ nettests
nettests running on port 25603
testing ping: OK
testing single-process pings: OK
testing multi-process pings: OK
testing DNS
DNS arecord for pdos.csail.mit.edu. is 128.52.129.126
DNS OK
all tests passed.
```

您应该确保`make grade`同意您的解决方案通过。
#### 提示

首先，将打印语句添加到`e1000_transmit()`和`e1000_recv()`，然后运行`make server`和（在xv6中）`nettests`。您应该从打印语句中看到，`nettests`生成对`e1000_transmit`的调用。

**实现`e1000_transmit`的一些提示：**

- 首先，通过读取`E1000_TDT`控制寄存器，向E1000询问等待下一个数据包的TX环索引。
- 然后检查环是否溢出。如果`E1000_TXD_STAT_DD`未在`E1000_TDT`索引的描述符中设置，则E1000尚未完成先前相应的传输请求，因此返回错误。
- 否则，使用`mbuffree()`释放从该描述符传输的最后一个`mbuf`（如果有）。
- 然后填写描述符。`m->head`指向内存中数据包的内容，`m->len`是数据包的长度。设置必要的cmd标志（请参阅E1000手册的第3.3节），并保存指向`mbuf`的指针，以便稍后释放。
- 最后，通过将一加到`E1000_TDT再对TX_RING_SIZE`取模来更新环位置。
- 如果`e1000_transmit()`成功地将`mbuf`添加到环中，则返回0。如果失败（例如，没有可用的描述符来传输`mbuf`），则返回-1，以便调用方知道应该释放`mbuf`。

**实现`e1000_recv`的一些提示：**

- 首先通过提取`E1000_RDT`控制寄存器并加一对`RX_RING_SIZE`取模，向E1000询问下一个等待接收数据包（如果有）所在的环索引。
- 然后通过检查描述符`status`部分中的`E1000_RXD_STAT_DD`位来检查新数据包是否可用。如果不可用，请停止。
- 否则，将`mbuf`的`m->len`更新为描述符中报告的长度。使用`net_rx()`将`mbuf`传送到网络栈。
- 然后使用`mbufalloc()`分配一个新的`mbuf`，以替换刚刚给`net_rx()`的`mbuf`。将其数据指针（`m->head`）编程到描述符中。将描述符的状态位清除为零。
- 最后，将`E1000_RDT`寄存器更新为最后处理的环描述符的索引。
- `e1000_init()`使用mbufs初始化RX环，您需要通过浏览代码来了解它是如何做到这一点的。
- 在某刻，曾经到达的数据包总数将超过环大小（16）；确保你的代码可以处理这个问题。

您将需要锁来应对xv6可能从多个进程使用E1000，或者在中断到达时在内核线程中使用E1000的可能性。

---
# 实验说明

## 实验难度

每个实验都具有相应的难度

- Easy：不到一个小时。这些锻炼通常是为后续锻炼做的热身运动。
    
- Moderate：1-2小时。
    
- Hard：超过2个小时。这些练习通常不需要很多代码，但是代码很难正确。

实验往往不需要很多行代码(几十到几百行) ，但是代码在概念上很复杂，而且细节往往很重要。所以，在你写任何代码之前，一定要完成实验室指定的阅读，通读相关文件，查阅文档(RISC-V手册等存放在了[参考页面](https://pdos.csail.mit.edu/6.828/2020/reference.html)上)。只有当你确定掌握了任务和解决方案，再开始编码。当你开始编写代码的时候，一小步一小步地实现你的解决方案(作业通常会建议如何将问题分解为更小的步骤)，并且在继续下一个步骤之前测试每个步骤是否正常工作。

## 调试技巧

确保你理解了c和指针。Kernighan和Ritchie的《c程序设计语言》一书对C语言进行了简要的描述。这里有一些有用的指针练习。除非你已经完全掌握了C语言，不要跳过或略读上面的指针练习。如果你不能真正理解C语言中的指针，你将在实验室中遭受难以言喻的痛苦，然后最终以一种艰难的方式来理解它们。相信我们，你不会想知道什么是“艰难的路”的。

一些常见的习惯用法特别值得记住:

- 如果`int *p = (int*)100`，那么`(int)p + 1`及`(int)(p + 1)`是不同的数字，第一个是101，但第二个是104。当向指针添加一个整数时，如第二种情况，整数被隐式地乘以指针指向的对象的大小。
    
- `p[i]`被定义为与`*(p+i)`相同，指向内存中p指向的第i个对象，当对象大于1字节时，上面所说的加法规则有利于此定义工作
    

虽然大多数C程序不需要在指针和整数之间进行强制转换，但操作系统经常需要这样做。每当您看到一个包含内存地址的加法时，问问自己它是整数加法还是指针加法，并确保所添加的值是否适当地相乘。

- 如果你有一个部分工作的练习，请通过提交代码来检查你的进度。如果您稍后破坏了某些东西，那么您可以回滚到您的检查点，然后以较小的步骤继续前进。要了解关于Git的更多信息，请查看[Git用户手册](http://www.kernel.org/pub/software/scm/git/docs/user-manual.html)，或者您可能会发现这个[面向计算机科学家的Git概述](http://eagain.net/articles/git-for-computer-scientists/)非常有用。
    
- 如果您没有通过测试，确保您了解为什么您的代码没有通过测试。插入打印(printf)语句，直到您理解正在发生的事情。
    
- 您可能会发现您的print语句可能会产生许多您想要搜索的输出；其中一种方法是在`script`内部运行`make qemu`（在您的机器上运行`man script`），它将所有控制台输出记录到一个文件中，然后您可以搜索该文件。别忘了退出`script`。
    
- 在许多情况下，print语句就足够了，但有时能够单步遍历一些汇编代码或检查堆栈上的变量是有帮助的。要在xv6中使用gdb，请在一个窗口中运行`make qemu-gdb`，在另一个窗口中运行`gdb`（或`riscv64-linux-gnu-gdb`），设置断点，后跟“`c`”（continue），xv6将一直运行，直到到达断点。（有关有用的GDB提示，请参阅[使用GNU调试器](https://pdos.csail.mit.edu/6.828/2019/lec/gdb_slides.pdf)。）
    
- 如果要查看编译器为内核生成的程序集是什么，或者要找出特定内核地址的指令是什么，请参阅文件`kernel.asm`，该文件在编译内核时由Makefile生成。（Makefile同时也为所有用户程序生成.asm文件。）
    
- 如果内核崩溃，它将打印一条错误消息，列出崩溃时程序计数器的值；您可以进行搜索`kernel.asm`找出程序计数器崩溃时在哪个函数中，或者可以运行`addr2line -e kernel/kernel pc-value`（有关详细信息，请运行`man addr2line`）。如果要获取回溯，请使用gdb重新启动：在一个窗口中运行'`make qemu-gdb`'，在另一个窗口中运行gdb（或`riscv64-linux-gnu-gdb`），在`panic`中设置断点（“`b panic`”），后跟“`c`”（continue）。当内核到达断点时，键入“bt”以获取回溯跟踪。
    
- 如果您的内核挂起(例如，由于死锁)或无法进一步执行(例如，由于在执行内核指令时出现页面错误)，您可以使用gdb查找挂起的位置。在一个窗口中运行“ `make qemu-gdb`”，在另一个窗口中运行 `gdb` (`riscv64-linux-gnu-gdb`) ，后跟“`c`”(continue)。当内核出现挂起时，在 `qemu-gdb` 窗口中按 `Ctrl-C` 并键入“`bt`”以获得回溯跟踪。
    
- qemu有一个“监视器”，允许您查询模拟机器的状态。您可以通过键入`<Ctrl>+a c`（c表示控制台）来获得它。一个特别有用的monitor命令`是info mem`，用于打印页表。您可能需要使用`cpu`命令来选择`info mem`查看哪一个核心，或者可以使用`make CPUS=1 qemu`启动qemu，以使其只有一个核心。
    

花时间学习上述工具是非常值得的