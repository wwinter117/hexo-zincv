---
title: jvm-spec-5. Loading, Linking, and Initializing
tags:
  - 官方文档翻译
  - jvm
date: 2022-9-2 22:46:49
---

The Java Virtual Machine dynamically loads, links, and initializes classes and interfaces. Loading is the process of finding the binary representation of a class or interface type with a particular name and creating a class or interface from that binary representation. Linking is the process of taking a class or interface and combining it into the run-time state of the Java Virtual Machine so that it can be executed. Initialization of a class or interface consists of executing the class or interface initialization method `<clinit>` (§2.9).

Java虚拟机动态地加载、链接和初始化类和接口。加载是找到具有特定名称的类或接口类型的二进制表示并从该二进制表示中创建一个类或接口的过程。链接是将类或接口整合到Java虚拟机的运行时状态中，以便能够执行。类或接口的初始化包括执行类或接口的初始化方法`<clinit>` (§2.9)。

In this chapter, §5.1 describes how the Java Virtual Machine derives symbolic references from the binary representation of a class or interface. §5.2 explains how the processes of loading, linking, and initialization are first initiated by the Java Virtual Machine. §5.3 specifies how binary representations of classes and interfaces are loaded by class loaders and how classes and interfaces are created. Linking is described in §5.4. §5.5 details how classes and interfaces are initialized. §5.6 introduces the notion of binding native methods. Finally, §5.7 describes when a Java Virtual Machine exits.

在本章中，§5.1描述了Java虚拟机如何从类或接口的二进制表示中派生符号引用。§5.2解释了Java虚拟机如何首先启动加载、链接和初始化的过程。§5.3规定了类加载器如何加载类和接口的二进制表示，以及如何创建类和接口。链接在§5.4中描述。§5.5详细说明了类和接口的初始化。§5.6介绍了绑定本地方法的概念。最后，§5.7描述了Java虚拟机何时退出。

---
## 5.1 The Run-Time Constant Pool

The Java Virtual Machine maintains a per-type constant pool (§2.5.5), a run-time data structure that serves many of the purposes of the symbol table of a conventional programming language implementation.

Java虚拟机维护一个按类型划分的常量池 (§2.5.5)，它是一个运行时数据结构，起到了传统编程语言实现中符号表的许多功能。

The `constant_pool` table (§4.4) in the binary representation of a class or interface is used to construct the run-time constant pool upon class or interface creation (§5.3). All references in the run-time constant pool are initially symbolic.

在类或接口的创建过程中 (§5.3)，二进制表示中的`constant_pool`表 (§4.4) 被用来构建运行时常量池。运行时常量池中的所有引用最初都是符号引用。

The symbolic references in the run-time constant pool are derived from structures in the binary representation of the class or interface as follows:

运行时常量池中的符号引用是从类或接口的二进制表示中的结构派生的，具体如下：

- A symbolic reference to a class or interface is derived from a `CONSTANT_Class_info` structure (§4.4.1) in the binary representation of a class or interface. Such a reference gives the name of the class or interface in the form returned by the `Class.getName` method, that is:
  - For a non-array class or an interface, the name is the binary name (§4.2.1) of the class or interface.
  - For an array class of n dimensions, the name begins with n occurrences of the ASCII "\[" character followed by a representation of the element type:
    - If the element type is a primitive type, it is represented by the corresponding field descriptor (§4.3.2).
    - Otherwise, if the element type is a reference type, it is represented by the ASCII "L" character followed by the binary name (§4.2.1) of the element type followed by the ASCII ";" character.

