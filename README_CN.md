# Parker

轻量级打包工具与容器运行时，一条指令将工作目录打包成可执行文件，并以轻量级容器的方式直接运行在目标机器上。


适用于 linux 系统，示例图

![](https://raw.githubusercontent.com/weiwenhao/pictures/main/blogs20230915104318.png)

示例是一个 c 语言编写的 ip 解析服务 `gcc -o ipservice`，其依赖 ipdb 资源文件。

使用 parker 将可执行文件 ipservice 和其依赖的 assert 压缩并打包成新的可执行文件 ipserviced。

在目标机器上运行 ipserviced 将会生成一个轻量的容器环境来运行原始的 ipservice 服务。

## 安装

从 [github releases](https://github.com/weiwenhao/parker/releases) 中下载并解压 Parker 安装包。推荐将解压后的 parker 文件夹移动到 `/usr/local/` 下，并将 `/usr/local/parker/bin` 目录加入到系统环境变量。

```
> parker --version
1.0.1
```

## 使用

cd 到工作目录，执行 `parker target` 该命令将 target 连同当前的工作目录一起打包成一个 targetd 可执行文件，将可执行文件放到目标机器上运行即可。

```
> cd :workdir && parker :target
```

#### 示例

上面的可执行文件+资源文件打包是**标准使用**示例，当然也有一些非标准的使用方式，比如以一个 python3.11 编写的 server 为例子

```
> tree .
├── bar.png
├── foo.txt
├── python # cp /usr/bin/python3.11 ./
└── server.py

0 directories, 4 files
```

server.py 内容如下

```python
from http.server import SimpleHTTPRequestHandler, HTTPServer

def run():
    print("listen on http://127.0.0.1:8000")
    server_address = ('127.0.0.1', 8000)
    httpd = HTTPServer(server_address, SimpleHTTPRequestHandler)
    httpd.serve_forever()

run()
```

cd 到工作目录中执行 `parser python`，你将会得到一个 `pythond` 文件，这就是打包好的可执行文件，将其上传到目标机器中执行即可。

```
------------------------------------------------------------------------ Stage 1
> tree .
.
└── pythond

0 directories, 1 file 

------------------------------------------------------------------------ Stage 2
> ./pythond server.py
listen on http://127.0.0.1:8000

------------------------------------------------------------------------ Stage 3
> curl 127.0.0.1:8000
...
<li><a href="bar.png">bar.png</a></li>
<li><a href="foo.txt">foo.txt</a></li>
<li><a href="python">python</a></li>
<li><a href="server.py">server.py</a></li>
...
```


现在只需要将 python 换成 pythond, 其他的启动参数都不需要改变。 pythond 会将参数传递给 python 进程。

> ❗️ parker 不解决 python 的动态库依赖问题。

## 运行说明

pythond 是 parker 构建的轻量级容器运行时，并且 pythond 是一个静态编译的可执行文件。其在执行时通过 linux namespace 创建了一个隔离环境，并解压工作目录运行目标 python 。

pythond 将监听 python 主进程的运行，一旦 python 进程停止或异常，pythond 会通过 cgroup 清理容器环境，并清理 python 的所有子进程。

用户像 pythond 中传递的所有参数和信号都会原封不动的传递给 python 进程。

## 环境依赖


容器运行时依赖 cgroup 和 namespace，需要 linux 内核版本大于 2.6.24。并且正确挂载了 cgroup。检查文件 `/sys/fs/cgroup/cgroup.controllers` 或 `/sys/fs/cgroup/freezer` 中任意一个目录存在即可。

测试环境: ubuntu:22 / ubuntu:20

## 源码构建

源代码由编程语言 [nature](https://github.com/nature-lang/nature) 开发，nature 编译器版本需要 >= 0.4.0。安装完成后在源码目录执行 `make amd64 && make install` 即可安装到 /usr/local/parker 目录下。

> nature 目前主要支持 amd64 构建，nature 构建的可执行文件体积更小，效率更高。如果需要构建等其他架构, 主仓库提供了 golang 版本实现。

## Thinks

[nature](https://github.com/nature-lang/nature) 是下一代系统级编程语言，将携手 c 一起进行高性能且高效的开发工作。

nature 社区可用版本即将发布，现在也可以先行体验并提供改进意见。并邀您一起进行标准库贡献，所有的标准库贡献都会合并至主仓库。

邀您加入 nature 编程语言交流群，添加微信号 `nature-lang`

## License

This project is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).

Copyright (c) 2020-2023 WEIWENHAO, all rights reserved.
