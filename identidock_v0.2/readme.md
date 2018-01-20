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
