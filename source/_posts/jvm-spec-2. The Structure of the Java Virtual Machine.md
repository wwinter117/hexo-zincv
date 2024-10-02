---
title: jvm-spec-2. The Structure of the Java Virtual Machine
tags:
  - 官方文档翻译
  - jvm
date: 2022-9-1 22:46:49
---

This document specifies an abstract machine. It does not describe any particular implementation of the Java Virtual Machine.

本文档指定了一个抽象机器，并未描述Java虚拟机的任何特定实现。

To implement the Java Virtual Machine correctly, you need only be able to read the class file format and correctly perform the operations specified therein. Implementation details that are not part of the Java Virtual Machine's specification would unnecessarily constrain the creativity of implementors. For example, the memory layout of run-time data areas, the garbage-collection algorithm used, and any internal optimization of the Java Virtual Machine instructions (for example, translating them into machine code) are left to the discretion of the implementor.

为了正确实现Java虚拟机，您只需能够读取类文件格式并正确执行其中指定的操作。那些不属于Java虚拟机规范的实现细节将不必要地限制实现者的创造力。例如，运行时数据区的内存布局、使用的垃圾回收算法，以及Java虚拟机指令的任何内部优化（例如将其翻译为机器代码）都由实现者自行决定。

All references to Unicode in this specification are given with respect to The Unicode Standard, Version 6.0.0, available at http://www.unicode.org/.

