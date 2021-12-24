---
layout: post
permalink: /docker-basic-commandline/
title: Docker 基本命令
date: 2020-07-17T21:47:00
lang: zh-CN
---

### 下载镜像
```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
# eg:
docker pull nvcr.io/nvidia/pytorch:20.06-py3
```

### 启动镜像
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
# eg:
docker run --gpus all -ti -v /:/data --ipc=host -p 8000:8000 --name lufficc nvcr.io/nvidia/pytorch:20.03-py3
```

- `-it` 交互模式运行
- `--rm` 容器退出时自动删除此容器
- `-v` 绑定磁盘，`/:/data` 即将容器下 `/data` 目录映射到主服务器的 `/` 目录
- `--ipc` IPC mode 
- `-p` 映射端口
- `--name` 容器名称

### 启动容器
```bash
docker start [OPTIONS] CONTAINER [CONTAINER...]
# eg:
docker start lufficc
```

### 停止容器
```bash
docker stop [OPTIONS] CONTAINER [CONTAINER...]
# eg:
docker stop lufficc
```
### 显示所有容器
```bash
docker container ls -a
```

### 进入容器
```bash
docker exec -it CONTAINER bash
# 如果使用 zsh
docker exec -it CONTAINER zsh
```

### 删除容器
```bash
docker rm CONTAINER
```

### 删除镜像
```bash
docker rmi IMAGE
```

### 参考
- [Docker command line](https://docs.docker.com/engine/reference/commandline/cli/)
- [NVIDIA NGC](https://ngc.nvidia.com/catalog/containers/nvidia:pytorch)

