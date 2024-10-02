---
title: 基于riscv的小型操作系统
tags:
  - riscv
  - 操作系统
  - 汇编
date: 2023-4-2 22:46:49
---
![](https://zincv.oss-cn-hangzhou.aliyuncs.com/images/riscv-7630a5e65eaa20a4f697de7b40bba24c.jpeg)

git@github.com:wwinter117/myrvos.git

# myrvos

一个运行在risc-v指令集架构之上的小型操作系统

## 环境依赖

- RISC-V工具链 (`riscv64-unknown-elf-gcc`, `riscv64-unknown-elf-objcopy`, `riscv64-unknown-elf-objdump`)
- QEMU系统模拟器 (`qemu-system-riscv32`)
- GDB多架构版本 (`gdb-multiarch`)

确保这些工具已经安装并且在系统路径中。

## 构建项目

使用以下命令来构建整个项目：

```shell
make all
```

这将会编译所有源代码，并将生成的文件放置在build目录下。

## 运行项目

一旦构建完成，可以使用以下命令来通过QEMU运行操作系统：

```shell
make run
```

这条命令将启动QEMU模拟器并加载操作系统。可以通过QEMU的控制台来与操作系统交互。

## 调试项目

如果需要调试操作系统，可以使用以下命令：

```shell
make debug
```

这会启动QEMU模拟器并等待GDB的连接。然后，可以使用GDB连接到QEMU，并开始调试。

## 查看汇编代码

如果想要查看操作系统的汇编代码，可以运行：

```shell
make code
```

这会使用objdump工具来反汇编操作系统，并通过less命令分页显示。

## 清理项目

要删除所有构建生成的文件，运行：

```shell
make clean
```

这会删除build目录下所有的文件，清理项目环境。