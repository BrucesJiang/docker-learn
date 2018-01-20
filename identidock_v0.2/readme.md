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
