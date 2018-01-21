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
```
===================================================================
第四步， Compose自动化实现

```shell
identidock:
  build: .
  ports:
    - "5000:5000"
  environment:
    ENV: DEV
  volumes:
    - ./app:/app
  links:
    - dnmonster
dnmonster:
  image: amouat/dnmonster:1.0
```

```shell
$ docker-compose up
```
=====================================================================
第五步： 实现缓存功能
通过引入redis模块实现缓存功能。

============================================================

第六步： 添加测试

```python
import unittest
import identidock

class TestCase(unittest.TestCase):

    def setUp(self):
        identidock.app.config["TESTING"] = True
        self.app = identidock.app.test_client()

    def test_get_mainpage(self):
        page = self.app.post("/", data=dict(name="Moby Dock"))
        assert page.status_code == 200
        assert 'Hello' in str(page.data)
        assert 'Moby Dock' in str(page.data)

    def test_html_escaping(self):
        page = self.app.post("/", data=dict(name='"><b>TEST</b><!--'))
        assert '<b>' not in str(page.data)

if __name__ == '__main__':
    unittest.main()
```

简单的测试文件，其中共有三个方法：
1. setUP 基于Flask Web应用，初始化一个它的测试脚本
2. test_get_mainpage 该测试方法以"Moby Dock"作为name字段输入并调用URL /。然后，它会检查
方法是否返回200状态码，以及返回的数据是否包含"Hello"和"Moby Dock"字符串。
3. test_html_escaping 测试输入中的HTML元素能否被正确转义。

```shell
#测试命令
$ docker build -t identidock .
$ docker run identidock python test.py
````
输出结果展示，第一个测试通过，第二个测试失败。因为我们没有将用户的输入正确转义。这个一个严重的安全问题，
在大型应用中可能导致数据泄漏和跨站脚本攻击(XSS)。例如，测试输入name = "> <b>pwned!</b><!--",必须包含
引号。攻击者可能向应用程序注入恶意的JavaScript代码，并诱使用户使用它。

修复代码：
```python
name = html.escape(request.form['name'], quote=True) #净化输入
```
执行测试代码

修改cmd.sh测试脚本，使测试能够自动化执行。

```shell
#!/bin/bash
set -e
if [ "$ENV" = 'DEV' ];then
  echo "Running Development Server"
  exec python "identidock.py"
else if [ "$ENV" = 'UNIT' ]; then
  echo "Running Unit Tests"
  exec python "test.py"
else
  echo "Running Production Server"
  exec uwsgi --http 0.0.0.0:9090 --wsgi-file /app/identidock.py \
  --callable app --stats 0.0.0.0:9191
fi
```
```shell
# 执行测试脚本
$ docker build -t identidock .
$ docker run -e "ENV=UNIT" identidock
```
还可以写更多的测试方法进行测试。为了利用单元测试来验证get_identicon方法，需要调用dnmonster和redis
服务的测试版本，可以通过`替身测试(test double)`。替身测试替代了真正提供服务的程序部分，通常
只是一个测试`桩(stub)`，返回一个固定答案，或者是一个`模拟对象(mock)`,它只能够以编写它的时候的预期来
调用。更多替身测试相关信息Python模拟模块(http://docs.python.org/3/library/unittest.mock.html)
以及专门的HTTP工具，如Pact(http://github.com/realestate-com-au/pact),Mountebnank(http://www.mbtest.org/)
和Mirage(https://mirage.readthedocs.org)


下一步是将测试放到持续集成服务器中自动运行，这样的话，当源代码提交到源码控制服务器的时候，源码测试便会自动被
执行，而这都是在源码进入准生产和生产环境前完成。

=======================================================================================

第七步：创建Jenkins容器，进行代码的自动化持续提交测试。