- 类或接口的符号引用是从类或接口二进制表示中的`CONSTANT_Class_info`结构 (§4.4.1) 派生的。此类引用提供了通过`Class.getName`方法返回的类或接口的名称，即：
  - 对于非数组类或接口，其名称是类或接口的二进制名称 (§4.2.1)。
  - 对于n维数组类，其名称以n个ASCII\\“\[”字符开头，后跟元素类型的表示：
    - 如果元素类型是基本类型，则使用相应的字段描述符 (§4.3.2) 表示。
    - 否则，如果元素类型是引用类型，则使用ASCII“L”字符，后跟元素类型的二进制名称 (§4.2.1)，再后跟ASCII“;”字符。

Whenever this chapter refers to the name of a class or interface, it should be understood to be in the form returned by the `Class.getName` method.

本章中提到类或接口的名称时，应理解为以`Class.getName`方法返回的形式。

- A symbolic reference to a field of a class or an interface is derived from a `CONSTANT_Fieldref_info` structure (§4.4.2) in the binary representation of a class or interface. Such a reference gives the name and descriptor of the field, as well as a symbolic reference to the class or interface in which the field is to be found.

- 类或接口字段的符号引用是从类或接口二进制表示中的`CONSTANT_Fieldref_info`结构 (§4.4.2) 派生的。此类引用提供字段的名称和描述符，以及包含该字段的类或接口的符号引用。

- A symbolic reference to a method of a class is derived from a `CONSTANT_Methodref_info` structure (§4.4.2) in the binary representation of a class or interface. Such a reference gives the name and descriptor of the method, as well as a symbolic reference to the class in which the method is to be found.

- 类方法的符号引用是从类或接口二进制表示中的`CONSTANT_Methodref_info`结构 (§4.4.2) 派生的。此类引用提供方法的名称和描述符，以及包含该方法的类的符号引用。

- A symbolic reference to a method of an interface is derived from a `CONSTANT_InterfaceMethodref_info` structure (§4.4.2) in the binary representation of a class or interface. Such a reference gives the name and descriptor of the interface method, as well as a symbolic reference to the interface in which the method is to be found.

- 接口方法的符号引用是从类或接口二进制表示中的`CONSTANT_InterfaceMethodref_info`结构 (§4.4.2) 派生的。此类引用提供接口方法的名称和描述符，以及包含该方法的接口的符号引用。

- A symbolic reference to a method handle is derived from a `CONSTANT_MethodHandle_info` structure (§4.4.8) in the binary representation of a class or interface. Such a reference gives a symbolic reference to a field of a class or interface, or a method of a class, or a method of an interface, depending on the kind of the method handle.

- 方法句柄的符号引用是从类或接口二进制表示中的`CONSTANT_MethodHandle_info`结构 (§4.4.8) 派生的。此类引用提供对类或接口的字段、类的方法或接口的方法的符号引用，具体取决于方法句柄的类型。

- A symbolic reference to a method type is derived from a `CONSTANT_MethodType_info` structure (§4.4.9) in the binary representation of a class or interface. Such a reference gives a method descriptor (§4.3.3).

- 方法类型的符号引用是从类或接口二进制表示中的`CONSTANT_MethodType_info`结构 (§4.4.9) 派生的。此类引用提供方法描述符 (§4.3.3)。

- A symbolic reference to a call site specifier is derived from a `CONSTANT_InvokeDynamic_info` structure (§4.4.10) in the binary representation of a class or interface. Such a reference gives:
  - a symbolic reference to a method handle, which will serve as a bootstrap method for an `invokedynamic` instruction (§invokedynamic);
  - a sequence of symbolic references (to classes, method types, and method handles), string literals, and run-time constant values which will serve as static arguments to a bootstrap method;
  - a method name and method descriptor.

- 调用站点说明符的符号引用是从类或接口二进制表示中的`CONSTANT_InvokeDynamic_info`结构 (§4.4.10) 派生的。此类引用提供：
  - 一个方法句柄的符号引用，该句柄将用作`invokedynamic`指令 (§invokedynamic) 的引导方法；
  - 一系列符号引用（类、方法类型和方法句柄的符号引用）、字符串字面量和运行时常量值，这些将作为引导方法的静态参数；
  - 方法名称和方法描述符。

In addition, certain run-time values which are not symbolic references are derived from items found in the `constant_pool` table:

此外，某些不是符号引用的运行时值是从`constant_pool`表中的项派生的：

- A string literal is a reference to an instance of class `String`, and is derived from a `CONSTANT_String_info` structure (§4.4.3) in the binary representation of a class or interface. The `CONSTANT_String_info` structure gives the sequence of Unicode code points constituting the string literal.

- 字符串字面量是对`String`类实例的引用，并且是从类或接口的二进制表示中的`CONSTANT_String_info`结构 (§4.4.3) 派生的。`CONSTANT_String_info`结构给出了组成字符串字面量的Unicode代码点序列。

The Java programming language requires that identical string literals (that is, literals that contain the same sequence of code points) must refer to the same instance of class `String` (JLS §3.10.5). In addition, if the method `String.intern` is called on any string, the result is a reference to the same class instance that would be returned if that string appeared as a literal. Thus, the following expression must have the value true:
`("a" + "b" + "c").intern() == "abc"`

Java编程语言要求相同的字符串字面量（即包含相同代码点序列的字面量）必须引用`String`类的同一实例 (JLS §3.10.5)。此外，如果对任何字符串调用`String.intern`方法，结果将是对同一类实例的引用，如果该字符串作为字面量出现则会返回该实例。因此，以下表达式的值必须为true：
`("a" + "b" + "c").intern() == "abc"`

To derive a string literal, the Java Virtual Machine examines the sequence of code points given by the `CONSTANT_String_info` structure.

要派生字符串字面量，Java虚拟机会检查`CONSTANT_String_info`结构给出的代码点序列。

- If the method `String.intern` has previously been called on an instance of class `String` containing a sequence of Unicode code points identical to that given by the `CONSTANT_String_info` structure, then the result of string literal derivation is a reference to that same instance of class `String`.

- 如果`String`类实例上之前已调用过`String.intern`方法，并且该实例包含的Unicode代码点序列与`CONSTANT_String_info`结构给出的一致，则字符串字面量派生的结果是对该`String`类同一实例的引用。

- Otherwise, a new instance of class `String` is created containing the sequence of Unicode code points given by the `CONSTANT_String_info` structure; a reference to that class instance is the result of string literal derivation. Finally, the `intern` method of the new `String` instance is invoked.

- 否则，将创建一个包含`CONSTANT_String_info`结构给出的Unicode代码点序列的`String`类新实例；对该类实例的引用是字符串字面量派生的结果。最后，调用新`String`实例的`intern`方法。

---
## 5.2 Java Virtual Machine Startup

The Java Virtual Machine starts up by creating an initial class, which is specified in an implementation-dependent manner, using the bootstrap class loader (§5.3.1). The Java Virtual Machine then links the initial class, initializes it, and invokes the public class method `void main(String[])`. The invocation of this method drives all further execution. Execution of the Java Virtual Machine instructions constituting the main method may cause linking (and consequently creation) of additional classes and interfaces, as well as invocation of additional methods.

Java虚拟机启动时，使用引导类加载器 (§5.3.1) 创建初始类，该初始类以实现依赖的方式指定。Java虚拟机随后链接初始类，初始化它，并调用公共类方法`void main(String[])`。此方法的调用驱动了所有进一步的执行。构成main方法的Java虚拟机指令的执行可能会导致其他类和接口的链接（从而创建），以及其他方法的调用。

In an implementation of the Java Virtual Machine, the initial class could be provided as a command line argument. Alternatively, the implementation could provide an initial class that sets up a class loader which in turn loads an application. Other choices of the initial class are possible so long as they are consistent with the specification given in the previous paragraph.

在Java虚拟机的实现中，初始类可以作为命令行参数提供。或者，实现可以提供一个初始类，该类设置一个类加载器，该加载器反过来加载应用程序。只要与前一段中给出的规范一致，初始类的其他选择也是可能的。

---
## 5.3 Creation and Loading

Creation of a class or interface C denoted by the name N consists of the construction in the method area of the Java Virtual Machine (§2.5.4) of an implementation-specific internal representation of C. Class or interface creation is triggered by another class or interface D, which references C through its run-time constant pool.

表示为名称N的类或接口C的创建包括在Java虚拟机 (§2.5.4) 的方法区中构建C的实现特定的内部表示。类或接口的创建是由另一个类或接口D触发的，D通过其运行时常量池引用C。

Class or interface creation may also be triggered by D invoking methods in certain Java SE platform class libraries (§2.12) such as reflection.

类或接口的创建还可能由D在某些Java SE平台类库中调用方法（例如反射）触发。

If C is not an array class, it is created by loading a binary representation of C (§4 (The class File Format)) using a class loader. Array classes do not have an external binary representation; they are created by the Java Virtual Machine rather than by a class loader.

如果C不是数组类，则通过使用类加载器加载C的二进制表示 (§4 (类文件格式)) 创建它。数组类没有外部二进制表示；它们是由Java虚拟机创建的，而不是由类加载器创建的。

There are two kinds of class loaders: the bootstrap class loader supplied by the Java Virtual Machine, and user-defined class loaders. Every user-defined class loader is an instance of a subclass of the abstract class `ClassLoader`. Applications employ user-defined class loaders in order to extend the manner in which the Java Virtual Machine dynamically loads and thereby creates classes. User-defined class loaders can be used to create classes that originate from user-defined sources. For example, a class could be downloaded across a network, generated on the fly, or extracted from an encrypted file.

有两种类加载器：Java虚拟机提供的引导类加载器和用户定义的类加载器。每个用户定义的类加载器都是抽象类`ClassLoader`子类的实例。应用程序使用用户定义的类加载器来扩展Java虚拟机动态加载并从而创建类的方式。用户定义的类加载器可用于创建源自用户定义源的类。例如，一个类可以通过网络下载，动态生成，或从加密文件中提取。

A class loader L may create C by defining it directly or by delegating to another class loader. If L creates C directly, we say that L defines C or, equivalently, that L is the defining loader of C.

类加载器L可以通过直接定义C或委托给另一个类加载器来创建C。如果L直接创建C，我们称L定义了C，或等同于说L是C的定义加载器。

When one class loader delegates to another class loader, the loader that initiates the loading is not necessarily the same loader that completes the loading and defines the class. If L creates C, either by defining it directly or by delegation, we say that L initiates loading of C or, equivalently, that L is an initiating loader of C.

当一个类加载器委托给另一个类加载器时，发起加载的加载器不一定是完成加载并定义类的加载器。如果L创建C，无论是直接定义还是通过委托，我们称L发起了C的加载，或等同于说L是C的发起加载器。

At run time, a class or interface is determined not by its name alone, but by a pair: its binary name (§4.2.1) and its defining class loader. Each such class or interface belongs to a single run-time package. The run-time package of a class or interface is determined by the package name and defining class loader of the class or interface.

在运行时，类或接口不仅由其名称确定，还由一对：其二进制名称 (§4.2.1) 和定义类加载器。每个此类类或接口都属于单一的运行时包。类或接口的运行时包由类或接口的包名称和定义类加载器确定。

The Java Virtual Machine uses one of three procedures to create class or interface C denoted by N:

Java虚拟机使用以下三种程序之一来创建表示为N的类或接口C：

- If N denotes a non-array class or an interface, one of the two following methods is used to load and thereby create C:
  - If D was defined by the bootstrap class loader, then the bootstrap class loader initiates loading of C (§5.3.1).
  - If D was defined by a user-defined class loader, then that same user-defined class loader initiates loading of C (§5.3.2).

- 如果N表示非数组类或接口，则使用以下两种方法之一来加载并从而创建C：
  - 如果D是由引导类加载器定义的，则由引导类加载器发起C的加载 (§5.3.1)。
  - 如果D是由用户定义的类加载器定义的，则由同一个用户定义的类加载器发起C的加载 (§5.3.2)。

- Otherwise N denotes an array class. An array class is created directly by the Java Virtual Machine (§5.3.3), not by a class loader. However, the defining class loader of D is used in the process of creating array class C.

- 否则N表示数组类。数组类直接由Java虚拟机创建 (§5.3.3)，而不是由类加载器创建。但是，在创建数组类C的过程中使用了D的定义类加载器。

If an error occurs during class loading, then an instance of a subclass of `LinkageError` must be thrown at a point in the program that (directly or indirectly) uses the class or interface being loaded.

如果在类加载过程中发生错误，则必须在程序中使用（直接或间接）正在加载的类或接口的地方抛出`LinkageError`子类的实例。

If the Java Virtual Machine ever attempts to load a class C during verification (§5.4.1) or resolution (§5.4.3) (but not initialization (§5.5)), and the class loader that is used to initiate loading of C throws an instance of `ClassNotFoundException`, then the Java Virtual Machine must throw an instance of `NoClassDefFoundError` whose cause is the instance of `ClassNotFoundException`.

如果Java虚拟机在验证 (§5.4.1) 或解析 (§5.4.3) （但不是初始化 (§5.5)）过程中尝试加载类C，并且用于发起C加载的类加载器抛出`ClassNotFoundException`的实例，则Java虚拟机必须抛出`NoClassDefFoundError`实例，其原因是`ClassNotFoundException`实例。

(A subtlety here is that recursive class loading to load superclasses is performed as part of resolution (§5.3.5, step 3). Therefore, a `ClassNotFoundException` that results from a class loader failing to load a superclass must be wrapped in a `NoClassDefFoundError`.)

（这里的一个微妙之处在于，加载超类的递归类加载作为解析 (§5.3.5, 第3步) 的一部分进行。因此，由类加载器无法加载超类而导致的`ClassNotFoundException`必须包装在`NoClassDefFoundError`中。）

A well-behaved class loader should maintain three properties:

一个表现良好的类加载器应该保持三个属性：

- Given the same name, a good class loader should always return the same `Class` object.

- 对于相同的名称，一个好的类加载器应始终返回相同的`Class`对象。

- If a class loader L delegates loading of a class C to another loader L', then for any type T that occurs as the direct superclass or a direct superinterface of C, or as the type of a field in C, or as the type of a formal parameter of a method or constructor in C, or as a return type of a method in C, L and L' should return the same `Class` object.

- 如果类加载器L将类C的加载委托给另一个加载器L'，那么对于作为C的直接超类或直接超接口出现的任何类型T，或作为C中的字段类型，或作为C中方法或构造函数的形式参数类型，或作为C中方法的返回类型，L和L'应返回相同的`Class`对象。

- If a user-defined classloader prefetches binary representations of classes and interfaces, or loads a group of related classes together, then it must reflect loading errors only at points in the program where they could have arisen without prefetching or group loading.

- 如果用户定义的类加载器预先获取类和接口的二进制表示，或一起加载一组相关类，则它必须仅在程序中可能发生错误的点上反映加载错误，而不会因预先获取或组加载而产生错误。

We will sometimes represent a class or interface using the notation `<N, L>`, where N denotes the name of the class or interface and L denotes the defining loader of the class or interface.

我们有时会使用符号 `<N, L>` 来表示一个类或接口，其中N表示类或接口的名称，L表示类或接口的定义加载器。

We will also represent a class or interface using the notation `N^Li`, where N denotes the name of the class or interface and Li denotes an initiating loader of the class or interface.

我们还将使用符号 `N^Li` 来表示一个类或接口，其中N表示类或接口的名称，Li表示类或接口的发起加载器。

---
### 5.3.1 Loading Using the Bootstrap Class Loader

The following steps are used to load and thereby create the non-array class or interface C denoted by N using the bootstrap class loader.

使用引导类加载器加载并从而创建非数组类或接口C的步骤如下：

1. First, the Java Virtual Machine determines whether the bootstrap class loader has already been recorded as an initiating loader of a class or interface denoted by N. If so, this class or interface is C, and no class creation is necessary.

1. 首先，Java虚拟机确定引导类加载器是否已被记录为由N表示的类或接口的发起加载器。如果是这样，这个类或接口就是C，不需要创建类。

2. Otherwise, the Java Virtual Machine passes the argument N to an invocation of a method on the bootstrap class loader to search for a purported representation of C in a platform-dependent manner. Typically, a class or interface will be represented using a file in a hierarchical file system, and the name of the class or interface will be encoded in the pathname of the file.

2. 否则，Java虚拟机会将参数N传递给引导类加载器上的方法调用，以平台依赖的方式搜索C的假定表示。通常，类或接口将使用分层文件系统中的文件表示，并且类或接口的名称将编码在文件的路径名中。

3. Note that there is no guarantee that a purported representation found is valid or is a representation of C. This phase of loading must detect the following error:
   - If no purported representation of C is found, loading throws an instance of `ClassNotFoundException`.

3. 请注意，找到的假定表示是否有效或是否为C的表示没有保证。加载的这个阶段必须检测以下错误：
   - 如果未找到C的假定表示，加载会抛出`ClassNotFoundException`实例。

4. Then the Java Virtual Machine attempts to derive a class denoted by N using the bootstrap class loader from the purported representation using the algorithm found in §5.3.5. That class is C.

4. 然后，Java虚拟机会尝试使用引导类加载器通过§5.3.5中找到的算法从假定表示中派生一个由N表示的类。那个类就是C。

---
### 5.3.2 Loading Using a User-defined Class Loader

The following steps are used to load and thereby create the non-array class or interface C denoted by N using a user-defined class loader L.

使用用户定义的类加载器L加载并从而创建由N表示的非数组类或接口C的步骤如下：

1. First, the Java Virtual Machine determines whether L has already been recorded as an initiating loader of a class or interface denoted by N. If so, this class or interface is C, and no class creation is necessary.

1. 首先，Java虚拟机确定L是否已被记录为由N表示的类或接口的发起加载器。如果是这样，这个类或接口就是C，不需要创建类。

2. Otherwise, the Java Virtual Machine invokes `loadClass(N)` on L. The value returned by the invocation is the created class or interface C. The Java Virtual Machine then records that L is an initiating loader of C (§5.3.4). The remainder of this section describes this process in more detail.

2. 否则，Java虚拟机在L上调用`loadClass(N)`。调用返回的值是创建的类或接口C。然后，Java虚拟机记录L是C的发起加载器 (§5.3.4)。本节的其余部分更详细地描述了这个过程。

3. When the `loadClass` method of the class loader L is invoked with the name N of a class or interface C to be loaded, L must perform one of the following two operations in order to load C:
   1. The class loader L can create an array of bytes representing C as the bytes of a `ClassFile` structure (§4.1); it then must invoke the method `defineClass` of class `ClassLoader`. Invoking `defineClass` causes the Java Virtual Machine to derive a class or interface denoted by N using L from the array of bytes using the algorithm found in §5.3.5.
   2. The class loader L can delegate the loading of C to some other class loader L'. This is accomplished by passing the argument N directly or indirectly to an invocation of a method on L' (typically the `loadClass` method). The result of the invocation is C.

3. 当类加载器L的`loadClass`方法被调用并指定要加载的类或接口C的名称N时，L必须执行以下两项操作之一以加载C：
   1. 类加载器L可以创建表示C的字节数组作为`ClassFile`结构 (§4.1) 的字节；然后必须调用`ClassLoader`类的`defineClass`方法。调用`defineClass`会导致Java虚拟机使用L从字节数组中使用§5.3.5中找到的算法派生一个由N表示的类或接口。
   2. 类加载器L可以将C的加载委托给其他类加载器L'。这是通过将参数N直接或间接传递给L'上的方法调用（通常是`loadClass`方法）来完成的。调用的结果是C。

4. In either (1) or (2), if the class loader L is unable to load a class or interface denoted by N for any reason, it must throw an instance of `ClassNotFoundException`.

4. 在(1)或(2)中，如果类加载器L因任何原因无法加载由N表示的类或接口，则必须抛出`ClassNotFoundException`实例。

Since JDK release 1.1, Oracle’s Java Virtual Machine implementation has invoked the `loadClass` method of a class loader in order to cause it to load a class or interface. The argument to `loadClass` is the name of the class or interface to be loaded. There is also a two-argument version of the `loadClass` method, where the second argument is a boolean that indicates whether the class or interface is to be linked or not. Only the two-argument version was supplied in JDK release 1.0.2, and Oracle’s Java Virtual Machine implementation relied on it to link the loaded class or interface. From JDK release 1.1 onward, Oracle’s Java Virtual Machine implementation links the class or interface directly, without relying on the class loader.

自JDK 1.1版本以来，Oracle的Java虚拟机实现已经调用类加载器的`loadClass`方法来促使它加载类或接口。`loadClass`的参数是要加载的类或接口的名称。还有一个带两个参数的`loadClass`方法版本，其中第二个参数是一个布尔值，指示类或接口是否要链接。JDK 1.0.2版本中只提供了带两个参数的版本，Oracle的Java虚拟机实现依赖它来链接加载的类或接口。从JDK 1.1版本开始，Oracle的Java虚拟机实现直接链接类或接口，而不依赖类加载器。

### 5.3.3 Creating Array Classes

The following steps are used to create the array class C denoted by N using class loader L. Class loader L may be either the bootstrap class loader or a user-defined class loader.

使用类加载器L创建由N表示的数组类C的步骤如下。类加载器L可以是引导类加载器或用户定义的类加载器。

1. If L has already been recorded as an initiating loader of an array class with the same component type as N, that class is C, and no array class creation is necessary.

1. 如果L已被记录为与N具有相同组件类型的数组类的发起加载器，则该类就是C，不需要创建数组类。

2. Otherwise, the following steps are performed to create C:
   1. If the component type is a reference type, the algorithm of this section (§5.3) is applied recursively using class loader L in order to load and thereby create the component type of C.
   2. The Java Virtual Machine creates a new array class with the indicated component type and number of dimensions.

2. 否则，执行以下步骤以创建C：
   1. 如果组件类型是引用类型，则使用类加载器L递归应用本节 (§5.3) 的算法以加载并从而创建C的组件类型。
   2. Java虚拟机创建具有指定组件类型和维数的新数组类。

3. If the component type is a reference type, C is marked as having been defined by the defining class loader of the component type. Otherwise, C is marked as having been defined by the bootstrap class loader.

3. 如果组件类型是引用类型，则C标记为由组件类型的定义类加载器定义。否则，C标记为由引导类加载器定义。

4. In any case, the Java Virtual Machine then records that L is an initiating loader for C (§5.3.4).

4. 无论如何，Java虚拟机随后记录L是C的发起加载器 (§5.3.4)。

5. If the component type is a reference type, the accessibility of the array class is determined by the accessibility of its component type. Otherwise, the accessibility of the array class is public.

5. 如果组件类型是引用类型，数组类的可访问性由其组件类型的可访问性决定。否则，数组类的可访问性为公共。

---
### 5.3.4 Loading Constraints

Ensuring type-safe linkage in the presence of class loaders requires special care. It is possible that when two different class loaders initiate loading of a class or interface denoted by N, the name N may denote a different class or interface in each loader.

在类加载器存在的情况下确保类型安全的链接需要特别小心。当两个不同的类加载器发起由N表示的类或接口的加载时，N可能在每个加载器中表示不同的类或接口。

When a class or interface C = `<N1, L1>` makes a symbolic reference to a field or method of another class or interface D = `<N2, L2>`, the symbolic reference includes a descriptor specifying the type of the field, or the return and argument types of the method. It is essential that any type name N mentioned in the field or method descriptor denote the same class or interface when loaded by L1 and when loaded by L2.

当类或接口C = `<N1, L1>` 对另一个类或接口D = `<N2, L2>` 的字段或方法进行符号引用时，符号引用包括指定字段类型或方法的返回和参数类型的描述符。至关重要的是，字段或方法描述符中提到的任何类型名N在L1加载和L2加载时表示相同的类或接口。

To ensure this, the Java Virtual Machine imposes loading constraints of the form `L1L2 N1 = N2` during preparation (§5.4.2) and resolution (§5.4.3). To enforce these constraints, the Java Virtual Machine will, at certain prescribed times (see §5.3.1, §5.3.2, §5.3.3, and §5.3.5), record that a particular loader is an initiating loader of a particular class. After recording that a loader is an initiating loader of a class, the Java Virtual Machine must immediately check to see if any loading constraints are violated. If so, the record is retracted, the Java Virtual Machine throws a `LinkageError`, and the loading operation that caused the recording to take place fails.

为确保这一点，Java虚拟机在准备 (§5.4.2) 和解析 (§5.4.3) 期间施加形式为 `L1L2 N1 = N2` 的加载约束。为了强制执行这些约束，Java虚拟机将在某些规定的时间点（见§5.3.1，§5.3.2，§5.3.3，和§5.3.5）记录特定加载器是特定类的发起加载器。记录加载器是类的发起加载器后，Java虚拟机必须立即检查是否违反了任何加载约束。如果是，记录将被撤回，Java虚拟机抛出`LinkageError`，并且导致记录发生的加载操作失败。

Similarly, after imposing a loading constraint (see §5.4.2, §5.4.3.2, §5.4.3.3, and §5.4.3.4), the Java Virtual Machine must immediately check to see if any loading constraints are violated. If so, the newly imposed loading constraint is retracted, the Java Virtual Machine throws a `LinkageError`, and the operation that caused the constraint to be imposed (either resolution or preparation, as the case may be) fails.

同样，在施加加载约束后（见§5.4.2，§5.4.3.2，§5.4.3.3和§5.4.3.4），Java虚拟机必须立即检查是否违反了任何加载约束。如果是，刚施加的加载约束将被撤回，Java虚拟机抛出`LinkageError`，并且导致施加该约束的操作（无论是解析还是准备）失败。

The situations described here are the only times at which the Java Virtual Machine checks whether any loading constraints have been violated. A loading constraint is violated if, and only if, all the following four conditions hold:

这里描述的情况是Java虚拟机检查是否违反任何加载约束的唯一时间。加载约束在且仅在以下四个条件全部成立时违反：

1. There exists a loader L such that L has been recorded by the Java Virtual Machine as an initiating loader of a class C named N.

1. 存在一个加载器L，该加载器已被Java虚拟机记录为名为N的类C的发起加载器。

2. There exists a loader L' such that L' has been recorded by the Java Virtual Machine as an initiating loader of a class C' named N.

2. 存在一个加载器L'，该加载器已被Java虚拟机记录为名为N的类C'的发起加载器。

3. The equivalence relation defined by the (transitive closure of the) set of imposed L1L2 constraints implies `N1 = N2`.

3. 由施加的L1L2约束集合（的传递闭包）定义的等价关系暗示 `N1 = N2`。

4. C ≠ C'.

4. C ≠ C'。

A full discussion of class loaders and type safety is beyond the scope of this specification.

关于类加载器和类型安全的全面讨论超出了本规范的范围。


For a more comprehensive discussion, readers are referred to *Dynamic Class Loading in the Java Virtual Machine* by Sheng Liang and Gilad Bracha (Proceedings of the 1998 ACM SIGPLAN Conference on Object-Oriented Programming Systems, Languages and Applications).

有关更全面的讨论，读者可以参考Sheng Liang和Gilad Bracha撰写的《Java虚拟机中的动态类加载》一文（发表于1998年ACM SIGPLAN面向对象编程系统、语言和应用程序会议论文集）。

---
### 5.3.5 Deriving a Class from a class File Representation

The following steps are used to derive a `Class` object for the non-array class or interface C denoted by N using loader L from a purported representation in class file format.

以下步骤用于从类文件格式的假定表示中使用加载器L派生表示为N的非数组类或接口C的`Class`对象。

1. First, the Java Virtual Machine determines whether it has already recorded that L is an initiating loader of a class or interface denoted by N. If so, this creation attempt is invalid and loading throws a `LinkageError`.

1. 首先，Java虚拟机确定是否已经记录L是由N表示的类或接口的发起加载器。如果是这样，这次创建尝试无效，加载会抛出`LinkageError`。

2. Otherwise, the Java Virtual Machine attempts to parse the purported representation. However, the purported representation may not in fact be a valid representation of C.

2. 否则，Java虚拟机会尝试解析假定表示。然而，假定表示可能实际上并不是C的有效表示。

This phase of loading must detect the following errors:

加载的这个阶段必须检测以下错误：

- If the purported representation is not a `ClassFile` structure (§4.1, §4.8), loading throws an instance of `ClassFormatError`.

- 如果假定表示不是`ClassFile`结构 (§4.1, §4.8)，加载会抛出`ClassFormatError`实例。

- Otherwise, if the purported representation is not of a supported major or minor version (§4.1), loading throws an instance of `UnsupportedClassVersionError`.

- 否则，如果假定表示不是受支持的主版本或次版本 (§4.1)，加载会抛出`UnsupportedClassVersionError`实例。

- Otherwise, if the purported representation does not actually represent a class named N, loading throws an instance of `NoClassDefFoundError` or an instance of one of its subclasses.

- 否则，如果假定表示实际上并不表示名为N的类，加载会抛出`NoClassDefFoundError`实例或其子类的实例。

3. If C has a direct superclass, the symbolic reference from C to its direct superclass is resolved using the algorithm of §5.4.3.1. Note that if C is an interface it must have `Object` as its direct superclass, which must already have been loaded. Only `Object` has no direct superclass.

3. 如果C有直接超类，则使用§5.4.3.1的算法解析C到其直接超类的符号引用。请注意，如果C是接口，则必须将`Object`作为其直接超类，并且必须已经加载。只有`Object`没有直接超类。

Any exceptions that can be thrown due to class or interface resolution can be thrown as a result of this phase of loading. In addition, this phase of loading must detect the following errors:

由于类或接口解析而可能抛出的任何异常都可能作为加载的这个阶段的结果抛出。此外，加载的这个阶段必须检测以下错误：

- If the class or interface named as the direct superclass of C is in fact an interface, loading throws an `IncompatibleClassChangeError`.

- 如果C的直接超类命名的类或接口实际上是接口，加载会抛出`IncompatibleClassChangeError`。

- Otherwise, if any of the superclasses of C is C itself, loading throws a `ClassCircularityError`.

- 否则，如果C的任何超类是C本身，加载会抛出`ClassCircularityError`。

4. If C has any direct superinterfaces, the symbolic references from C to its direct superinterfaces are resolved using the algorithm of §5.4.3.1.

4. 如果C有任何直接超接口，则使用§5.4.3.1的算法解析C到其直接超接口的符号引用。

Any exceptions that can be thrown due to class or interface resolution can be thrown as a result of this phase of loading. In addition, this phase of loading must detect the following errors:

由于类或接口解析而可能抛出的任何异常都可能作为加载的这个阶段的结果抛出。此外，加载的这个阶段必须检测以下错误：

- If any of the classes or interfaces named as direct superinterfaces of C is not in fact an interface, loading throws an `IncompatibleClassChangeError`.

- 如果C的直接超接口命名的类或接口实际上不是接口，加载会抛出`IncompatibleClassChangeError`。

- Otherwise, if any of the superinterfaces of C is C itself, loading throws a `ClassCircularityError`.

- 否则，如果C的任何超接口是C本身，加载会抛出`ClassCircularityError`。

5. The Java Virtual Machine marks C as having L as its defining class loader and records that L is an initiating loader of C (§5.3.4).

5. Java虚拟机将C标记为由L作为其定义类加载器，并记录L是C的发起加载器 (§5.3.4)。

---
## 5.4 Linking

Linking a class or interface involves verifying and preparing that class or interface, its direct superclass, its direct superinterfaces, and its element type (if it is an array type), if necessary. Resolution of symbolic references in the class or interface is an optional part of linking.

链接类或接口涉及验证和准备该类或接口、其直接超类、其直接超接口和其元素类型（如果是数组类型），如果有必要的话。解析类或接口中的符号引用是链接的可选部分。

This specification allows an implementation flexibility as to when linking activities (and, because of recursion, loading) take place, provided that all of the following properties are maintained:

本规范允许实现者在何时进行链接活动（并且由于递归，还包括加载）方面具有灵活性，前提是必须保持以下所有属性：

1. A class or interface is completely loaded before it is linked.

1. 类或接口在链接之前必须完全加载。

2. A class or interface is completely verified and prepared before it is initialized.

2. 类或接口在初始化之前必须完全验证并准备。

3. Errors detected during linkage are thrown at a point in the program where some action is taken by the program that might, directly or indirectly, require linkage to the class or interface involved in the error.

3. 在链接过程中检测到的错误会在程序中某个采取行动的点上抛出，该行动可能直接或间接地需要链接到涉及错误的类或接口。

For example, a Java Virtual Machine implementation may choose to resolve each symbolic reference in a class or interface individually when it is used ("lazy" or "late" resolution), or to resolve them all at once when the class is being verified ("eager" or "static" resolution). This means that the resolution process may continue, in some implementations, after a class or interface has been initialized. Whichever strategy is followed, any error detected during resolution must be thrown at a point in the program that (directly or indirectly) uses a symbolic reference to the class or interface.

例如，Java虚拟机实现可以选择在使用类或接口中的每个符号引用时单独解析它们（“惰性”或“延迟”解析），或者在验证类时一次性解析它们（“急切”或“静态”解析）。这意味着在某些实现中，解析过程可能在类或接口初始化之后继续进行。无论采用哪种策略，解析过程中检测到的任何错误必须在程序中使用符号引用类或接口的点上抛出。

Because linking involves the allocation of new data structures, it may fail with an `OutOfMemoryError`.

由于链接涉及分配新的数据结构，因此可能因`OutOfMemoryError`失败。
### 5.4.1 Verification

Verification (§4.10) ensures that the binary representation of a class or interface is structurally correct (§4.9). Verification may cause additional classes and interfaces to be loaded (§5.3) but need not cause them to be verified or prepared.

验证 (§4.10) 确保类或接口的二进制表示在结构上是正确的 (§4.9)。验证可能会导致加载其他类和接口 (§5.3)，但不必对它们进行验证或准备。

If the binary representation of a class or interface does not satisfy the static or structural constraints listed in §4.9, then a `VerifyError` must be thrown at the point in the program that caused the class or interface to be verified.

如果类或接口的二进制表示不满足§4.9中列出的静态或结构约束，则必须在导致类或接口被验证的程序点上抛出`VerifyError`。

If an attempt by the Java Virtual Machine to verify a class or interface fails because an error is thrown that is an instance of `LinkageError` (or a subclass), then subsequent attempts to verify the class or interface always fail with the same error that was thrown as a result of the initial verification attempt.

如果Java虚拟机尝试验证类或接口失败，因为抛出了`LinkageError`（或其子类）的实例，则随后对该类或接口的验证尝试将始终因最初验证尝试抛出的相同错误而失败。

### 5.4.2 Preparation

Preparation involves creating the static fields for a class or interface and initializing such fields to their default values (§2.3, §2.4). This does not require the execution of any Java Virtual Machine code; explicit initializers for static fields are executed as part of initialization (§5.5), not preparation.

准备包括为类或接口创建静态字段并将这些字段初始化为其默认值 (§2.3, §2.4)。这不需要执行任何Java虚拟机代码；静态字段的显式初始化器作为初始化 (§5.5) 的一部分执行，而不是准备的一部分。

During preparation of a class or interface C, the Java Virtual Machine also imposes loading constraints (§5.3.4). Let L1 be the defining loader of C. For each method m declared in C that overrides (§5.4.5) a method declared in a superclass or superinterface `<D, L2>`, the Java Virtual Machine imposes the following loading constraints:

在准备类或接口C期间，Java虚拟机还施加加载约束 (§5.3.4)。设L1为C的定义加载器。对于C中声明的每个覆盖 (§5.4.5) 超类或超接口 `<D, L2>` 声明的方法m，Java虚拟机施加以下加载约束：

1. Given that the return type of m is Tr, and that the formal parameter types of m are Tf1, ..., Tfn, then:

1. 给定m的返回类型是Tr，m的形式参数类型是Tf1，...，Tfn，那么：

- If Tr is not an array type, let Tr0 be Tr; otherwise, let Tr0 be the element type (§2.4) of Tr.

- 如果Tr不是数组类型，则设Tr0为Tr；否则，设Tr0为Tr的元素类型 (§2.4)。

- For i = 1 to n: If Tfi is not an array type, let Tfi0 be Tfi; otherwise, let Tfi0 be the element type (§2.4) of Tfi.

- 对于i = 1到n：如果Tfi不是数组类型，则设Tfi0为Tfi；否则，设Tfi0为Tfi的元素类型 (§2.4)。

- L1L2 Then Tr0 = Tr and Tfi0 = Tfi for i = 1 to n.

- L1L2 然后Tr0 = Tr且Tfi0 = Tfi，适用于i = 1到n。

Furthermore, if C implements a method m declared in a superinterface `<I, L3>` of C, but C does not itself declare the method m, then let `<D, L2>` be the superclass of C that declares the implementation of method m inherited by C. The Java Virtual Machine imposes the following constraints:

此外，如果C实现了超接口 `<I, L3>` 声明的方法m，但C本身并未声明该方法m，则设 `<D, L2>` 为C的超类，声明了C继承的方法m的实现。Java虚拟机施加以下约束：

1. Given that the return type of m is Tr, and that the formal parameter types of m are Tf1, ..., Tfn, then:

1. 给定m的返回类型是Tr，m的形式参数类型是Tf1，...，Tfn，那么：

- If Tr is not an array type, let Tr0 be Tr; otherwise, let Tr0 be the element type (§2.4) of Tr.

- 如果Tr不是数组类型，则设Tr0为Tr；否则，设Tr0为Tr的元素类型 (§2.4)。

- For i = 1 to n: If Tfi is not an array type, let Tfi0 be Tfi; otherwise, let Tfi0 be the element type (§2.4) of Tfi.

- 对于i = 1到n：如果Tfi不是数组类型，则设Tfi0为Tfi；否则，设Tfi0为Tfi的元素类型 (§2.4)。

- L23 Then Tr0 = Tr and Tfi0 = Tfi for i = 0 to n.

- L23 然后Tr0 = Tr且Tfi0 = Tfi，适用于i = 0到n。

Preparation may occur at any time following creation but must be completed prior to initialization.

准备可能发生在创建之后的任何时间，但必须在初始化之前完成。

### 5.4.3 Resolution

The Java Virtual Machine instructions `anewarray`, `checkcast`, `getfield`, `getstatic`, `instanceof`, `invokedynamic`, `invokeinterface`, `invokespecial`, `invokestatic`, `invokevirtual`, `ldc`, `ldc_w`, `multianewarray`, `new`, `putfield`, and `putstatic` make symbolic references to the run-time constant pool. Execution of any of these instructions requires resolution of its symbolic reference.

Java虚拟机指令`anewarray`、`checkcast`、`getfield`、`getstatic`、`instanceof`、`invokedynamic`、`invokeinterface`、`invokespecial`、`invokestatic`、`invokevirtual`、`ldc`、`ldc_w`、`multianewarray`、`new`、`putfield`和`putstatic`对运行时常量池进行了符号引用。执行任何这些指令都需要解析其符号引用。

Resolution is the process of dynamically determining concrete values from symbolic references in the run-time constant pool.

解析是从运行时常量池中的符号引用动态确定具体值的过程。

Resolution of the symbolic reference of one occurrence of an `invokedynamic` instruction does not imply that the same symbolic reference is considered resolved for any other `invokedynamic` instruction.

一次`invokedynamic`指令的符号引用解析并不意味着该符号引用在任何其他`invokedynamic`指令中都被认为已解析。

For all other instructions above, resolution of the symbolic reference of one occurrence of an instruction does imply that the same symbolic reference is considered resolved for any other non-`invokedynamic` instruction.

对于上述所有其他指令，一次指令的符号引用解析确实意味着该符号引用在任何其他非`invokedynamic`指令中都被认为已解析。

(The above text implies that the concrete value determined by resolution for a specific `invokedynamic` instruction is a call site object bound to that specific `invokedynamic` instruction.)

（上述文本暗示，解析为特定`invokedynamic`指令确定的具体值是绑定到该特定`invokedynamic`指令的调用站点对象。）

Resolution can be attempted on a symbolic reference that has already been resolved. An attempt to resolve a symbolic reference that has already successfully been resolved always succeeds trivially and always results in the same entity produced by the initial resolution of that reference.

解析可以尝试在已解析的符号引用上进行。尝试解析已经成功解析的符号引用总是轻而易举地成功，并且总是产生最初解析该引用时的相同实体。

If an error occurs during resolution of a symbolic reference, then an instance of `IncompatibleClassChangeError` (or a subclass) must be thrown at a point in the program that (directly or indirectly) uses the symbolic reference.

如果在解析符号引用期间发生错误，则必须在程序中使用（直接或间接）符号引用的点上抛出`IncompatibleClassChangeError`（或其子类）实例。

If an attempt by the Java Virtual Machine to resolve a symbolic reference fails because an error is thrown that is an instance of `LinkageError` (or a subclass), then subsequent attempts to resolve the reference always fail with the same error that was thrown as a result of the initial resolution attempt.

如果Java虚拟机尝试解析符号引用失败，因为抛出了`LinkageError`（或其子类）的实例，则随后对该引用的解析尝试将始终因最初解析尝试抛出的相同错误而失败。

A symbolic reference to a call site specifier by a specific `invokedynamic` instruction must not be resolved prior to execution of that instruction.

特定`invokedynamic`指令对调用站点说明符的符号引用不得在执行该指令之前解析。

In the case of failed resolution of an `invokedynamic` instruction, the bootstrap method is not re-executed on subsequent resolution attempts.

如果`invokedynamic`指令解析失败，则在随后的解析尝试中不会重新执行引导方法。

Certain of the instructions above require additional linking checks when resolving symbolic references. For instance, in order for a `getfield` instruction to successfully resolve the symbolic reference to the field on which it operates, it must not only complete the field resolution steps given in §5.4.3.2 but also check that the field is not static. If it is a static field, a linking exception must be thrown.

上述某些指令在解析符号引用时需要额外的链接检查。例如，为了使`getfield`指令成功解析它操作的字段的符号引用，它不仅必须完成§5.4.3.2中给出的字段解析步骤，还必须检查字段是否为静态。如果是静态字段，则必须抛出链接异常。

Notably, in order for an `invokedynamic` instruction to successfully resolve the symbolic reference to a call site specifier, the bootstrap method specified therein must complete normally and return a suitable call site object. If the bootstrap method completes abruptly or returns an unsuitable call site object, a linking exception must be thrown.

特别是，为了使`invokedynamic`指令成功解析调用站点说明符的符号引用，其中指定的引导方法必须正常完成并返回合适的调用站点对象。如果引导方法突然完成或返回不合适的调用站点对象，则必须抛出链接异常。

Linking exceptions generated by checks that are specific to the execution of a particular Java Virtual Machine instruction are given in the description of that instruction and are not covered in this general discussion of resolution. Note that such exceptions, although described as part of the execution of Java Virtual Machine instructions rather than resolution, are still properly considered failures of resolution.

由特定Java虚拟机指令执行特定检查生成的链接异常在该指令的描述中给出，并未包含在本一般解析讨论中。请注意，尽管这些异常作为Java虚拟机指令执行的一部分而非解析描述，但仍然被适当地认为是解析失败。

The following sections describe the process of resolving a symbolic reference in the run-time constant pool (§5.1) of a class or interface D. Details of resolution differ with the kind of symbolic reference to be resolved.

以下各节描述了在类或接口D的运行时常量池 (§5.1) 中解析符号引用的过程。解析的细节因符号引用的种类而异。

#### 5.4.3.1 Class and Interface Resolution

To resolve an unresolved symbolic reference from D to a class or interface C denoted by N, the following steps are performed:

要解析D到由N表示的类或接口C的未解析符号引用，执行以下步骤：

1. The defining class loader of D is used to create a class or interface denoted by N. This class or interface is C. The details of the process are given in §5.3.

1. 使用D的定义类加载器创建由N表示的类或接口。该类或接口是C。该过程的细节见§5.3。

Any exception that can be thrown as a result of failure of class or interface creation can thus be thrown as a result of failure of class and interface resolution.

由于类或接口创建失败而可能抛出的任何异常，因此也可能由于类和接口解析失败而抛出。

2. If C is an array class and its element type is a reference type, then a symbolic reference to the class or interface representing the element type is resolved by invoking the algorithm in §5.4.3.1 recursively.

2. 如果C是数组类并且其元素类型是引用类型，则通过递归调用§5.4.3.1中的算法解析表示元素类型的类或接口的符号引用。

3. Finally, access permissions to C are checked.

3. 最后，检查对C的访问权限。

- If C is not accessible (§5.4.4) to D, class or interface resolution throws an `IllegalAccessError`.

- 如果C对D不可访问 (§5.4.4)，类或接口解析会抛出`IllegalAccessError`。

This condition can occur, for example, if C is a class that was originally declared to be public but was changed to be non-public after D was compiled.

例如，如果C是一个类，最初声明为公共的，但在D编译后更改为非公共的，则可能会发生这种情况。

If steps 1 and 2 succeed but step 3 fails, C is still valid and usable. Nevertheless, resolution fails, and D is prohibited from accessing C.

如果步骤1和2成功，但步骤3失败，则C仍然有效且可用。然而，解析失败，并且禁止D访问C。

#### 5.4.3.2 Field Resolution

To resolve an unresolved symbolic reference from _D_ to a field in a class or interface
_C_, the symbolic reference to _C_ given by the field reference must first be resolved
(§5.4.3.1). Therefore, any exception that can be thrown as a result of failure of
resolution of a class or interface reference can be thrown as a result of failure of
field resolution. If the reference to _C_ can be successfully resolved, an exception
relating to the failure of resolution of the field reference itself can be thrown.

要解析类或接口D对类或接口C中字段的未解析符号引用，首先必须解析字段引用所给定的对 C 的符号引用（§5.4.3.1）。因此，解析类或接口引用失败可能引发的任何异常，也可以在字段解析失败时抛出。如果对C的引用能够成功解析，那么可能会抛出与字段引用解析失败相关的异常。如果C的引用可以成功解析，则可能抛出与字段引用解析失败相关的异常。

When resolving a field reference, field resolution first attempts to look up the referenced field in C and its superclasses:

在解析字段引用时，字段解析首先尝试在C及其超类中查找引用的字段：

1. If C declares a field with the name and descriptor specified by the field reference, field lookup succeeds. The declared field is the result of the field lookup.

1. 如果C声明了一个具有字段引用指定的名称和描述符的字段，则字段查找成功。声明的字段是字段查找的结果。

2. Otherwise, field lookup is applied recursively to the direct superinterfaces of the specified class or interface C.

2. 否则，字段查找递归地应用于指定类或接口C的直接超接口。

3. Otherwise, if C has a superclass S, field lookup is applied recursively to S.

3. 否则，如果C有一个超类S，字段查找递归地应用于S。

4. Otherwise, field lookup fails.

4. 否则，字段查找失败。

Then:

接着：

- If field lookup fails, field resolution throws a `NoSuchFieldError`.

- 如果字段查找失败，字段解析会抛出`NoSuchFieldError`。

- Otherwise, if field lookup succeeds but the referenced field is not accessible (§5.4.4) to D, field resolution throws an `IllegalAccessError`.

- 否则，如果字段查找成功，但对D不可访问 (§5.4.4)，字段解析会抛出`IllegalAccessError`。

- Otherwise, let `<E, L1>` be the class or interface in which the referenced field is actually declared and let `L2` be the defining loader of D.

- 否则，设 `<E, L1>` 为实际声明引用字段的类或接口，设 `L2` 为D的定义加载器。

Given that the type of the referenced field is `Tf`, let `Tf0` be `Tf` if `Tf` is not an array type, and let `Tf0` be the element type (§2.4) of `Tf` otherwise.

假定引用字段的类型为`Tf`，则如果`Tf`不是数组类型，则设`Tf0`为`Tf`，否则设`Tf0`为`Tf`的元素类型 (§2.4)。

The Java Virtual Machine must impose the loading constraint that `L1L2 Tf0 = Tf` (§5.3.4).

Java虚拟机必须施加加载约束 `L1L2 Tf0 = Tf` (§5.3.4)。

#### 5.4.3.3 Method Resolution

To resolve an unresolved symbolic reference from D to a method in a class C, the symbolic reference to C given by the method reference is first resolved (§5.4.3.1). Therefore, any exception that can be thrown as a result of failure of resolution of a class reference can be thrown as a result of failure of method resolution. If the reference to C can be successfully resolved, exceptions relating to the resolution of the method reference itself can be thrown.

要解析D到类C中的方法的未解析符号引用，首先要解析方法引用提供的C的符号引用 (§5.4.3.1)。因此，由于类引用解析失败而可能抛出的任何异常也可能由于方法解析失败而抛出。如果C的引用能够成功解析，则可能抛出与方法引用解析本身有关的异常。

When resolving a method reference:

在解析方法引用时：

1. If C is an interface, method resolution throws an `IncompatibleClassChangeError`.

1. 如果C是一个接口，方法解析会抛出`IncompatibleClassChangeError`。

2. Otherwise, method resolution attempts to locate the referenced method in C and its superclasses:

2. 否则，方法解析尝试在C及其超类中定位引用的方法：

- If C declares exactly one method with the name specified by the method reference, and the declaration is a signature polymorphic method (§2.9), then method lookup succeeds. All the class names mentioned in the descriptor are resolved (§5.4.3.1).

- 如果C声明了一个与方法引用指定的名称完全一致的方法，并且该声明是签名多态方法 (§2.9)，则方法查找成功。描述符中提到的所有类名都会被解析 (§5.4.3.1)。

- The resolved method is the signature polymorphic method declaration. It is not necessary for C to declare a method with the descriptor specified by the method reference.

- 解析出来的方法就是签名多态方法声明。C不需要声明与方法引用指定的描述符一致的方法。

- Otherwise, if C declares a method with the name and descriptor specified by the method reference, method lookup succeeds.

- 否则，如果C声明了一个具有方法引用指定的名称和描述符的方法，方法查找成功。

- Otherwise, if C has a superclass, step 2 of method resolution is recursively invoked on the direct superclass of C.

- 否则，如果C有一个超类，则在C的直接超类上递归调用方法解析的第2步。

3. Otherwise, method resolution attempts to locate the referenced method in the superinterfaces of the specified class C:

3. 否则，方法解析尝试在指定类C的超接口中定位引用的方法：

- If the maximally-specific superinterface methods of C for the name and descriptor specified by the method reference include exactly one method that does not have its `ACC_ABSTRACT` flag set, then this method is chosen and method lookup succeeds.

- 如果C的超接口中具有方法引用指定的名称和描述符的最具体的方法中恰好有一个没有设置`ACC_ABSTRACT`标志的方法，则选择此方法并且方法查找成功。

- Otherwise, if any superinterface of C declares a method with the name and descriptor specified by the method reference that has neither its `ACC_PRIVATE` flag nor its `ACC_STATIC` flag set, one of these is arbitrarily chosen and method lookup succeeds.

- 否则，如果C的任何超接口声明了一个具有方法引用指定的名称和描述符且没有设置`ACC_PRIVATE`或`ACC_STATIC`标志的方法，则从这些方法中任意选择一个，方法查找成功。

- Otherwise, method lookup fails.

- 否则，方法查找失败。

A maximally-specific superinterface method of a class or interface C for a particular method name and descriptor is any method for which all of the following are true:

对于类或接口C的特定方法名称和描述符，最具体的超接口方法是满足以下所有条件的任何方法：

1. The method is declared in a superinterface (direct or indirect) of C.

1. 该方法声明在C的超接口（直接或间接）中。

2. The method is declared with the specified name and descriptor.

2. 该方法以指定的名称和描述符声明。

3. The method has neither its `ACC_PRIVATE` flag nor its `ACC_STATIC` flag set.

3. 该方法没有设置`ACC_PRIVATE`或`ACC_STATIC`标志。

4. Where the method is declared in interface I, there exists no other maximally-specific superinterface method of C with the specified name and descriptor that is declared in a subinterface of I.

4. 当方法在接口I中声明时，C没有其他具有指定名称和描述符并且在I的子接口中声明的最具体的超接口方法。

The result of method resolution is determined by whether method lookup succeeds or fails:

方法解析的结果取决于方法查找的成功或失败：

- If method lookup fails, method resolution throws a `NoSuchMethodError`.

- 如果方法查找失败，方法解析会抛出`NoSuchMethodError`。

- Otherwise, if method lookup succeeds and the referenced method is not accessible (§5.4.4) to D, method resolution throws an `IllegalAccessError`.

- 否则，如果方法查找成功，但对D不可访问 (§5.4.4)，方法解析会抛出`IllegalAccessError`。

- Otherwise, let `<E, L1>` be the class or interface in which the referenced method m is actually declared, and let `L2` be the defining loader of D.

- 否则，设 `<E, L1>` 为实际声明引用方法m的类或接口，设 `L2` 为D的定义加载器。

Given that the return type of m is `Tr`, and that the formal parameter types of m are `Tf1, ..., Tfn`, then:

给定m的返回类型是`Tr`，m的形式参数类型是`Tf1, ..., Tfn`，那么：

- If `Tr` is not an array type, let `Tr0` be `Tr`; otherwise, let `Tr0` be the element type (§2.4) of `Tr`.

- 如果`Tr`不是数组类型，则设`Tr0`为`Tr`；否则，设`Tr0`为`Tr`的元素类型 (§2.4)。

- For i = 1 to n: If `Tfi` is not an array type, let `Tfi0` be `Tfi`; otherwise, let `Tfi0` be the element type (§2.4) of `Tfi`.

- 对于i = 1到n：如果`Tfi`不是数组类型，则设`Tfi0`为`Tfi`；否则，设`Tfi0`为`Tfi`的元素类型 (§2.4)。

- The Java Virtual Machine must impose the loading constraints `L1L2 Tr0 = Tr` and `L1L2 Tfi0 = Tfi` for i = 0 to n (§5.3.4).

- Java虚拟机必须施加加载约束 `L1L2 Tr0 = Tr` 和 `L1L2 Tfi0 = Tfi`，适用于i = 0到n (§5.3.4)。

When resolution searches for a method in the class's superinterfaces, the best outcome is to identify a maximally-specific non-abstract method. It is possible that this method will be chosen by method selection, so it is desirable to add class loader constraints for it.

当解析在类的超接口中查找方法时，最好的结果是确定一个最具体的非抽象方法。可能会选择此方法进行方法选择，因此为它添加类加载器约束是可取的。

Otherwise, the result is nondeterministic. This is not new: The Java® Virtual Machine Specification has never identified exactly which method is chosen, and how "ties" should be broken. Prior to Java SE 8, this was mostly an unobservable distinction. However, beginning with Java SE 8, the set of interface methods is more heterogeneous, so care must be taken to avoid problems with nondeterministic behavior. Thus:

否则，结果是非确定性的。这并不新鲜：Java®虚拟机规范从未明确指出究竟选择了哪个方法，以及如何“打破平局”。在Java SE 8之前，这大多是一个不可观察的区别。然而，从Java SE 8开始，接口方法的集合更加异质，因此必须小心避免非确定性行为的问题。因此：

1. Superinterface methods that are private and static are ignored by resolution. This is consistent with the Java programming language, where such interface methods are not inherited.

1. 私有和静态的超接口方法在解析时被忽略。这与Java编程语言一致，在Java中，这些接口方法不会被继承。

2. Any behavior controlled by the resolved method should not depend on whether the method is abstract or not.

2. 由解析方法控制的任何行为都不应依赖于方法是否是抽象的。

Note that if the result of resolution is an abstract method, the referenced class C may be non-abstract. Requiring C to be abstract would conflict with the nondeterministic choice of superinterface methods. Instead, resolution assumes that the runtime class of the invoked object has a concrete implementation of the method.

请注意，如果解析的结果是一个抽象方法，则引用的类C可能是非抽象的。要求C是抽象的将与超接口方法的非确定性选择相冲突。相反，解析假定被调用对象的运行时类具有该方法的具体实现。

#### 5.4.3.4 Interface Method Resolution

To resolve an unresolved symbolic reference from D to an interface method in an interface C, the symbolic reference to C given by the interface method reference is first resolved (§5.4.3.1). Therefore, any exception that can be thrown as a result of failure of resolution of an interface reference can be thrown as a result of failure of interface method resolution. If the reference to C can be successfully resolved, exceptions relating to the resolution of the interface method reference itself can be thrown.

要解析D到接口C中的接口方法的未解析符号引用，首先要解析接口方法引用提供的C的符号引用 (§5.4.3.1)。因此，由于接口引用解析失败而可能抛出的任何异常也可能由于接口方法解析失败而抛出。如果C的引用能够成功解析，则可能抛出与接口方法引用解析本身有关的异常。

When resolving an interface method reference:

在解析接口方法引用时：

1. If C is not an interface, interface method resolution throws an `IncompatibleClassChangeError`.

1. 如果C不是接口，接口方法解析会抛出`IncompatibleClassChangeError`。

2. Otherwise, if C declares a method with the name and descriptor specified by the interface method reference, method lookup succeeds.

2. 否则，如果C声明了一个具有接口方法引用指定的名称和描述符的方法，方法查找成功。

3. Otherwise, if the class `Object` declares a method with the name and descriptor specified by the interface method reference, which has its `ACC_PUBLIC` flag set and does not have its `ACC_STATIC` flag set, method lookup succeeds.

3. 否则，如果类`Object`声明了一个具有接口方法引用指定的名称和描述符的方法，并且该方法设置了`ACC_PUBLIC`标志但没有设置`ACC_STATIC`标志，方法查找成功。

4. Otherwise, if the maximally-specific superinterface methods (§5.4.3.3) of C for the name and descriptor specified by the method reference include exactly one method that does not have its `ACC_ABSTRACT` flag set, then this method is chosen and method lookup succeeds.

4. 否则，如果C的超接口中具有方法引用指定的名称和描述符的最具体的方法中恰好有一个没有设置`ACC_ABSTRACT`标志的方法，则选择此方法并且方法查找成功。

5. Otherwise, if any superinterface of C declares a method with the name and descriptor specified by the method reference that has neither its `ACC_PRIVATE` flag nor its `ACC_STATIC` flag set, one of these is arbitrarily chosen and method lookup succeeds.

5. 否则，如果C的任何超接口声明了一个具有方法引用指定的名称和描述符且没有设置`ACC_PRIVATE`或`ACC_STATIC`标志的方法，则从这些方法中任意选择一个，方法查找成功。

6. Otherwise, method lookup fails.

6. 否则，方法查找失败。

The result of interface method resolution is determined by whether method lookup succeeds or fails:

接口方法解析的结果取决于方法查找的成功或失败：

- If method lookup fails, interface method resolution throws a `NoSuchMethodError`.

- 如果方法查找失败，接口方法解析会抛出`NoSuchMethodError`。

- If method lookup succeeds and the referenced method is not accessible (§5.4.4) to D, interface method resolution throws an `IllegalAccessError`.

- 如果方法查找成功，但对D不可访问 (§5.4.4)，接口方法解析会抛出`IllegalAccessError`。

- Otherwise, let `<E, L1>` be the class or interface in which the referenced interface method m is actually declared, and let `L2` be the defining loader of D.

- 否则，设 `<E, L1>` 为实际声明引用接口方法m的类或接口，设 `L2` 为D的定义加载器。

Given that the return type of m is `Tr`, and that the formal parameter types of m are `Tf1, ..., Tfn`, then:

给定m的返回类型是`Tr`，m的形式参数类型是`Tf1, ..., Tfn`，那么：

- If `Tr` is not an array type, let `Tr0` be `Tr`; otherwise, let `Tr0` be the element type (§2.4) of `Tr`.

- 如果`Tr`不是数组类型，则设`Tr0`为`Tr`；否则，设`Tr0`为`Tr`的元素类型 (§2.4)。

- For i = 1 to n: If `Tfi` is not an array type, let `Tfi0` be `Tfi`; otherwise, let `Tfi0` be the element type (§2.4) of `Tfi`.

- 对于i = 1到n：如果`Tfi`不是数组类型，则设`Tfi0`为`Tfi`；否则，设`Tfi0`为`Tfi`的元素类型 (§2.4)。

- The Java Virtual Machine must impose the loading constraints `L1L2 Tr0 = Tr` and `L1L2 Tfi0 = Tfi` for i = 0 to n (§5.3.4).

- Java虚拟机必须施加加载约束 `L1L2 Tr0 = Tr` 和 `L1L2 Tfi0 = Tfi`，适用于i = 0到n (§5.3.4)。

The clause about accessibility is necessary because interface method resolution may pick a private method of interface C. (Prior to Java SE 8, the result of interface method resolution could be a non-public method of class `Object` or a static method of class `Object`; such results were not consistent with the inheritance model of the Java programming language, and are disallowed in Java SE 8 and above.)

关于可访问性的条款是必要的，因为接口方法解析可能会选择接口C的私有方法。（在Java SE 8之前，接口方法解析的结果可能是类`Object`的非公共方法或类`Object`的静态方法；这些结果与Java编程语言的继承模型不一致，在Java SE 8及以上版本中是不允许的。）

#### 5.4.3.5 Method Type and Method Handle Resolution

To resolve an unresolved symbolic reference to a method type, it is as if resolution occurs of unresolved symbolic references to classes and interfaces (§5.4.3.1) whose names correspond to the types given in the method descriptor (§4.3.3).

要解析对方法类型的未解析符号引用，仿佛解析了与方法描述符 (§4.3.3) 中给出的类型对应的类和接口的未解析符号引用 (§5.4.3.1)。

Any exception that can be thrown as a result of failure of resolution of a class reference can thus be thrown as a result of failure of method type resolution.

因此，由于类引用解析失败而可能抛出的任何异常也可能由于方法类型解析失败而抛出。

The result of successful method type resolution is a reference to an instance of `java.lang.invoke.MethodType` which represents the method descriptor.

成功的解析方法类型的结果是对`java.lang.invoke.MethodType`实例的引用，该实例表示方法描述符。

Method type resolution occurs regardless of whether the runtime constant pool actually contains symbolic references to classes and interfaces indicated in the method descriptor. Also, the resolution is deemed to occur on unresolved symbolic references, so a failure to resolve one method type will not necessarily lead to a later failure to resolve another method type with the same textual method descriptor, if suitable classes and interfaces can be loaded by the later time.

无论运行时常量池是否实际包含方法描述符中指示的类和接口的符号引用，方法类型解析都会发生。此外，解析被认为发生在未解析的符号引用上，因此解析一种方法类型的失败不会必然导致稍后解析具有相同文本方法描述符的另一种方法类型失败，如果稍后可以加载适当的类和接口。

Resolution of an unresolved symbolic reference to a method handle is more complicated. Each method handle resolved by the Java Virtual Machine has an equivalent instruction sequence called its bytecode behavior, indicated by the method handle's kind. The integer values and descriptions of the nine kinds of method handle are given in Table 5.4.3.5-A.

解析方法句柄的未解析符号引用更加复杂。由Java虚拟机解析的每个方法句柄都有一个等效的指令序列，称为其字节码行为，由方法句柄的种类指示。九种方法句柄的整数值和描述见表5.4.3.5-A。

Symbolic references by an instruction sequence to fields or methods are indicated by `C.x:T`, where x and T are the name and descriptor (§4.3.2, §4.3.3) of the field or method, and C is the class or interface in which the field or method is to be found.

指令序列对字段或方法的符号引用由 `C.x:T` 表示，其中x和T是字段或方法的名称和描述符 (§4.3.2, §4.3.3)，C是要找到字段或方法的类或接口。

**Table 5.4.3.5-A. Bytecode Behaviors for Method Handles**

| Kind | Description              | Interpretation           |
|------|--------------------------|--------------------------|
| 1    | `REF_getField`           | `getfield C.f:T`          |
| 2    | `REF_getStatic`          | `getstatic C.f:T`         |
| 3    | `REF_putField`           | `putfield C.f:T`          |
| 4    | `REF_putStatic`          | `putstatic C.f:T`         |
| 5    | `REF_invokeVirtual`      | `invokevirtual C.m:(A*)T` |
| 6    | `REF_invokeStatic`       | `invokestatic C.m:(A*)T`  |
| 7    | `REF_invokeSpecial`      | `invokespecial C.m:(A*)T` |
| 8    | `REF_newInvokeSpecial`   | `new C; dup; invokespecial C.<init>:(A*)V` |
| 9    | `REF_invokeInterface`    | `invokeinterface C.m:(A*)T` |

Let MH be the symbolic reference to a method handle (§5.1) being resolved. Then:

设MH为正在解析的方法句柄 (§5.1) 的符号引用。那么：

- Let R be the symbolic reference to the field or method contained within MH. (R is derived from the `CONSTANT_Fieldref`, `CONSTANT_Methodref`, or `CONSTANT_InterfaceMethodref` structure referred to by the `reference_index` item of the `CONSTANT_MethodHandle` from which MH is derived.)

- 设R为MH中包含的字段或方法的符号引用。（R派生自`CONSTANT_Fieldref`、`CONSTANT_Methodref`或`CONSTANT_InterfaceMethodref`结构，引用由MH派生的`CONSTANT_MethodHandle`的`reference_index`项。）

- Let T be the type of the field referenced by R, or the return type of the method referenced by R. Let A* be the sequence (perhaps empty) of parameter types of the method referenced by R.

- 设T为R引用的字段的类型，或R引用的方法的返回类型。设A*为R引用的方法的参数类型序列（可能为空）。

- (T and A* are derived from the `CONSTANT_NameAndType` structure referred to by the `name_and_type_index` item in the `CONSTANT_Fieldref`, `CONSTANT_Methodref`, or `CONSTANT_InterfaceMethodref` structure from which R is derived.)

- （T和A*派生自`CONSTANT_NameAndType`结构，引用由R派生的`CONSTANT_Fieldref`、`CONSTANT_Methodref`或`CONSTANT_InterfaceMethodref`结构中的`name_and_type_index`项。）

To resolve MH, all symbolic references to classes, interfaces, fields, and methods in MH's bytecode behavior are resolved, using the following three steps:

要解析MH，必须解析MH字节码行为中的所有符号引用，包括类、接口、字段和方法，使用以下三步：

1. First, R is resolved.

1. 首先，解析R。

2. Second, resolution occurs as if of unresolved symbolic references to classes and interfaces whose names correspond to each type in A*, and to the type T, in that order.

2. 其次，解析仿佛未解析的符号引用，依次对应A*中的每种类型和类型T的类和接口的名称。

3. Third, a reference to an instance of `java.lang.invoke.MethodType` is obtained as if by resolution of an unresolved symbolic reference to a method type that contains the method descriptor specified in **Table 5.4.3.5-B** for the kind of MH.

3. 第三，通过解析未解析的符号引用获得对`java.lang.invoke.MethodType`实例的引用，仿佛该引用包含表5.4.3.5-B中指定的与MH类型相对应的方法描述符。

It is as if the symbolic reference to a method handle contains a symbolic reference to the method type that the resolved method handle will eventually have. The detailed structure of the method type is obtained by inspecting **Table 5.4.3.5-B**.

仿佛方法句柄的符号引用包含对解析后的方法句柄最终将具有的方法类型的符号引用。通过检查表5.4.3.5-B获取方法类型的详细结构。

**Table 5.4.3.5-B. Method Descriptors for Method Handles**

| Kind | Description              | Method descriptor         |
|------|--------------------------|--------------------------|
| 1    | `REF_getField`           | `(C)T`                   |
| 2    | `REF_getStatic`          | `()T`                    |
| 3    | `REF_putField`           | `(C,T)V`                 |
| 4    | `REF_putStatic`          | `(T)V`                   |
| 5    | `REF_invokeVirtual`      | `(C,A*)T`                |
| 6    | `REF_invokeStatic`       | `(A*)T`                  |
| 7    | `REF_invokeSpecial`      | `(C,A*)T`                |
| 8    | `REF_newInvokeSpecial`   | `(A*)C`                  |
| 9    | `REF_invokeInterface`    | `(C,A*)T`                |

In each step, any exception that can be thrown as a result of failure of resolution of a class or interface or field or method reference can be thrown as a result of failure of method handle resolution.

在每一步中，由于类、接口、字段或方法引用解析失败而可能抛出的任何异常也可能由于方法句柄解析失败而抛出。

The intent is that resolving a method handle can be done in exactly the same circumstances that the Java Virtual Machine would successfully resolve the symbolic references in the bytecode behavior. In particular, method handles to private and protected members can be created in exactly those classes for which the corresponding normal accesses are legal.

目的是，在Java虚拟机成功解析字节码行为中的符号引用的情况下，可以在完全相同的情况下解析方法句柄。特别是，可以在对相应的正常访问合法的类中创建对私有和受保护成员的方法句柄。

#### 5.4.3.6 Call Site Specifier Resolution

To resolve an unresolved symbolic reference to a call site specifier involves three steps:

解析对调用站点说明符的未解析符号引用涉及三步：

1. A call site specifier gives a symbolic reference to a method handle which is to serve as the bootstrap method for a dynamic call site (§4.7.23). The method handle is resolved to obtain a reference to an instance of `java.lang.invoke.MethodHandle` (§5.4.3.5).

1. 调用站点说明符提供了一个方法句柄的符号引用，该句柄将作为动态调用站点的引导方法 (§4.7.23)。解析方法句柄以获得对`java.lang.invoke.MethodHandle`实例的引用 (§5.4.3.5)。

2. A call site specifier gives a method descriptor, `TD`. A reference to an instance of `java.lang.invoke.MethodType` is obtained as if by resolution of a symbolic reference to a method type with the same parameter and return types as `TD` (§5.4.3.5).

2. 调用站点说明符提供了一个方法描述符`TD`。解析一个符号引用，以仿佛解析具有与`TD`相同参数和返回类型的方法类型的符号引用，从而获得对`java.lang.invoke.MethodType`实例的引用 (§5.4.3.5)。

3. A call site specifier gives zero or more static arguments, which communicate application-specific metadata to the bootstrap method. Any static arguments which are symbolic references to classes, method handles, or method types are resolved, as if by invocation of the `ldc` instruction (§ldc), to obtain references to `Class` objects, `java.lang.invoke.MethodHandle` objects, and `java.lang.invoke.MethodType` objects respectively. Any static arguments that are string literals are used to obtain references to `String` objects.

3. 调用站点说明符提供了零个或多个静态参数，这些参数将应用程序特定的元数据传递给引导方法。任何作为类、方法句柄或方法类型的符号引用的静态参数都被解析，仿佛通过调用`ldc`指令 (§ldc)，分别获得对`Class`对象、`java.lang.invoke.MethodHandle`对象和`java.lang.invoke.MethodType`对象的引用。任何作为字符串字面量的静态参数都用于获得对`String`对象的引用。

The result of call site specifier resolution is a tuple consisting of:

调用站点说明符解析的结果是一个元组，包含以下内容：

1. The reference to an instance of `java.lang.invoke.MethodHandle`,

1. 对`java.lang.invoke.MethodHandle`实例的引用，


2. The reference to an instance of `java.lang.invoke.MethodType`,

2. 对`java.lang.invoke.MethodType`实例的引用，

3. The references to instances of `Class`, `java.lang.invoke.MethodHandle`, `java.lang.invoke.MethodType`, and `String`.

3. 对`Class`、`java.lang.invoke.MethodHandle`、`java.lang.invoke.MethodType`和`String`实例的引用。

During resolution of the symbolic reference to the method handle in the call site specifier, or resolution of the symbolic reference to the method type for the method descriptor in the call site specifier, or resolution of a symbolic reference to any static argument, any of the exceptions pertaining to method type or method handle resolution may be thrown (§5.4.3.5).

在解析对调用站点说明符中方法句柄的符号引用期间，或解析对调用站点说明符中方法描述符的方法类型的符号引用期间，或解析对任何静态参数的符号引用期间，可能会抛出与方法类型或方法句柄解析相关的任何异常 (§5.4.3.5)。

### 5.4.4 Access Control

A class or interface C is accessible to a class or interface D if and only if either of the following is true:

类或接口C对类或接口D是可访问的，当且仅当以下条件之一为真：


1. C is public.

1. C是公共的。

2. C and D are members of the same runtime package (§5.3).

2. C和D属于同一个运行时包 (§5.3)。

A field or method R is accessible to a class or interface D if and only if any of the following is true:

字段或方法R对类或接口D是可访问的，当且仅当以下条件之一为真：

1. R is public.

1. R是公共的。

2. R is protected and is declared in a class C, and D is either a subclass of C or C itself. Furthermore, if R is not static, then the symbolic reference to R must contain a symbolic reference to a class T, such that T is either a subclass of D, a superclass of D, or D itself.

2. R是受保护的，并且声明在类C中，并且D要么是C的子类，要么是C本身。此外，如果R不是静态的，则R的符号引用必须包含对类T的符号引用，T要么是D的子类，要么是D的超类，要么是D本身。

3. R is either protected or has default access (that is, neither public nor protected nor private), and is declared by a class in the same runtime package as D.

3. R是受保护的，或者具有默认访问权限（即，既不是公共的，也不是受保护的，也不是私有的），并且由与D在同一个运行时包中的类声明。

4. R is private and is declared in D.

4. R是私有的，并且在D中声明。

This discussion of access control omits a related restriction on the target of a protected field access or method invocation (the target must be of class D or a subtype of D). That requirement is checked as part of the verification process (§4.10.1.8); it is not part of link-time access control.

对于受保护字段访问或方法调用的目标，这一访问控制的讨论省略了相关的限制（目标必须是D类或D的子类型）。该要求作为验证过程的一部分进行检查 (§4.10.1.8)；它不是链接时访问控制的一部分。

### 5.4.5 Overriding

An instance method m declared in class C overrides another instance method m_CA declared in class A if and only if either m_C is the same as m_CA, or all of the following are true:

类C中声明的实例方法m_C覆盖类A中声明的另一个实例方法m_CA，当且仅当m_C与m_CA相同，或者以下所有条件为真：

1. C is a subclass of A.

1. C是A的子类。

2. m_C has the same name and descriptor as m_CA.

2. m_C与m_CA具有相同的名称和描述符。

3. m_C is not marked `ACC_PRIVATE`.

3. m_C未标记为`ACC_PRIVATE`。

4. One of the following is true:

4. 以下之一为真：

- m_CA is marked `ACC_PUBLIC`; or is marked `ACC_PROTECTED`; or is marked neither `ACC_PUBLIC` nor `ACC_PROTECTED` nor `ACC_PRIVATE` and A belongs to the same runtime package as C.

- m_CA标记为`ACC_PUBLIC`；或者标记为`ACC_PROTECTED`；或者未标记为`ACC_PUBLIC`、`ACC_PROTECTED`或`ACC_PRIVATE`，并且A与C属于同一运行时包。


- m_C overrides a method m'_CA (m'_CA distinct from m_C and m_CA) such that m'_CA overrides m_CA.

