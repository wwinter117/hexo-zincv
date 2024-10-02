---
title: HotSpot字节码解释器片段
tags:
  - jdk
  - hotspot
date: 2022-9-28 22:46:49
---
![](https://zincv.oss-cn-hangzhou.aliyuncs.com/images/jvm-3af4ecb7bbe2376bffb9225160a6a0dd.jpg)

以下代码片段是 `JVM` 中对象分配的核心逻辑，来自《深入理解java虚拟机》第三版p49。

```c++
// 确保常量池中的类已经被解析
if (!constants->tag_at(index).is_unresolved_Klass()) {
    // 获取常量池中的Klass对象
    oop entry = (Klassoop) *constants->obj_at_addr(index);
    
    // 断言该对象是一个已解析的Klass
    assert(entry->is_klass(), "Should be resolved klass");
    klassOop k_entry = (klassOop) entry;
    
    // 断言该类是一个实例类（instanceKlass）
    assert(k_entry->klass_part()->oop_is_instance(), "Should be instanceKlass");
    instanceKlass* ik = (instanceKlass*) k_entry->klass_part();

    // 确保类已经初始化并且可以快速分配
    if (ik->is_initialized() && ik->can_be_fastpath_allocated()) {
        // 计算对象大小
        size_t obj_size = ik->size_helper();
        oop result = NULL;
        
        // 判断是否需要将内存区域置零
        bool need_zero = !ZeroTLAB;

        // 尝试在TLAB中分配内存
        if (UseTLAB) {
            result = (oop) THREAD->tlab()->allocate(obj_size);
        }

        // 如果在TLAB分配失败，则尝试直接在Eden中分配
        if (result == NULL) {
            need_zero = true;
retry:
            HeapWord* compare_to = *Universe::heap()->top_addr();
            HeapWord* new_top = compare_to + obj_size;

            // 使用CAS操作安全地更新堆顶指针
            if (new_top <= *Universe::heap()->end_addr()) {
                if (Atomic::cmpxchg_ptr(new_top, Universe::heap()->top_addr(), compare_to) != compare_to) {
                    // 如果CAS失败，重试分配
                    goto retry;
                }

                result = (oop) compare_to;

                // 如果需要，将对象内存区域置零
                if (result != NULL && need_zero) {
                    HeapWord* to_zero = (HeapWord*) result + sizeof(oopDesc) / oopSize;
                    obj_size -= sizeof(oopDesc) / oopSize;
                    if (obj_size > 0) {
                        memset(to_zero, 0, obj_size * HeapWordSize);
                    }
                }
            }
        }

        // 根据是否启用偏向锁，设置对象头信息
        if (UseBiasedLocking) {
            result->set_mark(ik->prototype_header());
        } else {
            result->set_mark(markOopDesc::prototype());
            result->set_klass_gap(0);
        }
        result->set_klass(k_entry);

        // 将对象引用压入栈中，继续执行下一条字节码指令
        SET_STACK_OBJECT(result, 0);
        UPDATE_PC_AND_TOS_AND_CONTINUE(3, 1);
    }
}
```

### 代码简化和解释

#### 1. 确保常量池中存放的是已解析的类

```cpp
if (!constants->tag_at(index).is_unresolved_Klass()) {
    oop entry = (Klassoop) *constants->obj_at_addr(index);
    assert(entry->is_klass(), "Should be resolved klass");
    klassOop k_entry = (klassOop) entry;
    assert(k_entry->klass_part()->oop_is_instance(), "Should be instanceKlass");
    instanceKlass* ik = (instanceKlass*) k_entry->klass_part();
}
```

- 这部分代码首先检查常量池中的一个条目，确保它是一个已经解析过的类（即`Klass` 对象）。
- `assert` 语句用于验证这个条目是否确实是一个 `klassOop` （代表类的元数据对象）以及它是否是一个实例类（`instanceKlass`）。
- `instanceKlass` 是 `HotSpot` 中表示类元数据的一个重要结构，它包含类的字段、方法等信息。

#### 2. 确保对象所属类型已经经过初始化阶段

```cpp
if (ik->is_initialized() && ik->can_be_fastpath_allocated()) {
    size_t obj_size = ik->size_helper();
    oop result = NULL;
    bool need_zero = !ZeroTLAB;
```

- 在继续对象分配之前，确保这个类已经完成初始化阶段（即其静态初始化块已经执行）。
- `size_helper()` 计算对象的大小。
- `need_zero` 用于决定是否需要将对象的内存区域置为零。

#### 3. 在TLAB（Thread-Local Allocation Buffer）中分配对象

```cpp
if (UseTLAB) {
    result = (oop) THREAD->tlab()->allocate(obj_size);
    if (result == NULL) {
        need_zero = true;
    }
```

- 如果启用了 `TLAB`（线程本地分配缓存），则尝试在 `TLAB` 中分配对象。如果分配失败（`result == NULL`），将 `need_zero` 设置为 `true`，以确保稍后分配时内存区域会被清零。

#### 4. 直接在Eden空间分配对象

```cpp
HeapWord* compare_to = *Universe::heap()->top_addr();
HeapWord* new_top = compare_to + obj_size;
if (new_top <= *Universe::heap()->end_addr()) {
    if (Atomic::cmpxchg_ptr(new_top, Universe::heap()->top_addr(), compare_to) != compare_to) {
        goto retry;
    }
    result = (oop) compare_to;
    if (result != NULL) {
        if (need_zero) {
            HeapWord* to_zero = (HeapWord*) result + sizeof(oopDesc) / oopSize;
            obj_size -= sizeof(oopDesc) / oopSize;
            if (obj_size > 0) {
                memset(to_zero, 0, obj_size * HeapWordSize);
            }
        }
    }
}
```

- 如果在 `TLAB` 分配失败，就尝试在 `Eden` 空间分配对象。
- 通过 `CAS`（Compare-And-Swap，比较并交换）操作确保在多线程环境下安全地更新堆顶指针。若 `CAS` 失败，则通过`goto retry`重新尝试。
- 如果分配成功且 `need_zero` 为 `true`，则使用 `memset` 函数将对象的内存置零。

#### 5. 设置对象头信息

```cpp
if (UseBiasedLocking) {
    result->set_mark(ik->prototype_header());
} else {
    result->set_mark(markOopDesc::prototype());
    result->set_klass_gap(0);
}
result->set_klass(k_entry);
```

- 根据是否启用了偏向锁（`Biased Locking`）来设置对象头信息。
- 设置完对象头信息后，将对象的类指针（`klass`）设置为刚才解析的 `k_entry`。

#### 6. 将对象引用入栈，继续执行下一条指令

```cpp
SET_STACK_OBJECT(result, 0);
UPDATE_PC_AND_TOS_AND_CONTINUE(3, 1);
```

- 将新分配的对象引用压入Java栈中，并更新程序计数器（PC）和操作数栈顶指针，继续执行后续的字节码指令。
