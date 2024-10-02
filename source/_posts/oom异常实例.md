---
title: oom异常实例
tags:
  - jdk
date: 2022-8-1 22:46:49
---
![](https://zincv.oss-cn-hangzhou.aliyuncs.com/images/oom-Pasted%20image%2020240909195755.png)
### 1. 堆溢出

测试代码：

```java
package cn.wwinter;  
  
import java.util.ArrayList;  
import java.util.List;  
  
public class HeapOOM {  
    static class OOMObject {  
    }  
    public static void main(String[] args) {  
        List<OOMObject> list = new ArrayList<>();  
        while (true) {  
            list.add(new OOMObject());  
        }  
    }  
}
```

执行：`java -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError HeapOOM.java`： 

- `-Xms20m`：设置 `JVM` 初始堆内存大小为 `20MB`。
- `-Xmx20m`：设置 `JVM` 最大堆内存大小为 `20MB`。
- `-XX:+HeapDumpOnOutOfMemoryError`：当堆内存溢出时，`JVM` 会生成一个堆转储文件（`.hprof` 文件），用于后续的内存分析。

```bash
zhangdongdong:wwinter/ (master✗) $ java -Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError HeapOOM.java                                                                   [19:51:00]
java.lang.OutOfMemoryError: Java heap space
Dumping heap to java_pid18144.hprof ...
Heap dump file created [34168105 bytes in 0.052 secs]
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
        at java.base/java.util.Arrays.copyOf(Arrays.java:3512)
        at java.base/java.util.Arrays.copyOf(Arrays.java:3481)
        at java.base/java.util.ArrayList.grow(ArrayList.java:237)
        at java.base/java.util.ArrayList.grow(ArrayList.java:244)
        at java.base/java.util.ArrayList.add(ArrayList.java:454)
        at java.base/java.util.ArrayList.add(ArrayList.java:467)
        at cn.wwinter.HeapOOM.main(HeapOOM.java:13)
```


![](https://zincv.oss-cn-hangzhou.aliyuncs.com/images/oom-Pasted%20image%2020240909195755.png)


从堆转储文件来看，有以下几个关键点：
##### 1. `cn.wwinter.HeapOOM$OOMObject` 类的实例数量

- 该类的实例数量达到了 **810,326** 个，占用了 **12.97 MB** 的内存。
##### 2. `java.lang.Object[]` 和 `byte[]` 的内存占用

- `java.lang.Object[]` 占用了 **17.07 MB**。
- `byte[]` 占用了 **728.21 KB**。
##### 3. 内存泄漏或大内存消耗的根本原因

- 从分析结果来看，最大的内存消耗者是 `cn.wwinter.HeapOOM$OOMObject`，这表明代码中的 `list.add(new OOMObject());` 是导致堆内存溢出的主要原因。

### 2.  栈溢出

```java
package cn.wwinter;  
  
public class JavaVMStackSOF {  
    private int stackLength = 1;  
  
    public void stackLeak() {  
        stackLength++;  
        stackLeak();  
    }  
  
    public static void main(String[] args) {  
        JavaVMStackSOF sof = new JavaVMStackSOF();  
        try {  
            sof.stackLeak();  
        } catch (Throwable e) {  
            System.out.println("stack length: " + sof.stackLength);  
            throw e;  
        }  
    }  
}
```

```bash
stack length: 36574
Exception in thread "main" java.lang.StackOverflowError
	at cn.wwinter.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:8)
	at cn.wwinter.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:8)
	at cn.wwinter.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:8)
```

### 3.  方法区溢出（通常与元空间有关）

在 `Java 8` 及以后的版本中，方法区已经被移到元空间（`Metaspace`），而在 `Java 7` 及以前，方法区包含在永久代（`PermGen`）中。以下代码通过动态生成大量类，导致方法区溢出：

#### 示例代码：方法区溢出（元空间溢出）


```java
import net.sf.cglib.proxy.Enhancer;  
import net.sf.cglib.proxy.MethodInterceptor;  
  
public class MetaspaceOOM {  
    public static void main(String[] args) {  
        while (true) {  
            Enhancer enhancer = new Enhancer();  
            enhancer.setSuperclass(MetaspaceOOM.class);  
            enhancer.setUseCache(false);  
            enhancer.setCallback((MethodInterceptor) (obj, method, args1, proxy) -> proxy.invokeSuper(obj, args1));  
            enhancer.create();  
        }  
    }  
}
```

#### 说明：

- 这个代码使用  `CGLIB`  动态生成了大量的类，并不断地将这些类加载到 `JVM` 中，导致元空间（`Metaspace`）溢出。
    
- 运行时可以使用以下 `JVM` 参数来限制元空间的大小：
    
    `java -XX:MaxMetaspaceSize=10m MetaspaceOOM`
    
    当生成的类过多，导致元空间耗尽时，会抛出 `java.lang.OutOfMemoryError: Metaspace` 异常。
    
### 4. 运行时常量池溢出

在 `Java 7` 及以前，运行时常量池（`Runtime Constant Pool`）是在方法区（永久代）中的，而在 `Java 8` 及以后，它在堆中。下面的代码通过不断地向常量池中添加字符串来导致溢出。

#### 示例代码：运行时常量池溢出


```java
import java.util.ArrayList;  
import java.util.List;  
  
public class RuntimeConstantPoolOOM {  
    public static void main(String[] args) {  
        List<String> list = new ArrayList<>();  
        int i = 0;  
        while (true) {  
            list.add(String.valueOf(i++).intern());  
        }  
    }  
}
```

#### 说明：

- 这个代码不断生成新的字符串，并调用 `intern()` 方法将其放入运行时常量池中。
    
- 如果使用 `Java 7`，可以通过以下 `JVM` 参数来限制永久代大小，从而触发 `java.lang.OutOfMemoryError: PermGen space` 异常：
    
    `java -XX:PermSize=10m -XX:MaxPermSize=10m RuntimeConstantPoolOOM`
    
- 在 `Java 8` 及以后，这个代码将会导致堆内存溢出，因为运行时常量池在堆内存中：
    `java -Xmx10m RuntimeConstantPoolOOM`
    
    当堆内存耗尽时，会抛出 `java.lang.OutOfMemoryError: Java heap space` 异常。