本文档中提到的所有Unicode引用均基于《Unicode标准》第6.0.0版，你可以在[http://www.unicode.org/](http://www.unicode.org/)获取。

---
## 2.1 The class File Format（class文件格式）

Compiled code to be executed by the Java Virtual Machine is represented using a hardware- and operating system-independent binary format, typically (but not necessarily) stored in a file, known as the class file format. The class file format precisely defines the representation of a class or interface, including details such as byte ordering that might be taken for granted in a platform-specific object file format.

由Java虚拟机执行的编译代码使用一种与硬件和操作系统无关的二进制格式表示，这种格式通常（但不一定）存储在一个文件中，称为类文件格式。类文件格式精确地定义了类或接口的表示方式，包括诸如字节顺序等在特定平台的对象文件格式中可能被默认的细节。

Chapter 4, "The class File Format", covers the class file format in detail.

第四章，“The class File Format”，详细介绍了类文件格式。

---  
## 2.2 Data Types（数据类型）

Like the Java programming language, the Java Virtual Machine operates on two kinds of types: **primitive types and reference types**. There are, correspondingly, two kinds of values that can be stored in variables, passed as arguments, returned by methods, and operated upon: **primitive values and reference values**.

与Java编程语言类似，Java虚拟机处理两种类型：**原始类型（*primitive types*）和引用类型（*reference types*）**。相应地，也有两种值可以存储在变量中、作为参数传递、由方法返回以及进行操作：**原始值（*primitive values*）和引用值（*reference values*）**。

The Java Virtual Machine expects that nearly all type checking is done prior to run time, typically by a compiler, and does not have to be done by the Java Virtual Machine itself. Values of primitive types need not be tagged or otherwise be inspectable to determine their types at run time, or to be distinguished from values of reference types. Instead, the instruction set of the Java Virtual Machine distinguishes its operand types using instructions intended to operate on values of specific types. For instance, iadd, ladd, fadd, and dadd are all Java Virtual Machine instructions that add two numeric values and produce numeric results, but each is specialized for its operand type: int, long, float, and double, respectively. For a summary of type support in the Java Virtual Machine instruction set, see §2.11.1.

Java虚拟机假设几乎所有的类型检查在运行时之前都已经完成，通常由编译器来进行，而不需要由Java虚拟机本身完成。原始类型的值不需要在运行时进行标记或其他方式检查其类型，或者将它们与引用类型的值区分开来。相反，Java虚拟机的指令集通过专门用于特定类型值的指令来区分其操作数类型。例如，`iadd`、`ladd`、`fadd`和`dadd`都是Java虚拟机指令，这些指令用于将两个数值相加并产生数值结果，但它们各自专门用于其操作数类型：`int`、`long`、`float`和`double`。关于Java虚拟机指令集中类型支持的总结，请参见§2.11.1。
  
The Java Virtual Machine contains explicit support for objects. An object is either a dynamically allocated class instance or an array. A reference to an object is considered to have Java Virtual Machine type reference. Values of type reference can be thought of as pointers to objects. More than one reference to an object may exist. Objects are always operated on, passed, and tested via values of type reference.

Java虚拟机包含对对象的明确支持。对象要么是动态分配的类实例，要么是数组。对对象的引用被认为具有Java虚拟机的引用类型。引用类型的值可以被看作是指向对象的指针。多个引用可能指向同一个对象。对象总是通过引用类型的值进行操作、传递和测试。

---  
## 2.3 Primitive Types and Values（基本数据类型和值）

The primitive data types supported by the Java Virtual Machine are the numeric types, the boolean type (§2.3.4), and the returnAddress type (§2.3.3).

Java虚拟机支持的基本数据类型包括数值类型、布尔类型（见第2.3.4节）和返回地址类型（见第2.3.3节）。

The numeric types consist of the integral types (§2.3.1) and the floating-point types (§2.3.2).  

数值类型包括整型（见第2.3.1节）和浮点型（见第2.3.2节）。

The integral types are:
  
- byte, whose values are 8-bit signed two's-complement integers, and whose default value is zero  
- short, whose values are 16-bit signed two's-complement integers, and whose default value is zero  
- int, whose values are 32-bit signed two's-complement integers, and whose default value is zero  
- long, whose values are 64-bit signed two's-complement integers, and whose default value is zero  
- char, whose values are 16-bit unsigned integers representing Unicode code points in the Basic Multilingual Plane, encoded with UTF-16, and whose default value is the null code point ('\u0000')  

整型包括：
  
- `byte`：值为8位有符号二进制补码整数，默认值为0。
- `short`：值为16位有符号二进制补码整数，默认值为0。
- `int`：值为32位有符号二进制补码整数，默认值为0。
- `long`：值为64位有符号二进制补码整数，默认值为0。
- `char`：值为16位无符号整数，表示基本多语言平面的Unicode代码点，以UTF-16编码，默认值为空代码点('\u0000')。

The floating-point types are:
  
- float, whose values are elements of the float value set or, where supported, the float-extended-exponent value set, and whose default value is positive zero
- double, whose values are elements of the double value set or, where supported, the double-extended-exponent value set, and whose default value is positive zero

浮点型包括：

- `float`：值为`float`值集中的元素或在支持的情况下为`float-extended-exponent`值集中的元素，默认值为正零。
- `double`：值为`double`值集中的元素或在支持的情况下为`double-extended-exponent`值集中的元素，默认值为正零。

The values of the boolean type encode the truth values true and false, and the default value is false.

布尔类型的值编码了真和假的真值，默认值为假。

The First Edition of The Java® Virtual Machine Specification did not consider boolean to be a Java Virtual Machine type. However, boolean values do have limited support in the Java Virtual Machine. The Second Edition of The Java® Virtual Machine Specification clarified the issue by treating boolean as a type.

Java虚拟机规范的第一版没有考虑将布尔类型作为Java虚拟机类型。然而，布尔值在Java虚拟机中有一定的支持。Java虚拟机规范的第二版通过将布尔类型作为一种类型来澄清了这个问题。

The values of the returnAddress type are pointers to the opcodes of Java Virtual Machine instructions. Of the primitive types, only the returnAddress type is not directly associated with a Java programming language type.

返回地址类型的值是指向Java虚拟机指令操作码的指针。在基本类型中，只有返回地址类型没有直接与Java编程语言的类型关联。

### 2.3.1 Integral Types and Values（整型和值）

The values of the integral types of the Java Virtual Machine are:

+ For byte, from -128 to 127 (-2^7 to 2^7 - 1), inclusive
+ For short, from -32768 to 32767 (-2^15 to 2^15 - 1), inclusive
+ For int, from -2147483648 to 2147483647 (-2^31 to 2^31 - 1), inclusive
+ For long, from -9223372036854775808 to 9223372036854775807 (-2^63 to 2^63 - 1), inclusive
+ For char, from 0 to 65535 inclusive

Java虚拟机的整型值包括：

- `byte`：范围从-128到127（-2^7到2^7 - 1），包含在内。
- `short`：范围从-32768到32767（-2^15到2^15 - 1），包含在内。
- `int`：范围从-2147483648到2147483647（-2^31到2^31 - 1），包含在内。
- `long`：范围从-9223372036854775808到9223372036854775807（-2^63到2^63 - 1），包含在内。
- `char`：范围从0到65535，包含在内。

### 2.3.2 Floating-Point Types, Value Sets, and Values（浮点型，浮点值集和值）

The floating-point types are float and double, which are conceptually associated with the 32-bit single-precision and 64-bit double-precision format IEEE 754 values and operations as specified in IEEE Standard for Binary Floating-Point Arithmetic (ANSI/IEEE Std. 754-1985, New York).

浮点类型包括`float`和`double`，它们在概念上与IEEE 754标准中规定的32位单精度和64位双精度格式的值和操作相关。

The IEEE 754 standard includes not only positive and negative sign-magnitude numbers, but also positive and negative zeros, positive and negative infinities, and a special Not-a-Number value (hereafter abbreviated as "NaN"). The NaN value is used to represent the result of certain invalid operations such as dividing zero by zero.

IEEE 754标准不仅包括正负符号-幅度数字，还包括正负零、正负无穷大，以及特殊的非数字（NaN）值。NaN值用于表示某些无效操作的结果，例如零除以零。

Every implementation of the Java Virtual Machine is required to support two standard sets of floating-point values, called the float value set and the double value set. In addition, an implementation of the Java Virtual Machine may, at its option, support either or both of two extended-exponent floating-point value sets, called the float-extended-exponent value set and the double-extended-exponent value set. These extended-exponent value sets may, under certain circumstances, be used instead of the standard value sets to represent the values of type float or double.

每个Java虚拟机实现都需要支持两种标准浮点值集，称为`float`值集和`double`值集。此外，Java虚拟机的实现可以选择支持一种或两种扩展指数浮点值集，分别称为`float-extended-exponent`值集和`double-extended-exponent`值集。在某些情况下，这些扩展指数值集可以代替标准值集来表示类型`float`或`double`的值。

The finite nonzero values of any floating-point value set can all be expressed in the form s ⋅ m ⋅ 2^(e - N + 1), where s is +1 or -1, m is a positive integer less than 2^N, and e is an integer between E_min = -(2^K-1-2) and E_max = 2^K-1-1, inclusive, and where N and K are parameters that depend on the value set. Some values can be represented in this form in more than one way; for example, supposing that a value v in a value set might be represented in this form using certain values for s, m, and e, then if it happened that m were even and e were less than 2^K-1, one could halve m and increase e by 1 to produce a second representation for the same value v. A representation in this form is called normalized if m ≥ 2^N-1; otherwise the representation is said to be denormalized. If a value in a value set cannot be represented in such a way that m ≥ 2^N-1, then the value is said to be a denormalized value, because it has no normalized representation.

任何浮点值集的有限非零值都可以表示为`s ⋅ m ⋅ 2^(e - N + 1)`，其中`s`为+1或-1，`m`为小于2^N的正整数，`e`为E_min = -(2^K-1-2)和E_max = 2^K-1-1之间的整数，且N和K是取决于值集的参数。一些值可以以多种方式表示在此形式中；例如，假设值集中的某个值v可以使用特定的s、m和e值表示，那么如果m为偶数且e小于2^K-1，可以将m减半并将e加1，以生成相同值v的第二种表示形式。如果m ≥ 2^N-1，则此形式的表示称为规范化；否则，表示被称为非规范化。如果一个值集中的值无法以m ≥ 2^N-1的方式表示，则该值被称为非规范化值，因为它没有规范化表示。

The constraints on the parameters N and K (and on the derived parameters E_min and E_max) for the two required and two optional floating-point value sets are summarized in Table 2.3.2-A.  

表2.3.2-A总结了两个必需和两个可选浮点值集的参数N和K（以及导出的参数E_min和E_max）的约束。

Table 2.3.2-A. Floating-point value set parameters

| Parameter | float | float-extended-exponent | double | double-extended-exponent |  
|---------------|-----------|-----------------------------|------------|-----------------------------|  
| N             | 24        | 24                          | 53         | 53                          |  
| K             | 8         | ≥ 11                        | 11         | ≥ 15                        |  
| E_max         | +127      | ≥ +1023                     | +1023      | ≥ +16383                    |  
| E_min         | -126      | ≤ -1022                     | -1022      | ≤ -16382                    |  

Where one or both extended-exponent value sets are supported by an implementation, then for each supported extended-exponent value set there is a specific implementation-dependent constant K, whose value is constrained by Table 2.3.2-A; this value K in turn dictates the values for E_min and E_max.

如果一个实现支持一种或两种扩展指数值集，那么对于每种支持的扩展指数值集，都有一个特定的实现依赖常量K，其值受表2.3.2-A的约束；这个K值又决定了E_min和E_max的值。

Each of the four value sets includes not only the finite nonzero values that are ascribed to it above, but also the five values positive zero, negative zero, positive infinity, negative infinity, and NaN.

四个值集中的每一个不仅包括上面提到的有限非零值，还包括五个值：正零、负零、正无穷大、负无穷大和NaN。
  
Note that the constraints in Table 2.3.2-A are designed so that every element of the float value set is necessarily also an element of the float-extended-exponent value set, the double value set, and the double-extended-exponent value set. Likewise, each element of the double value set is necessarily also an element of the double-extended-exponent value set. Each extended-exponent value set has a larger range of exponent values than the corresponding standard value set, but does not have more precision.

注意，表2.3.2-A中的约束设计为使float值集中的每个元素都必然也是float-extended-exponent值集、double值集和double-extended-exponent值集中的元素。同样，double值集中的每个元素也必然是double-extended-exponent值集中的元素。每个扩展指数值集的指数值范围都比对应的标准值集更大，但并没有更高的精度。

The elements of the float value set are exactly the values that can be represented using the single floating-point format defined in the IEEE 754 standard, except that there is only one NaN value (IEEE 754 specifies 2^24-2 distinct NaN values). The elements of the double value set are exactly the values that can be represented using the double floating-point format defined in the IEEE 754 standard, except that there is only one NaN value (IEEE 754 specifies 2^53-2 distinct NaN values). Note, however, that the elements of the float-extended-exponent and double-extended-exponent value sets defined here do not correspond to the values that can be represented using IEEE 754 single extended and double extended formats, respectively. This specification does not mandate a specific representation for the values of the floating-point value sets except where floating-point values must be represented in the class file format (§4.4.4, §4.4.5).

float值集中的元素完全是可以使用IEEE 754标准中定义的单精度浮点格式表示的值，除了只有一个NaN值（IEEE 754指定了2^24-2个不同的NaN值）。double值集中的元素完全是可以使用IEEE 754标准中定义的双精度浮点格式表示的值，除了只有一个NaN值（IEEE 754指定了2^53-2个不同的NaN值）。然而，注意这里定义的float-extended-exponent和double-extended-exponent值集中的元素并不对应于可以使用IEEE 754单扩展和双扩展格式表示的值。此规范并未规定浮点值集的特定表示，除非浮点值必须在类文件格式中表示（§4.4.4，§4.4.5）。

The float, float-extended-exponent, double, and double-extended-exponent value sets are not types. It is always correct for an implementation of the Java Virtual Machine to use an element of the float value set to represent a value of type float; however, it may be permissible in certain contexts for an implementation to use an element of the float-extended-exponent value set instead. Similarly, it is always correct for an implementation to use an element of the double value set to represent a value of type double; however, it may be permissible in certain contexts for an implementation to use an element of the double-extended-exponent value set instead.

`float`、`float-extended-exponent`、`double`和`double-extended-exponent`值集不是类型。对于Java虚拟机的实现来说，使用`float`值集中的元素来表示类型为`float`的值总是正确的；然而，在某些情况下，允许实现使用`float-extended-exponent`值集中的元素来代替。同样，使用`double`值集中的元素来表示类型为`double`的值总是正确的；然而，在某些情况下，允许实现使用`double-extended-exponent`值集中的元素来代替。

Except for NaNs, values of the floating-point value sets are ordered. When arranged from smallest to largest, they are negative infinity, negative finite values, positive and negative zero, positive finite values, and positive infinity.

除了NaN以外，浮点值集中的值是有序的。当从最小到最大排列时，它们依次是负无穷大、负有限值、正负零、正有限值和正无穷大。

Floating-point positive zero and floating-point negative zero compare as equal, but there are other operations that can distinguish them; for example, dividing 1.0 by 0.0 produces positive infinity, but dividing 1.0 by -0.0 produces negative infinity.

浮点正零和浮点负零比较时相等，但有其他操作可以区分它们；例如，1.0除以0.0会产生正无穷大，但1.0除以-0.0会产生负无穷大。

NaNs are unordered, so numerical comparisons and tests for numerical equality have the value false if either or both of their operands are NaN. In particular, a test for numerical equality of a value against itself has the value false if and only if the value is NaN. A test for numerical inequality has the value true if either operand is NaN.

NaN是无序的，因此如果操作数之一或两个都是NaN，则数值比较和数值相等的测试结果为false。特别地，如果值是NaN，那么对自身的数值相等测试的结果为false，且只有在值是NaN时结果为false。如果任一操作数是NaN，则数值不等测试的结果为true。

### 2.3.3 The returnAddress Type and Values（返回地址类型和值）

The returnAddress type is used by the Java Virtual Machine's jsr, ret, and jsr_w instructions (§jsr, §ret, §jsr_w). The values of the returnAddress type are pointers to the opcodes of Java Virtual Machine instructions. Unlike the numeric primitive types, the returnAddress type does not correspond to any Java programming language type and cannot be modified by the running program.

返回地址类型由Java虚拟机的`jsr`、`ret`和`jsr_w`指令使用（§jsr，§ret，§jsr_w）。返回地址类型的值是指向Java虚拟机指令操作码的指针。与数值原始类型不同，返回地址类型不对应于任何Java编程语言类型，并且不能由运行的程序修改。

### 2.3.4 The boolean Type（布尔类型）

Although the Java Virtual Machine defines a boolean type, it only provides very limited support for it. There are no Java Virtual Machine instructions solely dedicated to operations on boolean values. Instead, expressions in the Java programming language that operate on boolean values are compiled to use values of the Java Virtual Machine int data type.  

尽管Java虚拟机定义了布尔类型，但它对布尔类型的支持非常有限。Java虚拟机没有专门用于操作布尔值的指令。相反，Java编程语言中操作布尔值的表达式被编译为使用Java虚拟机的`int`数据类型的值。

The Java Virtual Machine does directly support boolean arrays. Its newarray instruction (§newarray) enables creation of boolean arrays. Arrays of type boolean are accessed and modified using the byte array instructions baload and bastore (§baload, §bastore).  

Java虚拟机直接支持布尔数组。它的`newarray`指令（§newarray）允许创建布尔数组。布尔类型的数组通过字节数组指令`baload`和`bastore`（§baload，§bastore）进行访问和修改。

In Oracle’s Java Virtual Machine implementation, boolean arrays in the Java programming language are encoded as Java Virtual Machine byte arrays, using 8 bits per boolean element.  

在Oracle的Java虚拟机实现中，Java编程语言中的布尔数组被编码为Java虚拟机的字节数组，每个布尔元素使用8位。

The Java Virtual Machine encodes boolean array components using 1 to represent true and 0 to represent false. Where Java programming language boolean values are mapped by compilers to values of Java Virtual Machine type int, the compilers must use the same encoding.  

Java虚拟机使用1编码布尔数组组件来表示true，使用0来表示false。当Java编程语言的布尔值被编译器映射到Java虚拟机的`int`类型值时，编译器必须使用相同的编码。  

---  
### 2.4 Reference Types and Values（引用类型和值集）  

There are three kinds of reference types: class types, array types, and interface types. Their values are references to dynamically created class instances, arrays, or class instances or arrays that implement interfaces, respectively.  

有三种引用类型：类类型、数组类型和接口类型。它们的值分别是对动态创建的类实例、数组或实现接口的类实例或数组的引用。  

An array type consists of a component type with a single dimension (whose length is not given by the type). The component type of an array type may itself be an array type. If, starting from any array type, one considers its component type, and then (if that is also an array type) the component type of that type, and so on, eventually one must reach a component type that is not an array type; this is called the element type of the array type. The element type of an array type is necessarily either a primitive type, or a class type, or an interface type.  

数组类型由具有单一维度的组件类型组成（其长度不是由类型给出的）。数组类型的组件类型本身也可以是数组类型。如果从任何数组类型开始，考虑其组件类型，然后（如果那也是数组类型）考虑该类型的组件类型，依此类推，最终必须到达一个不是数组类型的组件类型；这被称为数组类型的元素类型。数组类型的元素类型必须是基本类型、类类型或接口类型。  

A reference value may also be the special null reference, a reference to no object, which will be denoted here by null. The null reference initially has no run-time type, but may be cast to any type. The default value of a reference type is null.  

引用值还可以是特殊的`null`引用，即对无对象的引用，这里将其表示为`null`。`null`引用最初没有运行时类型，但可以转换为任何类型。引用类型的默认值是`null`。  

This specification does not mandate a concrete value encoding null.  

此规范并未规定具体的`null`值编码。  

---  
## 2.5 Run-Time Data Areas（运行时数据区域）  
  
The Java Virtual Machine defines various run-time data areas that are used during execution of a program. Some of these data areas are created on Java Virtual Machine start-up and are destroyed only when the Java Virtual Machine exits. Other data areas are per thread. Per-thread data areas are created when a thread is created and destroyed when the thread exits.  

Java虚拟机定义了在程序执行期间使用的各种运行时数据区。这些数据区中的一些是在Java虚拟机启动时创建的，只有在Java虚拟机退出时才被销毁。其他数据区是每个线程的。每个线程的数据区是在线程创建时创建的，并在线程退出时销毁。  

### 2.5.1 The pc Register（pc寄存器）  

The Java Virtual Machine can support many threads of execution at once (JLS §17). Each Java Virtual Machine thread has its own pc (program counter) register. At any point, each Java Virtual Machine thread is executing the code of a single method, namely the current method (§2.6) for that thread. If that method is not native, the pc register contains the address of the Java Virtual Machine instruction currently being executed. If the method currently being executed by the thread is native, the value of the Java Virtual Machine's pc register is undefined. The Java Virtual Machine's pc register is wide enough to hold a returnAddress or a native pointer on the specific platform.  

Java虚拟机可以同时支持多个执行线程（JLS §17）。每个Java虚拟机线程都有自己的pc（程序计数器）寄存器。在任何时候，每个Java虚拟机线程都在执行一个方法的代码，即该线程的当前方法（§2.6）。如果该方法不是本机方法，则pc寄存器包含当前正在执行的Java虚拟机指令的地址。如果线程当前正在执行的方法是本机方法，则Java虚拟机的pc寄存器的值未定义。Java虚拟机的pc寄存器足够宽，可以在特定平台上容纳一个`returnAddress`或本地指针。  

### 2.5.2 Java Virtual Machine Stacks（Java虚拟机栈）  

Each Java Virtual Machine thread has a private Java Virtual Machine stack, created at the same time as the thread. A Java Virtual Machine stack stores frames (§2.6). A Java Virtual Machine stack is analogous to the stack of a conventional language such as C: it holds local variables and partial results, and plays a part in method invocation and return. Because the Java Virtual Machine stack is never manipulated directly except to push and pop frames, frames may be heap allocated. The memory for a Java Virtual Machine stack does not need to be contiguous.  

每个Java虚拟机线程都有一个私有的Java虚拟机栈，该栈与线程同时创建。Java虚拟机栈存储帧（§2.6）。Java虚拟机栈类似于传统语言（如C）的栈：它保存局部变量和部分结果，并在方法调用和返回中发挥作用。由于Java虚拟机栈除了推入和弹出帧之外，永远不会直接操作，因此帧可能在堆上分配。Java虚拟机栈的内存不需要是连续的。  

In the First Edition of The Java® Virtual Machine Specification, the Java Virtual Machine stack was known as the Java stack.  

在《Java®虚拟机规范》的第一版中，Java虚拟机栈被称为Java栈。  

This specification permits Java Virtual Machine stacks either to be of a fixed size or to dynamically expand and contract as required by the computation. If the Java Virtual Machine stacks are of a fixed size, the size of each Java Virtual Machine stack may be chosen independently when that stack is created.  

此规范允许Java虚拟机栈具有固定大小，或者根据计算需求动态扩展和收缩。如果Java虚拟机栈的大小是固定的，那么每个Java虚拟机栈的大小可以在创建该栈时独立选择。  

A Java Virtual Machine implementation may provide the programmer or the user control over the initial size of Java Virtual Machine stacks, as well as, in the case of dynamically expanding or contracting Java Virtual Machine stacks, control over the maximum and minimum sizes.  

Java虚拟机的实现可以提供给程序员或用户对Java虚拟机栈初始大小的控制，以及在动态扩展或收缩的情况下，对最大和最小大小的控制。 

The following exceptional conditions are associated with Java Virtual Machine stacks:  

+ If the computation in a thread requires a larger Java Virtual Machine stack than is permitted, the Java Virtual Machine throws a StackOverflowError.  

+ If Java Virtual Machine stacks can be dynamically expanded, and expansion is attempted but insufficient memory can be made available to effect the expansion, or if insufficient memory can be made available to create the initial Java Virtual Machine stack for a new thread, the Java Virtual Machine throws an OutOfMemoryError.  

以下异常条件与Java虚拟机栈相关：  

- 如果线程中的计算需要比允许的更大的Java虚拟机栈，Java虚拟机会抛出`StackOverflowError`。  
- 如果Java虚拟机栈可以动态扩展，并且尝试扩展但无法提供足够的内存以实现扩展，或者无法提供足够的内存来为新线程创建初始Java虚拟机栈，Java虚拟机会抛出`OutOfMemoryError`。  

### 2.5.3 Heap（堆）  

The Java Virtual Machine has a heap that is shared among all Java Virtual Machine threads. The heap is the run-time data area from which memory for all class instances and arrays is allocated.  
Java虚拟机具有一个堆，该堆在所有Java虚拟机线程之间共享。堆是所有类实例和数组的内存分配的运行时数据区。  

The heap is created on virtual machine start-up. Heap storage for objects is reclaimed by an automatic storage management system (known as a garbage collector); objects are never explicitly deallocated. The Java Virtual Machine assumes no particular type of automatic storage management system, and the storage management technique may be chosen according to the implementor's system requirements. The heap may be of a fixed size or may be expanded as required by the computation and may be contracted if a larger heap becomes unnecessary. The memory for the heap does not need to be contiguous.  

堆是在虚拟机启动时创建的。对象的堆存储由自动存储管理系统（称为垃圾回收器）回收；对象从不被显式释放。Java虚拟机不假设任何特定类型的自动存储管理系统，存储管理技术可以根据实现者的系统要求选择。堆可以是固定大小的，也可以根据计算需求扩展，如果不再需要更大的堆，还可以收缩。堆的内存不需要是连续的。  

A Java Virtual Machine implementation may provide the programmer or the user control over the initial size of the heap, as well as, if the heap can be dynamically expanded or contracted, control over the maximum and minimum heap size.  

Java虚拟机的实现可以提供给程序员或用户对堆初始大小的控制，以及如果堆可以动态扩展或收缩的情况下，对最大和最小堆大小的控制。  

The following exceptional condition is associated with the heap:  

+ If a computation requires more heap than can be made available by the automatic storage management system, the Java Virtual Machine throws an OutOfMemoryError.  

以下异常条件与堆相关：  

- 如果计算需要比自动存储管理系统可以提供的更多的堆，Java虚拟机会抛出`OutOfMemoryError`。  

### 2.5.4 Method Area（方法区）  

The Java Virtual Machine has a method area that is shared among all Java Virtual Machine threads. The method area is analogous to the storage area for compiled code of a conventional language or analogous to the "text" segment in an operating system process. It stores per-class structures such as the run-time constant pool, field and method data, and the code for methods and constructors, including the special methods (§2.9) used in class and instance initialization and interface initialization.  

Java虚拟机具有一个方法区，该方法区在所有Java虚拟机线程之间共享。方法区类似于传统语言的编译代码存储区或类似于操作系统进程中的“文本”段。它存储按类划分的结构，例如运行时常量池、字段和方法数据，以及方法和构造函数的代码，包括类和实例初始化以及接口初始化中使用的特殊方法（§2.9）。  

The method area is created on virtual machine start-up. Although the method area is logically part of the heap, simple implementations may choose not to either garbage collect or compact it. This specification does not mandate the location of the method area or the policies used to manage compiled code. The method area may be of a fixed size or may be expanded as required by the computation and may be contracted if a larger method area becomes unnecessary. The memory for the method area does not need to be contiguous.  

方法区在虚拟机启动时创建。尽管方法区在逻辑上是堆的一部分，但简单的实现可以选择不对其进行垃圾回收或压缩。此规范不强制规定方法区的位置或用于管理编译代码的策略。方法区可以是固定大小的，也可以根据计算需求扩展，如果不再需要更大的方法区，还可以收缩。方法区的内存不需要是连续的。  

A Java Virtual Machine implementation may provide the programmer or the user control over the initial size of the method area, as well as, in the case of a varying-size method area, control over the maximum and minimum method area size.  

Java虚拟机的实现可以提供给程序员或用户对方法区初始大小的控制，以及在方法区大小可变的情况下，对最大和最小方法区大小的控制。  

The following exceptional condition is associated with the method area:  

+ If memory in the method area cannot be made available to satisfy an allocation request, the Java Virtual Machine throws an OutOfMemoryError.  

以下异常条件与方法区相关：  

- 如果方法区中的内存无法提供以满足分配请求，Java虚拟机会抛出`OutOfMemoryError`。  

### 2.5.5 Run-Time Constant Pool（运行时常量池）

A run-time constant pool is a per-class or per-interface run-time representation of the constant_pool table in a class file (§4.4). It contains several kinds of constants, ranging from numeric literals known at compile-time to method and field references that must be resolved at run-time. The run-time constant pool serves a function similar to that of a symbol table for a conventional programming language, although it contains a wider range of data than a typical symbol table.  

运行时常量池是每个类或接口在运行时的常量池表（`constant_pool`）在类文件（§4.4）中的运行时表示。它包含几种类型的常量，从编译时已知的数值字面量到必须在运行时解析的方法和字段引用。运行时常量池的功能类似于传统编程语言的符号表，尽管它包含的数据范围比典型的符号表更广泛。  

Each run-time constant pool is allocated from the Java Virtual Machine's method area (§2.5.4). The run-time constant pool for a class or interface is constructed when the class or interface is created (§5.3) by the Java Virtual Machine.  

每个运行时常量池都从Java虚拟机的方法区（`method area`，§2.5.4）中分配。类或接口的运行时常量池是在该类或接口被Java虚拟机创建时（§5.3）构建的。  

The following exceptional condition is associated with the construction of the run-time constant pool for a class or interface:  

- When creating a class or interface, if the construction of the run-time constant pool requires more memory than can be made available in the method area of the Java Virtual Machine, the Java Virtual Machine throws an `OutOfMemoryError`.  

以下异常情况与类或接口的运行时常量池构建相关：  

- 在创建类或接口时，如果运行时常量池的构建需要的内存超过了Java虚拟机的方法区中可用的内存，Java虚拟机会抛出`OutOfMemoryError`错误。  

See §5 (Loading, Linking, and Initializing) for information about the construction of the run-time constant pool.  

关于运行时常量池的构建，请参见§5（加载、链接和初始化）。  

### 2.5.6 Native Method Stacks（本地方法栈）

An implementation of the Java Virtual Machine may use conventional stacks, colloquially called "C stacks," to support native methods (methods written in a language other than the Java programming language). Native method stacks may also be used by the implementation of an interpreter for the Java Virtual Machine's instruction set in a language such as C. Java Virtual Machine implementations that cannot load native methods and that do not themselves rely on conventional stacks need not supply native method stacks. If supplied, native method stacks are typically allocated per thread when each thread is created.  

Java虚拟机的实现可以使用常规栈（通常称为“C栈”）来支持本地方法（用非Java编程语言编写的方法）。本地方法栈还可以用于以C语言等实现的Java虚拟机指令集解释器。如果Java虚拟机的实现不能加载本地方法，并且本身也不依赖常规栈，则不需要提供本地方法栈。如果提供，本地方法栈通常会在每个线程创建时为每个线程分配。  
  
This specification permits native method stacks either to be of a fixed size or to dynamically expand and contract as required by the computation. If the native method stacks are of a fixed size, the size of each native method stack may be chosen independently when that stack is created.  

本规范允许本地方法栈的大小可以是固定的，也可以根据计算需求动态扩展和收缩。如果本地方法栈是固定大小的，则每个本地方法栈的大小可以在栈创建时独立选择。  

A Java Virtual Machine implementation may provide the programmer or the user control over the initial size of the native method stacks, as well as, in the case of varying-size native method stacks, control over the maximum and minimum method stack sizes.  

Java虚拟机的实现可以让程序员或用户控制本地方法栈的初始大小，以及在可变大小的本地方法栈的情况下，控制最大和最小栈大小。  

The following exceptional conditions are associated with native method stacks:  

- If the computation in a thread requires a larger native method stack than is permitted, the Java Virtual Machine throws a `StackOverflowError`.  
- If native method stacks can be dynamically expanded and native method stack expansion is attempted but insufficient memory can be made available, or if insufficient memory can be made available to create the initial native method stack for a new thread, the Java Virtual Machine throws an `OutOfMemoryError`.  

以下异常情况与本地方法栈相关：  

- 如果线程中的计算需要比允许的更大的本地方法栈，Java虚拟机会抛出`StackOverflowError`错误。 
- 如果本地方法栈可以动态扩展，并且尝试扩展时内存不足，或者为新线程创建初始本地方法栈时内存不足，Java虚拟机会抛出`OutOfMemoryError`错误。

---  
## 2.6 Frames（帧）  

A frame is used to store data and partial results, as well as to perform dynamic linking, return values for methods, and dispatch exceptions.   

帧用于存储数据和部分结果，以及执行动态链接、返回方法的值和分发异常。  

A new frame is created each time a method is invoked. A frame is destroyed when its method invocation completes, whether that completion is normal or abrupt (it throws an uncaught exception). Frames are allocated from the Java Virtual Machine stack (§2.5.2) of the thread creating the frame. Each frame has its own array of local variables (§2.6.1), its own operand stack (§2.6.2), and a reference to the runtime constant pool (§2.5.5) of the class of the current method.  

每次调用方法时都会创建一个新的帧。当方法调用完成时，无论该完成是正常的还是异常的（抛出未捕获的异常），帧都会被销毁。帧是从创建该帧的线程的Java虚拟机堆栈（§2.5.2）中分配的。每个帧都有自己的局部变量数组（§2.6.1）、自己的操作数堆栈（§2.6.2），以及对当前方法所属类的运行时常量池（§2.5.5）的引用。  

A frame may be extended with additional implementation-specific information, such as debugging information.

帧可以通过额外的实现特定信息进行扩展，例如调试信息。

The sizes of the local variable array and the operand stack are determined at compile-time and are supplied along with the code for the method associated with the frame (§4.7.3). Thus the size of the frame data structure depends only on the implementation of the Java Virtual Machine, and the memory for these structures can be allocated simultaneously on method invocation.

局部变量数组和操作数堆栈的大小在编译时确定，并随与帧相关联的方法代码一起提供（§4.7.3）。因此，帧数据结构的大小仅取决于Java虚拟机的实现，且这些结构的内存可以在方法调用时同时分配。

Only one frame, the frame for the executing method, is active at any point in a given thread of control. This frame is referred to as the current frame, and its method is known as the current method. The class in which the current method is defined is the current class. Operations on local variables and the operand stack are typically with reference to the current frame.

在给定的控制线程中的任何时刻，只有一个帧（用于正在执行的方法）处于活动状态。这个帧被称为当前帧，其方法被称为当前方法。定义当前方法的类被称为当前类。对局部变量和操作数堆栈的操作通常是相对于当前帧进行的。

A frame ceases to be current if its method invokes another method or if its method completes. When a method is invoked, a new frame is created and becomes current when control transfers to the new method. On method return, the current frame passes back the result of its method invocation, if any, to the previous frame. The current frame is then discarded as the previous frame becomes the current one.

如果当前帧的方法调用了另一个方法或其方法完成，帧将不再是当前帧。当一个方法被调用时，会创建一个新的帧，并在控制转移到新方法时成为当前帧。在方法返回时，当前帧将其方法调用的结果（如果有）传递给前一个帧。当前帧随后被丢弃，而前一个帧成为当前帧。

Note that a frame created by a thread is local to that thread and cannot be referenced by any other thread.

请注意，由一个线程创建的帧是该线程的局部帧，不能被任何其他线程引用。

### 2.6.1 Local Variables（局部变量）

Each frame (§2.6) contains an array of variables known as its local variables. The length of the local variable array of a frame is determined at compile-time and supplied in the binary representation of a class or interface along with the code for the method associated with the frame (§4.7.3).  

每个帧（§2.6）包含一个称为局部变量的数组。帧的局部变量数组的长度在编译时确定，并在类或接口的二进制表示中与与帧相关联的方法代码一起提供（§4.7.3）。  

A single local variable can hold a value of type boolean, byte, char, short, int, float, reference, or returnAddress. A pair of local variables can hold a value of type long or double.  

一个局部变量可以保存一个`boolean`、`byte`、`char`、`short`、`int`、`float`、`reference`或`returnAddress`类型的值。一对局部变量可以保存一个`long`或`double`类型的值。  

Local variables are addressed by indexing. The index of the first local variable is zero. An integer is considered to be an index into the local variable array if and only if that integer is between zero and one less than the size of the local variable array.  

局部变量通过索引进行访问。第一个局部变量的索引为0。只有当一个整数在0到局部变量数组大小减一之间时，该整数才被视为局部变量数组的索引。  

A value of type long or type double occupies two consecutive local variables. Such a value may only be addressed using the lesser index. For example, a value of type double stored in the local variable array at index n actually occupies the local variables with indices n and n+1; however, the local variable at index n+1 cannot be loaded from. It can be stored into. However, doing so invalidates the contents of local variable n.  

`long`或`double`类型的值占用两个连续的局部变量。这样的值只能通过较小的索引进行访问。例如，存储在局部变量数组索引n处的`double`类型的值实际上占用了索引n和n+1的局部变量；然而，不能从索引n+1的局部变量加载值。它可以被存储进去。但这样做会使局部变量n的内容无效。  

The Java Virtual Machine does not require n to be even. In intuitive terms, values of types long and double need not be 64-bit aligned in the local variables array. Implementors are free to decide the appropriate way to represent such values using the two local variables reserved for the value.  

Java虚拟机不要求n必须是偶数。直观地说，`long`和`double`类型的值在局部变量数组中不需要64位对齐。实现者可以自由决定使用为该值保留的两个局部变量来表示该值的适当方式。  

The Java Virtual Machine uses local variables to pass parameters on method invocation. On class method invocation, any parameters are passed in consecutive local variables starting from local variable 0. On instance method invocation, local variable 0 is always used to pass a reference to the object on which the instance method is being invoked (this in the Java programming language). Any parameters are subsequently passed in consecutive local variables starting from local variable 1.  

Java虚拟机使用局部变量在方法调用时传递参数。在类方法调用时，任何参数都会从局部变量0开始依次传递。在实例方法调用时，局部变量0始终用于传递对调用实例方法的对象的引用（在Java编程语言中称为`this`）。任何参数随后从局部变量1开始依次传递。  

### 2.6.2 Operand Stacks（操作数栈）  

Each frame (§2.6) contains a last-in-first-out (LIFO) stack known as its operand stack. The maximum depth of the operand stack of a frame is determined at compile-time and is supplied along with the code for the method associated with the frame (§4.7.3).  

每个帧（§2.6）包含一个后进先出（LIFO）的栈，称为其操作数栈。帧的操作数栈的最大深度在编译时确定，并与与帧相关联的方法代码一起提供（§4.7.3）。  

Where it is clear by context, we will sometimes refer to the operand stack of the current frame as simply the operand stack.  

在上下文明确的情况下，我们有时会将当前帧的操作数堆栈简称为操作数栈。

The operand stack is empty when the frame that contains it is created. The Java Virtual Machine supplies instructions to load constants or values from local variables or fields onto the operand stack. Other Java Virtual Machine instructions take operands from the operand stack, operate on them, and push the result back onto the operand stack. The operand stack is also used to prepare parameters to be passed to methods and to receive method results.  

当包含操作数栈的帧被创建时，操作数栈是空的。Java虚拟机提供指令将常量或值从局部变量或字段加载到操作数栈上。其他Java虚拟机指令从操作数栈获取操作数，对其进行操作，然后将结果推回操作数栈。操作数栈还用于准备传递给方法的参数并接收方法的结果。  

For example, the iadd instruction (§iadd) adds two int values together. It requires that the int values to be added be the top two values of the operand stack, pushed there by previous instructions. Both of the int values are popped from the operand stack. They are added, and their sum is pushed back onto the operand stack. Subcomputations may be nested on the operand stack, resulting in values that can be used by the encompassing computation.  

例如，`iadd`指令（§iadd）将两个`int`值相加。它要求被相加的`int`值是操作数栈的顶部两个值，这些值是由之前的指令推入栈的。这两个`int`值从操作数栈中弹出。它们被相加，并且它们的和被推回操作数栈。子计算可以嵌套在操作数栈上，从而生成可由包含的计算使用的值。  

Each entry on the operand stack can hold a value of any Java Virtual Machine type, including a value of type long or double.  

操作数栈上的每个条目可以保存任何Java虚拟机类型的值，包括`long`或`double`类型的值。  

Values from the operand stack must be operated upon in ways appropriate to their types. It is not possible, for example, to push two int values and subsequently treat them as a long or to push two float values and subsequently add them with an iadd instruction.  

从操作数栈中取出的值必须按照适合其类型的方式进行操作。例如，不可能先推入两个`int`值，然后将它们当作`long`处理，或者先推入两个`float`值，然后用`iadd`指令将它们相加。  

A small number of Java Virtual Machine instructions (the dup instructions (§dup) and swap (§swap)) operate on runtime data areas as raw values without regard to their specific types; these instructions are defined in such a way that they cannot be used to modify or break up individual values. These restrictions on operand stack manipulation are enforced through class file verification (§4.10).  

少量Java虚拟机指令（如`dup`指令（§dup）和`swap`（§swap））对运行时数据区作为原始值进行操作，而不考虑它们的具体类型；这些指令的定义方式使得它们不能被用于修改或分解单个值。这些对操作数栈操作的限制通过类文件验证（§4.10）强制执行。  

At any point in time, an operand stack has an associated depth, where a value of type long or double contributes two units to the depth and a value of any other type contributes one unit.  

在任何时候，操作数栈都有一个相关的深度，其中`long`或`double`类型的值贡献两个单位深度，任何其他类型的值贡献一个单位深度。  

### 2.6.3 Dynamic Linking（动态链接）  

Each frame (§2.6) contains a reference to the runtime constant pool (§2.5.5) for the type of the current method to support dynamic linking of the method code.  

每个帧（§2.6）包含一个对当前方法类型的运行时常量池（§2.5.5）的引用，以支持方法代码的动态链接。  

The class file code for a method refers to methods to be invoked and variables to be accessed via symbolic references. Dynamic linking translates these symbolic method references into concrete method references, loading classes as necessary to resolve as-yet-undefined symbols, and translates variable accesses into appropriate offsets in storage structures associated with the runtime location of these variables.  

方法的类文件代码通过符号引用来引用要调用的方法和要访问的变量。动态链接将这些符号方法引用转换为具体的方法引用，根据需要加载类以解析尚未定义的符号，并将变量访问转换为与这些变量的运行时位置相关的存储结构中的适当偏移。  

This late binding of the methods and variables makes changes in other classes that a method uses less likely to break this code.  

这种方法和变量的晚绑定使得方法所使用的其他类的更改不太可能破坏该代码。  

### 2.6.4 Normal Method Invocation Completion（方法调用正常完成）  

A method invocation completes normally if that invocation does not cause an exception (§2.10) to be thrown, either directly from the Java Virtual Machine or as a result of executing an explicit throw statement.  

如果方法调用不会导致异常（§2.10）被抛出，无论是直接从Java虚拟机抛出还是由于执行显式的`throw`语句引发，则该方法调用正常完成。  

If the invocation of the current method completes normally, then a value may be returned to the invoking method. This occurs when the invoked method executes one of the return instructions (§2.11.8), the choice of which must be appropriate for the type of the value being returned (if any).  

如果当前方法的调用正常完成，那么可能会有一个值返回给调用方法。这发生在被调用的方法执行其中一个返回指令（§2.11.8）时，所选择的返回指令必须适合要返回的值的类型（如果有的话）。  

The current frame (§2.6) is used in this case to restore the state of the invoker, including its local variables and operand stack, with the program counter of the invoker appropriately incremented to skip past the method invocation instruction. Execution then continues normally in the invoking method's frame with the returned value (if any) pushed onto the operand stack of that frame.  

在这种情况下，当前帧（§2.6）用于恢复调用者的状态，包括其局部变量和操作数堆栈，并适当增加调用者的程序计数器以跳过方法调用指令。然后，执行在调用方法的帧中正常继续，返回的值（如果有的话）会被推入该帧的操作数堆栈。  

### 2.6.5 Abrupt Method Invocation Completion（方法调用异常完成）  

A method invocation completes abruptly if execution of a Java Virtual Machine instruction within the method causes the Java Virtual Machine to throw an exception (§2.10), and that exception is not handled within the method.  

如果方法中的Java虚拟机指令的执行导致Java虚拟机抛出异常（§2.10），并且该异常未在方法内处理，则该方法调用会异常完成。  

Execution of an athrow instruction (§athrow) also causes an exception to be explicitly thrown and, if the exception is not caught by the current method, results in abrupt method invocation completion. A method invocation that completes abruptly never returns a value to its invoker.  

执行`athrow`指令（§athrow）也会导致显式抛出异常，并且如果该异常未被当前方法捕获，将导致方法调用异常完成。异常完成的方法调用永远不会将值返回给其调用者。  

---  
### 2.7 Representation of Objects（对象的表示）  

The Java Virtual Machine does not mandate any particular internal structure for objects.

Java虚拟机不强制规定对象的任何特定内部结构。

In some of Oracle’s implementations of the Java Virtual Machine, a reference to a class instance is a pointer to a handle that is itself a pair of pointers: one to a table containing the methods of the object and a pointer to the Class object that represents the type of the object, and the other to the memory allocated from the heap for the object data.

在Oracle的某些Java虚拟机实现中，对类实例的引用是一个指向句柄的指针，该句柄本身是一对指针：一个指向包含对象方法的表和一个指向表示对象类型的`Class`对象的指针，另一个指向为对象数据从堆中分配的内存。

---  
### 2.8 Floating-Point Arithmetic（浮点数计算）  

The Java Virtual Machine incorporates a subset of the floating-point arithmetic specified in IEEE Standard for Binary Floating-Point Arithmetic (ANSI/IEEE Std. 754-1985, New York).

Java虚拟机包含了IEEE二进制浮点运算标准（ANSI/IEEE Std. 754-1985，纽约）中指定的浮点运算的一个子集。

### 2.8.1 Java Virtual Machine Floating-Point Arithmetic and IEEE 754（Java虚拟机中的浮点计算和IEEE 754）

The key differences between the floating-point arithmetic supported by the Java Virtual Machine and the IEEE 754 standard are:

Java虚拟机支持的浮点运算与IEEE 754标准的主要区别在于：

- The floating-point operations of the Java Virtual Machine do not throw exceptions, trap, or otherwise signal the IEEE 754 exceptional conditions of invalid operation, division by zero, overflow, underflow, or inexact. The Java Virtual Machine has no signaling NaN value.

  Java虚拟机的浮点运算不会抛出异常、陷阱或以其他方式指示IEEE 754规定的无效操作、除零、溢出、下溢或不精确的异常情况。Java虚拟机没有信号NaN值。

- The Java Virtual Machine does not support IEEE 754 signaling floating-point comparisons.  

  Java虚拟机不支持IEEE 754信号浮点比较。

- The rounding operations of the Java Virtual Machine always use IEEE 754 round to nearest mode. Inexact results are rounded to the nearest representable value, with ties going to the value with a zero least-significant bit. This is the IEEE 754 default mode. But Java Virtual Machine instructions that convert values of floating-point types to values of integral types round toward zero. The Java Virtual Machine does not give any means to change the floating-point rounding mode.

  Java虚拟机的舍入操作始终使用IEEE 754的舍入到最近模式。不精确的结果被舍入到最近的可表示值，如果两个最近的可表示值一样近，则选择最不重要位为零的值。这是IEEE 754的默认模式。但将浮点类型的值转换为整数类型的值的Java虚拟机指令会朝零舍入。Java虚拟机没有提供任何更改浮点舍入模式的方法。

- The Java Virtual Machine does not support either the IEEE 754 single extended or double extended format, except insofar as the double and double-extended-exponent value sets may be said to support the single extended format. The float-extended-exponent and double-extended-exponent value sets, which may optionally be supported, do not correspond to the values of the IEEE 754 extended formats: the IEEE 754 extended formats require extended precision as well as extended exponent range.  

  Java虚拟机不支持IEEE 754单扩展或双扩展格式，除非可以说双扩展和双扩展指数值集支持单扩展格式。浮点扩展指数和双扩展指数值集（可选支持）不对应IEEE 754扩展格式的值：IEEE 754扩展格式要求扩展精度以及扩展指数范围。

### 2.8.2 Floating-Point Modes（浮点模式）  

Every method has a floating-point mode, which is either FP-strict or not FP-strict. The floating-point mode of a method is determined by the setting of the ACC_STRICT flag of the access_flags item of the method_info structure (§4.6) defining the method. A method for which this flag is set is FP-strict; otherwise, the method is not FP-strict. 

每个方法都有一个浮点模式，它要么是`FP-strict`，要么不是。方法的浮点模式由定义该方法的`method_info`结构的`access_flags`项的`ACC_STRICT`标志的设置决定（§4.6）。如果设置了此标志的方法是`FP-strict`的，否则该方法不是`FP-strict`的。

Note that this mapping of the ACC_STRICT flag implies that methods in classes compiled by a compiler in JDK release 1.1 or earlier are effectively not FP-strict.

请注意，这种`ACC_STRICT`标志的映射意味着由JDK 1.1或更早版本的编译器编译的类中的方法实际上不是`FP-strict`的。

We will refer to an operand stack as having a given floating-point mode when the method whose invocation created the frame containing the operand stack has that floating-point mode. Similarly, we will refer to a Java Virtual Machine instruction as having a given floating-point mode when the method containing that instruction has that floating-point mode.  

当调用创建包含操作数堆栈的帧的方法具有某种浮点模式时，我们将称操作数堆栈具有该浮点模式。类似地，当包含该指令的方法具有某种浮点模式时，我们将称Java虚拟机指令具有该浮点模式。

If a float-extended-exponent value set is supported (§2.3.2), values of type float on an operand stack that is not FP-strict may range over that value set except where prohibited by value set conversion (§2.8.3). If a double-extended-exponent value set is supported (§2.3.2), values of type double on an operand stack that is not FP-strict may range over that value set except where prohibited by value set conversion.

如果支持浮点扩展指数值集（§2.3.2），则在非`FP-strict`的操作数堆栈上的`float`类型值可以在该值集内变动，除非值集转换（§2.8.3）禁止这样做。如果支持双扩展指数值集（§2.3.2），则在非`FP-strict`的操作数堆栈上的`double`类型值可以在该值集内变动，除非值集转换禁止这样做。

In all other contexts, whether on the operand stack or elsewhere, and regardless of floating-point mode, floating-point values of type float and double may only range over the float value set and double value set, respectively. In particular, class and instance fields, array elements, local variables, and method parameters may only contain values drawn from the standard value sets.

在所有其他上下文中，无论是在操作数堆栈上还是在其他地方，并且无论浮点模式如何，`float`和`double`类型的浮点值只能在`float`值集和`double`值集内变动。特别是，类和实例字段、数组元素、局部变量和方法参数只能包含来自标准值集的值。

### 2.8.3 Value Set Conversion（值集转换）

An implementation of the Java Virtual Machine that supports an extended floating-point value set is permitted or required, under specified circumstances, to map a value of the associated floating-point type between the extended and the standard value sets. Such a value set conversion is not a type conversion, but a mapping between the value sets associated with the same type.

支持扩展浮点值集的Java虚拟机实现被允许或在特定情况下被要求在扩展值集和标准值集之间映射与浮点类型相关的值。这种值集转换不是类型转换，而是与同一类型相关的值集之间的映射。

Where value set conversion is indicated, an implementation is permitted to perform one of the following operations on a value:

在指示进行值集转换的地方，实现允许对一个值执行以下操作之一：

- If the value is of type float and is not an element of the float value set, it maps the value to the nearest element of the float value set.

  如果值的类型是`float`，并且不是`float`值集的元素，它会将该值映射到`float`值集中最近的元素。

- If the value is of type double and is not an element of the double value set, it maps the value to the nearest element of the double value set.

  如果值的类型是`double`，并且不是`double`值集的元素，它会将该值映射到`double`值集中最近的元素。

In addition, where value set conversion is indicated, certain operations are required:  

此外，在指示进行值集转换的地方，某些操作是必须的：  

- Suppose execution of a Java Virtual Machine instruction that is not FP-strict causes a value of type float to be pushed onto an operand stack that is FP-strict, passed as a parameter, or stored into a local variable, a field, or an element of an array. If the value is not an element of the float value set, it maps the value to the nearest element of the float value set.

  假设执行一个非`FP-strict`的Java虚拟机指令导致一个`float`类型的值被推送到一个`FP-strict`的操作数堆栈中，作为参数传递，或者存储到局部变量、字段或数组的元素中。如果该值不是`float`值集的元素，它会将该值映射到`float`值集中最近的元素。

- Suppose execution of a Java Virtual Machine instruction that is not FP-strict causes a value of type double to be pushed onto an operand stack that is FP-strict, passed as a parameter, or stored into a local variable, a field, or an element of an array. If the value is not an element of the double value set, it maps the value to the nearest element of the double value set.

  假设执行一个非`FP-strict`的Java虚拟机指令导致一个`double`类型的值被推送到一个`FP-strict`的操作数堆栈中，作为参数传递，或者存储到局部变量、字段或数组的元素中。如果该值不是`double`值集的元素，它会将该值映射到`double`值集中最近的元素。

Such required value set conversions may occur as a result of passing a parameter of a floating-point type during method invocation, including native method invocation; returning a value of a floating-point type from a method that is not FP-strict to a method that is FP-strict; or storing a value of a floating-point type into a local variable, a field, or an array in a method that is not FP-strict.

这些必需的值集转换可能是由于在方法调用期间传递浮点类型的参数（包括本机方法调用）、从非`FP-strict`的方法返回浮点类型的值到`FP-strict`的方法，或者将浮点类型的值存储到非`FP-strict`的方法中的局部变量、字段或数组中所导致的。

Not all values from an extended-exponent value set can be mapped exactly to a value in the corresponding standard value set. If a value being mapped is too large to be represented exactly (its exponent is greater than that permitted by the standard value set), it is converted to a (positive or negative) infinity of the corresponding type. If a value being mapped is too small to be represented exactly (its exponent is smaller than that permitted by the standard value set), it is rounded to the nearest of a representable denormalized value or zero of the same sign.

并非扩展指数值集中的所有值都可以准确地映射到对应的标准值集中的值。如果被映射的值过大而无法精确表示（其指数大于标准值集允许的范围），则它会被转换为对应类型的（正或负）无穷大。如果被映射的值过小而无法精确表示（其指数小于标准值集允许的范围），则它会被舍入为可表示的非标准化值或同符号的零。

Value set conversion preserves infinities and NaNs and cannot change the sign of the value being converted. Value set conversion has no effect on a value that is not of a floating-point type.

值集转换保留无穷大和NaN，并且不能改变被转换值的符号。值集转换对非浮点类型的值没有影响。

---  
### 2.9 Special Methods（特殊方法）

At the level of the Java Virtual Machine, every constructor written in the Java programming language (JLS §8.8) appears as an instance initialization method that has the special name \<init>. This name is supplied by a compiler. Because the name \<init> is not a valid identifier, it cannot be used directly in a program written in the Java programming language. Instance initialization methods may be invoked only within the Java Virtual Machine by the invokespecial instruction (§invokespecial), and they may be invoked only on uninitialized class instances. An instance initialization method takes on the access permissions (JLS §6.6) of the constructor from which it was derived.

在Java虚拟机级别，用Java编程语言编写的每个构造函数（JLS §8.8）都表现为一个实例初始化方法，该方法具有特殊名称`<init>`。这个名称由编译器提供。由于名称`<init>`不是有效的标识符，因此不能在用Java编写的程序中直接使用。实例初始化方法只能在Java虚拟机内通过`invokespecial`指令（§invokespecial）调用，并且它们只能在未初始化的类实例上调用。实例初始化方法继承了它所派生的构造函数的访问权限（JLS §6.6）。

A class or interface has at most one class or interface initialization method and is initialized (§5.5) by invoking that method. The initialization method of a class or interface has the special name \<clinit>, takes no arguments, and is void (§4.3.3).

一个类或接口最多有一个类或接口初始化方法，并通过调用该方法初始化（§5.5）。类或接口的初始化方法具有特殊名称`<clinit>`，不接受任何参数，并且是`void`类型（§4.3.3）。

Other methods named \<clinit> in a class file are of no consequence. They are not class or interface initialization methods. They cannot be invoked by any Java Virtual Machine instruction and are never invoked by the Java Virtual Machine itself.

类文件中其他名为`<clinit>`的方法没有任何意义。它们不是类或接口初始化方法。它们不能被任何Java虚拟机指令调用，也不会被Java虚拟机本身调用。

In a class file whose version number is 51.0 or above, the method must additionally have its ACC_STATIC flag (§4.6) set in order to be the class or interface initialization method.

在版本号为51.0或更高的类文件中，该方法还必须设置`ACC_STATIC`标志（§4.6）才能成为类或接口初始化方法。

This requirement was introduced in Java SE 7. In a class file whose version number is 50.0 or below, a method named \<clinit> that is void and takes no arguments is considered the class or interface initialization method regardless of the setting of its ACC_STATIC flag.

此要求是在Java SE 7中引入的。在版本号为50.0或更低的类文件中，无论`ACC_STATIC`标志的设置如何，一个名为`<clinit>`且为`void`类型并且不接受任何参数的方法都被视为类或接口初始化方法。

The name \<clinit> is supplied by a compiler. Because the name \<clinit> is not a valid identifier, it cannot be used directly in a program written in the Java programming language. Class and interface initialization methods are invoked implicitly by the Java Virtual Machine; they are never invoked directly from any Java Virtual Machine instruction, but are invoked only indirectly as part of the class initialization process.

名称`<clinit>`由编译器提供。由于名称`<clinit>`不是有效的标识符，因此不能在用Java编写的程序中直接使用。类和接口初始化方法由Java虚拟机隐式调用；它们从不会直接从任何Java虚拟机指令中调用，而只是作为类初始化过程的一部分间接调用。

A method is signature polymorphic if all of the following are true:

- It is declared in the java.lang.invoke.MethodHandle class.
- It has a single formal parameter of type Object\[].
- It has a return type of Object.
- It has the ACC_VARARGS and ACC_NATIVE flags set.

如果满足以下所有条件，则一个方法是签名多态的：

+ 它在`java.lang.invoke.MethodHandle`类中声明。
+ 它有一个类型为`Object[]`的正式参数。
+ 它的返回类型是`Object`。
+ 它设置了`ACC_VARARGS`和`ACC_NATIVE`标志。

In Java SE 8, the only signature polymorphic methods are the invoke and invokeExact methods of the class java.lang.invoke.MethodHandle.

在Java SE 8中，唯一的签名多态方法是`java.lang.invoke.MethodHandle`类的`invoke`和`invokeExact`方法。

The Java Virtual Machine gives special treatment to signature polymorphic methods in the invokevirtual instruction (§invokevirtual), in order to effect invocation of a method handle. A method handle is a strongly typed, directly executable reference to an underlying method, constructor, field, or similar low-level operation (§5.4.3.5), with optional transformations of arguments or return values. These transformations are quite general, and include such patterns as conversion, insertion, deletion, and substitution. See the java.lang.invoke package in the Java SE platform API for more information.

Java虚拟机在`invokevirtual`指令（§invokevirtual）中对签名多态方法进行了特殊处理，以便有效地调用方法句柄。方法句柄是对底层方法、构造函数、字段或类似低级操作的强类型、直接可执行的引用（§5.4.3.5），并可以选择性地对参数或返回值进行转换。这些转换非常通用，包括转换、插入、删除和替换等模式。有关更多信息，请参阅Java SE平台API中的`java.lang.invoke`包。

---  
### 2.10 Exceptions（异常）

An exception in the Java Virtual Machine is represented by an instance of the class Throwable or one of its subclasses. Throwing an exception results in an immediate nonlocal transfer of control from the point where the exception was thrown.

在Java虚拟机中，异常由`Throwable`类或其子类的实例表示。抛出异常会导致从抛出异常的点立即进行非局部控制转移。

Most exceptions occur synchronously as a result of an action by the thread in which they occur. An asynchronous exception, by contrast, can potentially occur at any point in the execution of a program. The Java Virtual Machine throws an exception for one of three reasons:  

大多数异常是由于它们发生的线程的操作而同步发生的。相比之下，异步异常可能会在程序执行的任何点发生。Java虚拟机因以下三种原因之一抛出异常：

- An athrow instruction (§athrow) was executed.
- An abnormal execution condition was synchronously detected by the Java Virtual Machine.These exceptions are not thrown at an arbitrary point in the program, but only synchronously after execution of an instruction that either:
  - Specifies the exception as a possible result, such as:
      - When the instruction embodies an operation that violates the semantics of the Java programming language, for example indexing outside the bounds of an array.
      - When an error occurs in loading or linking part of the program.
  - Causes some limit on a resource to be exceeded, for example when too much memory is used.
- An asynchronous exception occurred because:
  - The stop method of class Thread or ThreadGroup was invoked, or
  - An internal error occurred in the Java Virtual Machine implementation.

+ 执行了`athrow`指令（§athrow）。
+ Java虚拟机同步检测到异常执行条件。这些异常不是在程序的任意点抛出的，而是仅在执行以下指令之后同步抛出的：
  + 指定异常作为可能的结果，例如：
      + 当指令体现了违反Java编程语言语义的操作时，例如索引超出数组的边界。
      + 当加载或链接程序的某部分时发生错误。
  + 导致超出资源限制，例如使用了过多的内存。
+ 因以下原因发生异步异常：
  + 调用了`Thread`或`ThreadGroup`类的`stop`方法，或
  + Java虚拟机实现中发生内部错误。

The stop methods may be invoked by one thread to affect another thread or all the threads in a specified thread group. They are asynchronous because they may occur at any point in the execution of the other thread or threads. An internal error is considered asynchronous (§6.3).

`stop`方法可以由一个线程调用，以影响另一个线程或指定线程组中的所有线程。它们是异步的，因为它们可能在其他线程或线程的执行过程中的任何点发生。内部错误被认为是异步的（§6.3）。

A Java Virtual Machine may permit a small but bounded amount of execution to occur before an asynchronous exception is thrown. This delay is permitted to allow optimized code to detect and throw these exceptions at points where it is practical to handle them while obeying the semantics of the Java programming language.

Java虚拟机可能允许在抛出异步异常之前进行少量但有限的执行。这种延迟被允许，以便优化代码可以在实践中处理这些异常的点检测并抛出这些异常，同时遵守Java编程语言的语义。  

A simple implementation might poll for asynchronous exceptions at the point of each control transfer instruction. Since a program has a finite size, this provides a bound on the total delay in detecting an asynchronous exception. Since no asynchronous exception will occur between control transfers, the code generator has some flexibility to reorder computation between control transfers for greater performance. The paper "Polling Efficiently on Stock Hardware" by Marc Feeley, Proc. 1993 Conference on Functional Programming and Computer Architecture, Copenhagen, Denmark, pp. 179–187, is recommended as further reading.

一个简单的实现可能会在每个控制转移指令点轮询异步异常。由于程序具有有限的大小，这为检测异步异常的总延迟提供了限制。由于在控制转移之间不会发生异步异常，代码生成器在控制转移之间重新排序计算以提高性能方面具有一定的灵活性。推荐阅读Marc Feeley的论文“Polling Efficiently on Stock Hardware”，该论文发表在1993年函数编程与计算机架构会议（Copenhagen, Denmark）上，第179-187页。

Exceptions thrown by the Java Virtual Machine are precise: when the transfer of control takes place, all effects of the instructions executed before the point from which the exception is thrown must appear to have taken place. No instructions that occur after the point from which the exception is thrown may appear to have been evaluated. If optimized code has speculatively executed some of the instructions which follow the point at which the exception occurs, such code must be prepared to hide this speculative execution from the user-visible state of the program.

Java虚拟机抛出的异常是精确的：当发生控制转移时，必须显示在抛出异常点之前执行的指令的所有效果。在抛出异常点之后发生的任何指令都不能被认为已执行。如果优化代码已经推测性地执行了异常发生点之后的一些指令，则该代码必须准备好隐藏这些推测执行，以免暴露在程序的用户可见状态中。

Each method in the Java Virtual Machine may be associated with zero or more exception handlers. An exception handler specifies the range of offsets into the Java Virtual Machine code implementing the method for which the exception handler is active, describes the type of exception that the exception handler is able to handle, and specifies the location of the code that is to handle that exception. An exception matches an exception handler if the offset of the instruction that caused the exception is in the range of offsets of the exception handler and the exception type is the same class as or a subclass of the class of exception that the exception handler handles. When an exception is thrown, the Java Virtual Machine searches for a matching exception handler in the current method. If a matching exception handler is found, the system branches to the exception handling code specified by the matched handler.

Java虚拟机中的每个方法可以与零个或多个异常处理程序关联。异常处理程序指定在Java虚拟机代码中实现该方法的偏移范围，该范围内异常处理程序处于活动状态，描述异常处理程序能够处理的异常类型，并指定处理该异常的代码位置。如果导致异常的指令的偏移量在异常处理程序的偏移范围内，并且异常类型与异常处理程序处理的异常类型相同或为其子类，则该异常与异常处理程序匹配。当抛出异常时，Java虚拟机会在当前方法中搜索匹配的异常处理程序。如果找到匹配的异常处理程序，系统将分支到由匹配的处理程序指定的异常处理代码。

If no such exception handler is found in the current method, the current method invocation completes abruptly (§2.6.5). On abrupt completion, the operand stack and local variables of the current method invocation are discarded, and its frame is popped, reinstating the frame of the invoking method. The exception is then rethrown in the context of the invoker's frame and so on, continuing up the method invocation chain. If no suitable exception handler is found before the top of the method invocation chain is reached, the execution of the thread in which the exception was thrown is terminated.

如果在当前方法中未找到此类异常处理程序，则当前方法调用会异常完成（§2.6.5）。在异常完成时，当前方法调用的操作数堆栈和局部变量会被丢弃，其帧会被弹出，恢复调用方法的帧。然后在调用者的帧上下文中重新抛出异常，依此类推，继续沿着方法调用链查找。如果在到达方法调用链的顶部之前未找到合适的异常处理程序，则抛出异常的线程的执行将终止。

The order in which the exception handlers of a method are searched for a match is important. Within a class file, the exception handlers for each method are stored in a table (§4.7.3). At run time, when an exception is thrown, the Java Virtual Machine searches the exception handlers of the current method in the order that they appear in the corresponding exception handler table in the class file, starting from the beginning of that table.

搜索方法的异常处理程序以匹配顺序是很重要的。在类文件中，每个方法的异常处理程序都存储在一个表中（§4.7.3）。在运行时，当抛出异常时，Java虚拟机会按照它们在类文件中的相应异常处理程序表中的出现顺序搜索当前方法的异常处理程序，从该表的开头开始。

Note that the Java Virtual Machine does not enforce nesting of or any ordering of the exception table entries of a method. The exception handling semantics of the Java programming language are implemented only through cooperation with the compiler (§3.12). When class files are generated by some other means, the defined search procedure ensures that all Java Virtual Machine implementations will behave consistently.

请注意，Java虚拟机不强制执行方法的异常表条目的嵌套或任何顺序。Java编程语言的异常处理语义仅通过与编译器的协作来实现（§3.12）。当通过其他方式生成类文件时，定义的搜索过程确保所有Java虚拟机实现将一致地表现。

---  
### 2.11 Instruction Set Summary（指令集总结）

A Java Virtual Machine instruction consists of a one-byte opcode specifying the operation to be performed, followed by zero or more operands supplying arguments or data that are used by the operation. Many instructions have no operands and consist only of an opcode.

Java虚拟机指令由一个指定要执行的操作的单字节操作码组成，后跟零个或多个提供操作所用参数或数据的操作数。许多指令没有操作数，仅由操作码组成。

Ignoring exceptions, the inner loop of a Java Virtual Machine interpreter is effectively:

忽略异常，Java虚拟机解释器的内循环实际上是：

- atomically calculate pc and fetch opcode at pc;

  原子地计算pc值，并获取pc指向指令的操作码；

- if (operands) fetch operands;

  如果有操作数，则获取操作数；

- execute the action for the opcode;

  执行操作码对应的操作；

- while (there is more to do);

  当有更多操作时继续循环；

The number and size of the operands are determined by the opcode. If an operand is more than one byte in size, then it is stored in big-endian order - high-order byte first. For example, an unsigned 16-bit index into the local variables is stored as two unsigned bytes, byte1 and byte2, such that its value is (byte1 << 8) | byte2.

操作数的数量和大小由操作码决定。如果操作数的大小超过一个字节，则按大端顺序存储——高位字节在前。例如，局部变量中的一个无符号16位索引存储为两个无符号字节，`byte1`和`byte2`，其值为`(byte1 << 8) | byte2`。

The bytecode instruction stream is only single-byte aligned. The two exceptions are the lookupswitch and tableswitch instructions (§lookupswitch, §tableswitch), which are padded to force internal alignment of some of their operands on 4-byte boundaries.

字节码指令流仅单字节对齐。两个例外是`lookupswitch`和`tableswitch`指令（§lookupswitch，§tableswitch），它们通过填充来强制其某些操作数在4字节边界上对齐。

The decision to limit the Java Virtual Machine opcode to a byte and to forgo data alignment within compiled code reflects a conscious bias in favor of compactness, possibly at the cost of some performance in naive implementations. A one-byte opcode also limits the size of the instruction set. Not assuming data alignment means that immediate data larger than a byte must be constructed from bytes at run time on many machines.

将Java虚拟机操作码限制为一个字节并放弃在编译代码中对数据进行对齐的决定反映了一种有意识的偏向，即偏向于紧凑性，可能以在简单实现中牺牲一些性能为代价。一个字节的操作码也限制了指令集的大小。不假设数据对齐意味着在许多机器上，大小超过一个字节的立即数必须在运行时由字节构建。

### 2.11.1 Types and the Java Virtual Machine（指令类型和JVM）

Most of the instructions in the Java Virtual Machine instruction set encode type information about the operations they perform. For instance, the iload instruction (§iload) loads the contents of a local variable, which must be an int, onto the operand stack. The fload instruction (§fload) does the same with a float value. The two instructions may have identical implementations, but have distinct opcodes.

Java虚拟机指令集中大多数指令都会对它们执行的操作编码类型信息。例如，`iload`指令（§iload）将一个局部变量的内容加载到操作数堆栈上，该局部变量必须是`int`类型的。`fload`指令（§fload）对`float`值执行相同的操作。这两条指令可能具有相同的实现，但具有不同的操作码。

For the majority of typed instructions, the instruction type is represented explicitly in the opcode mnemonic by a letter: i for an int operation, l for long, s for short, b for byte, c for char, f for float, d for double, and a for reference. Some instructions for which the type is unambiguous do not have a type letter in their mnemonic. For instance, arraylength always operates on an object that is an array. Some instructions, such as goto, an unconditional control transfer, do not operate on typed operands.

对于大多数类型指令，指令类型在操作码助记符中由一个字母显式表示：`i`表示`int`操作，`l`表示`long`，`s`表示`short`，`b`表示`byte`，`c`表示`char`，`f`表示`float`，`d`表示`double`，`a`表示`reference`。对于类型明确的某些指令，其助记符中没有类型字母。例如，`arraylength`始终对一个对象（数组）操作。某些指令，如`goto`（无条件控制转移），不对类型化的操作数操作。

Given the Java Virtual Machine's one-byte opcode size, encoding types into opcodes places pressure on the design of its instruction set. If each typed instruction supported all of the Java Virtual Machine's run-time data types, there would be more instructions than could be represented in a byte. Instead, the instruction set of the Java Virtual Machine provides a reduced level of type support for certain operations. In other words, the instruction set is intentionally not orthogonal. Separate instructions can be used to convert between unsupported and supported data types as necessary.

考虑到Java虚拟机的单字节操作码大小，将类型编码到操作码中对其指令集的设计造成了压力。如果每种类型化的指令都支持Java虚拟机的所有运行时数据类型，那么指令数量将超过一个字节所能表示的数量。因此，Java虚拟机的指令集为某些操作提供了较少的类型支持。换句话说，指令集有意不是正交的。可以根据需要使用单独的指令在不支持和支持的数据类型之间进行转换。

Table 2.11.1-A summarizes the type support in the instruction set of the Java Virtual Machine. A specific instruction, with type information, is built by replacing the T in the instruction template in the opcode column by the letter in the type column. If the type column for some instruction template and type is blank, then no instruction exists supporting that type of operation. For instance, there is a load instruction for type int, iload, but there is no load instruction for type byte.

表2.11.1-A总结了Java虚拟机指令集中的类型支持。通过将操作码列中的指令模板中的`T`替换为类型列中的字母，可以构建特定的带有类型信息的指令。如果某个指令模板和类型的类型列为空，则不存在支持该类型操作的指令。例如，有一个用于`int`类型的加载指令`iload`，但没有用于`byte`类型的加载指令。

Note that most instructions in Table 2.11.1-A do not have forms for the integral types byte, char, and short. None have forms for the boolean type. A compiler encodes loads of literal values of types byte and short using Java Virtual Machine instructions that sign-extend those values to values of type int at compile-time or run-time. Loads of literal values of types boolean and char are encoded using instructions that zero-extend the literal to a value of type int at compile-time or run-time. Likewise, loads from arrays of values of type boolean, byte, short, and char are encoded using Java Virtual Machine instructions that sign-extend or zero-extend the values to values of type int. Thus, most operations on values of actual types boolean, byte, char, and short are correctly performed by instructions operating on values of computational type int.

请注意，表2.11.1-A中的大多数指令都没有针对整型类型`byte`、`char`和`short`的形式。没有任何指令具有`boolean`类型的形式。编译器使用Java虚拟机指令对`byte`和`short`类型的字面值加载进行编码，这些指令在编译时或运行时将这些值符号扩展为`int`类型的值。对`boolean`和`char`类型的字面值加载进行编码的指令在编译时或运行时将这些字面值零扩展为`int`类型的值。同样，从`boolean`、`byte`、`short`和`char`类型的值数组加载值的操作使用Java虚拟机指令进行编码，这些指令将值符号扩展或零扩展为`int`类型的值。因此，实际上大多数对`boolean`、`byte`、`char`和`short`类型值的操作都通过对计算类型为`int`的值进行操作的指令正确执行。

**Table 2.11.1-A. Type support in the Java Virtual Machine instruction set**

| opcode    | byte    | short   | int       | long    | float   | double  | char    | reference |  
| --------- | ------- | ------- | --------- | ------- | ------- | ------- | ------- | --------- |  
| Tipush    | bipush  | sipush  |           |         |         |         |         |           |  
| Tconst    |         |         | iconst    | lconst  | fconst  | dconst  |         | aconst    |  
| Tload     |         |         | iload     | lload   | fload   | dload   |         | aload     |  
| Tstore    |         |         | istore    | lstore  | fstore  | dstore  |         | astore    |  
| Tinc      |         |         | iinc      |         |         |         |         |           |  
| Taload    | baload  | saload  | iaload    | laload  | faload  | daload  | caload  | aaload    |  
| Tastore   | bastore | sastore | iastore   | lastore | fastore | dastore | castore | aastore   |  
| Tadd      |         |         | iadd      | ladd    | fadd    | dadd    |         |           |  
| Tsub      |         |         | isub      | lsub    | fsub    | dsub    |         |           |  
| Tmul      |         |         | imul      | lmul    | fmul    | dmul    |         |           |  
| Tdiv      |         |         | idiv      | ldiv    | fdiv    | ddiv    |         |           |  
| Trem      |         |         | irem      | lrem    | frem    | drem    |         |           |  
| Tneg      |         |         | ineg      | lneg    | fneg    | dneg    |         |           |  
| Tshl      |         |         | ishl      | lshl    |         |         |         |           |  
| Tshr      |         |         | ishr      | lshr    |         |         |         |           |  
| Tushr     |         |         | iushr     | lushr   |         |         |         |           |  
| Tand      |         |         | iand      | land    |         |         |         |           |  
| Tor       |         |         | ior       | lor     |         |         |         |           |  
| Txor      |         |         | ixor      | lxor    |         |         |         |           |  
| i2T       | i2b     | i2s     | i2l       | i2f     | i2d     |         |         |           |  
| l2T       |         |         | l2i       | l2f     | l2d     |         |         |           |  
| f2T       |         |         | f2i       | f2l     | f2d     |         |         |           |  
| d2T       |         |         | d2i       | d2l     | d2f     |         |         |           |  
| Tcmp      |         |         | lcmp      |         |         |         |         |           |  
| Tcmpl     |         |         |           |         | fcmpl   | dcmpl   |         |           |  
| Tcmpg     |         |         |           |         | fcmpg   | dcmpg   |         |           |  
| if_TcmpOP |         |         | if_icmpOP |         |         |         |         | if_acmpOP |  
| Treturn   |         |         | ireturn   | lreturn | freturn | dreturn |         | areturn   |  
  
The mapping between Java Virtual Machine actual types and Java Virtual Machine computational types is summarized by Table 2.11.1-B.

Java虚拟机实际类型和Java虚拟机计算类型之间的映射总结在表2.11.1-B中。

Certain Java Virtual Machine instructions such as pop and swap operate on the operand stack without regard to type; however, such instructions are constrained to use only on values of certain categories of computational types, also given in Table 2.11.1-B.

某些Java虚拟机指令（如`pop`和`swap`）在操作数堆栈上操作时不考虑类型；然而，这些指令仅限于在某些类别的计算类型值上使用，这些类别也在表2.11.1-B中给出。

**Table 2.11.1-B. Actual and Computational types in the Java Virtual Machine**

| Actual type    | Computational type | Category |  
|----------------|--------------------|----------|  
| boolean        | int                | 1        |  
| byte           | int                | 1        |  
| char           | int                | 1        |  
| short          | int                | 1        |  
| int            | int                | 1        |  
| float          | float              | 1        |  
| reference      | reference          | 1        |  
| returnAddress  | returnAddress      | 1        |  
| long           | long               | 2        |  
| double         | double             | 2        |  

### 2.11.2 Load and Store Instructions（加载和存储指令）

The load and store instructions transfer values between the local variables (§2.6.1) and the operand stack (§2.6.2) of a Java Virtual Machine frame (§2.6):

加载和存储指令在Java虚拟机帧（§2.6）的局部变量（§2.6.1）和操作数堆栈（§2.6.2）之间传递值：

- Load a local variable onto the operand stack: iload, iload_\<n>, lload, lload_\<n>, fload, fload_\<n>, dload, dload_\<n>, aload, aload_\<n>.

  将局部变量加载到操作数堆栈：`iload`, `iload_<n>`，`lload`，`lload_<n>`，`fload`，`fload_<n>`，`dload`，`dload_<n>`，`aload`，`aload_<n>`。

- Store a value from the operand stack into a local variable: istore, istore_\<n>, lstore, lstore_\<n>, fstore, fstore_\<n>, dstore, dstore_\<n>, astore, astore_\<n>.

  将值从操作数堆栈存储到局部变量：`istore`, `istore_<n>`，`lstore`，`lstore_<n>`，`fstore`，`fstore_<n>`，`dstore`，`dstore_<n>`，`astore`，`astore_<n>`。

- Load a constant onto the operand stack: bipush, sipush, ldc, ldc_w, ldc2_w, aconst_null, iconst_m1, iconst_\<i>, lconst_\<l>, fconst_\<f>, dconst_\<d>.

  将常量加载到操作数堆栈：`bipush`, `sipush`，`ldc`，`ldc_w`，`ldc2_w`，`aconst_null`，`iconst_m1`，`iconst_<i>`，`lconst_<l>`，`fconst_<f>`，`dconst_<d>`。

- Gain access to more local variables using a wider index, or to a larger immediate operand: wide.

  使用更宽的索引或更大的立即操作数访问更多局部变量：`wide`。

Instructions that access fields of objects and elements of arrays (§2.11.5) also transfer data to and from the operand stack.

访问对象字段和数组元素的指令（§2.11.5）也在操作数堆栈中传输数据。

Instruction mnemonics shown above with trailing letters between angle brackets (for instance, iload_\<n>) denote families of instructions (with members iload_0, iload_1, iload_2, and iload_3 in the case of iload_\<n>). Such families of instructions are specializations of an additional generic instruction (iload) that takes one operand. For the specialized instructions, the operand is implicit and does not need to be stored or fetched. The semantics are otherwise the same (iload_0 means the same thing as iload with the operand 0). The letter between the angle brackets specifies the type of the implicit operand for that family of instructions: for \<n>, a nonnegative integer; for \<i>, an int; for \<l>, a long; for \<f>, a float; and for \<d>, a double. Forms for type int are used in many cases to perform operations on values of type byte, char, and short (§2.11.1).

上面显示的带有尖括号之间尾随字母的指令助记符（例如`iload_<n>`）表示指令系列（例如`iload_<n>`的成员包括`iload_0`，`iload_1`，`iload_2`和`iload_3`）。这些指令系列是一个额外通用指令（`iload`）的特化形式，该指令需要一个操作数。对于特化的指令，操作数是隐式的，不需要存储或获取。语义相同（`iload_0`与操作数为0的`iload`具有相同的含义）。尖括号之间的字母指定了该指令系列的隐式操作数的类型：`<n>`表示非负整数，`<i>`表示`int`，`<l>`表示`long`，`<f>`表示`float`，`<d>`表示`double`。在许多情况下，`int`类型的形式用于对`byte`、`char`和`short`类型的值执行操作（§2.11.1）。

This notation for instruction families is used throughout this specification.

指令系列的这种表示法在整个规范中使用。

### 2.11.3 Arithmetic Instructions（算术指令）

The arithmetic instructions compute a result that is typically a function of two values on the operand stack, pushing the result back on the operand stack. There are two main kinds of arithmetic instructions: those operating on integer values and those operating on floating-point values.

算术指令计算的结果通常是操作数堆栈上两个值的函数，并将结果推回操作数堆栈。算术指令主要有两种：一种是对整数值进行操作的指令，另一种是对浮点数值进行操作的指令。

Within each of these kinds, the arithmetic instructions are specialized to Java Virtual Machine numeric types. There is no direct support for integer arithmetic on values of the byte, short, and char types (§2.11.1), or for values of the boolean type; those operations are handled by instructions operating on type int.

在这两种类型中，算术指令专门用于Java虚拟机的数值类型。对于`byte`、`short`和`char`类型的整数运算（§2.11.1）或`boolean`类型的值没有直接支持；这些操作由`int`类型操作的指令处理。

Integer and floating-point instructions also differ in their behavior on overflow and divide-by-zero.

整数和浮点数指令在溢出和除零操作上的行为也有所不同。

The arithmetic instructions are as follows:

算术指令如下：

- Add: iadd, ladd, fadd, dadd.

  加法：`iadd`，`ladd`，`fadd`，`dadd`。

- Subtract: isub, lsub, fsub, dsub.

  减法：`isub`，`lsub`，`fsub`，`dsub`。

- Multiply: imul, lmul, fmul, dmul.

  乘法：`imul`，`lmul`，`fmul`，`dmul`。

- Divide: idiv, ldiv, fdiv, ddiv.
  
  除法：`idiv`，`ldiv`，`fdiv`，`ddiv`。

- Remainder: irem, lrem, frem, drem.

  取余：`irem`，`lrem`，`frem`，`drem`。

- Negate: ineg, lneg, fneg, dneg.

  取反：`ineg`，`lneg`，`fneg`，`dneg`。

- Shift: ishl, ishr, iushr, lshl, lshr, lushr.

  移位：`ishl`，`ishr`，`iushr`，`lshl`，`lshr`，`lushr`。

- Bitwise OR: ior, lor.

  按位或：`ior`，`lor`。

- Bitwise AND: iand, land.

  按位与：`iand`，`land`。

- Bitwise exclusive OR: ixor, lxor.

  按位异或：`ixor`，`lxor`。

- Local variable increment: iinc.

  局部变量增量：`iinc`。

- Comparison: dcmpg, dcmpl, fcmpg, fcmpl, lcmp.

  比较：`dcmpg`，`dcmpl`，`fcmpg`，`fcmpl`，`lcmp`。

The semantics of the Java programming language operators on integer and floating-point values (JLS §4.2.2, JLS §4.2.4) are directly supported by the semantics of the Java Virtual Machine instruction set.

Java编程语言在整数和浮点数值上的操作符语义（JLS §4.2.2，JLS §4.2.4）由Java虚拟机指令集的语义直接支持。

The Java Virtual Machine does not indicate overflow during operations on integer data types. The only integer operations that can throw an exception are the integer divide instructions (idiv and ldiv) and the integer remainder instructions (irem and lrem), which throw an ArithmeticException if the divisor is zero.

Java虚拟机在整数数据类型的操作期间不指示溢出。唯一可能抛出异常的整数操作是整数除法指令（`idiv`和`ldiv`）以及整数取余指令（`irem`和`lrem`），如果除数为零，它们会抛出`ArithmeticException`。

Java Virtual Machine operations on floating-point numbers behave as specified in IEEE 754. In particular, the Java Virtual Machine requires full support of IEEE 754 denormalized floating-point numbers and gradual underflow, which make it easier to prove desirable properties of particular numerical algorithms.

Java虚拟机在浮点数上的操作行为如IEEE 754所规定的那样。特别是，Java虚拟机要求完全支持IEEE 754的非标准化浮点数和逐渐下溢，这使得证明特定数值算法的理想属性更加容易。

The Java Virtual Machine requires that floating-point arithmetic behave as if every floating-point operator rounded its floating-point result to the result precision. Inexact results must be rounded to the representable value nearest to the infinitely precise result; if the two nearest representable values are equally near, the one having a least significant bit of zero is chosen. This is the IEEE 754 standard's default rounding mode, known as round to nearest mode.

Java虚拟机要求浮点数运算的行为就像每个浮点运算符将其浮点结果舍入到结果精度一样。不精确的结果必须舍入到最接近无限精确结果的可表示值；如果两个最接近的可表示值一样近，则选择最不重要位为零的值。这是IEEE 754标准的默认舍入模式，称为舍入到最近模式。

The Java Virtual Machine uses the IEEE 754 round towards zero mode when converting a floating-point value to an integer. This results in the number being truncated; any bits of the significand that represent the fractional part of the operand value are discarded. Round towards zero mode chooses as its result the type's value closest to, but no greater in magnitude than, the infinitely precise result.

Java虚拟机在将浮点值转换为整数时使用IEEE 754的朝零舍入模式。这导致数字被截断；表示操作数值的小数部分的有效位被丢弃。朝零舍入模式选择其结果为最接近但不大于无限精确结果的类型值。

The Java Virtual Machine's floating-point operators do not throw run-time exceptions (not to be confused with IEEE 754 floating-point exceptions).

Java虚拟机的浮点运算符不会抛出运行时异常（不要与IEEE 754的浮点异常混淆）。

An operation that overflows produces a signed infinity, an operation that underflows produces a denormalized value or a signed zero, and an operation that has no mathematically definite result produces NaN. All numeric operations with NaN as an operand produce NaN as a result.

溢出操作产生一个有符号无穷大，下溢操作产生一个非标准化值或有符号零，而没有数学上确定结果的操作会产生NaN。所有以NaN为操作数的数值操作都会产生NaN结果。

Comparisons on values of type long (lcmp) perform a signed comparison. Comparisons on values of floating-point types (dcmpg, dcmpl, fcmpg, fcmpl) are performed using IEEE 754 nonsignaling comparisons.

对`long`类型值的比较（`lcmp`）执行有符号比较。对浮点数类型值的比较（`dcmpg`、`dcmpl`、`fcmpg`、`fcmpl`）使用IEEE 754非信号比较执行。

###  2.11.4 Type Conversion Instructions（类型转换指令）

The type conversion instructions allow conversion between Java Virtual Machine numeric types. These may be used to implement explicit conversions in user code or to mitigate the lack of orthogonality in the instruction set of the Java Virtual Machine.

类型转换指令允许在Java虚拟机数值类型之间进行转换。这些指令可以用于在用户代码中实现显式转换，或者缓解Java虚拟机指令集中正交性不足的问题。

The Java Virtual Machine directly supports the following widening numeric conversions:  

Java虚拟机直接支持以下数值扩展转换：

- int to long, float, or double

  `int`到`long`、`float`或`double`

- long to float or double

  `long`到`float`或`double`

- float to double

  `float`到`double`

The widening numeric conversion instructions are i2l, i2f, i2d, l2f, l2d, and f2d. The mnemonics for these opcodes are straightforward given the naming conventions for typed instructions and the punning use of 2 to mean "to." For instance, the i2d instruction converts an int value to a double.

数值扩展转换指令包括`i2l`、`i2f`、`i2d`、`l2f`、`l2d`和`f2d`。这些操作码的助记符是直接的，考虑到类型化指令的命名约定和使用`2`来表示“到”的双关语。例如，`i2d`指令将`int`值转换为`double`。

Most widening numeric conversions do not lose information about the overall magnitude of a numeric value. Indeed, conversions widening from int to long and int to double do not lose any information at all; the numeric value is preserved exactly. Conversions widening from float to double that are FP-strict (§2.8.2) also preserve the numeric value exactly; only such conversions that are not FP-strict may lose information about the overall magnitude of the converted value.

大多数数值扩展转换不会丢失关于数值整体幅度的信息。实际上，从`int`到`long`和`int`到`double`的扩展转换根本不会丢失任何信息；数值完全保留。从`float`到`double`的`FP-strict`转换（§2.8.2）也完全保留数值；只有非`FP-strict`的此类转换可能会丢失关于转换值整体幅度的信息。

Conversions from int to float, or from long to float, or from long to double, may lose precision, that is, may lose some of the least significant bits of the value; the resulting floating-point value is a correctly rounded version of the integer value, using IEEE 754 round to nearest mode.

从`int`到`float`，或从`long`到`float`，或从`long`到`double`的转换可能会丢失精度，即可能会丢失值的某些最低有效位；生成的浮点值是整数值的正确舍入版本，使用IEEE 754的舍入到最近模式。

Despite the fact that loss of precision may occur, widening numeric conversions never cause the Java Virtual Machine to throw a run-time exception (not to be confused with an IEEE 754 floating-point exception).

尽管可能会发生精度丧失，但数值扩展转换从不会导致Java虚拟机抛出运行时异常（不要与IEEE 754的浮点异常混淆）。

A widening numeric conversion of an int to a long simply sign-extends the two's-complement representation of the int value to fill the wider format. A widening numeric conversion of a char to an integral type zero-extends the representation of the char value to fill the wider format.

将`int`扩展为`long`的数值扩展转换仅将`int`值的二进制补码表示符号扩展以填充更宽的格式。将`char`扩展为整型的数值扩展转换通过零扩展`char`值的表示来填充更宽的格式。

Note that widening numeric conversions do not exist from integral types byte, char, and short to type int. As noted in §2.11.1, values of type byte, char, and short are internally widened to type int, making these conversions implicit.

请注意，没有从整型类型`byte`、`char`和`short`到`int`类型的数值扩展转换。如§2.11.1中所述，`byte`、`char`和`short`类型的值在内部扩展为`int`类型，使得这些转换是隐式的。

The Java Virtual Machine also directly supports the following narrowing numeric conversions:

Java虚拟机还直接支持以下数值缩减转换：

- int to byte, short, or char

  `int`到`byte`、`short`或`char`

- long to int

  `long`到`int`

- float to int or long

  `float`到`int`或`long`

- double to int, long, or float

  `double`到`int`、`long`或`float`

The narrowing numeric conversion instructions are i2b, i2c, i2s, l2i, f2i, f2l, d2i, d2l, and d2f. A narrowing numeric conversion can result in a value of different sign, a different order of magnitude, or both; it may thereby lose precision.

数值缩减转换指令包括`i2b`、`i2c`、`i2s`、`l2i`、`f2i`、`f2l`、`d2i`、`d2l`和`d2f`。数值缩减转换可能导致符号、数量级或两者都不同的值；因此它可能会丢失精度。

A narrowing numeric conversion of an int or long to an integral type T simply discards all but the n lowest-order bits, where n is the number of bits used to represent type T. This may cause the resulting value not to have the same sign as the input value.

将`int`或`long`缩减为整型`T`的数值缩减转换仅丢弃所有除n个最低位之外的位，其中n是表示`T`类型所用的位数。这可能会导致生成的值与输入值的符号不同。

In a narrowing numeric conversion of a floating-point value to an integral type T, where T is either int or long, the floating-point value is converted as follows:

在将浮点值缩减为整型`T`的数值缩减转换中，其中`T`可以是`int`或`long`，浮点值的转换如下：

- If the floating-point value is NaN, the result of the conversion is an int or long 0.

  如果浮点值为NaN，则转换的结果是`int`或`long`的0。

- Otherwise, if the floating-point value is not an infinity, the floating-point value is rounded to an integer value V using IEEE 754 round towards zero mode. There are two cases:

  否则，如果浮点值不是无穷大，则使用IEEE 754的朝零舍入模式将浮点值舍入为整数值V。有两种情况：

  - If T is long and this integer value can be represented as a long, then the result is the long value V.

      如果`T`是`long`，并且该整数值可以表示为`long`，则结果是`long`值V。

  - If T is of type int and this integer value can be represented as an int, then the result is the int value V.

      如果`T`是`int`，并且该整数值可以表示为`int`，则结果是`int`值V。

- Otherwise:

  否则：

  - Either the value must be too small (a negative value of large magnitude or negative infinity), and the result is the smallest representable value of type int or long.

      值必须太小（大幅度负值或负无穷大），结果是`int`或`long`类型的最小可表示值。

  - Or the value must be too large (a positive value of large magnitude or positive infinity), and the result is the largest representable value of type int or long.

      值必须太大（大幅度正值或正无穷大），结果是`int`或`long`类型的最大可表示值。

A narrowing numeric conversion from double to float behaves in accordance with IEEE 754. The result is correctly rounded using IEEE 754 round to nearest mode. A value too small to be represented as a float is converted to a positive or negative zero of type float; a value too large to be represented as a float is converted to a positive or negative infinity. A double NaN is always converted to a float NaN.

将`double`缩减为`float`的数值缩减转换按照IEEE 754标准进行。结果使用IEEE 754的舍入到最近模式进行正确舍入。一个太小而不能表示为`float`的值将转换为`float`类型的正零或负零；一个太大而不能表示为`float`的值将转换为正无穷大或负无穷大。`double`的NaN始终转换为`float`的NaN。

Despite the fact that overflow, underflow, or loss of precision may occur, narrowing conversions among numeric types never cause the Java Virtual Machine to throw a run-time exception (not to be confused with an IEEE 754 floating-point exception).

尽管可能会发生溢出、下溢或精度丧失，但数值类型之间的缩减转换从不会导致Java虚拟机抛出运行时异常（不要与IEEE 754的浮点异常混淆）。

### 2.11.5 Object Creation and Manipulation（对象创建和操作指令）

Although both class instances and arrays are objects, the Java Virtual Machine creates and manipulates class instances and arrays using distinct sets of instructions:

虽然类实例和数组都是对象，但Java虚拟机使用不同的指令集来创建和操作类实例和数组：

- Create a new class instance: new.

  创建一个新的类实例：`new`。

- Create a new array: newarray, anewarray, multianewarray.

  创建一个新的数组：`newarray`、`anewarray`、`multianewarray`。

- Access fields of classes (static fields, known as class variables) and fields of class instances (non-static fields, known as instance variables): getstatic, putstatic, getfield, putfield.

  访问类的字段（静态字段，称为类变量）和类实例的字段（非静态字段，称为实例变量）：`getstatic`、`putstatic`、`getfield`、`putfield`。

- Load an array component onto the operand stack: baload, caload, saload, iaload, laload, faload, daload, aaload.

  将数组组件加载到操作数堆栈：`baload`、`caload`、`saload`、`iaload`、`laload`、`faload`、`daload`、`aaload`。

- Store a value from the operand stack as an array component: bastore, castore, sastore, iastore, lastore, fastore, dastore, aastore.

  将值从操作数堆栈存储为数组组件：`bastore`、`castore`、`sastore`、`iastore`、`lastore`、`fastore`、`dastore`、`aastore`。

- Get the length of array: arraylength.

  获取数组的长度：`arraylength`。

- Check properties of class instances or arrays: instanceof, checkcast.

  检查类实例或数组的属性：`instanceof`、`checkcast`。

### 2.11.6 Operand Stack Management Instructions（操作数栈管理指令）

A number of instructions are provided for the direct manipulation of the operand stack: pop, pop2, dup, dup2, dup_x1, dup2_x1, dup_x2, dup2_x2, swap.

提供了一些指令用于直接操作操作数栈：`pop`、`pop2`、`dup`、`dup2`、`dup_x1`、`dup2_x1`、`dup_x2`、`dup2_x2`、`swap`。

### 2.11.7 Control Transfer Instructions（控制转移指令）

The control transfer instructions conditionally or unconditionally cause the Java Virtual Machine to continue execution with an instruction other than the one following the control transfer instruction. They are:

控制转移指令有条件或无条件地使Java虚拟机继续执行控制转移指令后面的指令。它们是：

- Conditional branch: ifeq, ifne, iflt, ifle, ifgt, ifge, ifnull, ifnonnull, if_icmpeq, if_icmpne, if_icmplt, if_icmple, if_icmpgt, if_icmpge, if_acmpeq, if_acmpne.

  条件分支：`ifeq`、`ifne`、`iflt`、`ifle`、`ifgt`、`ifge`、`ifnull`、`ifnonnull`、`if_icmpeq`、`if_icmpne`、`if_icmplt`、`if_icmple`、`if_icmpgt`、`if_icmpge`、`if_acmpeq`、`if_acmpne`。

- Compound conditional branch: tableswitch, lookupswitch.

  复合条件分支：`tableswitch`、`lookupswitch`。

- Unconditional branch: goto, goto_w, jsr, jsr_w, ret.

  无条件分支：`goto`、`goto_w`、`jsr`、`jsr_w`、`ret`。  

The Java Virtual Machine has distinct sets of instructions that conditionally branch on comparison with data of int and reference types. It also has distinct conditional branch instructions that test for the null reference and thus it is not required to specify a concrete value for null (§2.4).

Java虚拟机有不同的指令集用于根据`int`和引用类型数据进行条件分支。它还有专门的条件分支指令用于测试空引用，因此不需要为`null`指定具体值（§2.4）。

Conditional branches on comparisons between data of types boolean, byte, char, and short are performed using int comparison instructions (§2.11.1). A conditional branch on a comparison between data of types long, float, or double is initiated using an instruction that compares the data and produces an int result of the comparison (§2.11.3). A subsequent int comparison instruction tests this result and effects the conditional branch. Because of its emphasis on int comparisons, the Java Virtual Machine provides a rich complement of conditional branch instructions for type int.

对于`boolean`、`byte`、`char`和`short`类型数据之间比较的条件分支使用`int`比较指令执行（§2.11.1）。对于`long`、`float`或`double`类型数据之间比较的条件分支使用比较数据并生成`int`比较结果的指令启动（§2.11.3）。后续的`int`比较指令测试此结果并执行条件分支。由于其对`int`比较的强调，Java虚拟机为`int`类型提供了丰富的条件分支指令。

All int conditional control transfer instructions perform signed comparisons.

所有`int`条件控制转移指令执行有符号比较。

### 2.11.8 Method Invocation and Return Instructions（方法调用和返回指令）

The following five instructions invoke methods:

以下五个指令用于调用方法：

- invokevirtual invokes an instance method of an object, dispatching on the (virtual) type of the object. This is the normal method dispatch in the Java programming language.  
  
  `invokevirtual`调用对象的实例方法，基于对象的（虚拟）类型进行分派。这是Java编程语言中的正常方法分派。  
  
- invokeinterface invokes an interface method, searching the methods implemented by the particular run-time object to find the appropriate method.

  `invokeinterface`调用接口方法，搜索特定运行时对象实现的方法以找到合适的方法。

- invokespecial invokes an instance method requiring special handling, whether an instance initialization method (§2.9), a private method, or a superclass method.

  `invokespecial`调用需要特殊处理的实例方法，无论是实例初始化方法（§2.9）、私有方法还是超类方法。

- invokestatic invokes a class (static) method in a named class.

  `invokestatic`调用命名类中的类（静态）方法。

- invokedynamic invokes the method which is the target of the call site object bound to the invokedynamic instruction. The call site object was bound to a specific lexical occurrence of the invokedynamic instruction by the Java Virtual Machine as a result of running a bootstrap method before the first execution of the instruction. Therefore, each occurrence of an invokedynamic instruction has a unique linkage state, unlike the other instructions which invoke methods.

  `invokedynamic`调用绑定到`invokedynamic`指令的调用站点对象的目标方法。调用站点对象是在指令首次执行之前，由Java虚拟机在运行引导方法时绑定到`invokedynamic`指令的特定词法出现。因此，每次出现`invokedynamic`指令时，都会具有唯一的链接状态，这与其他调用方法的指令不同。

The method return instructions, which are distinguished by return type, are ireturn (used to return values of type boolean, byte, char, short, or int), lreturn, freturn, dreturn, and areturn. In addition, the return instruction is used to return from methods declared to be void, instance initialization methods, and class or interface initialization methods.

方法返回指令按返回类型区分为`ireturn`（用于返回`boolean`、`byte`、`char`、`short`或`int`类型的值）、`lreturn`、`freturn`、`dreturn`和`areturn`。此外，`return`指令用于从声明为`void`的方法、实例初始化方法以及类或接口初始化方法中返回。

### 2.11.9 Throwing Exceptions（异常抛出指令）

An exception is thrown programmatically using the athrow instruction. Exceptions can also be thrown by various Java Virtual Machine instructions if they detect an abnormal condition.  

使用`athrow`指令以编程方式抛出异常。如果Java虚拟机的各种指令检测到异常情况，也会抛出异常。

### 2.11.10 Synchronization（同步指令）

The Java Virtual Machine supports synchronization of both methods and sequences of instructions within a method by a single synchronization construct: the monitor.

Java虚拟机通过单一的同步结构——监视器（monitor），支持方法和方法内指令序列的同步。

Method-level synchronization is performed implicitly, as part of method invocation and return (§2.11.8). A synchronized method is distinguished in the run-time constant pool's method_info structure (§4.6) by the ACC_SYNCHRONIZED flag, which is checked by the method invocation instructions. When invoking a method for which ACC_SYNCHRONIZED is set, the executing thread enters a monitor, invokes the method itself, and exits the monitor whether the method invocation completes normally or abruptly. During the time the executing thread owns the monitor, no other thread may enter it. If an exception is thrown during invocation of the synchronized method and the synchronized method does not handle the exception, the monitor for the method is automatically exited before the exception is rethrown out of the synchronized method.  

方法级别的同步隐式执行，作为方法调用和返回的一部分（§2.11.8）。同步方法在运行时常量池的`method_info`结构（§4.6）中由`ACC_SYNCHRONIZED`标志区分，方法调用指令检查该标志。当调用设置了`ACC_SYNCHRONIZED`的 方法时，执行线程进入一个监视器，调用方法本身，并在方法调用正常或突然完成时退出监视器。在执行线程拥有监视器期间，其他线程无法进入。如果在调用同步方法期间抛出异常并且同步方法未处理该异常，则在异常从同步方法重新抛出之前，自动退出该方法的监视器。

Synchronization of sequences of instructions is typically used to encode the synchronized block of the Java programming language. The Java Virtual Machine supplies the monitorenter and monitorexit instructions to support such language constructs. Proper implementation of synchronized blocks requires cooperation from a compiler targeting the Java Virtual Machine (§3.14).

指令序列的同步通常用于编码Java编程语言的同步块。Java虚拟机提供了`monitorenter`和`monitorexit`指令来支持这种语言结构。同步块的正确实现需要编译器与针对Java虚拟机的协作（§3.14）。

Structured locking is the situation when, during a method invocation, every exit on a given monitor matches a preceding entry on that monitor. Since there is no assurance that all code submitted to the Java Virtual Machine will perform structured locking, implementations of the Java Virtual Machine are permitted but not required to enforce both of the following two rules guaranteeing structured locking. Let T be a thread and M be a monitor. Then:

结构化锁定是指在方法调用期间，给定监视器上的每个退出都与之前在该监视器上的进入相匹配。由于无法保证提交给Java虚拟机的所有代码都会执行结构化锁定，因此Java虚拟机的实现可以但不必强制执行以下两条规则来保证结构化锁定。假设T为一个线程，M为一个监视器。那么：

1. The number of monitor entries performed by T on M during a method invocation must equal the number of monitor exits performed by T on M during the method invocation whether the method invocation completes normally or abruptly.

   在方法调用期间，线程T在监视器M上执行的进入次数必须等于线程T在监视器M上执行的退出次数，无论方法调用是正常完成还是突然完成。

2. At no point during a method invocation may the number of monitor exits performed by T on M since the method invocation exceed the number of monitor entries performed by T on M since the method invocation.

   在方法调用期间的任何时候，线程T在监视器M上执行的退出次数都不得超过方法调用以来线程T在监视器M上执行的进入次数。

Note that the monitor entry and exit automatically performed by the Java Virtual Machine when invoking a synchronized method are considered to occur during the calling method's invocation.

请注意，Java虚拟机在调用同步方法时自动执行的监视器进入和退出被认为发生在调用方法的调用期间。

---  
## 2.12 Class Libraries（类库）

The Java Virtual Machine must provide sufficient support for the implementation of the class libraries of the Java SE platform. Some of the classes in these libraries cannot be implemented without the cooperation of the Java Virtual Machine.

Java虚拟机必须提供足够的支持来实现Java SE平台的类库。这些类库中的某些类在没有Java虚拟机的协作下无法实现。

Classes that might require special support from the Java Virtual Machine include those that support:

可能需要Java虚拟机特别支持的类包括以下支持的类：

- Reflection, such as the classes in the package java.lang.reflect and the class Class.

  反射，例如`java.lang.reflect`包中的类和`Class`类。

- Loading and creation of a class or interface. The most obvious example is the class ClassLoader.

  类或接口的加载和创建。最明显的例子是`ClassLoader`类。

- Linking and initialization of a class or interface. The example classes cited above fall into this category as well.

  类或接口的链接和初始化。上面提到的示例类也属于此类别。

- Security, such as the classes in the package java.security and other classes such as SecurityManager.

  安全性，例如`java.security`包中的类和其他类如`SecurityManager`。

- Multithreading, such as the class Thread.

  多线程，例如`Thread`类。

- Weak references, such as the classes in the package java.lang.ref.

  弱引用，例如`java.lang.ref`包中的类。

The list above is meant to be illustrative rather than comprehensive. An exhaustive list of these classes or of the functionality they provide is beyond the scope of this specification. See the specifications of the Java SE platform class libraries for details.

上面的列表是示例性的，而不是全面的。对这些类或它们提供的功能的详尽列表超出了本规范的范围。有关详细信息，请参见Java SE平台类库的规范。

---  
## 2.13 Public Design, Private Implementation（公共设计和私有实现）

Thus far this specification has sketched the public view of the Java Virtual Machine: the class file format and the instruction set. These components are vital to the hardware-, operating system-, and implementation-independence of the Java Virtual Machine. The implementor may prefer to think of them as a means to securely communicate fragments of programs between hosts each implementing the Java SE platform, rather than as a blueprint to be followed exactly.

到目前为止，本规范已勾勒出Java虚拟机的公共视图：类文件格式和指令集。这些组件对于Java虚拟机的硬件、操作系统和实现的独立性至关重要。实现者可能更愿意将它们视为在各自实现Java SE平台的主机之间安全传递程序片段的一种手段，而不是必须精确遵循的蓝图。

It is important to understand where the line between the public design and the private implementation lies. A Java Virtual Machine implementation must be able to read class files and must exactly implement the semantics of the Java Virtual Machine code therein. One way of doing this is to take this document as a specification and to implement that specification literally. But it is also perfectly feasible and desirable for the implementor to modify or optimize the implementation within the constraints of this specification. So long as the class file format can be read and the semantics of its code are maintained, the implementor may implement these semantics in any way. What is "under the hood" is the implementor's business, as long as the correct external interface is carefully maintained.

理解公共设计与私有实现之间的界限至关重要。Java虚拟机的实现必须能够读取类文件，并且必须准确地实现其中的Java虚拟机代码的语义。一种实现方法是将本文档作为规范，并严格按照该规范实现。但是，在本规范的约束范围内，修改或优化实现也是完全可行和可取的。只要可以读取类文件格式并且保持其代码的语义，实施者可以以任何方式实现这些语义。只要正确维护外部接口，内部如何实现由实现者决定。

There are some exceptions: debuggers, profilers, and just-in-time code generators can each require access to elements of the Java Virtual Machine that are normally considered to be "under the hood." Where appropriate, Oracle works with other Java Virtual Machine implementors and with tool vendors to develop common interfaces to the Java Virtual Machine for use by such tools, and to promote those interfaces across the industry.

有一些例外：调试器、分析器和即时代码生成器可能需要访问Java虚拟机的某些通常被认为是“内部实现”的元素。在适当的情况下，Oracle与其他Java虚拟机实现者和工具供应商合作，开发用于这些工具的Java虚拟机的通用接口，并在整个行业中推广这些接口。

The implementor can use this flexibility to tailor Java Virtual Machine implementations for high performance, low memory use, or portability. What makes sense in a given implementation depends on the goals of that implementation. The range of implementation options includes the following:

实现者可以利用这种灵活性来为高性能、低内存使用或可移植性定制Java虚拟机的实现。特定实现中有意义的内容取决于该实现的目标。实现选项的范围包括以下内容：

- Translating Java Virtual Machine code at load-time or during execution into the instruction set of another virtual machine.

  在加载时或执行期间将Java虚拟机代码转换为另一种虚拟机的指令集。

- Translating Java Virtual Machine code at load-time or during execution into the native instruction set of the host CPU (sometimes referred to as just-in-time, or JIT, code generation).

  在加载时或执行期间将Java虚拟机代码转换为主机CPU的本机指令集（有时称为即时（JIT）代码生成）。

The existence of a precisely defined virtual machine and object file format need not significantly restrict the creativity of the implementor. The Java Virtual Machine is designed to support many different implementations, providing new and interesting solutions while retaining compatibility between implementations.

精确定义的虚拟机和对象文件格式的存在并不需要显著限制实现者的创造力。Java虚拟机旨在支持多种不同的实现，提供新的和有趣的解决方案，同时保持实现之间的兼容性。