- m_C覆盖了方法m'_CA（m'_CA不同于m_C和m_CA），使得m'_CA覆盖m_CA。

---
## 5.5 Initialization

Initialization of a class or interface consists of executing its class or interface initialization method (§2.9).

类或接口的初始化包括执行其类或接口初始化方法 (§2.9)。

A class or interface C may be initialized only as a result of:

类或接口C只能作为以下结果之一进行初始化：

1. The execution of any one of the Java Virtual Machine instructions `new`, `getstatic`, `putstatic`, or `invokestatic` that references C (§new, §getstatic, §putstatic, §invokestatic). These instructions reference a class or interface directly or indirectly through either a field reference or a method reference.

1. 执行引用C的任何一个Java虚拟机指令`new`、`getstatic`、`putstatic`或`invokestatic`（§new、§getstatic、§putstatic、§invokestatic）。这些指令通过字段引用或方法引用直接或间接地引用类或接口。

- Upon execution of a `new` instruction, the referenced class is initialized if it has not been initialized already.

- 执行`new`指令时，如果引用的类尚未初始化，则初始化它。

- Upon execution of a `getstatic`, `putstatic`, or `invokestatic` instruction, the class or interface that declared the resolved field or method is initialized if it has not been initialized already.

- 执行`getstatic`、`putstatic`或`invokestatic`指令时，如果声明了解析字段或方法的类或接口尚未初始化，则初始化它。

