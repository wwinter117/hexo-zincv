---
title: jvm-类加载流程
tags:
  - jvm
date: 2022-8-7 22:46:49
---

当使用 `java -jar HelloApp.jar` 启动一个 Java 应用程序时，JVM 的类加载过程是由多个步骤组成的，并且不同的类加载器在不同阶段扮演着不同的角色。下面是详细的过程：

### 1. 启动JVM

 当运行 `java -jar HelloApp.jar` 时，JVM 启动并初始化。这时，JVM 会先加载一些必要的系统类，这些类是由 **Bootstrap ClassLoader（引导类加载器）** 负责加载的。

### 2. 引导类加载器加载核心类库

**Bootstrap ClassLoader** 是 `JVM` 自带的一个特殊的类加载器，它使用本地代码实现，负责加载 `JVM` 自身需要的核心类库，如 `java.lang.*`、`java.util.*` 等，这些类通常位于 `rt.jar` 或者在模块化 `JDK` 中的基础模块。

具体来说，`Bootstrap ClassLoader` 会加载类似 `java.lang.Object`、`java.lang.String`、`java.lang.ClassLoader` 等类。这些是 `JVM` 启动和运行 `Java` 应用程序所必需的类。

### 3. 扩展类加载器加载扩展类库

`JVM` 然后使用 **Extension ClassLoader（扩展类加载器）**，它加载位于 `jre/lib/ext` 目录下的扩展类库（在模块化 `JDK` 中，这个加载器主要加载某些模块）。这些类库通常是标准扩展，但并不是所有应用程序都需要。

### 4. 应用类加载器加载应用程序类

接下来，**Application ClassLoader（应用类加载器）** 加载应用程序的类。对于 `java -jar Hello.jar`，应用类加载器会加载 JAR 文件中的主类和其他所有相关类。此时，`Application ClassLoader` 会从 `JAR` 文件的 `classpath` 中找到 `HelloApp.class` 并将其加载到内存中。

### 5. 加载并解析主类

应用类加载器加载主类（即包含 `public static void main(String[] args)` 方法的类），并解析它的字节码。

在解析过程中，如果主类依赖于其他类，这些依赖类将按照**双亲委派模型**被相应的类加载器加载。

### 6. 执行主类的 `main` 方法

当主类被加载并初始化后，`JVM` 调用主类的 `main` 方法，程序正式开始执行。

### 7. 类加载的双亲委派模型

在整个类加载过程中，`JVM` 使用双亲委派模型来加载类。即当一个类加载器尝试加载一个类时，它首先会把请求委派给父类加载器（如 `Application ClassLoader` 会委派给 `Extension ClassLoader`，`Extension ClassLoader` 又委派给 `Bootstrap ClassLoader`）。如果父类加载器无法加载该类，那么子加载器才会尝试自己加载。

这种模型保证了核心类库不会被错误的加载（例如，防止用户定义的 `java.lang.String` 类覆盖系统类库中的同名类）。

### 8. 动态类加载

如果在 `main` 方法执行期间，某些类是第一次被引用，`JVM` 会动态加载这些类。这个加载过程依旧遵循双亲委派模型，由对应的类加载器加载这些新类。

### 总结

当使用 `java -jar Hello.jar` 启动一个 Java 程序时，`JVM` 依次使用 `Bootstrap ClassLoader`、`Extension ClassLoader` 和 `Application ClassLoader` 来加载所需的类库。`Bootstrap ClassLoader` 负责加载核心类库（包括 `java.lang.ClassLoader`），随后，应用类加载器负责加载 `JAR` 文件中的应用程序类。程序开始执行时，`JVM` 已经准备好了所有必要的类，并开始调用 `main` 方法来启动应用。