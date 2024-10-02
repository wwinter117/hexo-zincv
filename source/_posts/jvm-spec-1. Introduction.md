---
title: jvm-spec-1. Introduction
tags:
  - 官方文档翻译
  - jvm
date: 2022-9-1 22:46:49
---

## 1.1 A Bit of History（一些历史）

The Java® programming language is a general-purpose, concurrent, object-oriented language. Its syntax is similar to C and C++, but it omits many of the features that make C and C++ complex, confusing, and unsafe. The Java platform was initially developed to address the problems of building software for networked consumer devices. It was designed to support multiple host architectures and to allow secure delivery of software components. To meet these requirements, compiled code had to survive transport across networks, operate on any client, and assure the client that it was safe to run.

Java编程语言是一种通用的、并发的、面向对象的语言。它的语法类似于C和C++，但省去了许多使C和C++变得复杂、混乱和不安全的特性。Java平台最初开发是为了应对为联网设备构建软件时遇到的问题。它的设计目标是支持多种主机架构，并允许安全地交付软件组件。为了满足这些要求，编译后的代码必须能够在网络上传输、生存，并在任何客户端上运行，同时确保客户端运行时是安全的。

The popularization of the World Wide Web made these attributes much more interesting. Web browsers enabled millions of people to surf the Net and access media-rich content in simple ways. At last there was a medium where what you saw and heard was essentially the same regardless of the machine you were using and whether it was connected to a fast network or a slow modem.

万维网的普及使这些特性变得更加有趣。Web浏览器使数百万人能够以简单的方式上网并访问媒体丰富的内容。最终，无论你使用的是什么机器，也无论它连接的是快速网络还是慢速调制解调器，你所看到和听到的内容本质上都是一样的。

Web enthusiasts soon discovered that the content supported by the Web's HTML document format was too limited. HTML extensions, such as forms, only highlighted those limitations, while making it clear that no browser could include all the features users wanted. Extensibility was the answer.

网络爱好者很快发现，Web的HTML文档格式支持的内容过于有限。HTML扩展，例如表单，仅仅突出了这些限制，并且显而易见，没有任何浏览器能够包含用户想要的所有功能。可扩展性是答案。

The HotJava browser first showcased the interesting properties of the Java programming language and platform by making it possible to embed programs inside HTML pages. Programs are transparently downloaded into the browser along with the HTML pages in which they appear. Before being accepted by the browser, programs are carefully checked to make sure they are safe. Like HTML pages, compiled programs are network- and host-independent. The programs behave the same way regardless of where they come from or what kind of machine they are being loaded into and run on.

HotJava浏览器首次展示了Java编程语言和平台的有趣特性，使得可以将程序嵌入到HTML页面中。程序随着它们所在的HTML页面一起透明地下载到浏览器中。在被浏览器接受之前，程序会被仔细检查以确保它们是安全的。与HTML页面一样，编译后的程序是网络和主机无关的。这些程序无论来源何处，或被加载到何种机器上运行，行为方式都是相同的。

---
## 1.2 The Java Virtual Machine（Java虚拟机）

The Java Virtual Machine is the cornerstone of the Java platform. It is the component of the technology responsible for its hardware- and operating system-independence, the small size of its compiled code, and its ability to protect users from malicious programs.

Java虚拟机是Java平台的基石。它是负责实现技术的硬件和操作系统独立性、编译代码的小尺寸以及保护用户免受恶意程序侵害的组件。

The Java Virtual Machine is an abstract computing machine. Like a real computing machine, it has an instruction set and manipulates various memory areas at run time. It is reasonably common to implement a programming language using a virtual machine; the best-known virtual machine may be the P-Code machine of UCSD Pascal.

Java虚拟机是一种抽象的计算机。与实际的计算机一样，它有一套指令集，并在运行时操作各种内存区域。使用虚拟机来实现编程语言是相当常见的做法；最著名的虚拟机可能是UCSD Pascal的P-Code机。

