---
title: jvm-spec-3. Compiling for the Java Virtual Machine
tags:
  - 官方文档翻译
  - jvm
date: 2022-9-2 22:46:49
---

The Java Virtual Machine machine is designed to support the Java programming language. Oracle's JDK software contains a compiler from source code written in the Java programming language to the instruction set of the Java Virtual Machine, and a run-time system that implements the Java Virtual Machine itself. Understanding how one compiler utilizes the Java Virtual Machine is useful to the prospective compiler writer, as well as to one trying to understand the Java Virtual Machine itself. The numbered sections in this chapter are not normative.

Java虚拟机（Java Virtual Machine，JVM）的设计是为了支持Java编程语言。Oracle的JDK软件包含一个编译器，它将用Java编程语言编写的源代码编译成JVM的指令集，并且JDK中还包含一个实现JVM的运行时系统。了解一个编译器如何利用JVM，对于未来的编译器编写者以及试图理解JVM本身的人都是有帮助的。本章中编号的章节不是规范性的。

Note that the term "compiler" is sometimes used when referring to a translator from the instruction set of a Java Virtual Machine to the instruction set of a specific CPU. One example of such a translator is a just-in-time (JIT) code generator, which generates platform-specific instructions only after Java Virtual Machine code has been loaded. This chapter does not address issues associated with code generation, only those associated with compiling source code written in the Java programming language to Java Virtual Machine instructions.

需要注意的是，“编译器”一词有时也用来指从Java虚拟机指令集到特定CPU指令集的翻译器。此类翻译器的一个例子是即时编译器（JIT，Just-In-Time）代码生成器，它只在Java虚拟机代码被加载后生成平台特定的指令。本章不讨论与代码生成相关的问题，仅讨论将用Java编程语言编写的源代码编译为Java虚拟机指令时涉及的问题。

---
## 3.1 Format of Examples

This chapter consists mainly of examples of source code together with annotated listings of the Java Virtual Machine code that the javac compiler in Oracle’s JDK release 1.0.2 generates for the examples. The Java Virtual Machine code is written in the informal “virtual machine assembly language” output by Oracle's javap utility, distributed with the JDK release. You can use javap to generate additional examples of compiled methods.

本章主要由源代码示例以及Oracle JDK 1.0.2版本中的`javac`编译器为这些示例生成的Java虚拟机代码的带注释列表组成。Java虚拟机代码以非正式的“虚拟机汇编语言”编写，这种语言由Oracle的`javap`工具输出，该工具与JDK发行版一起分发。你可以使用`javap`生成编译方法的其他示例。

The format of the examples should be familiar to anyone who has read assembly code. Each instruction takes the form:

示例的格式对于读过汇编代码的人来说应该是熟悉的。每条指令的形式如下：

```
<index> <opcode> [ <operand1> [ <operand2>... ]] [<comment>]
```

The \<index> is the index of the opcode of the instruction in the array that contains the bytes of Java Virtual Machine code for this method. Alternatively, the \<index> may be thought of as a byte offset from the beginning of the method. The \<opcode> is the mnemonic for the instruction's opcode, and the zero or more \<operandN> are the operands of the instruction. The optional \<comment> is given in end-of-line comment syntax:

\<index> 是指令在包含该方法的Java虚拟机代码字节数组中的索引。或者说，\<index> 可以被视为从方法起始处开始的字节偏移量。\<opcode> 是指令的操作码的助记符，而一个或多个\<operandN> 是该指令的操作数。可选的 \<comment> 以行尾注释的语法给出：

```
8 bipush 100 // Push int constant 100
```

Some of the material in the comments is emitted by javap; the rest is supplied by the authors. The \<index> prefacing each instruction may be used as the target of a control transfer instruction. For instance, a goto 8 instruction transfers control to the instruction at index 8. Note that the actual operands of Java Virtual Machine control transfer instructions are offsets from the addresses of the opcodes of those instructions; these operands are displayed by javap (and are shown in this chapter) as more easily read offsets into their methods.

注释中的部分内容由`javap`输出，其他内容则由作者提供。前置于每条指令的\<index>可以作为控制转移指令的目标。例如，goto 8指令会将控制转移到索引为8的指令处。需要注意的是，Java虚拟机控制转移指令的实际操作数是从这些指令的操作码地址起的偏移量；这些操作数由`javap`显示（也在本章中显示）为更易读的方法偏移量。

We preface an operand representing a run-time constant pool index with a hash sign and follow the instruction by a comment identifying the run-time constant pool item referenced, as in:

我们在表示运行时常量池索引的操作数前加上井号，并在指令后面附上一个注释，用以标识引用的运行时常量池项，例如：

```
10 ldc #1 // Push float constant 100.0
```

```
9 invokevirtual #4 // Method Example.addTwo(II)I
```

For the purposes of this chapter, we do not worry about specifying details such as operand sizes.

出于本章的目的，我们不必关心指定操作数大小等细节。

## 3.2 Use of Constants, Local Variables, and Control Constructs

Java Virtual Machine code exhibits a set of general characteristics imposed by the Java Virtual Machine's design and use of types. In the first example we encounter many of these, and we consider them in some detail.

Java虚拟机代码表现出一组由Java虚拟机的设计和类型使用所决定的一般特性。在第一个示例中，我们会遇到许多这样的特性，并会详细讨论它们。

The spin method simply spins around an empty for loop 100 times:

`spin`方法简单地在一个空的`for`循环中运行100次：

---

```java
void spin() {
    int i;
    for (i = 0; i < 100; i++) {
        ; // Loop body is empty
    }
}
```

A compiler might compile spin to:

编译器可能会将`spin`编译为：

```
0 iconst_0       // Push int constant 0
1 istore_1       // Store into local variable 1 (i=0)
2 goto 8         // First time through don't increment
5 iinc 1 1       // Increment local variable 1 by 1 (i++)
8 iload_1        // Push local variable 1 (i)
9 bipush 100     // Push int constant 100
11 if_icmplt 5   // Compare and loop if less than (i < 100)
14 return        // Return void when done
```

The Java Virtual Machine is stack-oriented, with most operations taking one or more operands from the operand stack of the Java Virtual Machine's current frame or pushing results back onto the operand stack. A new frame is created each time a method is invoked, and with it is created a new operand stack and set of local variables for use by that method (§2.6). At any one point of the computation, there are thus likely to be many frames and equally many operand stacks per thread of control, corresponding to many nested method invocations. Only the operand stack in the current frame is active.

Java虚拟机是基于栈的，大多数操作从当前栈帧的操作数栈中获取一个或多个操作数，或者将结果推回到操作数栈。每次调用方法时都会创建一个新的栈帧，同时为该方法创建一个新的操作数栈和一组本地变量。在计算的任何一点上，控制线程通常会有许多栈帧和同样多的操作数栈，对应于许多嵌套的方法调用。只有当前栈帧中的操作数栈是活动的。

The instruction set of the Java Virtual Machine distinguishes operand types by using distinct bytecodes for operations on its various data types. The method spin operates only on values of type int. The instructions in its compiled code chosen to operate on typed data (iconst_0, istore_1, iinc, iload_1, if_icmplt) are all specialized for type int.

Java虚拟机的指令集通过使用不同的数据类型操作的字节码来区分操作数类型。`spin`方法仅处理`int`类型的值。其编译代码中的指令（如`iconst_0`、`istore_1`、`iinc`、`iload_1`、`if_icmplt`）都是专门为`int`类型定制的。

The two constants in spin, 0 and 100, are pushed onto the operand stack using two different instructions. The 0 is pushed using an iconst_0 instruction, one of the family of iconst_\<i> instructions. The 100 is pushed using a bipush instruction, which fetches the value it pushes as an immediate operand.

`spin`中的两个常量0和100使用了两种不同的指令将它们推送到操作数栈中。0使用`iconst_0`指令推送，它属于`iconst_<i>`指令家族。100则使用`bipush`指令推送，这个指令将值作为立即数操作数获取并推送。

The Java Virtual Machine frequently takes advantage of the likelihood of certain operands (int constants -1, 0, 1, 2, 3, 4 and 5 in the case of the iconst_\<i> instructions) by making those operands implicit in the opcode. Because the iconst_0 instruction knows it is going to push an int 0, iconst_0 does not need to store an operand to tell it what value to push, nor does it need to fetch or decode an operand. Compiling the push of 0 as bipush 0 would have been correct, but would have made the compiled code for spin one byte longer. A simple virtual machine would have also spent additional time fetching and decoding the explicit operand each time around the loop. Use of implicit operands makes compiled code more compact and efficient.

Java虚拟机经常利用某些操作数的可能性（例如`iconst_<i>`指令中的`int`常量-1、0、1、2、3、4和5），通过使这些操作数在操作码中隐含，从而优化性能。由于`iconst_0`指令知道它将推送一个`int 0`，因此`iconst_0`不需要存储操作数来指示它要推送的值，也不需要获取或解码操作数。将推送0编译为`bipush 0`虽然是正确的，但会使`spin`的编译代码增加一个字节长度。一个简单的虚拟机在每次循环时也会花费额外的时间来获取和解码显式操作数。使用隐式操作数可以使编译后的代码更加紧凑和高效。