2. The first invocation of a `java.lang.invoke.MethodHandle` instance which was the result of method handle resolution (§5.4.3.5) for a method handle of kind 2 (`REF_getStatic`), 4 (`REF_putStatic`), 6 (`REF_invokeStatic`), or 8 (`REF_newInvokeSpecial`).

2. 第一次调用`java.lang.invoke.MethodHandle`实例，该实例是方法句柄解析的结果 (§5.4.3.5)，用于类型2 (`REF_getStatic`)、4 (`REF_putStatic`)、6 (`REF_invokeStatic`) 或8 (`REF_newInvokeSpecial`) 的方法句柄。

- This implies that the class of a bootstrap method is initialized when the bootstrap method is invoked for an `invokedynamic` instruction (§invokedynamic), as part of the continuing resolution of the call site specifier.

- 这意味着当引导方法为`invokedynamic`指令 (§invokedynamic) 调用时，引导方法的类会初始化，作为继续解析调用站点说明符的一部分。

3. Invocation of certain reflective methods in the class library (§2.12), for example, in class `Class` or in package `java.lang.reflect`.

3. 调用类库中的某些反射方法 (§2.12)，例如在`Class`类中或在`java.lang.reflect`包中。

4. If C is a class, the initialization of one of its subclasses.

4. 如果C是一个类，则初始化其子类之一。

