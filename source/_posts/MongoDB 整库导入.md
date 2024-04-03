---
 title: "MongoDB 整库导入"
 date: 2022-02-24
 tags: [MongoDB]
 categories: [前端笔记]
---

「这是我参与2022首次更文挑战的第27天，活动详情查看：[2022首次更文挑战](https://juejin.cn/post/7052884569032392740 "https://juejin.cn/post/7052884569032392740")」

背景
--

兄弟部门扔了个文件夹过来，说是mongo导出的部分数据，需要测试一下。

文件夹里大概长这样：

![image.png](../imgs/503f78a59ccd4cb58576e9fc02767ecd.png)

我表示现在的前端可真不好干。。。

启动MongoDB
---------

既然是测试，那么本地使用docker起一个MongoDB是最方便的，编写docker-compose.yml如下：

```yaml
version: '3.1'

services:

  mongo:
    image: mongo
    restart: always
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/
```

启动命令：`docker-compose up -d --no-recreate`

### 查看MongoDB

可以用mongo-express，打开`http://localhost:8081/`

![image.png](../imgs/b14906e8bf9d42adabf68728b85f47cc.png)

或者安装一个MongoDB Compass

![image.png](../imgs/b1b144ee16394b449e0d9cb9952ebcce.png)

将数据库文件传入docker 容器
-----------------

首先查看docker container id，运行`docker container ls`

![image.png](../imgs/fea1fc3b1a6e456d95dc097e7636e292.png) 可以使用docker cp命令，`docker cp from containerId:to`，将文件赋值到目标路径。

### 进入docker container

控制台输入命令： `docker exec -it containerid bash`，进入docker container

![image.png](../imgs/35379bb1ef624db4868454ba2df4a31a.png)

### 检查文件是否已经全部传输完成

进入文件所在目录，通过`ls`命令查看文件是否已经传过来；通过`ls -l | grep "^-" | wc -l`命令检查文件数量是否和源文件夹一直。

使用mongorestore导入数据
------------------

[mongorestore](https://docs.mongodb.com%2Fdatabase-tools%2Fmongorestore%2F "https://docs.mongodb.com/database-tools/mongorestore/")能够创建数据库，并将整个文件夹导入，命令为`mongorestore -d db_name 文件夹目录`。

*   `--db=<database>, -d=<database>`指定将数据还原到哪个数据库，如果不存在则会创建它

如果命令运行成功，会看到输出\* document(s) restored successfully. 0 document(s) failed to restore.\*

通过Compass，可以看到我们导入的新数据库：

![image.png](../imgs/5c1163271f3d47b4a911d8b08e248644.png)

### 常见报错及解决办法

*   运行命令`mongorestore -d db_test ./example`,将example中的文件导入db\_test库，报错_error reading database: (Unauthorized) command listCollections requires authentication_
    
    ![image.png](../imgs/cddfa26a428743cc8389ba3ccf05f2c5.png) 需要增加`--uri`参数:
    
    ```ini
    mongorestore -d db_test ./example --uri="mongodb://username:password@host:port"
    ```
    
*   运行命令`mongorestore -d db_test ./example --uri="mongodb://xxx:xxx@127.0.0.1:27017"`，报错_unable to authenticate using mechanism "SCRAM-SHA-1": (AuthenticationFailed) Authentication failed_
    
    ![image.png](../imgs/124a2ee904c54b6fb2e7f064b8d05946.png) 需要对`--uri`增加`?authSource=admin`
    
    ```ini
    mongorestore -d db_test ./example --uri="mongodb://***:***@host:port/?authSource=admin"
    ```