The int i in spin is stored as Java Virtual Machine local variable 1. Because most Java Virtual Machine instructions operate on values popped from the operand stack rather than directly on local variables, instructions that transfer values between local variables and the operand stack are common in code compiled for the Java Virtual Machine. These operations also have special support in the instruction set. In spin, values are transferred to and from local variables using the istore_1 and iload_1 instructions, each of which implicitly operates on local variable 1. The istore_1 instruction pops an int from the operand stack and stores it in local variable 1. The iload_1 instruction pushes the value in local variable 1 on to the operand stack.

`spin`中的`int i`被存储为Java虚拟机的本地变量1。由于大多数Java虚拟机指令在操作数栈中弹出值进行操作，而不是直接对本地变量进行操作，因此在为Java虚拟机编译的代码中，值在本地变量和操作数栈之间传递的指令是很常见的。这些操作在指令集中也得到了特殊支持。在`spin`中，值通过`istore_1`和`iload_1`指令在本地变量和操作数栈之间传递，每条指令都隐式地操作本地变量1。`istore_1`指令从操作数栈中弹出一个`int`并将其存储在本地变量1中。`iload_1`指令则将本地变量1中的值推送到操作数栈上。

The use (and reuse) of local variables is the responsibility of the compiler writer. The specialized load and store instructions should encourage the compiler writer to reuse local variables as much as is feasible. The resulting code is faster, more compact, and uses less space in the frame.

本地变量的使用（以及重用）是编译器编写者的责任。专用的加载和存储指令应鼓励编译器编写者尽可能多地重用本地变量。这样产生的代码速度更快、更紧凑，并且在栈帧中占用更少的空间。

Certain very frequent operations on local variables are catered to specially by the Java Virtual Machine. The iinc instruction increments the contents of a local variable by a one-byte signed value. The iinc instruction in spin increments the first local variable (its first operand) by 1 (its second operand). The iinc instruction is very handy when implementing looping constructs.

Java虚拟机特别支持某些非常频繁的本地变量操作。`iinc`指令通过一个字节的有符号值增加本地变量的内容。`spin`中的`iinc`指令将第一个本地变量（其第一个操作数）增加1（其第二个操作数）。`iinc`指令在实现循环结构时非常方便。

The for loop of spin is accomplished mainly by these instructions:

`spin`的`for`循环主要通过以下指令实现：

```asm
5  iinc 1 1        // Increment local variable 1 by 1 (i++)
8  iload_1         // Push local variable 1 (i)
9  bipush 100      // Push int constant 100
11 if_icmplt 5     // Compare and loop if less than (i < 100)
```

The bipush instruction pushes the value 100 onto the operand stack as an int, then the if_icmplt instruction pops that value off the operand stack and compares it against i. If the comparison succeeds (the variable i is less than 100), control is transferred to index 5 and the next iteration of the for loop begins. Otherwise, control passes to the instruction following the if_icmplt.

`bipush`指令将值100作为`int`推送到操作数栈上，然后`if_icmplt`指令将该值从操作数栈中弹出并与`i`进行比较。如果比较成功（变量`i`小于100），控制转移到索引5，并开始`for`循环的下一次迭代。否则，控制转移到`if_icmplt`之后的指令。

If the spin example had used a data type other than int for the loop counter, the compiled code would necessarily change to reflect the different data type. For instance, if instead of an int the spin example uses a double, as shown:

如果`spin`示例为循环计数器使用了`int`以外的数据类型，则编译后的代码必然会改变以反映不同的数据类型。例如，如果将`spin`示例中的`int`替换为`double`，如下所示：

```java
void dspin() {
    double i;
    for (i = 0.0; i < 100.0; i++) {
        ; // Loop body is empty
    }
}
```

the compiled code is:

编译后的代码为：

```asm
0  dconst_0      // Push double constant 0.0
1  dstore_1      // Store into local variables 1 and 2
2  goto 9        // First time through don't increment
5  dload_1       // Push local variables 1 and 2
6  dconst_1      // Push double constant 1.0
7  dadd          // Add; there is no dinc instruction
8  dstore_1      // Store result in local variables 1 and 2
9  dload_1       // Push local variables 1 and 2
10 ldc2_w #4     // Push double constant 100.0
13 dcmpg         // There is no if_dcmplt instruction
14 iflt 5        // Compare and loop if less than (i < 100.0)
17 return        // Return void when done
```

The instructions that operate on typed data are now specialized for type double. (The ldc2_w instruction will be discussed later in this chapter.)

现在操作类型化数据的指令专门用于`double`类型。（`ldc2_w`指令将在本章稍后讨论。）

Recall that double values occupy two local variables, although they are only accessed using the lesser index of the two local variables. This is also the case for values of type long. Again for example,

请记住，`double`值占用两个本地变量，尽管它们只使用两个本地变量中较小的索引进行访问。对于`long`类型的值也是如此。再例如，

```java
double doubleLocals(double d1, double d2) {
    return d1 + d2;
}
```

becomes
编译后为

```
0 dload_1 // First argument in local variables 1 and 2
1 dload_3 // Second argument in local variables 3 and 4
2 dadd
3 dreturn
```

Note that local variables of the local variable pairs used to store double values in doubleLocals must never be manipulated individually.

请注意，用于在`doubleLocals`中存储`double`值的本地变量对中的本地变量，绝不能单独操作。

The Java Virtual Machine's opcode size of 1 byte results in its compiled code being very compact. However, 1-byte opcodes also mean that the Java Virtual Machine instruction set must stay small. As a compromise, the Java Virtual Machine does not provide equal support for all data types: it is not completely orthogonal (Table 2.11.1-A).

Java虚拟机的操作码大小为1字节，这使得其编译后的代码非常紧凑。然而，1字节的操作码也意味着Java虚拟机指令集必须保持较小的规模。作为一种妥协，Java虚拟机并未对所有数据类型提供相同的支持：它并非完全正交（参见表2.11.1-A）。

For example, the comparison of values of type int in the for statement of example spin can be implemented using a single if_icmplt instruction; however, there is no single instruction in the Java Virtual Machine instruction set that performs a conditional branch on values of type double. Thus, dspin must implement its comparison of values of type double using a dcmpg instruction followed by an iflt instruction.

例如，在`spin`示例的`for`语句中，对`int`类型值的比较可以使用单个`if_icmplt`指令来实现；然而，在Java虚拟机指令集中没有单个指令可以对`double`类型的值执行条件分支。因此，`dspin`必须使用`dcmpg`指令，然后跟随`iflt`指令来实现对`double`类型值的比较。

The Java Virtual Machine provides the most direct support for data of type int. This is partly in anticipation of efficient implementations of the Java Virtual Machine's operand stacks and local variable arrays. It is also motivated by the frequency of int data in typical programs. Other integral types have less direct support. There are no byte, char, or short versions of the store, load, or add instructions, for instance. Here is the spin example written using a short:

Java虚拟机为`int`类型的数据提供了最直接的支持。这部分是为了预期Java虚拟机操作数栈和本地变量数组的高效实现。它也是由于在典型程序中`int`数据的频繁使用。其他整数类型的支持则较少。例如，没有`byte`、`char`或`short`版本的存储、加载或添加指令。以下是使用`short`编写的`spin`示例：

```java
void sspin() {
    short i;
    for (i = 0; i < 100; i++) {
        ; // Loop body is empty
    }
}
```

It must be compiled for the Java Virtual Machine, as follows, using instructions operating on another type, most likely int, converting between short and int values as necessary to ensure that the results of operations on short data stay within the appropriate range:

它必须为Java虚拟机编译，如下所示，使用在另一种类型（最可能是`int`）上操作的指令，在`short`和`int`值之间进行转换，以确保对`short`数据的操作结果保持在适当的范围内：

```
0  iconst_0
1  istore_1
2  goto 10
5  iload_1
6  iconst_1
7  iadd
8  i2s
9  istore_1
10 iload_1
11 bipush 100
13 if_icmplt 5
16 return
```

The lack of direct support for byte, char, and short types in the Java Virtual Machine is not particularly painful, because values of those types are internally promoted to int (byte and short are sign-extended to int, char is zero-extended). Operations on byte, char, and short data can thus be done using int instructions. The only additional cost is that of truncating the values of int operations to valid ranges.

Java虚拟机对`byte`、`char`和`short`类型缺乏直接支持并不是特别麻烦，因为这些类型的值在内部被提升为`int`（`byte`和`short`被符号扩展为`int`，`char`被零扩展）。因此，对`byte`、`char`和`short`数据的操作可以使用`int`指令进行。唯一的额外成本是将`int`操作的值截断为有效范围。