The first prototype implementation of the Java Virtual Machine, done at Sun Microsystems, Inc., emulated the Java Virtual Machine instruction set in software hosted by a handheld device that resembled a contemporary Personal Digital Assistant (PDA). Oracle's current implementations emulate the Java Virtual Machine on mobile, desktop and server devices, but the Java Virtual Machine does not assume any particular implementation technology, host hardware, or host operating system. It is not inherently interpreted, but can just as well be implemented by compiling its instruction set to that of a silicon CPU. It may also be implemented in microcode or directly in silicon.

Java虚拟机的第一个原型实现是在Sun Microsystems, Inc.完成的，它在一个类似当代个人数字助理（PDA）的手持设备上模拟了Java虚拟机的指令集。Oracle目前的实现版本在移动设备、桌面设备和服务器设备上模拟了Java虚拟机，但Java虚拟机并不假设任何特定的实现技术、主机硬件或主机操作系统。它并非天生就是解释执行的，但也可以通过将其指令集编译成硅基CPU的指令集来实现。它还可以通过微代码或直接在硅片上实现。

The Java Virtual Machine knows nothing of the Java programming language, only of a particular binary format, the class file format. A class file contains Java Virtual Machine instructions (or bytecodes) and a symbol table, as well as other ancillary information.

Java虚拟机对Java编程语言一无所知，它只了解一种特定的二进制格式，即类文件格式。类文件包含Java虚拟机指令（或字节码）和符号表，以及其他辅助信息。

For the sake of security, the Java Virtual Machine imposes strong syntactic and structural constraints on the code in a class file. **However, any language with functionality that can be expressed in terms of a valid class file can be hosted by the Java Virtual Machine.** Attracted by a generally available, machine-independent platform, implementors of other languages can turn to the Java Virtual Machine as a delivery vehicle for their languages.

为了安全起见，Java虚拟机对类文件中的代码施加了强烈的语法和结构约束。**然而，任何功能可以用有效类文件来表达的语言都可以由Java虚拟机托管。** 由于Java虚拟机是一个广泛可用的、与机器无关的平台，其他语言的实现者可以将Java虚拟机作为他们语言的交付工具。

---
## 1.3 Organization of the Specification（规范文档的组织结构）

Chapter 2 gives an overview of the Java Virtual Machine architecture.

第2章概述了Java虚拟机架构。

Chapter 3 introduces compilation of code written in the Java programming language into the instruction set of the Java Virtual Machine.

第3章介绍了将用Java编程语言编写的代码编译为Java虚拟机指令集。

Chapter 4 specifies the class file format, the hardware- and operating system-independent binary format used to represent compiled classes and interfaces.

第4章规定了类文件格式，这是一种用于表示编译后的类和接口的硬件和操作系统无关的二进制格式。

Chapter 5 specifies the start-up of the Java Virtual Machine and the loading, linking, and initialization of classes and interfaces.

第5章规定了Java虚拟机的启动以及类和接口的加载、链接和初始化。

Chapter 6 specifies the instruction set of the Java Virtual Machine, presenting the instructions in alphabetical order of opcode mnemonics.

第6章规定了Java虚拟机的指令集，并按操作码助记符的字母顺序排列指令。

Chapter 7 gives a table of Java Virtual Machine opcode mnemonics indexed by opcode value.

第7章提供了按操作码值索引的Java虚拟机操作码助记符表。

In the Second Edition of The Java® Virtual Machine Specification, Chapter 2 gave an overview of the Java programming language that was intended to support the specification of the Java Virtual Machine but was not itself a part of the specification. In The Java Virtual Machine Specification, Java SE 8 Edition, the reader is referred to The Java Language Specification, Java SE 8 Edition for information about the Java programming language. References of the form: (JLS §x.y) indicate where this is necessary.

