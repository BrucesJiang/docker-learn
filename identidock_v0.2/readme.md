```shell
$ cd identidock_v0.2
$ docker build -t identidock .
$ docker run identidock whoami
$ docker run -p 9090:9090 -p 9191:9191 identidock
$ curl localhost:9090
$ curl localhost:9191
```
在浏览器访问
http://localhost:9090 结果
http://localhpst:9191 查看服务器状态


绑定挂载，修改可以直接体现在项目中。

===================================================
第二步：净化Dockfile, 使用配置文件和辅助脚本

简单的项目，将所有的东西都放在Dockfile内。但是随着应用程序的增长，最好需要将它的内容移动到辅助文件
或脚本文件。例如这里将uWSGI的配置移动到一个.ini文件中。将pip的依赖关系移动到requirements.txt文件中

```shell
$ chmod +x cmd.sh
$ docker build -t identidock .
$ docker run -e "ENV=DEV" -p 5000:5000 identidock
```

=======================================================
第三步: 通过Compose实现自动化

Docker Compose（http://docs.docker.com/compose/）旨在建立和运行Docker开发环境。大体上，
它使用YAML文件来存储不同容器的配置，节省开发者重复且容易出错的输入，以及避免了自行开发解决方案的负担。
Compose将使我们免于自己维护用于服务编排的脚本，包括启动、连接、更新和停止容器。

```shell
identidock:
  build: .
  ports:
    - "5000:5000"
  environment:
    ENV: DEV
  volumes:
    - ./app:/app
```

1. 第一行声明构建容器的名称。一个YAML文件可以定义多个容器(在Compose的术语中称为服务)
2. 这里的build关键字告诉Compose,这个容器的镜像是通过当前目录(.)下的Dockerfile构建。每个容器的定义必须
包括一个build关键字或image关键字。image关键字的值用于启动容器的镜像的标签或ID与Docker run 命令的参数相同
3. ports关键字相当于docker run命令的-p参数，用于声明对外开放的端口。这里我们把容器的5000端口映射到主机的5000
端口。列出端口时可以不带引号，但是最好避免这么做。因为当遇到像56:56这样的值的时候，YAML会把它解析为以60为基数六十
进制数字。
4. environment关键字相当于docker run命令的-e参数，用来设置容器的环境变量。为了运行用于开发的Flask Web服务器，
这里我们将ENV设置成DEV。
5. volumes关键字可以在Compose的YAML文件中设置，通常它们都与Docker run的参数有一对一的对应关系

```shell
$ docker-compose up
```

============================================================
第三步：创建网页，实现identcon

```shell
$ docker build -t identidock .
$ docker run --name dnmonster amouat/dnmonster:1.0
$ docker run -p 5000:5000 -e "ENV=DEV" --link dnmonster:dnmonster identidock