5. If C is an interface that declares a non-abstract, non-static method, the initialization of a class that implements C directly or indirectly.

5. 如果C是声明了非抽象、非静态方法的接口，则初始化直接或间接实现C的类。

6. If C is a class, its designation as the initial class at Java Virtual Machine startup (§5.2).

6. 如果C是一个类，则在Java虚拟机启动时指定为初始类 (§5.2)。

Prior to initialization, a class or interface must be linked, that is, verified, prepared, and optionally resolved.

在初始化之前，类或接口必须链接，即验证、准备，并可选地解析。

Because the Java Virtual Machine is multithreaded, initialization of a class or interface requires careful synchronization, since some other thread may be trying to initialize the same class or interface at the same time. There is also the possibility that initialization of a class or interface may be requested recursively as part of the initialization of that class or interface. The implementation of the Java Virtual Machine is responsible for taking care of synchronization and recursive initialization by using the following procedure. It assumes that the `Class` object has already been verified and prepared, and that the `Class` object contains state that indicates one of four situations:

由于Java虚拟机是多线程的，因此类或接口的初始化需要仔细同步，因为其他线程可能会尝试同时初始化相同的类或接口。也有可能作为类或接口初始化的一部分递归地请求类或接口的初始化。Java虚拟机的实现负责通过使用以下过程处理同步和递归初始化。假定`Class`对象已经过验证和准备，并且`Class`对象包含指示以下四种情况之一的状态：