The long and floating-point types have an intermediate level of support in the Java Virtual Machine, lacking only the full complement of conditional control transfer instructions.

`long`和浮点类型在Java虚拟机中具有中级支持，仅缺少完整的条件控制转移指令集。

---
## 3.3 Arithmetic

The Java Virtual Machine generally does arithmetic on its operand stack. (The exception is the iinc instruction, which directly increments the value of a local variable.) For instance, the align2grain method aligns an int value to a given power of 2:

Java虚拟机通常在其操作数栈上进行算术运算。（例外情况是`iinc`指令，它直接增加本地变量的值。）例如，`align2grain`方法将一个`int`值对齐到一个给定的2的幂次方：

```java
int align2grain(int i, int grain) {
    return ((i + grain-1) & ~(grain-1));
}
```

Operands for arithmetic operations are popped from the operand stack, and the results of operations are pushed back onto the operand stack. Results of arithmetic subcomputations can thus be made available as operands of their nesting computation. For instance, the calculation of ~(grain-1) is handled by these instructions:

算术运算的操作数从操作数栈中弹出，运算结果被推回到操作数栈上。因此，算术子计算的结果可以作为嵌套计算的操作数使用。例如，`~(grain-1)`的计算由以下指令处理：

```asm
5 iload_2    // Push grain
6 iconst_1   // Push int constant 1
7 isub       // Subtract; push result
8 iconst_m1  // Push int constant -1
9 ixor       // Do XOR; push result
```

First grain-1 is calculated using the contents of local variable 2 and an immediate int value 1. These operands are popped from the operand stack and their difference pushed back onto the operand stack. The difference is thus immediately available for use as one operand of the ixor instruction. (Recall that ~x == -1^x.) Similarly, the result of the ixor instruction becomes an operand for the subsequent iand instruction.

首先使用本地变量2的内容和立即数`int`值1计算`grain-1`。这些操作数从操作数栈中弹出，它们的差值被推回到操作数栈上。因此，这个差值可以立即作为`ixor`指令的一个操作数使用。（回忆一下，`~x == -1^x`。）类似地，`ixor`指令的结果成为后续`iand`指令的一个操作数。

The code for the entire method follows:

整个方法的代码如下：

```asm
Method int align2grain(int,int)
0  iload_1
1  iload_2
2  iadd
3  iconst_1
4  isub
5  iload_2
6  iconst_1
7  isub
8  iconst_m1
9  ixor
10 iand
11 ireturn
```

---
## 3.4 Accessing the Run-Time Constant Pool

Many numeric constants, as well as objects, fields, and methods, are accessed via the run-time constant pool of the current class. Object access is considered later (§3.8). Data of types int, long, float, and double, as well as references to instances of class String, are managed using the ldc, ldc_w, and ldc2_w instructions.

许多数值常量、对象、字段和方法都是通过当前类的运行时常量池访问的。对象访问将在后面讨论（§3.8）。`int`、`long`、`float`和`double`类型的数据以及对`String`类实例的引用都是使用`ldc`、`ldc_w`和`ldc2_w`指令管理的。

The ldc and ldc_w instructions are used to access values in the run-time constant pool (including instances of class String) of types other than double and long. The ldc_w instruction is used in place of ldc only when there is a large number of run-time constant pool items and a larger index is needed to access an item. The ldc2_w instruction is used to access all values of types double and long; there is no non-wide variant.

`ldc`和`ldc_w`指令用于访问运行时常量池中的值（包括`String`类实例），这些值的类型不是`double`和`long`。仅当运行时常量池项数量较多且需要较大的索引来访问某个项时，才使用`ldc_w`代替`ldc`。`ldc2_w`指令用于访问`double`和`long`类型的所有值；没有非宽版的变体。

Integral constants of types byte, char, or short, as well as small int values, may be compiled using the bipush, sipush, or iconst_\<i> instructions (§3.2). Certain small floating-point constants may be compiled using the fconst_\<f> and dconst_\<d> instructions.

`byte`、`char`或`short`类型的整数常量以及小的`int`值可以使用`bipush`、`sipush`或`iconst_<i>`指令编译（§3.2）。某些小的浮点数常量可以使用`fconst_<f>`和`dconst_<d>`指令编译。

In all of these cases, compilation is straightforward. For instance, the constants for:

在所有这些情况下，编译都是直接的。例如，以下方法的常量：

```java
void useManyNumeric() {
    int i = 100;
    int j = 1000000;
    long l1 = 1;
    long l2 = 0xffffffff;
    double d = 2.2;
    ...do some calculations...
}
```

are set up as follows:

被设置如下：

```
Method void useManyNumeric()
0  bipush 100    // Push small int constant with bipush
2  istore_1
3  ldc #1        // Push large int constant (1000000) with ldc
5  istore_2
6  lconst_1      // A tiny long value uses small fast lconst_1
7  lstore_3
8  ldc2_w #6     // Push long 0xffffffff (that is, an int -1)
11 lstore 5
13 ldc2_w #8    // Push double constant 2.200000
16 dstore 7
...do those calculations...
```

---
## 3.5 More Control Examples

Compilation of for statements was shown in an earlier section (§3.2). Most of the Java programming language's other control constructs (if-then-else, do, while, break, and continue) are also compiled in the obvious ways. The compilation of switch statements is handled in a separate section (§3.10), as are the compilation of exceptions (§3.12) and the compilation of finally clauses (§3.13).

前面章节（§3.2）展示了`for`语句的编译。Java编程语言的大多数其他控制结构（如`if-then-else`、`do`、`while`、`break`和`continue`）也以显而易见的方式编译。`switch`语句的编译将在单独的章节中讨论（§3.10），异常的编译（§3.12）和`finally`子句的编译（§3.13）也将分别处理。

As a further example, a while loop is compiled in an obvious way, although the specific control transfer instructions made available by the Java Virtual Machine vary by data type. As usual, there is more support for data of type int, for example:

作为进一步的示例，`while`循环以显而易见的方式编译，尽管Java虚拟机提供的特定控制转移指令因数据类型而异。通常情况下，`int`类型的数据支持更多，例如：

```java
void whileInt() {
    int i = 0;
    while (i < 100) {
        i++;
    }
}
```

is compiled to:

编译后的代码为：

```
Method void whileInt()
0 iconst_0
1 istore_1
2 goto 8
5 iinc 1 1
8 iload_1
9 bipush 100
11 if_icmplt 5
14 return
```

Note that the test of the while statement (implemented using the if_icmplt instruction) is at the bottom of the Java Virtual Machine code for the loop. (This was also the case in the spin examples earlier.) The test being at the bottom of the loop forces the use of a goto instruction to get to the test prior to the first iteration of the loop. If that test fails, and the loop body is never entered, this extra instruction is wasted. However, while loops are typically used when their body is expected to be run, often for many iterations. For subsequent iterations, putting the test at the bottom of the loop saves a Java Virtual Machine instruction each time around the loop: if the test were at the top of the loop, the loop body would need a trailing goto instruction to get back to the top.

请注意，`while`语句的测试（使用`if_icmplt`指令实现）位于循环的Java虚拟机代码的底部。（在前面的`spin`示例中也是如此。）测试位于循环的底部，这迫使使用`goto`指令在循环的第一次迭代之前进入测试。如果测试失败，并且从未进入循环体，那么这条额外的指令将被浪费。然而，`while`循环通常在预计其循环体会运行时使用，通常是多次迭代。在后续的迭代中，将测试放在循环底部可以每次循环节省一个Java虚拟机指令：如果测试位于循环顶部，则循环体需要一个尾随`goto`指令来返回顶部。

Control constructs involving other data types are compiled in similar ways, but must use the instructions available for those data types. This leads to somewhat less efficient code because more Java Virtual Machine instructions are needed, for example:

涉及其他数据类型的控制结构以类似的方式编译，但必须使用适用于这些数据类型的指令。这导致代码效率较低，因为需要更多的Java虚拟机指令，例如：

```java
void whileDouble() {
    double i = 0.0;
    while (i < 100.1) {
        i++;
    }
}
```

is compiled to:

编译后的代码为：

```
Method void whileDouble()
0 dconst_0
1 dstore_1
2 goto 9
5 dload_1
6 dconst_1
7 dadd
8 dstore_1
9 dload_1
10 ldc2_w #4
13 dcmpg
14 iflt 5
17 return
```

Each floating-point type has two comparison instructions: fcmpl and fcmpg for type float, and dcmpl and dcmpg for type double. The variants differ only in their treatment of NaN. NaN is unordered (§2.3.2), so all floating-point comparisons fail if either of their operands is NaN. The compiler chooses the variant of the comparison instruction for the appropriate type that produces the same result whether the comparison fails on non-NaN values or encounters a NaN. For instance:

