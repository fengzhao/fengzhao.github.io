---
title: Docker-compose简明教程
date: 2019-01-03 23:00:49
tags: docker 
---


### 简介

docker-compose 是一个定义和运行多个 docker 容器实例的工具。它使用一个 docker-compose.yml（yaml格式）来定义一组关联的容器应用为一个项目（porject）。通过一个简单的命令，可以创建启动yml中配置的多个容器。它可以在所有的环境中使用，包括生产，开发，测试，持续集成环境。

<!-- more -->

docker-compose中有两个重要概念：

- 服务（service）：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

- 项目（project）：由一组关联的应用容器组成的一个完整业务单元。

可以看出，一个项目可以由多个服务（容器）关联而成，Compose 面向项目进行管理。

docker-compose有很多命令，用于容器应用的整个生命周期：

- 启动，停止，重建服务。
- 查看运行中服务的状态。
- 流式输出运行中服务的日志。
- 对服务运行的命令。

### 安装

一般在 Linux 中直接下载二进制文件即可：

```shell
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

$ docker-compose --version
docker-compose version 1.22.0, build f46880fe

```

### 开始使用docker-compose

```python
#创建文件夹 dockertest ，在该目录中编写app.py，内容为

#!/usr/bin/env python
# -*- coding: utf-8 -*-

import time
import redis
from flask import Flask


app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
    
    
    
#编写 Dockerfile 文件，内容为

FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install redis flask
CMD ["python", "app.py"]


#编写 docker-compose.yml 文件，内容为


version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"

```



docker-compose.yml 文件中定义两个服务：web 和redis。

web：使用了Dockerfile中构建的镜像，把容器中的5000端口映射到宿主机的8300端口。

redis：使用了从 Docker Hub registry 中拉取的官方 redis 镜像。

### 使用说明

docker-compose的基本使用格式如下：

```shell
Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose --help
  docker-compose command --help
```

#### 选项说明

- `-f  FILE` 	指定使用的 compose 模板文件（显示指定文件路径），默认为当前目录下的 docker-compose.yml ，如果没有则报错。
- `-p NAME` 指定项目名称，默认将使用所在目录名称作为项目名。
- `--verbose`  显示详细输出。
- `--log-level LEVEL`  设置日志级别 (DEBUG, INFO, WARNING, ERROR, CRITICAL)。
- `-v, --version`  打印版本并退出。

#### 命令说明

##### build

构建或重建项目中的服务容器。

```shell
Usage: build [options] [--build-arg key=val...] [SERVICE...]
```

服务容器一旦构建后，将会带上一个标记名，例如对于 web 项目中的一个 db 容器，可能是 web_db。
可以随时在项目目录下运行 `docker-compose build` 来重新构建服务。

选项包括：

- `--force-rm` 删除构建过程中的临时容器。
- `--no-cache`  构建镜像过程中不使用 cache（这将加长构建过程）。
- `--pull` 始终尝试通过 pull 来获取更新版本的镜像。
- `-m, --memory MEM ` 为创建的容器设置内存限制

