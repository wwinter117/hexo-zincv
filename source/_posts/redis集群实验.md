---
title: redis集群实验
tags:
  - redis
  - 实验
date: 2022-6-11 22:46:49
---

在前一篇博客中，介绍了如何在 `Ubuntu` 上使用 `Docker` 搭建 `Redis 6.0.0` 集群。搭建完成后，接下来我们可以进行一些实验，来对 `Redis` 集群的工作原理和新特性进行一个初步认识。本文将介绍如何通过一些常见的实验来测试和验证 `Redis` 集群的功能，包括数据分片、故障转移、主从同步等等

## 环境准备

在进行以下实验之前，确保已经按照前文的方法成功搭建了一个包含6个节点的`Redis 6.0.0` 集群。
## 实验一：数据分片

### 实验目的

验证 `Redis` 集群如何在不同节点间分片存储数据。

### 实验步骤

1. 使用 `redis-cli` 连接到集群中的某一个节点：

```bash
zhangdongdong:~/ $ docker exec -it redis-1 redis-cli -c -p 6371
```

2. 插入一些键值对，观察它们在不同节点间的分布：

```bash
zhangdongdong:~/ $ docker exec -it redis-1 redis-cli -c -p 6371     
127.0.0.1:6371> SET key1 "value1"
-> Redirected to slot [9189] located at 127.0.0.1:6372
OK
127.0.0.1:6372> SET key2 "value2"
-> Redirected to slot [4998] located at 127.0.0.1:6371
OK
127.0.0.1:6371> SET key3 "value3"
OK
127.0.0.1:6371> SET key4 "value4"
-> Redirected to slot [13120] located at 127.0.0.1:6373
OK
127.0.0.1:6373> SET key5 "value5"
-> Redirected to slot [9057] located at 127.0.0.1:6372
OK
127.0.0.1:6372> CLUSTER KEYSLOT key1
(integer) 9189
127.0.0.1:6372> CLUSTER KEYSLOT key2
(integer) 4998
127.0.0.1:6372> CLUSTER KEYSLOT key3
(integer) 935
127.0.0.1:6372> CLUSTER KEYSLOT key4
(integer) 13120
127.0.0.1:6372> CLUSTER KEYSLOT key5
(integer) 9057
127.0.0.1:6372>
```

+ 设置键值对 `key1`:

`Redis` 计算出 `key1` 应该存储在哈希槽 `9189`。当前连接的是 `127.0.0.1:6371`，但哈希槽 `9189` 归属于节点 `127.0.0.1:6372`，因此 `Redis` 自动将请求重定向到 `127.0.0.1:6372` 节点，并在该节点上存储 `key1`。

+ 设置键值对 `key2`:

`Redis` 计算出 `key2` 应该存储在哈希槽 `4998`，这个哈希槽归属于当前连接的节点 `127.0.0.1:6371`，因此数据直接存储在 `127.0.0.1:6371`，没有进行重定向。

+ 设置键值对 `key3`:

被分配到的哈希槽同样位于节点 `127.0.0.1:6371`，因此数据直接存储在该节点，无需重定向。

+ 设置键值对 `key4`:

`Redis` 计算出 `key4` 应该存储在哈希槽 `13120`，该哈希槽归属于节点 `127.0.0.1:6373`，因此数据被重定向到 `127.0.0.1:6373` 并存储在那里。

+ 设置键值对 `key5`:

`key5` 被分配到的哈希槽 `9057` 位于节点 `127.0.0.1:6372`，因此请求被重定向到该节点并存储数据。

+ 检查哈希槽分配:

使用 `CLUSTER KEYSLOT` 命令可以查询每个键对应的哈希槽编号。通过哈希槽编号，`Redis` 决定每个键值对应该存储在哪个节点上。这展示了 `Redis` 集群如何通过哈希槽将数据均匀分布在不同的节点上。

**总结：**

`Redis` 集群通过哈希槽（共16384个）将数据分配到不同的节点上。每个键被哈希后对应一个哈希槽编号，`Redis` 根据哈希槽编号确定数据的存储节点。这种分片机制保证了集群的横向扩展能力，即使数据量增加，集群也能够有效地分散负载，确保高效运行。

## 实验二：故障转移

### 实验目的

验证 `Redis` 集群的自动故障转移能力，当一个主节点失效时，从节点是否能够自动接管。
### 实验步骤

1. 找到一个主节点，并停止该节点的容器：

```bash
zhangdongdong:~/ $ docker stop redis-1                              
redis-1
```

2. 等待几秒钟后，查看集群状态，验证新的主节点是否已自动选出：

```bash
zhangdongdong:~/ $ docker exec -it redis-2 redis-cli -p 6372 CLUSTER NODES 
d0f53aca9ceefb82026d395e825a39554bc4280f 127.0.0.1:6376@16376 slave 0e2a32d078ecc37ae1126e658953fc790fe04fff 0 1724588629137 6 connected

f0a458466e522417340d66773def7b8ca596a96e 127.0.0.1:6375@16375 slave 23ace6314d5bfc6630ea02c2d86083140bdd48a7 0 1724588628110 5 connected

23ace6314d5bfc6630ea02c2d86083140bdd48a7 127.0.0.1:6372@16372 myself,master - 0 1724588628000 2 connected 5461-10922

0e2a32d078ecc37ae1126e658953fc790fe04fff 127.0.0.1:6373@16373 master - 0 1724588628625 3 connected 10923-16383

2aa62f953d746c7ee36c8c79282dcf79a75375d9 127.0.0.1:6374@16374 master - 0 1724588628110 7 connected 0-5460

eb921b7aad0b10aed38006870d849bc7572cb1fa 127.0.0.1:6371@16371 master,fail - 1724588614566 1724588612000 1 disconnected
```

