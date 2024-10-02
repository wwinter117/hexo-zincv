---
title: jvm-对象的内存布局
tags:
  - jvm
date: 2022-8-12 22:46:49
---

阅读《深入理解java虚拟机》第三版，结合`openjdk-17u` 中的`hotspo`t源码简单记录一下。

### 1. 对象的内存布局

在JVM中，每个Java对象在内存中的布局通常分为三个部分：

#### 1.1 对象头 (Object Header)

 **Mark Word**
 
 它是对象头中的一部分，用于存储对象的运行时信息，包括对象的哈希码（`hashcode`）、`GC` 状态（是否被标记为垃圾回收）、锁标志位（用于同步锁的状态）、偏向锁的线程 `ID`、偏向时间戳等。在不同的情况下，`Mark Word` 的内容会有所不同。以下是来自`openjdk-17u`的`hotspot`源码中关于 `Mark Word` 部分的注释

`src/hotspot/share/oops/markWord.cpp: `

```c++
// The markWord describes the header of an object.  
//  
// Bit-format of an object header (most significant first, big endian layout below):  
//  
//  32 bits:  
//  --------  
//             hash:25 ------------>| age:4    biased_lock:1 lock:2 (normal object)  
//             JavaThread*:23 epoch:2 age:4    biased_lock:1 lock:2 (biased object)  
//  
//  64 bits:  
//  --------  
//  unused:25 hash:31 -->| unused_gap:1   age:4    biased_lock:1 lock:2 (normal object)  
//  JavaThread*:54 epoch:2 unused_gap:1   age:4    biased_lock:1 lock:2 (biased object)  
//  
//  - hash contains the identity hash value: largest value is  
//    31 bits, see os::random().  Also, 64-bit vm's require  
//    a hash value no bigger than 32 bits because they will not  
//    properly generate a mask larger than that: see library_call.cpp  
//  
//  - the biased lock pattern is used to bias a lock toward a given  
//    thread. When this pattern is set in the low three bits, the lock  
//    is either biased toward a given thread or "anonymously" biased,  
//    indicating that it is possible for it to be biased. When the  
//    lock is biased toward a given thread, locking and unlocking can  
//    be performed by that thread without using atomic operations.  
//    When a lock's bias is revoked, it reverts back to the normal  
//    locking scheme described below.  
//  
//    Note that we are overloading the meaning of the "unlocked" state  
//    of the header. Because we steal a bit from the age we can  
//    guarantee that the bias pattern will never be seen for a truly  
//    unlocked object.  
//  
//    Note also that the biased state contains the age bits normally  
//    contained in the object header. Large increases in scavenge  
//    times were seen when these bits were absent and an arbitrary age  
//    assigned to all biased objects, because they tended to consume a  
//    significant fraction of the eden semispaces and were not  
//    promoted promptly, causing an increase in the amount of copying  
//    performed. The runtime system aligns all JavaThread* pointers to  
//    a very large value (currently 128 bytes (32bVM) or 256 bytes (64bVM))  
//    to make room for the age bits & the epoch bits (used in support of  
//    biased locking).  
//  
//    [JavaThread* | epoch | age | 1 | 01]       lock is biased toward given thread  
//    [0           | epoch | age | 1 | 01]       lock is anonymously biased  
//  
//  - the two lock bits are used to describe three states: locked/unlocked and monitor.  
//  
//    [ptr             | 00]  locked             ptr points to real header on stack  
//    [header      | 0 | 01]  unlocked           regular object header  
//    [ptr             | 10]  monitor            inflated lock (header is wapped out)  
//    [ptr             | 11]  marked             used to mark an object  
//    [0 ............ 0| 00]  inflating          inflation in progress  
//  
//    We assume that stack/thread pointers have the lowest two bits cleared.  
//  
//  - INFLATING() is a distinguished markword value of all zeros that is  
//    used when inflating an existing stack-lock into an ObjectMonitor.  
//    See below for is_being_inflated() and INFLATING().
```

 **Class Pointer**（类型指针）
 
 指向对象所属类的 `Class` 对象。`JVM` 通过这个指针确定对象的类型信息。

 **数组长度（仅对数组对象有效）**
 
 如果是数组对象，头部还会存储数组的长度。

#### 1.2 实例数据 (Instance Data)

这是对象实际存储成员变量（`field`）的部分。这个区域包含了所有的实例字段，按声明顺序排列。`JVM` 可能会对这些字段进行对齐优化。

- **基本类型字段**：如`int`、`float`、`boolean`等，这些字段按类型和声明顺序存储。
    
- **引用类型字段**：如对象引用，这些字段存储的是指向实际对象的引用（通常是指针）。
    

#### 1.3 对齐填充 (Padding)

`JVM` 为了保证对象大小是 `8` 字节的倍数，可能会在对象的末尾添加一些填充字节（`padding`）。这是为了满足硬件的内存对齐要求，提升访问效率。