每种浮点类型都有两种比较指令：`fcmpl`和`fcmpg`用于`float`类型，`dcmpl`和`dcmpg`用于`double`类型。这些变体仅在处理`NaN`（非数字）方面有所不同。`NaN`是无序的（§2.3.2），因此如果操作数之一为`NaN`，则所有浮点比较都会失败。编译器为适当的类型选择比较指令的变体，使得无论比较失败于非`NaN`值还是遇到`NaN`，都能产生相同的结果。例如：

```java
int lessThan100(double d) {
    if (d < 100.0) {
        return 1;
    } else {
        return -1;
    }
}
```

compiles to:

编译后的代码为：

```
Method int lessThan100(double)
0  dload_1
1  ldc2_w #4
4  dcmpg
5  ifge 10
8  iconst_1
9  ireturn
10 iconst_m1
11 ireturn
```

If d is not NaN and is less than 100.0, the dcmpg instruction pushes an int -1 onto the operand stack, and the ifge instruction does not branch. Whether d is greater than 100.0 or is NaN, the dcmpg instruction pushes an int 1 onto the operand stack, and the ifge branches. If d is equal to 100.0, the dcmpg instruction pushes an int 0 onto the operand stack, and the ifge branches.

如果`d`不是`NaN`并且小于100.0，`dcmpg`指令将`int -1`推送到操作数栈上，并且`ifge`指令不会分支。无论`d`是否大于100.0或是`NaN`，`dcmpg`指令都会将`int 1`推送到操作数栈上，并且`ifge`指令将分支。如果`d`等于100.0，`dcmpg`指令将`int 0`推送到操作数栈上，并且`ifge`指令将分支。

The dcmpl instruction achieves the same effect if the comparison is reversed:

如果比较结果相反，`dcmpl`指令可以实现相同的效果：

```java
int greaterThan100(double d) {
    if (d > 100.0) {
        return 1;
    } else {
        return -1;
    }
}
```

becomes:

编译后的代码为：

```
Method int greaterThan100(double)
0 dload_1
1 ldc2_w #4
4 dcmpl
5 ifle 10
8 iconst_1
9 ireturn
10 iconst_m1
11 ireturn
```

Once again, whether the comparison fails on a non-NaN value or because it is passed a NaN, the dcmpl instruction pushes an int value onto the operand stack that causes the ifle to branch. If both of the dcmp instructions did not exist, one of the example methods would have had to do more work to detect NaN.

再次强调，无论比较失败于非`NaN`值还是因为遇到`NaN`，`dcmpl`指令都会将一个`int`值推送到操作数栈上，导致`ifle`分支。如果没有这两个`dcmp`指令，示例方法之一将不得不进行更多工作来检测`NaN`。

---
## 3.6 Receiving Arguments

If n arguments are passed to an instance method, they are received, by convention, in the local variables numbered 1 through n of the frame created for the new method invocation. The arguments are received in the order they were passed. For example:

如果将n个参数传递给实例方法，根据惯例，这些参数将被接收并存储在为新方法调用创建的栈帧的本地变量1到n中。参数的接收顺序与传递顺序一致。例如：

```java
int addTwo(int i, int j) {
    return i + j;
}
```

compiles to:

编译后的代码为：

```asm
Method int addTwo(int,int)
0 iload_1   // Push value of local variable 1 (i)
1 iload_2   // Push value of local variable 2 (j)
2 iadd      // Add; leave int result on operand stack
3 ireturn   // Return int result
```

By convention, an instance method is passed a reference to its instance in local variable 0. In the Java programming language the instance is accessible via the this keyword.

按照惯例，实例方法在本地变量0中传递对其实例的引用。在Java编程语言中，该实例可以通过`this`关键字访问。

Class (static) methods do not have an instance, so for them this use of local variable 0 is unnecessary. A class method starts using local variables at index 0. If the addTwo method were a class method, its arguments would be passed in a similar way to the first version:

类（静态）方法没有实例，因此对于它们来说，本地变量0的使用是没有必要的。类方法从索引0开始使用本地变量。如果`addTwo`方法是一个类方法，其参数将以与第一个版本相似的方式传递：

```java
static int addTwoStatic(int i, int j) {
    return i + j;
}
```

compiles to:

编译后的代码为：

```
Method int addTwoStatic(int,int)
0 iload_0
1 iload_1
2 iadd
3 ireturn
```

The only difference is that the method arguments appear starting in local variable 0 rather than 1.

唯一的区别是方法参数从本地变量0开始，而不是从1开始。

---
## 3.7 Invoking Methods

The normal method invocation for a instance method dispatches on the run-time type of the object. (They are virtual, in C++ terms.) Such an invocation is implemented using the invokevirtual instruction, which takes as its argument an index to a run-time constant pool entry giving the internal form of the binary name of the class type of the object, the name of the method to invoke, and that method's descriptor (§4.3.3). To invoke the addTwo method, defined earlier as an instance method, we might write:

实例方法的常规方法调用基于对象的运行时类型进行调度。（用C++的术语来说，它们是虚拟的。）这种调用是通过`invokevirtual`指令实现的，该指令将运行时常量池项的索引作为其参数，提供对象的类类型的二进制名称的内部形式、要调用的方法名称以及该方法的描述符（§4.3.3）。要调用前面定义的实例方法`addTwo`，我们可以这样写：

```java
int add12and13() {
    return addTwo(12, 13);
}
```

This compiles to:

编译后的代码为：

```asm
Method int add12and13()    
0 aload_0                  // Push local variable 0 (this)
1 bipush 12                // Push int constant 12
3 bipush 13                // Push int constant 13
5 invokevirtual #4         // Method Example.addtwo(II)I
8 ireturn                  // Return int on top of operand stack;it is the int result of addTwo()
```

The invocation is set up by first pushing a reference to the current instance, this, on to the operand stack. The method invocation's arguments, int values 12 and 13, are then pushed. When the frame for the addTwo method is created, the arguments passed to the method become the initial values of the new frame's local variables. That is, the reference for this and the two arguments, pushed onto the operand stack by the invoker, will become the initial values of local variables 0, 1, and 2 of the invoked method.

方法调用的设置首先通过将当前实例的引用`this`推送到操作数栈上。然后推送方法调用的参数，`int`值12和13。当为`addTwo`方法创建栈帧时，传递给该方法的参数成为新栈帧本地变量的初始值。也就是说，由调用者推送到操作数栈上的`this`引用和两个参数，将成为被调用方法的本地变量0、1和2的初始值。

Finally, addTwo is invoked. When it returns, its int return value is pushed onto the operand stack of the frame of the invoker, the add12and13 method. The return value is thus put in place to be immediately returned to the invoker of add12and13.

最后，调用`addTwo`。当它返回时，其`int`返回值被推送到调用者栈帧的操作数栈上，即`add12and13`方法的操作数栈。返回值因此被放置在适当位置，立即返回给`add12and13`的调用者。

The return from add12and13 is handled by the ireturn instruction of add12and13. The ireturn instruction takes the int value returned by addTwo, on the operand stack of the current frame, and pushes it onto the operand stack of the frame of the invoker. It then returns control to the invoker, making the invoker's frame current. The Java Virtual Machine provides distinct return instructions for many of its numeric and reference data types, as well as a return instruction for methods with no return value. The same set of return instructions is used for all varieties of method invocations.

从`add12and13`返回由`add12and13`的`ireturn`指令处理。`ireturn`指令获取`addTwo`返回的`int`值，位于当前栈帧的操作数栈上，并将其推送到调用者栈帧的操作数栈上。然后它将控制权返回给调用者，使调用者的栈帧成为当前栈帧。Java虚拟机为其许多数值和引用数据类型提供了不同的返回指令，以及用于无返回值方法的返回指令。同一组返回指令用于所有类型的方法调用。

