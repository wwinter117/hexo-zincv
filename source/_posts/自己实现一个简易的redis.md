---
title: 自己实现一个简易的redis
tags:
  - c语言
  - redis
date: 2023-7-12 22:46:49
---
![](https://zincv.oss-cn-hangzhou.aliyuncs.com/images/redis-5fb76d3ec9fd434fe46a579d9c1c8a83.jpeg)

项目地址：https://github.com/wwinter117/mini-redis.git

不定期更新中，目前实现了：

+ 实现了RESP通信协议
+ 支持字符串数据结构
+ 支持RDB持久化

### 编译

```sh
zhangdongdong:mini-redis/ (main✗) $ make clean                                   
rm -f mini-redis-server
zhangdongdong:mini-redis/ (main✗) $ make                                         
gcc -Wall -W -pedantic -std=c99 -o mini-redis-server main.c

```
### 启动

```sh
zhangdongdong:mini-redis/ (main✗) $ ./mini-redis-server                          
[Mini-Redis] is starting...
[Mini-Redis] RDB Version: 1.0.0
[Mini-Redis] start recovery
[Mini-Redis] found database: 0
found: hi-hu
found: zdd-ji
found: bh-asd
found: huh-datf
found: kk1-hiu
[Mini-Redis] found database: 1
found: kj1-huib
found: kj2-hub
[Mini-Redis] found database: 2
found: qw-scd
found: hu-daus
[Mini-Redis] done
[Mini-Redis] started
[Mini-Redis] Waiting for connections...

```
### 连接

因为实现了`RESP`通信协议，因此可以直接使用`redis`官方客户端工具`redis-cli`进行连接使用

```sh
zhangdongdong:github/ $ redis-cli
127.0.0.1:6379>
```

### 使用

```sh
**zhangdongdong**:**github/** $ redis-cli                                       
127.0.0.1:6379> SET hello world
OK
127.0.0.1:6379> GET hello
"world"
127.0.0.1:6379>
```


# mini-redis  
  
简易版本的redis(Redis 2.0)  
  
## C-S通信协议  
  
> 详细介绍：[Redis-RESP 官方文档](https://redis.io/docs/latest/develop/reference/protocol-spec/)  
  
RESP（REdis Serialization Protocol）是Redis使用的一种序列化协议，用于客户端和服务器之间的通信。RESP协议设计简单高效，支持以下几种类型的数据：  
  
+ 简单字符串（Simple Strings）  
+ 错误（Errors）  
+ 整数（Integers）  
+ 批量字符串（Bulk Strings）  
+ 数组（Arrays）  
  
### RESP 数据类型格式  
  
**1. 简单字符串（Simple Strings）**  
  
简单字符串以 + 开头，后跟字符串本身，最后以 \r\n 结尾。  
  
简单字符串用于传输短的非二进制字符串，开销最小。例如，许多Redis命令在成功时仅返回“OK”。这个简单字符串的编码为以下5个字节：  
```text  
+OK\r\n  
```  
  
**2. 简单错误（Errors）**  
  
错误消息以 - 开头，后跟错误信息，最后以 \r\n 结尾。  
  
Redis仅在出现问题时返回错误，例如，当尝试对错误的数据类型进行操作或命令不存在时。客户端在收到错误回复时应引发异常  
  
示例：  
```text  
-ERR unknown command 'foobar'\r\n  
```  
  
**3. 整数（Integers）**  
  
整数以 : 开头，后跟整数值，最后以 \r\n 结尾。  
  
示例：  
  
```text  
:1000\r\n  
```  
  
**4. 复杂字符串（Bulk Strings）**  
  
批量字符串以 $ 开头，后跟字符串的长度，最后以 \r\n 结尾。然后是字符串内容，再次以 \r\n 结尾。如果长度为-1，表示空值。  
  
示例：  
  
```text  
$6\r\nfoobar\r\n  
$-1\r\n  
```  
  
**5. 数组（Arrays）**  
  
数组以 * 开头，后跟元素数量，最后以 \r\n 结尾。然后是数组中各元素的 RESP 表示。如果数量为-1，表示空数组。  
```text  
*<元素数量>\r\n<元素1>...<元素n>  
```  
  
示例：  
  
+ 空数组：  
  
```text  
*0\r\n  
```  
  
+ 包含两个大块字符串“hello”和“world”的数组的编码为：  
  
```text  
*2\r\n$5\r\nhello\r\n$5\r\nworld\r\n  
```  
  
+ 一个包含三个整数的数组编码  
  
```text  
*3\r\n:1\r\n:2\r\n:3\r\n  
```  
  
+ 数组可以包含混合数据类型。以下编码表示一个包含四个整数和一个大块字符串的列表：  
  
```text  
*5\r\n  
:1\r\n  
:2\r\n  
:3\r\n  
:4\r\n  
$5\r\n  
hello\r\n  
```  
  
**下表总结了Redis支持的RESP数据类型：**  
  
| RESP数据类型 | 最小协议版本 | 分类  | 第一个字节 |  
|----------|--------|-----|-------|  
| 简单字符串    |    RESP2 |    简单 |   +    |  
| 简单错误     |    RESP2 |    简单 |   -    |  
| 整数       |    RESP2 |    简单 |   :    |  
| 大块字符串    |    RESP2 |    聚合 |   $    |  
| 数组       |    RESP2 |    聚合 |   *    |  
| 空值       |    RESP3 |    简单 |   _    |  
| 布尔值      |    RESP3 |    简单 |   #    |  
| 双精度数     |    RESP3 |    简单 |   ,    |  
| 大数       |    RESP3 |    简单 |   (    |  
| 大块错误     |    RESP3 |    聚合 |   !    |  
| 逐字字符串    |    RESP3 |    聚合 |   =    |  
| 映射       |    RESP3 |    聚合 |   %    |  
| 集合       |    RESP3 |    聚合 |   ~    |  
| 推送       |    RESP3 |    聚合 |   \>   |