在第二版《Java®虚拟机规范》中，第2章概述了Java编程语言，该语言旨在支持Java虚拟机规范，但它本身并不是规范的一部分。在《Java虚拟机规范，Java SE 8版》中，读者可参考《Java语言规范，Java SE 8版》以获取有关Java编程语言的信息。格式为（JLS §x.y）的引用表明何处需要参考。

In the Second Edition of The Java® Virtual Machine Specification, Chapter 8 detailed the low-level actions that explained the interaction of Java Virtual Machine threads with a shared main memory. In The Java Virtual Machine Specification, Java SE 8 Edition, the reader is referred to Chapter 17 of The Java Language Specification, Java SE 8 Edition for information about threads and locks. Chapter 17 reflects The Java Memory Model and Thread Specification produced by the JSR 133 Expert Group.

在第二版《Java®虚拟机规范》中，第8章详细描述了解释Java虚拟机线程与共享主存交互的低级操作。在《Java虚拟机规范，Java SE 8版》中，读者可参考《Java语言规范，Java SE 8版》的第17章以获取有关线程和锁的信息。第17章反映了JSR 133专家组编写的Java内存模型和线程规范。

---
## 1.4 Notation（标注）

Throughout this specification we refer to classes and interfaces drawn from the Java SE platform API. Whenever we refer to a class or interface (other than those declared in an example) using a single identifier N, the intended reference is to the class or interface named N in the package java.lang. We use the fully qualified name for classes or interfaces from packages other than java.lang.

在本规范中，我们提到的类和接口均来自Java SE平台API。每当我们使用单个标识符N引用一个类或接口（示例中声明的除外）时，其引用指向的是java.lang包中名为N的类或接口。对于来自java.lang之外的包的类或接口，我们使用全限定名称。

Whenever we refer to a class or interface that is declared in the package java or any of its subpackages, the intended reference is to that class or interface as loaded by the bootstrap class loader (§5.3.1).

每当我们引用在java包或其任何子包中声明的类或接口时，其引用指向的是由引导类加载器（§5.3.1）加载的该类或接口。

Whenever we refer to a subpackage of a package named java, the intended reference is to that subpackage as determined by the bootstrap class loader.

每当我们引用名为java的包的子包时，其引用指向的是由引导类加载器确定的该子包。

The use of fonts in this specification is as follows:

本规范中字体的使用如下：

- A fixed width font is used for Java Virtual Machine data types, exceptions, errors, class file structures, Prolog code, and Java code fragments.

- 固定宽度字体用于Java虚拟机数据类型、异常、错误、类文件结构、Prolog代码和Java代码片段。

- Italic is used for Java Virtual Machine "assembly language", its opcodes and operands, as well as items in the Java Virtual Machine's run-time data areas. It is also used to introduce new terms and simply for emphasis.

- 斜体用于Java虚拟机的“汇编语言”、其操作码和操作数，以及Java虚拟机的运行时数据区中的项目。它也用于引入新术语和简单的强调。

Non-normative information, designed to clarify the specification, is given in smaller, indented text.

非规范性信息，用于澄清规范，以较小的、缩进的文本形式给出。

This is non-normative information. It provides intuition, rationale, advice, examples, etc.

这是非规范性信息。它提供直觉、基本原理、建议、示例等。

---
## 1.5 Feedback（反馈）

Readers are invited to report technical errors and ambiguities in The Java® Virtual Machine Specification to [jls-jvms-spec-comments@openjdk.java.net](mailto:jls-jvms-spec-comments@openjdk.java.net).

邀请读者将《Java®虚拟机规范》中的技术错误和含糊之处报告至[jls-jvms-spec-comments@openjdk.java.net](mailto:jls-jvms-spec-comments@openjdk.java.net)。

Questions concerning the generation and manipulation of class files by javac (the reference compiler for the Java programming language) may be sent to [compiler-dev@openjdk.java.net](mailto:compiler-dev@openjdk.java.net).

有关javac（Java编程语言的参考编译器）生成和操作类文件的问题可以发送至[compiler-dev@openjdk.java.net](mailto:compiler-dev@openjdk.java.net)。
