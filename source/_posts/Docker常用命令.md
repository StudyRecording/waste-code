---
title: Docker常用命令
tags:
  - Docker
categories:
  - 运维与部署
excerpt: docker的简单使用, 包括基本的启动停止、 查看日志等
thumbnail: https://t.mwm.moe/ycy
cover: https://t.mwm.moe/pc
sticky: 2
date: 2022-08-13 05:06:20
---

# Docker常用使用命令
![头像](Docker常用命令/avatar.jpg)
## 基本命令
- `docker version` docker版本
- `docker info` docker信息
- `docker --help` docker命令帮助
## 容器命令
- 根据镜像新建并启动容器`docker run -d -p {宿主机端口}:{容器端口} -v {宿主机文件路径}:{容器文件路径} --name {容器名别名} {镜像名}:{镜像版本}`
  - `-d`: 后台运行
  - `-p`：端口映射
  - `-v`：文件路径映射
  - `--name` ：容器命名
- 列出当前所有正在运行的容器`docker ps`
- 列出所有容器`docker ps -a`
- 启动容器`docker start {容器id/容器名}`
- 重新启动容器`docker restart {容器id/容器名}`
- 停止容器`docker stop {容器id/容器名}`
- 强制停止容器`docker kill {容器id/容器名}`
- 删除容器`docker rm {容器id/容器名}`
- 强制删除容器`docker rm -f {容器id/容器名}`
- 查看容器日志`docker logs -f -t --since --tail {容器id/容器名}`
  - eg : `docker logs -f -t --since="2022-02-28" --tail=10 redis`
  - `-f`: 实时查看日志
  - `-t` : 显示日志时间
  - `--since="2022-02-28"` ： 只输出2022-02-28及其之后的日志
  - `--tail=10`：查看最后10条日志
- 查看容器内运行的进程`docker top {容器id/容器名}`
- 进入到容器内`docker exec -it {容器id} bash`
- 将容器内文件拷贝到宿主机`docker cp {容器id}:{容器内文件路径} {宿主机文件路径}`
  - eg：`docker cp 2c003a469ae3:/usr/local/etc/redis/redis.conf /Users/hpc/DockerFileSystem/redis/conf/`

## 镜像命令
- 查看镜像`docker images`
- 列出本地所有镜像`docker images -a`
- 拉取镜像`docker pull {镜像名}:{镜像版本}`
- 删除镜像`docker rmi {镜像名}`
- 强制删除镜像`docker rmi -f {镜像名/镜像id}`