1. This `Class` object is verified and prepared but not initialized.

1. 该`Class`对象已验证和准备，但尚未初始化。

2. This `Class` object is being initialized by some particular thread.

2. 该`Class`对象正在由某个特定线程初始化。

3. This `Class` object is fully initialized and ready for use.

3. 该`Class`对象已完全初始化并可供使用。

4. This `Class` object is in an erroneous state, perhaps because initialization was attempted and failed.

4. 该`Class`对象处于错误状态，可能是因为初始化尝试失败。

For each class or interface C, there is a unique initialization lock `LC`. The mapping from C to `LC` is left to the discretion of the Java Virtual Machine implementation. For example, `LC` could be the `Class` object for C, or the monitor associated with that `Class` object. The procedure for initializing C is then as follows:

对于每个类或接口C，存在一个唯一的初始化锁`LC`。从C到`LC`的映射由Java虚拟机实现自行决定。例如，`LC`可以是C的`Class`对象，也可以是与该`Class`对象相关联的监视器。初始化C的过程如下：

1. Synchronize on the initialization lock, `LC`, for C. This involves waiting until the current thread can acquire `LC`.

1. 在C的初始化锁`LC`上进行同步。这涉及等待当前线程可以获取`LC`。

2. If the `Class` object for C indicates that initialization is in progress for C by some other thread, then release `LC` and block the current thread until informed that the in-progress initialization has completed, at which time repeat this procedure.

