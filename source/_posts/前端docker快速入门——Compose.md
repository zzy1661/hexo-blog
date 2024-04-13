---
 title: "前端docker快速入门——Compose"
 date: 2022-03-06
 tags: [Docker]
 categories: [前端笔记]
---

前言
==

前文介绍了Dockerfile文件，但在实际应用中还有一个常见的文件——docker-compose.yml，本文便介绍Compose的使用方法。

Compose
=======

Compose定位是“定义和运行多个Docker容器的应用”。

docker可以很方便地起一个应用容器，但是我们项目实践中往往需要多个应用。比如一个web项目，可能需要启动nginx,redis,mysql等多个应用容器，它们之间可能还需要相互配合。Compose可以让用户通过编写一个简单的模板文件，快速地创建和管理基于Docker容器的应用集群。

服务和项目
-----

Compose中有两个重要的概念：服务（service）和项目（project）。

*   服务：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
*   项目：由一组关联的应用容器组成的一个完整业务单元。

模板文件
----

Compose默认的模板文件名是docker-compose.yml。下面是docker-compose.yml文件的一个例子：

```yml
version: "3"

services:
  webapp:
    image: examples/web
    ports:
      - "80:80"
    volumes:
      - "/data"
```

其中webapp是服务名，image、ports、volumes都是指令关键字。

### 常用指令

Compose模板文件中常用指令如下：

*   **image和build** 一个服务必须指定镜像，可以用image指定，也可以用build指定Dockerfile所在的文件目录
*   **command** 覆盖容器启动后默认执行的命令。
*   **depends\_on** 指定容器启动顺序
*   **env\_file和environment** 指定环境变量，env\_file的值是.env文件，将从文件中读取；environment则直接设置环境变量。

模板文件默认支持读取系统环境变量和当前目录下的.env文件中的变量，模板中可以用`${}`占位。

```arduino
image: "mongo:${MONGO_VERSION}"
```

*   **ports** 将端口映射到主机
*   **volumes** 设置数据卷挂载路径

Compose 命令
----------

Compose中的命令只需要记住一个：**up**。  
`docker-compose up [options] [SERVICE...]`  
它将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作。大部分时候都可以直接通过该命令来启动一个项目。

常用的选项有：

*   `-d` 在后台运行服务容器
*   `--no-recreate` 如果容器已经存在了，则不重新创建

实战
--

项目需要启动mongo、redis、rabbitmq三个应用，编写docker-compose.yml如下：

```yaml
version: '3.1'
networks:
  xxx:

services:

  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: xxx
      MONGO_INITDB_ROOT_PASSWORD: xxx
    networks:
      - xxx

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: xxx
      ME_CONFIG_MONGODB_ADMINPASSWORD: xxx
      ME_CONFIG_MONGODB_URL: mongodb://xxx:xxx@mongo:27017/
    networks:
      - xxx

  redis:
    image: redis:alpine
    ports:
      - 6379:6379
    command: redis-server
    networks:
      - xxx

  rabbitmq:
    image: rabbitmq:3.8.3-management
    container_name: rabbitmq
    restart: always
    hostname: myRabbitmq
    ports:
      - 15672:15672
      - 5672:5672
    environment:
      - RABBITMQ_DEFAULT_USER=xxx
      - RABBITMQ_DEFAULT_PASS=xxx
    networks:
      - xxx
```

*   构建并启动 执行`docker-compose up -d --no-recreate`

![image.png](../imgs/4f79340e5802405c8528233799680046.png)

*   查看容器 执行`docker-compose ps` ![image.png](../imgs/a777431b0212407d999884280b44aaa1.png)
*   删除容器 可以使用`stop`命令停止运行中的容器，然后再运行`rm`命令。但还有更简单的办法：`down`，它能停止并删除`up`命令创建的容器。

![image.png](../imgs/01a207b620f34eaea21cfea282c3f4ba.png)