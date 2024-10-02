---
title: docker加速方法总结
tags:
  - docker
date: 2022-4-26 22:46:49
---
![](https://zincv.oss-cn-hangzhou.aliyuncs.com/images/docker-12bb9bcf3a20e5eb0255e561a8bacba6.jpeg)

有的时候我们在使用 `docker` 下载镜像时遇到速度很慢的情况。主要原因是 `Docker Hub` 的服务器位于国外，网络传输速度较慢。本文将总结一下网络上有的常见几种解决方案，帮助你加速 `Docker` 镜像的下载速度。

## 1. 使用国内镜像源

国内很多公司提供了 `Docker` 镜像加速服务，可以通过配置 `Docker` 的 `daemon.json` 文件来使用这些加速器。

### 配置步骤：

1. 打开或创建 `/etc/docker/daemon.json` 文件

```Bash
root@wwinter-00:~# cd /etc/docker/
root@wwinter-00:/etc/docker# ls
root@wwinter-00:/etc/docker# 
```

这里看到我的服务器中没有这个 `daemon.json` 文件，没关系，可使用 `vim` 来创建并添加以下内容

```Bash
root@wwinter-00:/etc/docker# vi daemon.json
root@wwinter-00:/etc/docker# 
root@wwinter-00:/etc/docker# cat daemon.json 
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
```
  
2. 保存文件并重启 `Docker` 服务：

```bash
root@wwinter-00:/etc/docker# systemctl daemon-reload
root@wwinter-00:/etc/docker# systemctl restart docker
```

## 2. 使用代理

如果你有 `vpn` 服务可以网络环境支持代理，可以配置 `HTTP` 或 `HTTPS` 代理来加速 `Docker Pull` 的速度。
### 配置步骤：

1. 在 `Docker` 的配置文件 `/etc/systemd/system/docker.service.d/http-proxy.conf` 中添加代理配置：

```
    [Service]
    Environment="HTTP_PROXY=http://your-proxy.com:port"
    Environment="HTTPS_PROXY=https://your-proxy.com:port"
```

2. 保存并重启 Docker 服务：

```bash
    systemctl daemon-reload
    systemctl restart docker
 ```

## 3. 检查网络连接

有时 `Docker Pull` 慢是由于本地网络问题，比如 `DNS` 配置问题、路由问题等。可以通过以下方法排查：

- 检查本地网络的稳定性。
- 使用 `ping` 或 `traceroute` 命令测试与 `Docker Hub` 的连接状况。

通过以上方法，可以有效加速 `Docker` 镜像的下载速度。