2. 如果C的`Class`对象指示其他线程正在进行C的初始化，则释放`LC`并阻塞当前线程，直到通知初始化已完成为止，此时重复此过程。

- Thread interrupt status is unaffected by execution of the initialization procedure.

- 初始化过程的执行不会影响线程中断状态。

3. If the `Class` object for C indicates that initialization is in progress for C by the current thread, then this must be a recursive request for initialization. Release `LC` and complete normally.

3. 如果C的`Class`对象指示当前线程正在进行C的初始化，则这必须是递归初始化请求。释放`LC`并正常完成。

4. If the `Class` object for C indicates that C has already been initialized, then no further action is required. Release `LC` and complete normally.

4. 如果C的`Class`对象指示C已经初始化，则不需要进一步操作。释放`LC`并正常完成。

5. If the `Class` object for C is in an erroneous state, then initialization is not possible. Release `LC` and throw a `NoClassDefFoundError`.

5. 如果C的`Class`对象处于错误状态，则无法初始化。释放`LC`并抛出`NoClassDefFoundError`。

6. Otherwise, record the fact that initialization of the `Class` object for C is in progress by the current thread, and release `LC`.

6. 否则，记录当前线程正在进行C的`Class`对象的初始化，并释放`LC`。

- Then, initialize each final static field of C with the constant value in its `ConstantValue` attribute (§4.7.2), in the order the fields appear in the `ClassFile` structure.

