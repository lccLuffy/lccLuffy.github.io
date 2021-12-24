---
layout: post
permalink: /code-snippets/
title: 常用代码片段
date: 2017-10-10T12:00:00
lang: zh-CN
---

### Ubuntu 安装最新 Nginx：
```bash
sudo -s
nginx=stable # use nginx=development for latest development version
add-apt-repository ppa:nginx/$nginx
apt-get update
apt-get install nginx
```

### 查看 Mysql 各数据表大小：
```sql
SELECT table_schema                                        "DB Name", 
   Round(Sum(data_length + index_length) / 1024 / 1024, 1) "DB Size in MB" 
FROM   information_schema.tables 
GROUP  BY table_schema; 
```

### 安装55：
```bash
sudo apt-get update
sudo apt-get install python3-pip
sudo pip3 install shadowsocks
sudo mkdir /var/ss
sudo vim /var/ss/ss.json
ssserver -c /var/ss/ss.json -d start
```

```json
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"mypassword",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

### Useful
```bash
# NVIDIA GPUs 
nvidia-smi

# folder size. -h: human readable, -s: for summary
du -hs /path/to/directory
# 查看 CUDA 版本
cat /usr/local/cuda/version.txt
# 查看 cuDNN 版本
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
```

### 解压
```bash
tar -cvf myfolder.tar myfolder
tar -xf archive.tar -C /target/directory
tar -xvzf archive.tar.gz -C /target/directory
```

### 文件
```bash
# 查看当前文件夹文件数量
find . -type f | wc -l
# 查询文件行数
wc -l a.txt
# 查询文件单词个数
wc -w a.txt
# 输出整个文件，并在每行前面添加行号
cat -n a.txt 
# 查看磁盘空间及目录容量
df -hl
du -sh [目录名] 返回该目录的大小
```

### SSH
```bash
# 将远程服务器 <remote-ip> 的 127.0.0.1:6006 端口转发到本地 16006 端口。即本地输入
# localhost:16006 实际上访问的是远程服务器的 127.0.0.1:6006。
ssh -L 16006:127.0.0.1:6006 <username>@<remote-ip> -p <port>

ssh -N -f -L localhost:16006:localhost:6006 <user@remote>
-N : no remote commands
-f : put ssh in the background
-L <machine1>:<portA>:<machine2>:<portB> : forward <machine2>:<portB> (remote scope) to <machine1>:<portA> (local scope)
```

### Copy & Sync
```bash
# -a will keep permissions,etc, and -h will be human readable. 
# --progress2 which shows the overall percentage
rsync -ah --info=progress2 source destination
```

### 安装 pyenv
```bash
sudo apt-get install --no-install-recommends make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
git clone https://github.com/pyenv/pyenv.git ~/.pyenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.zshrc
exec "$SHELL"
pyenv install 3.6.5
pyenv global 3.6.5
exec "$SHELL"
```

### 安装 cocoapi
```bash
git clone https://github.com/cocodataset/cocoapi.git
cd cocoapi/PythonAPI
python setup.py build_ext install
```
