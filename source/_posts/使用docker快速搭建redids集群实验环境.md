---
title: 使用docker快速搭建redids集群实验环境
tags:
  - redis
  - docker
date: 2022-5-26 22:46:49
---
![](https://zincv.oss-cn-hangzhou.aliyuncs.com/images/docker-12bb9bcf3a20e5eb0255e561a8bacba6.jpeg)

`Redis` 是一个开源的内存数据结构存储系统，广泛应用于缓存、会话管理、排行榜、计数器等场景。为了深入了解和实验 `Redis` 的集群模式，使用 `Docker` 来搭建 `Redis` 集群是一个非常方便的方法。本文将介绍如何在 `Ubuntu` 上使用 `Docker` 快速搭建一个 `Redis` 集群。

## 步骤一：拉取Redis镜像

首先拉取最新的 `Redis` 镜像：

```bash
docker pull redis:latest
```

这将下载官方的 `Redis` 镜像，后续的容器都将基于这个镜像创建。

## 步骤二：创建Redis节点容器

为了搭建一个 `Redis` 集群，我们需要至少6个 `Redis` 节点（3个主节点和3个从节点）。使用以下命令创建这些容器：

```bash
for i in `seq 1 6`; do   docker run -d --name redis-$i --net host redis redis-server --appendonly yes --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --cluster-announce-ip 127.0.0.1 --port 637$i; done
```

以上命令将启动6个 `Redis` 实例，并将它们分别绑定到 `6371` 到 `6376` 端口。

- `--net host` 选项使容器使用宿主机的网络，这是为了简化集群配置时的网络问题。
- `--cluster-enabled yes` 启用了 `Redis` 集群模式。
- `--cluster-announce-ip` 配置了集群中每个节点的 `IP` 地址，在这里我们简单地使用了本地回环地址 `127.0.0.1`。

## 步骤三：配置Redis集群

启动完所有 `Redis` 节点后，可以通过以下命令来配置并启动 `Redis` 集群：

```bash
docker exec -it redis-1 redis-cli --cluster create 127.0.0.1:6371 127.0.0.1:6372 127.0.0.1:6373 127.0.0.1:6374 127.0.0.1:6375 127.0.0.1:6376 --cluster-replicas 1
```

- `--cluster-replicas 1` 指定每个主节点有一个从节点。
- 该命令将自动选择节点作为主节点或从节点，并形成完整的 `Redis` 集群。

在执行此命令后，会看到 `Redis` 要求确认配置，输入 `yes` 并按回车即可完成集群的创建。

## 步骤四：验证集群状态

创建完集群后，可以通过以下命令验证集群的状态：

```bash
docker exec -it redis-1 redis-cli -p 6371 cluster nodes
```

该命令会显示每个节点的角色（主或从）、连接信息等。如果所有节点的状态均为 `connected`，则说明集群已经成功搭建。

## 步骤五：实验与测试

现在可以在这个集群上进行各种Redis集群模式下的实验，例如：

- 数据分片
- 故障转移
- 主从同步
- 节点扩展

可以通过 `Redis` 的 `CLI` 工具来操作集群中的数据，观察其在不同节点间的分布与容错机制。