- 然后，按字段在`ClassFile`结构中出现的顺序，用其`ConstantValue`属性中的常量值 (§4.7.2) 初始化C的每个最终静态字段。

7. Next, if C is a class rather than an interface, and its superclass has not yet been initialized, then let `SC` be its superclass and let `SI1, ..., SIn` be all superinterfaces of C (whether direct or indirect) that declare at least one non-abstract, non-static method. The order of superinterfaces is given by a recursive enumeration over the superinterface hierarchy of each interface directly implemented by C. For each interface I directly implemented by C (in the order of the `interfaces` array of C), the enumeration recurs on I's superinterfaces (in the order of the `interfaces` array of I) before returning I.

7. 接下来，如果C是一个类而不是接口，并且它的超类尚未初始化，则设`SC`为其超类，设`SI1, ..., SIn`为C的所有超接口（无论是直接的还是间接的），它们声明了至少一个非抽象的非静态方法。超接口的顺序由C直接实现的每个接口的超接口层次结构的递归枚举给出。对于C直接实现的每个接口I（按C的`interfaces`数组的顺序），枚举在I的超接口上递归（按I的`interfaces`数组的顺序）然后返回I。

- For each `S` in the list `[ SC, SI1, ..., SIn ]`, recursively perform this entire procedure for `S`. If necessary, verify and prepare `S` first.

- 对于列表 `[ SC, SI1, ..., SIn ]` 中的每个 `S`，递归地为 `S` 执行整个过程。如果必要，先验证并准备 `S`。

- If the initialization of `S` completes abruptly because of a thrown exception, then acquire `LC`, label the `Class` object for C as erroneous, notify all waiting threads, release `LC`, and complete abruptly, throwing the same exception that resulted from initializing `SC`.

- 如果`S`的初始化由于抛出异常而突然完成，则获取`LC`，将C的`Class`对象标记为错误，通知所有等待的线程，释放`LC`并突然完成，抛出与初始化`SC`相同的异常。

8. Next, determine whether assertions are enabled for C by querying its defining class loader.

8. 接下来，通过查询其定义类加载器确定是否为C启用了断言。

9. Next, execute the class or interface initialization method of C.

9. 接下来，执行C的类或接口初始化方法。

10. If the execution of the class or interface initialization method completes normally, then acquire `LC`, label the `Class` object for C as fully initialized, notify all waiting threads, release `LC`, and complete this procedure normally.

10. 如果类或接口初始化方法的执行正常完成，则获取`LC`，将C的`Class`对象标记为完全初始化，通知所有等待的线程，释放`LC`并正常完成此过程。

11. Otherwise, the class or interface initialization method must have completed abruptly by throwing some exception `E`. If the class of `E` is not `Error` or one of its subclasses, then create a new instance of the class `ExceptionInInitializerError` with `E` as the argument, and use this object in place of `E` in the following step. If a new instance of `ExceptionInInitializerError` cannot be created because an `OutOfMemoryError` occurs, then use an `OutOfMemoryError` object in place of `E`.

11. 否则，类或接口初始化方法必须通过抛出某个异常`E`突然完成。如果`E`的类不是`Error`或其子类之一，则使用`E`作为参数创建一个新的`ExceptionInInitializerError`类的实例，并在以下步骤中使用此对象代替`E`。如果由于发生`OutOfMemoryError`而无法创建`ExceptionInInitializerError`的实例，则使用`OutOfMemoryError`对象代替`E`。

12. Acquire `LC`, label the `Class` object for C as erroneous, notify all waiting threads, release `LC`, and complete this procedure abruptly with reason `E` or its replacement as determined in the previous step.

12. 获取`LC`，将C的`Class`对象标记为错误，通知所有等待的线程，释放`LC`并突然完成此过程，原因是`E`或在前一步骤中确定的替换。

A Java Virtual Machine implementation may optimize this procedure by eliding the lock acquisition in step 1 (and release in step 4/5) when it can determine that the initialization of the class has already completed, provided that, in terms of the Java memory model, all happens-before orderings (JLS §17.4.5) that would exist if the lock were acquired, still exist when the optimization is performed.

Java虚拟机实现可以通过在步骤1中省略锁获取（在步骤4/5中省略释放）来优化此过程，当它可以确定类的初始化已经完成时，前提是，根据Java内存模型，如果获取了锁，所有会存在的发生先于顺序（JLS §17.4.5）在优化执行时仍然存在。

---
## 5.6 Binding Native Method Implementations

Binding is the process by which a function written in a language other than the Java programming language and implementing a native method is integrated into the Java Virtual Machine so that it can be executed. Although this process is traditionally referred to as linking, the term binding is used in the specification to avoid confusion with linking of classes or interfaces by the Java Virtual Machine.

绑定是指将用非Java编程语言编写并实现本地方法的函数集成到Java虚拟机中，使其可以执行的过程。尽管这个过程传统上称为链接，但在规范中使用绑定一词是为了避免与Java虚拟机对类或接口的链接混淆。

---
## 5.7 Java Virtual Machine Exit

The Java Virtual Machine exits when some thread invokes the `exit` method of class `Runtime` or class `System`, or the `halt` method of class `Runtime`, and the exit or halt operation is permitted by the security manager.

当某个线程调用`Runtime`类或`System`类的`exit`方法，或`Runtime`类的`halt`方法，并且安全管理器允许退出或停止操作时，Java虚拟机退出。

In addition, the JNI (Java Native Interface) Specification describes termination of the Java Virtual Machine when the JNI Invocation API is used to load and unload the Java Virtual Machine.

此外，JNI（Java本地接口）规范描述了在使用JNI调用API加载和卸载Java虚拟机时Java虚拟机的终止。