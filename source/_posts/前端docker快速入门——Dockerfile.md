---
 title: "前端docker快速入门——Dockerfile"
 date: 
 tags: [Docker]
 categories: 
---

前言
--

前文介绍了docker中最最常用的几个命令，但漏了一个build命令。build构建镜像通常需要编写一个Dockerfile的文件，这个文件上手也很简单，只要记住几个常用的指令就行了。

docker镜像的分层存储
-------------

这里需要先补充一个关于镜像概念：分层存储。

前文将镜像类比为一个装机文件，但实际上镜像并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。

> 联合文件系统（UnionFS）是一种轻量级的高性能分层文件系统，它支持将文件系统中的修改信息作为一次提交，并层层叠加，同时可以将不同目录挂载到同一个虚拟文件系统下，应用看到的是挂载的最终结果。联合文件系统是实现Docker镜像的技术基础。

分层能有哪些好处呢？这里可以举两个好理解的例子。比如镜像升级的时候，会创建一个新层，而不会改动之前的文件，那么其他人更新镜像的时候，只需要添加新层，而不必替换原来的整个镜像。另外，当我们拉取镜像的时候，是先拿到该镜像的配置文件，从中读取镜像layer的信息，并且只会拉取本地没有的layer，因为很多镜像都基于同样的基础镜像，这种设计就提高了镜像拉取速度和存储效率。

Dockerfile
----------

### 介绍

Dockerfile是用来定制镜像的，镜像的定制实际上就是定制每一层所添加的配置、文件。

Dockerfile以指令记录每一层的修改、安装、构建、操作,每一条指令构建一层。

比如我们构建一个nginx镜像，并自定义它web页面，那么Dockerfile可能是这样的：

```docker
FROM nginx
COPY ./index.html /usr/share/nginx/html
```

其中`FROM`、`COPY`就是指令。

### 指令

*   **FROM** 指定所创建镜像的基础镜像，如果本地不存在，则默认会去Docker Hub下载指定镜像。Dockerfile中的第一条指令必须为FROM指令.
*   **RUN** 运行指定命令。格式为RUN或RUN \["executable", "param1", "param2"\]（因为会被解析问json数组，所以必须是双引号）。  
    每条RUN指令将在当前镜像的基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用\\来换行。
*   **CMD** CMD指令用来指定启动容器时默认执行的命令。每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。 `CMD ["executable", "param1", "param2"]`使用exec执行；`CMD command param1 param2`在/bin/sh中执行，提供给需要交互的应用;`CMD ["param1", "param2"]`提供给ENTRYPOINT的默认参数。
*   **LABELLABEL** 用来指定生成镜像的元数据标签信息。格式为`LABEL <key>=<value> <key>=<value> <key>=<value> ...`。
*   **EXPOSE** 声明镜像内服务所监听的端口。格式为`EXPOSE <port> [<port>...]`。但仅是声明，启动容器并不会自动完成端口映射。
*   **ENV** 指定环境变量，在镜像生成过程中会被后续RUN指令使用，在镜像启动的容器中也会存在。格式为`ENV<key><value>`或`ENV<key>=<value>...`。
*   **ADD** 该命令将复制指定的路径下的内容到容器中的路径下。格式为`ADD<src> <dest>`。 src可以是Dockerfile所在目录的一个相对路径，或者一个URL。如果是tar文件，会自动解压缩。
*   **COPY** ADD的简化版，不支持URL,也不会自动解压缩，docker官方更推荐COPY。格式为`COPY <src> <dest>`。
*   **ENTRYPOINT** 指定镜像的默认入口命令，该入口命令会在启动容器时作为根命令执行，所有传入值作为该命令的参数。 每个Dockerfile中只能有一个ENTRYPOINT。格式为`ENTRYPOINT ["executable", "param1", "param2"]`（exec调用执行）或`ENTRYPOINT command param1 param2`（shell中执行）。
*   VOLUME 创建一个数据卷挂载点。格式为`VOLUME ["/data"]`。

docker容器删除后，其运行期间产生的数据也会被删除。而数据卷就是解决这一问题的常用手段。

_数据卷是一个可供一个或多个容器使用的特殊目录，数据卷的中的更新会立即生效，它的主要用途是持久化数据。_

*   **WORKDIR** 为后续的RUN、CMD和ENTRYPOINT指令配置工作目录。格式为`WORKDIR /path/to/workdir`。

下面的指令不太常用：

*   **USER** 指定运行容器时的用户名或UID
*   **MAINTAINER** 指定维护者信息，格式为MAINTAINER
*   **ARG** 指定一些镜像内使用的参数（例如版本号信息等），这些参数在执行docker build命令时可以以--build-arg=格式传入。格式为`ARG<name>[=<default value>]`。
*   **ONBUILD** 配置当所创建的镜像作为其他镜像的基础镜像时，所执行的创建操作指令。
*   **STOPSIGNAL** 指定所创建镜像启动的容器接收退出的信号值
*   **HEALTHCHECK** 配置所启动容器如何进行健康检查。
*   **SHELL** 指定其他命令使用shell时的默认shell类型。默认值为\["/bin/sh", "-c"\]。

### docker build

编写完成Dockerfile之后，可以通过docker build命令来创建镜像。

```perl
docker build -t my-nginx .
```

常用的参数有：

*   **\--tag, -t:**  镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。
*   **\-f :** 指定要使用的Dockerfile路径；
*   **\-build-arg=\[\] :** 设置镜像创建时的变量；

`docker build` 命令最后有一个 `.`,`.` 表示当前目录，也可以改成其他路径。这个路径并非用来指定Dockerfile的位置，而是命令执行的上下文，ADD和COPY命令只能操作这个上下文中的文件。

docker build的构建过程发生在服务端（docker引擎），因此会将该上下文中的所有文件全部打包传给docker 引擎，如果这些文件中有一部分不需要，那么可以编写.dockerignore忽略这些文件，就类似.gitignore一样。