The operand of the invokevirtual instruction (in the example, the run-time constant pool index #4) is not the offset of the method in the class instance. The compiler does not know the internal layout of a class instance. Instead, it generates symbolic references to the methods of an instance, which are stored in the run-time constant pool. Those run-time constant pool items are resolved at run-time to determine the actual method location. The same is true for all other Java Virtual Machine instructions that access class instances.

`invokevirtual`指令的操作数（在示例中为运行时常量池索引#4）不是类实例中方法的偏移量。编译器并不知道类实例的内部布局。相反，它生成对实例方法的符号引用，这些引用存储在运行时常量池中。这些运行时常量池项在运行时解析以确定实际的方法位置。所有其他访问类实例的Java虚拟机指令也是如此。

Invoking addTwoStatic, a class (static) variant of addTwo, is similar, as shown:

调用`addTwoStatic`，`addTwo`的类（静态）变体，与实例方法类似，如下所示：

```java
int add12and13() {
    return addTwoStatic(12, 13);
}
```

although a different Java Virtual Machine method invocation instruction is used:

尽管使用了不同的Java虚拟机方法调用指令：

```
Method int add12and13()
0 bipush 12
2 bipush 13
4 invokestatic #3       // Method Example.addTwoStatic(II)I
7 ireturn
```

Compiling an invocation of a class (static) method is very much like compiling an invocation of an instance method, except this is not passed by the invoker. The method arguments will thus be received beginning with local variable 0 (§3.6). The invokestatic instruction is always used to invoke class methods.

编译类（静态）方法的调用与编译实例方法的调用非常相似，只不过`this`不会由调用者传递。因此，方法参数将从本地变量0开始接收（§3.6）。`invokestatic`指令始终用于调用类方法。

The invokespecial instruction must be used to invoke instance initialization methods (§3.8). It is also used when invoking methods in the superclass (super) and when invoking private methods. For instance, given classes Near and Far declared as:

`invokespecial`指令必须用于调用实例初始化方法（§3.8）。它还用于调用超类（`super`）中的方法和私有方法。例如，定义了`Near`和`Far`类如下：

```java
class Near {
    int it;
    public int getItNear() {
        return getIt();
    }
    private int getIt() {
        return it;
    }
}
class Far extends Near {
    int getItFar() {
        return super.getItNear();
    }
}
```

the method Near.getItNear (which invokes a private method) becomes:

`Near.getItNear`方法（调用了一个私有方法）编译后为：

```
Method int getItNear()
0 aload_0
1 invokespecial #5      // Method Near.getIt()I
4 ireturn
```

The method Far.getItFar (which invokes a superclass method) becomes:

`Far.getItFar`方法（调用了一个超类方法）编译后为：

```
Method int getItFar()
0 aload_0
1 invokespecial #4      // Method Near.getItNear()I
4 ireturn
```

Note that methods called using the invokespecial instruction always pass this to the invoked method as its first argument. As usual, it is received in local variable 0.

请注意，使用`invokespecial`指令调用的方法始终将`this`作为其第一个参数传递给被调用方法。与往常一样，它在本地变量0中接收。

To invoke the target of a method handle, a compiler must form a method descriptor that records the actual argument and return types. A compiler may not perform method invocation conversions on the arguments; instead, it must push them on the stack according to their own unconverted types. The compiler arranges for a reference to the method handle object to be pushed on the stack before the arguments, as usual. The compiler emits an invokevirtual instruction that references a descriptor which describes the argument and return types. By special arrangement with method resolution (§5.4.3.3), an invokevirtual instruction which invokes the invokeExact or invoke methods of java.lang.invoke.MethodHandle will always link, provided the method descriptor is syntactically well-formed and the types named in the descriptor can be resolved.

要调用方法句柄的目标，编译器必须形成一个记录实际参数和返回类型的方法描述符。编译器不得对参数执行方法调用转换；相反，它必须根据其未转换的类型将它们推送到栈上。编译器安排在参数之前将方法句柄对象的引用推送到栈上，如常规操作一样。编译器发出一个`invokevirtual`指令，该指令引用一个描述参数和返回类型的描述符。通过与方法解析（§5.4.3.3）的特殊安排，调用`java.lang.invoke.MethodHandle`的`invokeExact`或`invoke`方法的`invokevirtual`指令将始终链接，前提是方法描述符在语法上是正确的，并且描述符中命名的类型可以解析。

---
## 3.8 Working with Class Instances

Java Virtual Machine class instances are created using the Java Virtual Machine's new instruction. Recall that at the level of the Java Virtual Machine, a constructor appears as a method with the compiler-supplied name \<init>. This specially named method is known as the instance initialization method (§2.9). Multiple instance initialization methods, corresponding to multiple constructors, may exist for a given class. Once the class instance has been created and its instance variables, including those of the class and all of its superclasses, have been initialized to their default values, an instance initialization method of the new class instance is invoked. For example:

Java虚拟机类实例是使用Java虚拟机的`new`指令创建的。请记住，在Java虚拟机层面上，构造函数显示为一个带有编译器提供名称`<init>`的方法。这个特别命名的方法被称为实例初始化方法（§2.9）。对于给定类，可能存在多个实例初始化方法，对应多个构造函数。一旦类实例被创建，并且其实例变量（包括该类及其所有超类的实例变量）已被初始化为默认值，新类实例的实例初始化方法就会被调用。例如：

```java
Object create() {
    return new Object();
}
```

compiles to:

编译后的代码为：

```
Method java.lang.Object create()
0 new #1             // Class java.lang.Object
3 dup
4 invokespecial #4   // Method java.lang.Object.<init>()V
7 areturn
```

Class instances are passed and returned (as reference types) very much like numeric values, although type reference has its own complement of instructions, for example:

类实例的传递和返回（作为引用类型）与数值非常相似，尽管引用类型有自己的一套指令，例如：

```java
int i;      // An instance variable
MyObj example() {
    MyObj o = new MyObj();
    return silly(o);
}
MyObj silly(MyObj o) {
    if (o != null) {
        return o;
    } else {
        return o;
    }
}
```

becomes:

编译后的代码为：

```
Method MyObj example()
0 new #2               // Class MyObj
3 dup
4 invokespecial #5     // Method MyObj.<init>()V
7 astore_1
8 aload_0
9 aload_1
10 invokevirtual #4    // Method Example.silly(LMyObj;)LMyObj;
13 areturn


Method MyObj silly(MyObj) 
0 aload_1  
1 ifnull 6  
4 aload_1
5 areturn 
6 aload_1 
7 areturn

```

The fields of a class instance (instance variables) are accessed using the getfield and putfield instructions. If i is an instance variable of type int, the methods setIt and getIt, defined as:

类实例的字段（实例变量）通过`getfield`和`putfield`指令访问。如果`i`是一个`int`类型的实例变量，定义的方法`setIt`和`getIt`如下：

```java
void setIt(int value) {
    i = value;
}
int getIt() {
    return i;
}
```

become:

编译后的代码为：

```
Method void setIt(int)
0 aload_0
1 iload_1
2 putfield #4       // Field Example.i I
5 return

Method int getIt()
0 aload_0
1 getfield #4       // Field Example.i I
4 ireturn
```

As with the operands of method invocation instructions, the operands of the putfield and getfield instructions (the run-time constant pool index #4) are not the offsets of the fields in the class instance. The compiler generates symbolic references to the fields of an instance, which are stored in the run-time constant pool. Those run-time constant pool items are resolved at run-time to determine the location of the field within the referenced object.

与方法调用指令的操作数一样，`putfield`和`getfield`指令的操作数（运行时常量池索引#4）不是类实例中字段的偏移量。编译器为实例的字段生成符号引用，这些引用存储在运行时常量池中。这些运行时常量池项在运行时解析以确定引用对象中字段的位置。

---
## 3.9 Arrays

Java Virtual Machine arrays are also objects. Arrays are created and manipulated using a distinct set of instructions. The newarray instruction is used to create an array of a numeric type. The code:

Java虚拟机的数组也是对象。数组的创建和操作使用了一套不同的指令。`newarray`指令用于创建某种数值类型的数组。如下代码：

```java
void createBuffer() {
    int buffer[];
    int bufsz = 100;
    int value = 12;
    buffer = new int[bufsz];
    buffer[10] = value;
    value = buffer[11];
}
```

might be compiled to:

可能会被编译为：

```asm
Method void createBuffer()
0  bipush 100    // Push int constant 100 (bufsz)
2  istore_2      // Store bufsz in local variable 2
3  bipush 12     // Push int constant 12 (value)
5  istore_3      // Store value in local variable 3
6  iload_2       // Push bufsz...
7  newarray int  // ...and create new int array of that length
9  astore_1      // Store new array in buffer
10 aload_1       // Push buffer
11 bipush 10     // Push int constant 10
13 iload_3       // Push value
14 iastore       // Store value at buffer[10]
15 aload_1       // Push buffer
16 bipush 11     // Push int constant 11
18 iaload        // Push value at buffer[11]...
19 istore_3      // ...and store it in value
20 return
```

The anewarray instruction is used to create a one-dimensional array of object references, for example:

`anewarray`指令用于创建一个一维对象引用数组，例如：

```java
void createThreadArray() {
    Thread threads[];
    int count = 10;
    threads = new Thread[count];
    threads[0] = new Thread();
}
```

becomes:

编译后的代码为：

```
Method void createThreadArray()
0 bipush 10             // Push int constant 10
2 istore_2              // Initialize count to that
3 iload_2               // Push count, used by anewarray
4 anewarray class #1    // Create new array of class Thread        
7 astore_1              // Store new array in threads
8 aload_1               // Push value of threads
9 iconst_0              // Push int constant 0
10 new #1               // Create instance of class Thread
13 dup                  // Make duplicate reference...
14 invokespecial #5     // ...for Thread's constructor; Method java.lang.Thread.<init>()V
17 aastore              // Store new Thread in array at 0
18 return            
```

The anewarray instruction can also be used to create the first dimension of a multidimensional array. Alternatively, the multianewarray instruction can be used to create several dimensions at once. For example, the three-dimensional array:

`anewarray`指令也可用于创建多维数组的第一维。或者，可以使用`multianewarray`指令一次性创建多个维度。例如，三维数组：

```java
int[][][] create3DArray() {
    int grid[][][];
    grid = new int[10][5][];
    return grid;
}
```

is created by:

可通过以下指令创建：

```
Method int create3DArray()[][][]
0 bipush 10                   // Push int 10 (dimension one)
2 iconst_5                    // Push int 5 (dimension two)
3 multianewarray #1 dim #2    // Class [[[I, a three-dimensional;int array;only create the first two dimensions
7 astore_1                    // Store new array...        
8 aload_1                     // ...then prepare to return it
9 areturn
```

The first operand of the multianewarray instruction is the run-time constant pool index to the array class type to be created. The second is the number of dimensions of that array type to actually create. The multianewarray instruction can be used to create all the dimensions of the type, as the code for create3DArray shows. Note that the multidimensional array is just an object and so is loaded and returned by an aload_1 and areturn instruction, respectively. For information about array class names, see §4.4.1.

`multianewarray`指令的第一个操作数是要创建的数组类类型的运行时常量池索引。第二个是该数组类型实际创建的维数。`multianewarray`指令可用于创建该类型的所有维度，正如`create3DArray`代码所示。注意，多维数组只是一个对象，因此分别通过`aload_1`和`areturn`指令加载和返回。有关数组类名称的信息，请参见§4.4.1。


All arrays have associated lengths, which are accessed via the `arraylength` instruction.

所有数组都有相关的长度，可以通过`arraylength`指令访问。

---
## 3.10 Compiling Switches

Compilation of switch statements uses the `tableswitch` and `lookupswitch` instructions. The `tableswitch` instruction is used when the cases of the switch can be efficiently represented as indices into a table of target offsets. The default target of the switch is used if the value of the expression of the switch falls outside the range of valid indices. For instance:

Switch语句的编译使用`tableswitch`和`lookupswitch`指令。当Switch的case可以有效地表示为目标偏移量表中的索引时，使用`tableswitch`指令。如果Switch表达式的值超出了有效索引范围，则使用Switch的默认目标。例如：

```java
int chooseNear(int i) {
    switch (i) {
        case 0:  return  0;
        case 1:  return  1;
        case 2:  return  2;
        default: return -1;
    }
}
```

compiles to:

编译为：

```java
Method int chooseNear(int)
0 iload_1
1 tableswitch 0 to 2:
    0: 28
    1: 30
    2: 32
default:34
28 iconst_0
29 ireturn
30 iconst_1
31 ireturn
32 iconst_2
33 ireturn
34 iconst_m1
35 ireturn
```

The Java Virtual Machine's `tableswitch` and `lookupswitch` instructions operate only on `int` data. Because operations on `byte`, `char`, or `short` values are internally promoted to `int`, a switch whose expression evaluates to one of those types is compiled as though it evaluated to type `int`. If the `chooseNear` method had been written using type `short`, the same Java Virtual Machine instructions would have been generated as when using type `int`. Other numeric types must be narrowed to type `int` for use in a switch.

`tableswitch`和`lookupswitch`指令只能操作`int`类型的数据。由于`byte`、`char`或`short`值的操作会被内部提升为`int`，因此，如果Switch的表达式计算为这些类型之一，它们会被编译为`int`类型。如果`chooseNear`方法是用`short`类型编写的，生成的Java虚拟机指令将与使用`int`类型时生成的指令相同。其他数值类型必须缩小为`int`类型才能在Switch中使用。

Where the cases of the switch are sparse, the table representation of the `tableswitch` instruction becomes inefficient in terms of space. The `lookupswitch` instruction may be used instead. The `lookupswitch` instruction pairs `int` keys (the values of the case labels) with target offsets in a table. When a `lookupswitch` instruction is executed, the value of the expression of the switch is compared against the keys in the table. If one of the keys matches the value of the expression, execution continues at the associated target offset. If no key matches, execution continues at the default target. For instance, the compiled code for:

当Switch的case稀疏时，`tableswitch`指令的表表示在空间方面变得低效。这时可以使用`lookupswitch`指令。`lookupswitch`指令将`int`键（case标签的值）与表中的目标偏移量配对。当执行`lookupswitch`指令时，Switch表达式的值会与表中的键进行比较。如果其中一个键与表达式的值匹配，执行会继续在相关的目标偏移量处。如果没有键匹配，执行会继续在默认目标处。例如，以下代码的编译：

```java
int chooseFar(int i) {
    switch (i) {
        case -100: return -1;
        case 0:    return  0;
        case 100:  return  1;
        default:   return -1;
    }
}
```

looks just like the code for `chooseNear`, except for the `lookupswitch` instruction:

看起来与`chooseNear`的代码相似，除了使用了`lookupswitch`指令：

```java
Method int chooseFar(int)
0 iload_1
1 lookupswitch 3:
    -100: 36
       0: 38
     100: 40
 default: 42
36 iconst_m1
37 ireturn
38 iconst_0
39 ireturn
40 iconst_1
41 ireturn
42 iconst_m1
43 ireturn
```

The Java Virtual Machine specifies that the table of the `lookupswitch` instruction must be sorted by key so that implementations may use searches more efficient than a linear scan. Even so, the `lookupswitch` instruction must search its keys for a match rather than simply perform a bounds check and index into a table like `tableswitch`. Thus, a `tableswitch` instruction is probably more efficient than a `lookupswitch` where space considerations permit a choice.

Java虚拟机规定，`lookupswitch`指令的表必须按键排序，以便实现可以使用比线性扫描更高效的搜索方法。尽管如此，`lookupswitch`指令必须在其键中搜索匹配，而不是像`tableswitch`那样简单地执行边界检查并索引到表中。因此，在空间允许的情况下，`tableswitch`指令可能比`lookupswitch`更高效。

---
## 3.11 Operations on the Operand Stack

The Java Virtual Machine has a large complement of instructions that manipulate the contents of the operand stack as untyped values. These are useful because of the Java Virtual Machine's reliance on deft manipulation of its operand stack. For instance:

Java虚拟机有大量的指令可以将操作数栈中的内容作为无类型值进行操作。这些指令很有用，因为Java虚拟机依赖于对其操作数栈的灵活操作。例如：

```java
public long nextIndex() {
    return index++;
}

private long index = 0;
```

is compiled to:

编译为：

```java
Method long nextIndex()
0 aload_0
1 dup
2 getfield #4
5 dup2_x1
6 lconst_1
7 ladd
8 putfield #4
11 lreturn
```

Note that the Java Virtual Machine never allows its operand stack manipulation instructions to modify or break up individual values on the operand stack.

需要注意的是，Java虚拟机不允许其操作数栈操作指令修改或分解操作数栈中的单个值。

---
## 3.12 Throwing and Handling Exceptions

Exceptions are thrown from programs using the `throw` keyword. Its compilation is simple:

程序使用`throw`关键字抛出异常。它的编译很简单：

```java
void cantBeZero(int i) throws TestExc {
    if (i == 0) {
        throw new TestExc();
    }
}
```

becomes:

编译为：

```java
Method void cantBeZero(int)
0 iload_1
1 ifne 12
4 new #1
7 dup
8 invokespecial #7
11 athrow
12 return
```

Compilation of try-catch constructs is straightforward. For example:

`try-catch`结构的编译很简单。例如：

```java
void catchOne() {
    try {
        tryItOut();
    } catch (TestExc e) {
        handleExc(e);
    }
}
```

is compiled as:

编译为：

```java
Method void catchOne()
0 aload_0
1 invokevirtual #6
4 return
5 astore_1
6 aload_0
7 aload_1
8 invokevirtual #5
11 return
Exception table:
From To Target
0 4 5
```

Looking more closely, the try block is compiled just as it would be if the try were not present:

仔细看，`try`块的编译与没有`try`时是一样的：

```java
Method void catchOne()
0 aload_0
1 invokevirtual #6
4 return
```

If no exception is thrown during the execution of the try block, it behaves as though the try were not there: `tryItOut` is invoked and `catchOne` returns.

如果在执行`try`块期间没有抛出异常，它的行为就像没有`try`一样：调用`tryItOut`并返回`catchOne`。

Following the try block is the Java Virtual Machine code that implements the single catch clause:

在`try`块之后是实现单个`catch`子句的Java虚拟机代码：

```java
5 astore_1
6 aload_0
7 aload_1
8 invokevirtual #5
11 return
Exception table:
From To Target
0 4 5
```

The invocation of `handleExc`, the contents of the catch clause, is also compiled like a normal method invocation. However, the presence of a catch clause causes the compiler to generate an exception table entry (§2.10, §4.7.3). The exception table for the `catchOne` method has one entry corresponding to the one argument (an instance of class `TestExc`) that the catch clause of `catchOne` can handle. If some value that is an instance of `TestExc` is thrown during execution of the instructions between indices 0 and 4 in `catchOne`, control is transferred to the Java Virtual Machine code at index 5, which implements the block of the catch clause. If the value that is thrown is not an instance of `TestExc`, the catch clause of `catchOne` cannot handle it. Instead, the value is rethrown to the invoker of `catchOne`.

`handleExc`的调用（即`catch`子句的内容）也像普通方法调用一样被编译。然而，`catch`子句的存在导致编译器生成一个异常表条目（§2.10，§4.7.3）。`catchOne`方法的异常表有一个条目，对应于`catchOne`的`catch`子句可以处理的参数（类`TestExc`的一个实例）。如果在`catchOne`中执行0到4索引之间的指令期间抛出了`TestExc`实例的某个值，控制将转移到索引5处的Java虚拟机代码，该代码实现`catch`子句的块。如果抛出的值不是`TestExc`的实例，`catchOne`的`catch`子句无法处理它，而是将该值重新抛出给`catchOne`的调用者。

A try may have multiple catch clauses:

`try`语句可以有多个`catch`子句：

```java
void catchTwo() {
    try {
        tryItOut();
    } catch (TestExc1 e) {
        handleExc(e);
    } catch (TestExc2 e) {
        handleExc(e);
    }
}
```

Multiple catch clauses of a given try statement are compiled by simply appending the Java Virtual Machine code for each catch clause one after the other and adding entries to the exception table, as shown:

对于给定的`try`语句的多个`catch`子句，编译器通过简单地将每个`catch`子句的Java虚拟机代码依次附加起来，并在异常表中添加条目，如下所示：

```java
Method void catchTwo()
0 aload_0
1 invokevirtual #5
4 return
5 astore_1
6 aload_0
7 aload_1
8 invokevirtual #7
11 return
12 astore_1
13 aload_0
14 aload_1
15 invokevirtual #7
18 return
Exception table:
From To Target
0 4 5
0 4 12
```

If during the execution of the try clause (between indices 0 and 4) a value is thrown that matches the parameter of one or more of the catch clauses (the value is an instance of one or more of the parameters), the first (innermost) such catch clause is selected. Control is transferred to the Java Virtual Machine code for the block of that catch clause. If the value thrown does not match the parameter of any of the catch clauses of `catchTwo`, the Java Virtual Machine rethrows the value without invoking code in any catch clause of `catchTwo`.

如果在执行`try`子句（索引0到4之间）期间抛出一个值，该值与一个或多个`catch`子句的参数匹配（该值是一个或多个参数的实例），则选择第一个（最内层的）`catch`子句。控制转移到该`catch`子句块的Java虚拟机代码。如果抛出的值与`catchTwo`的任何`catch`子句的参数不匹配，Java虚拟机将重新抛出该值，而不会调用`catchTwo`中的任何`catch`子句的代码。

Nested try-catch statements are compiled very much like a try statement with multiple catch clauses:

嵌套的`try-catch`语句的编译方式与具有多个`catch`子句的`try`语句非常相似：

```java
void nestedCatch() {
    try {
        try {
            tryItOut();
        } catch (TestExc1 e) {
            handleExc1(e);
        }
    } catch (TestExc2 e) {
        handleExc2(e);
    }
}
```

becomes:

编译为：

```java
Method void nestedCatch()
0 aload_0
1 invokevirtual #8
4 return
5 astore_1
6 aload_0
7 aload_1
8 invokevirtual #7
11 return
12 astore_1
13 aload_0
14 aload_1
15 invokevirtual #6
18 return
Exception table:
From To Target
0 4 5
0 12 12
```

The nesting of catch clauses is represented only in the exception table. The Java Virtual Machine does not enforce nesting of or any ordering of the exception table entries (§2.10). However, because try-catch constructs are structured, a compiler can always order the entries of the exception handler table such that, for any thrown exception and any program counter value in that method, the first exception handler that matches the thrown exception corresponds to the innermost matching catch clause.

`catch`子句的嵌套只在异常表中表示。Java虚拟机不强制执行异常表条目的嵌套或任何顺序（§2.10）。然而，由于`try-catch`结构是结构化的，编译器总是可以对异常处理器表的条目进行排序，以便对于在该方法中抛出的任何异常和任何程序计数器值，第一个与抛出异常匹配的异常处理器对应于最内层的匹配`catch`子句。

For instance, if the invocation of `tryItOut` (at index 1) threw an instance of `TestExc1`, it would be handled by the catch clause that invokes `handleExc1`. This is so even though the exception occurs within the bounds of the outer catch clause (catching `TestExc2`) and even though that outer catch clause might otherwise have been able to handle the thrown value.

例如，如果`tryItOut`（在索引1处的调用）抛出了`TestExc1`的实例，它将由调用`handleExc1`的`catch`子句处理。这种情况即使发生在外部`catch`子句（捕获`TestExc2`）的范围内，也仍然适用，即使外部`catch`子句本来可以处理该抛出的值。

As a subtle point, note that the range of a catch clause is inclusive on the "from" end and exclusive on the "to" end (§4.7.3). Thus, the exception table entry for the catch clause catching `TestExc1` does not cover the return instruction at offset 4. However, the exception table entry for the catch clause catching `TestExc2` does cover the return instruction at offset 11. Return instructions within nested catch clauses are included in the range of instructions covered by nesting catch clauses.

需要注意的是，`catch`子句的范围在“from”端是包含的，在“to”端是不包含的（§4.7.3）。因此，捕获`TestExc1`的`catch`子句的异常表条目不覆盖偏移量为4的return指令。然而，捕获`TestExc2`的`catch`子句的异常表条目确实覆盖了偏移量为11的return指令。嵌套`catch`子句中的return指令包含在嵌套`catch`子句覆盖的指令范围内。

---
## 3.13 Compiling finally

(This section assumes a compiler generates class files with version number 50.0 or below, so that the `jsr` instruction may be used. See also §4.10.2.5.)

（本节假设编译器生成版本号为50.0或以下的类文件，因此可以使用`jsr`指令。另见§4.10.2.5。）

Compilation of a try-finally statement is similar to that of try-catch. Prior to transferring control outside the try statement, whether that transfer is normal or abrupt, because an exception has been thrown, the finally clause must first be executed. For this simple example:

`try-finally`语句的编译类似于`try-catch`。在将控制权转移到`try`语句之外之前，无论这种转移是正常的还是突然的（例如抛出异常），都必须首先执行`finally`子句。以下是一个简单的例子：

```java
void tryFinally() {
    try {
        tryItOut();
    } finally {
        wrapItUp();
    }
}
```

the compiled code is:

编译后的代码为：

```java
Method void tryFinally()
0 aload_0
1 invokevirtual #6
4 jsr 14
7 return
8 astore_1
9 jsr 14
12 aload_1
13 athrow
14 astore_2
15 aload_0
16 invokevirtual #5
19 ret 2
Exception table:
From To Target
0 4 8 any
```

There are four ways for control to pass outside of the try statement: by falling through the bottom of that block, by returning, by executing a break or continue statement, or by raising an exception. If `tryItOut` returns without raising an exception, control is transferred to the finally block using a `jsr` instruction. The `jsr 14` instruction at index 4 makes a "subroutine call" to the code for the finally block at index 14 (the finally block is compiled as an embedded subroutine). When the finally block completes, the `ret 2` instruction returns control to the instruction following the `jsr` instruction at index 4.

有四种方式可以将控制权传递到`try`语句之外：通过块的底部正常退出、通过返回、通过执行`break`或`continue`语句，或通过引发异常。如果`tryItOut`在不引发异常的情况下返回，控制将通过`jsr`指令转移到`finally`块。索引4处的`jsr 14`指令对索引14处的`finally`块代码进行了“子程序调用”（`finally`块被编译为嵌入式子程序）。当`finally`块完成时，`ret 2`指令将控制权返回到索引4处`jsr`指令之后的指令。

In more detail, the subroutine call works as follows: The `jsr` instruction pushes the address of the following instruction (return at index 7) onto the operand stack before jumping. The `astore_2` instruction that is the jump target stores the address on the operand stack into local variable 2. The code for the finally block (in this case the `aload_0` and `invokevirtual` instructions) is run. Assuming execution of that code completes normally, the `ret` instruction retrieves the address from local variable 2 and resumes execution at that address. The return instruction is executed, and `tryFinally` returns normally.

更详细地说，子程序调用的工作原理如下：`jsr`指令在跳转之前将下一个指令的地址（索引7处的`return`）压入操作数栈。作为跳转目标的`astore_2`指令将操作数栈上的地址存储到局部变量2中。然后执行`finally`块的代码（在本例中是`aload_0`和`invokevirtual`指令）。假设该代码正常完成，`ret`指令会从局部变量2中检索地址，并在该地址恢复执行。接着执行`return`指令，`tryFinally`方法正常返回。

A try statement with a finally clause is compiled to have a special exception handler, one that can handle any exception thrown within the try statement. If `tryItOut` throws an exception, the exception table for `tryFinally` is searched for an appropriate exception handler. The special handler is found, causing execution to continue at index 8. The `astore_1` instruction at index 8 stores the thrown value into local variable 1. The following `jsr` instruction does a subroutine call to the code for the finally block. Assuming that code returns normally, the `aload_1` instruction at index 12 pushes the thrown value back onto the operand stack, and the following `athrow` instruction rethrows the value.

带有`finally`子句的`try`语句被编译为具有特殊异常处理程序的形式，这种处理程序可以处理`try`语句中抛出的任何异常。如果`tryItOut`抛出异常，会在`tryFinally`的异常表中搜索合适的异常处理程序。找到特殊处理程序后，执行会继续在索引8处进行。索引8处的`astore_1`指令将抛出的值存储到局部变量1中。接下来的`jsr`指令会对子程序进行调用，执行`finally`块的代码。假设该代码正常返回，索引12处的`aload_1`指令会将抛出的值重新压入操作数栈，随后`athrow`指令会重新抛出该值。

Compiling a try statement with both a catch clause and a finally clause is more complex:

编译带有`catch`和`finally`子句的`try`语句会更加复杂：

```java
void tryCatchFinally() {
    try {
        tryItOut();
    } catch (TestExc e) {
        handleExc(e);
    } finally {
        wrapItUp();
    }
}
```

becomes:

编译为：

```java
Method void tryCatchFinally()
0 aload_0
1 invokevirtual #4
4 goto 16
7 astore_3
8 aload_0
9 aload_3
10 invokevirtual #6
13 goto 16
16 jsr 26
19 return
20 astore_1
21 jsr 26
24 aload_1
25 athrow
26 astore_2
27 aload_0
28 invokevirtual #5
31 ret 2
Exception table:
From To Target
0 4 7
0 16 20
```

If the try statement completes normally, the `goto` instruction at index 4 jumps to the subroutine call for the finally block at index 16. The finally block at index 26 is executed, control returns to the return instruction at index 19, and `tryCatchFinally` returns normally.

如果`try`语句正常完成，索引4处的`goto`指令会跳转到索引16处`finally`块的子程序调用。索引26处的`finally`块被执行，控制权返回索引19处的`return`指令，`tryCatchFinally`方法正常返回。

If `tryItOut` throws an instance of `TestExc`, the first (innermost) applicable exception handler in the exception table is chosen to handle the exception. The code for that exception handler, beginning at index 7, passes the thrown value to `handleExc` and on its return makes the same subroutine call to the finally block at index 26 as in the normal case. If an exception is not thrown by `handleExc`, `tryCatchFinally` returns normally.

如果`tryItOut`抛出`TestExc`的实例，异常表中的第一个（最内层的）适用异常处理程序会被选择处理该异常。该异常处理程序的代码从索引7开始，将抛出的值传递给`handleExc`，并在返回时对索引26处的`finally`块进行与正常情况相同的子程序调用。如果`handleExc`没有抛出异常，`tryCatchFinally`方法正常返回。

If `tryItOut` throws a value that is not an instance of `TestExc` or if `handleExc` itself throws an exception, the condition is handled by the second entry in the exception table, which handles any value thrown between indices 0 and 16. That exception handler transfers control to index 20, where the thrown value is first stored in local variable 1. The code for the finally block at index 26 is called as a subroutine. If it returns, the thrown value is retrieved from local variable 1 and rethrown using the `athrow` instruction. If a new value is thrown during execution of the finally clause, the finally clause aborts, and `tryCatchFinally` returns abruptly, throwing the new value to its invoker.

如果`tryItOut`抛出的值不是`TestExc`的实例，或者`handleExc`本身抛出了异常，则由异常表中的第二个条目处理该情况，该条目处理在索引0到16之间抛出的任何值。该异常处理程序将控制权转移到索引20，在那里首先将抛出的值存储到局部变量1中。然后调用索引26处`finally`块的代码作为子程序。如果它返回，抛出的值会从局部变量1中检索出来，并使用`athrow`指令重新抛出。如果在`finally`子句的执行期间抛出了新值，`finally`子句会中止，并且`tryCatchFinally`会突然返回，将新值抛给其调用者。

---
## 3.14 Synchronization

Synchronization in the Java Virtual Machine is implemented by monitor entry and exit, either explicitly (by use of the `monitorenter` and `monitorexit` instructions) or implicitly (by the method invocation and return instructions).

在Java虚拟机中，同步是通过进入和退出监视器来实现的，可以显式地通过使用`monitorenter`和`monitorexit`指令，也可以隐式地通过方法调用和返回指令来实现。

For code written in the Java programming language, perhaps the most common form of synchronization is the `synchronized` method. A `synchronized` method is not normally implemented using `monitorenter` and `monitorexit`. Rather, it is simply distinguished in the run-time constant pool by the `ACC_SYNCHRONIZED` flag, which is checked by the method invocation instructions (§2.11.10).

对于用Java编写的代码，最常见的同步形式可能是`synchronized`方法。通常，同步方法不会使用`monitorenter`和`monitorexit`实现。相反，它只是通过运行时常量池中的`ACC_SYNCHRONIZED`标志来区分，方法调用指令会检查该标志（§2.11.10）。

The `monitorenter` and `monitorexit` instructions enable the compilation of `synchronized` statements. For example:

`monitorenter`和`monitorexit`指令允许编译`synchronized`语句。例如：

```java
void onlyMe(Foo f) {
    synchronized(f) {
        doSomething();
    }
}
```

is compiled to:

编译为：

```java
Method void onlyMe(Foo)
0 aload_1
1 dup
2 astore_2
3 monitorenter
4 aload_0
5 invokevirtual #5
8 aload_2
9 monitorexit
10 goto 18
13 astore_3
14 aload_2
15 monitorexit
16 aload_3
17 athrow
18 return
Exception table:
From To Target
4 10 13 any
13 16 13 any
```

The compiler ensures that at any method invocation completion, a `monitorexit` instruction will have been executed for each `monitorenter` instruction executed since the method invocation. This is the case whether the method invocation completes normally (§2.6.4) or abruptly (§2.6.5). To enforce proper pairing of `monitorenter` and `monitorexit` instructions on abrupt method invocation completion, the compiler generates exception handlers (§2.10) that will match any exception and whose associated code executes the necessary `monitorexit` instructions.

编译器确保在任何方法调用完成时，每个自方法调用以来执行的`monitorenter`指令都会有一个对应的`monitorexit`指令执行。这适用于方法调用无论是正常完成（§2.6.4）还是突然完成（§2.6.5）。为了在方法调用突然完成时强制执行`monitorenter`和`monitorexit`指令的正确配对，编译器生成异常处理程序（§2.10），这些处理程序将匹配任何异常，并执行必要的`monitorexit`指令。

---
## 3.15 Annotations

The representation of annotations in class files is described in §4.7.16-§4.7.22. These sections make it clear how to represent annotations on declarations of classes, interfaces, fields, methods, method parameters, and type parameters, as well as annotations on types used in those declarations. Annotations on package declarations require additional rules, given here.

类文件中注解的表示在§4.7.16-§4.7.22中进行了描述。这些部分清楚地说明了如何表示类、接口、字段、方法、方法参数和类型参数声明上的注解，以及这些声明中使用的类型上的注解。包声明上的注解需要额外的规则，这里给出了这些规则。

When the compiler encounters an annotated package declaration that must be made available at run time, it emits a class file with the following properties:

当编译器遇到必须在运行时可用的带注解包声明时，它会生成具有以下属性的类文件：

- The class file represents an interface, that is, the `ACC_INTERFACE` and `ACC_ABSTRACT` flags of the `ClassFile` structure are set (§4.1).
- If the class file version number is less than 50.0, then the ACC_SYNTHETIC flag is unset; if the class file version number is 50.0 or above, then the ACC_SYNTHETIC flag is set.
- The interface has package access (JLS §6.6.1).
- The interface's name is the internal form (§4.2.1) of `package-name.package-info`.
- The interface has no superinterfaces.
- The interface's only members are those implied by The Java Language Specification, Java SE 8 Edition (JLS §9.2).
- The annotations on the package declaration are stored as `RuntimeVisibleAnnotations` and `RuntimeInvisibleAnnotations` attributes in the attributes table of the `ClassFile` structure.

- 类文件表示一个接口，即设置了`ClassFile`结构的`ACC_INTERFACE`和`ACC_ABSTRACT`标志（§4.1）。
- 如果类文件的版本号小于50.0，则`ACC_SYNTHETIC`标志未设置；如果类文件版本号为50.0或更高，则设置`ACC_SYNTHETIC`标志。
- 该接口具有包访问权限（JLS §6.6.1）。
- 该接口的名称是`package-name.package-info`的内部形式（§4.2.1）。
- 该接口没有超接口。
- 该接口的唯一成员是《Java语言规范，Java SE 8版》（JLS §9.2）中规定的那些。
- 包声明上的注解存储为`ClassFile`结构中属性表中的`RuntimeVisibleAnnotations`和`RuntimeInvisibleAnnotations`属性。