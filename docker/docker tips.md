Docker Tips
===========

*本文亦载于[Gevin的博客](http://blog.igevin.info/2015/02/05/docker-tips/)*

##镜像

###获取镜像
可以使用 `docker pull` 命令来从仓库获取所需要的镜像

    e.g. sudo docker pull ubuntu:12.04

###列出本地镜像

    sudo docker images


###修改已有镜像来创建新镜像

```
sudo docker commit -m "change source list" -a "gevin" 0b2616b0e5a8 gevin/ubuntu:v1
```

-m 来指定提交的说明信息，跟我们使用的版本控制工具一样；-a 可以指定更新的用户信息；之后是用来创建镜像的容器的 ID；最后指定目标镜像的仓库名和 tag 信息。创建成功后会返回这个镜像的 ID 信息

###载入镜像

使用 `docker load` 从导出的本地文件中再导入到本地镜像库，例如

```
$ sudo docker load --input ubuntu_14.04.tar
或
$ sudo docker load < ubuntu_14.04.tar
```

###导出镜像

使用 `docker save` 导出镜像到本地，如：

```
docker save mynewimage_id > /tmp/mynewimage.tar
```
详细的导出方法如下：

The current (and correct) behavior is as follows:

1.
```
docker save repo
```
Saves all tagged images + parents in the repo, and creates a repositories file listing the tags

2.
```
docker save repo:tag
```
Saves tagged image + parents in repo, and creates a repositories file listing the tag

3.
```
docker save imageid
```
Saves image + parents, does not create repositories file. The save relates to the image only, and tags are left out by design and left as an exercise for the user to populate based on their own naming convention.

###移除本地镜像

如果要移除本地的镜像，可以使用 `docker rmi` 命令。注意 docker rm 命令是移除容器。
```
$ sudo docker rmi training/sinatra
```    
##容器

###启动容器

####新建并启动

所需要的命令主要为 `docker run`
交互式的容器:
```bash
$ sudo docker run -t -i ubuntu:14.04 /bin/bash
root@af8bae53bdd3:/#
```
`-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开

####守护态运行

让 Docker 容器在后台以守护态（Daemonized）形式运行。此时，可以通过添加 -d 参数来实现。

例如下面的命令会在后台运行容器。

```
$ sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
1e5535038e285177d5214659a068137486f96ee5c2e85a4ac52dc83f2ebe4147
```

####启动已终止容器

可以利用 docker start 命令，直接将一个已经终止的容器启动运行。

###终止容器

可以使用 `docker stop` 来终止一个运行中的容器

###查看容器信息

通过 `docker ps` 命令来查看容器信息。

```
sudo docker ps
or
sudo docker ps -a
```

###获取容器的输出信息

要获取容器的输出信息，可以通过 `docker logs` 命令

```
$ sudo docker logs container_name
hello world
hello world
hello world
. . .
```

###进入容器

在使用 -d 参数时，容器启动后会进入后台。 某些时候需要进入容器进行操作，有很多种方法，包括使用 docker attach 命令或 nsenter 工具等。

####attach 命令

docker attach 是Docker自带的命令。下面示例如何使用该命令。
```
$ sudo docker run -idt ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED         STATUS         PORTS   NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago  Up 17 seconds          nostalgi_hypatia
$sudo docker attach nostalgic_hypatia
root@243c32535da7:/#
```
但是使用 attach 命令有时候并不方便。当多个窗口同时 attach 到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时,其他窗口也无法执行操作了。

####nsenter 命令参考[这里](http://yeasy.gitbooks.io/docker_practice/content/container/enter.html)

###导出容器

如果要导出本地某个容器，可以使用 docker export 命令。
```
$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:14.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ sudo docker export 7691a814370e > ubuntu.tar
```
这样将导出容器快照到本地文件。

###导入容器快照

可以使用 docker import 从容器快照文件中再导入为镜像，例如
```bash
$ cat ubuntu.tar | sudo docker import - test/buntu:v1.0
$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
test/ubuntu         v1.0                9d37a6082e97        About a minute ago   171.3 MB
```
此外，也可以通过指定 URL 或者某个目录来导入，例如
```
$sudo docker import http://example.com/exampleimage.tgz example/imagerepo
```