**详细分析：**

**节点 127.0.0.1:6371 状态**

+ 节点ID：`eb921b7aad0b10aed38006870d849bc7572cb1fa`

+ 状态：`master,fail - 1724588614566 1724588612000 1 disconnected`

+ 分析: 该节点原为主节点，但现在显示为 `fail` 状态，这意味着集群检测到它已失效并断开连接。这是因为在之前的步骤中停止了 `redis-1` 容器。由于这个节点负责哈希槽 `0-5460`，这些槽的数据需要通过集群的故障转移机制进行接管。

**节点 127.0.0.1:6372 状态**

+ 节点ID：`23ace6314d5bfc6630ea02c2d86083140bdd48a7`

+ 状态：`myself,master - 0 1724588628000 2 connected 5461-10922`

+ 分析: 该节点显示为 `myself`,`master`，表示这是当前连接的节点，并且它是主节点，负责哈希槽 `5461-10922`。这个节点在集群中的状态为 `connected`，说明它正常工作。

**节点 127.0.0.1:6373 状态**

+ 节点ID：`0e2a32d078ecc37ae1126e658953fc790fe04fff`

+ 状态：`master - 0 1724588628625 3 connected 10923-16383`

+ 分析: 该节点是集群中的另一个主节点，负责哈希槽 `10923-16383`，状态为 `connected`，表明正常工作。

**节点 127.0.0.1:6374 状态**

+ 节点ID：`2aa62f953d746c7ee36c8c79282dcf79a75375d9`

+ 状态：`master - 0 1724588628110 7 connected 0-5460`

+ 分析: 该节点现在接管了哈希槽 `0-5460`，这部分槽原本属于已失败的 `127.0.0.1:6371`。这说明集群中的故障转移机制已运行，将原 `6371` 的主节点职责转移到 `6374` 以保证集群的正常运行。

**从节点状态**

`127.0.0.1:6375` 和 `127.0.0.1:6376` 均为从节点，它们各自复制主节点的数据，并在集群中保持 `connected` 状态。

**哈希槽 0-5460 由原来的 127.0.0.1:6371 转移到了 127.0.0.1:6374，这就是故障转移机制的体现。**

## 实验三：主从同步

### 实验目的

验证 `Redis` 集群中主从节点的数据同步机制。

### 实验步骤

刚才我们停止了 `redis-1` 容器，现在重新运行

```Bash
zhangdongdong:~/ $ docker start redis-1                             
redis-1
```

1. 找到一个主节点和其对应的从节点。可以通过 `CLUSTER NODES` 命令查看主从关系：

```bash
zhangdongdong:~/ $ docker exec -it redis-1 redis-cli -p 6371 CLUSTER NODES  

f0a458466e522417340d66773def7b8ca596a96e 127.0.0.1:6375@16375 slave 23ace6314d5bfc6630ea02c2d86083140bdd48a7 0 1724589362522 5 connected

23ace6314d5bfc6630ea02c2d86083140bdd48a7 127.0.0.1:6372@16372 master - 0 1724589362523 2 connected 5461-10922

2aa62f953d746c7ee36c8c79282dcf79a75375d9 127.0.0.1:6374@16374 master - 0 1724589363040 7 connected 0-5460

d0f53aca9ceefb82026d395e825a39554bc4280f 127.0.0.1:6376@16376 slave 0e2a32d078ecc37ae1126e658953fc790fe04fff 0 1724589363552 6 connected

eb921b7aad0b10aed38006870d849bc7572cb1fa 127.0.0.1:6371@16371 myself,slave 2aa62f953d746c7ee36c8c79282dcf79a75375d9 0 1724589362000 1 connected

0e2a32d078ecc37ae1126e658953fc790fe04fff 127.0.0.1:6373@16373 master - 0 1724589361495 3 connected 10923-16383
```

总结以上主从关系:

`master` 主节点：`6372` `6373` `6374` 

`slave` 从节点：`6371` `6375` `6376`

2. 在主节点上 `6372` 写入数据：

```bash
zhangdongdong:~/ $ docker exec -it redis-2 redis-cli -p 6372 SET mykey "myvalue"                                                    
(error) MOVED 14687 127.0.0.1:6373
zhangdongdong:~/ $ 
```

发现返回错误：`(error) MOVED 14687 127.0.0.1:6373`，说明当前 `key` `被映射到6373` 节点，所以我们可以换到 `6373` 进行写入

```Bash
zhangdongdong:~/ $ docker exec -it redis-3 redis-cli -p 6373 SET mykey "myvalue"                                                    
OK
zhangdongdong:~/ $ docker exec -it redis-3 redis-cli -p 6373 GET mykey                                                              
"myvalue"
zhangdongdong:~/ $    
```

成功

3. 在对应的从节点上检查数据是否已经同步：

```bash
zhangdongdong:~/ $ docker exec -it redis-6 redis-cli -p 6376        
127.0.0.1:6376> KEYS *
1) "mykey"
2) "key4"
127.0.0.1:6376> 
```

可以看见从节点 `redis-6` 已经有了在主节点 `redis-2` 添加的 `key4`