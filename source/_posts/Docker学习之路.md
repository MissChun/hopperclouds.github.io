title: Docker学习之路
author: liudong
date: 2016-10-10 20:00:00
# 类别
categories:
  - 技术
# 标签
tags:
  - 运维
---
作者：liudong

## Docker简介
### Docker是什么？
Docker 是一个开源项目,Go 语言实现,遵从了 Apache 2.0 协议，项目代码在 GitHub 上进行维护。Docker 项目的目标是实现轻量级的操作系统虚拟化解决方案。Docker 的基础是 Linux 容器（LXC）等技术。

下面的图片比较了 Docker 和传统虚拟化方式的不同之处，可见容器是在操作系统层面上实现虚拟化，直接复用本地主机的操作系统，而传统方式则是在硬件层面实现。
![2016-10-10-截图](http://img.pinbot.me:8080/uploads/2016/10/10/blob_1476068288967.png "blob_1476068288967.png")
![2016-10-10-截图](http://img.pinbot.me:8080/uploads/2016/10/10/blob_1476068344578.png "blob_1476068344578.png")

<!--more-->

### 为什么要使用 Docker？
首先，Docker 容器的启动可以在秒级实现，这相比传统的虚拟机方式要快得多。 其次，Docker 对系统资源的利用率很高，一台主机上可以同时运行数千个 Docker 容器。
容器除了运行其中应用外，基本不消耗额外的系统资源，使得应用的性能很高，同时系统的开销尽量小。传统虚拟机方式运行 10 个不同的应用就要起 10 个虚拟机，而Docker 只需要启动 10 个隔离的应用即可。

##### 更快速的交付和部署
开发者可以使用一个标准的镜像来构建一套开发容器，开发完成之后，运维人员可以直接使用这个容器来部署代码。 Docker 可以快速创建容器，快速迭代应用程序，并让整个过程全程可见，使团队中的其他成员更容易理解应用程序是如何创建和工作的。 Docker 容器很轻很快！容器的启动时间是秒级的，大量地节约开发、测试、部署的时间。
##### 更高效的虚拟化
Docker 容器的运行不需要额外的 hypervisor 支持，它是内核级的虚拟化，因此可以实现更高的性能和效率。
##### 更轻松的迁移和扩展
Docker 容器几乎可以在任意的平台上运行，包括物理机、虚拟机、公有云、私有云、个人电脑、服务器等。 这种兼容性可以让用户把一个应用程序从一个平台直接迁移到另外一个。
##### 更简单的管理
使用 Docker，只需要小小的修改，就可以替代以往大量的更新工作。所有的修改都以增量的方式被分发和更新，从而实现自动化并且高效的管理。

##### 对比传统虚拟机总结
|特性	       |容器		       |虚拟机	      |
|------------|----------------|------------|
|启动			|秒级		|分钟级		|
|硬盘使用		|一般为 MB		|一般为 GB		|
|性能		|接近原生		|弱于		|
|系统支持量		|单机支持上千个容器		|一般几十个		|

### Docker能做什么？
Docker可以解决虚拟机能够解决的问题，同时也能够解决虚拟机由于资源要求过高而无法解决的问题。Docker能处理的事情包括：
隔离应用依赖
创建应用镜像并进行复制
创建容易分发的即启即用的应用
允许实例简单、快速地扩展
测试应用并随后销毁它们

Docker背后的想法是创建软件程序可移植的轻量容器，让其可以在任何安装了Docker的机器上运行，而不用关心底层操作系统
基本概念

## Docker 镜像 （Image）
镜像原理：Docker的镜像类似虚拟机的快照，但更轻量，非常非常轻量。Docker 使用 Union FS 将这些不同的层结合到一个镜像中去。
通常 Union FS 有两个用途, 一方面可以实现不借助 LVM、RAID 将多个 disk 挂到同一个目录下,另一个更常用的就是将一个只读的分支和一个可写的分支联合在一起，Live CD 正是基于此方法可以允许在镜像不变的基础上允许用户在其上进行一些写操作；
创建Docker镜像有几种方式，多数是在一个现有镜像基础上创建新镜像，因为几乎你需要的任何东西都有了公共镜像，包括所有主流Linux发行版，你应该不会找不到你需要的镜像。不过，就算你想从头构建一个镜像，也有好几种方法。
要创建一个镜像，你可以拿一个镜像，对它进行修改来创建它的子镜像。实现前述目的的方式有两种：在一个文件中指定一个基础镜像及需要完成的修改；或通过“运行”一个镜像，对其进行修改并提交。不同方式各有优点，不过一般会使用文件来指定所做的变化。
Docker 镜像（Image）就是一个只读的模板，可以用来创建 Docker 容器。

简单命令  （Ubuntu系统）
安装Docker
```
$ sudo apt-get update
$ wget -qO- https://get.docker.com/ | sh
```
注：系统会提示你输入sudo密码，输入完成之后，就会下载脚本并且安装Docker及依赖包。

Docker命令工具需要root权限才能工作。你可以将你的用户放入docker组来避免每次都要使用sudo。
```
$ sudo docker pull ubuntu:latest
```
列出docker镜像
```
$ sudo docker images
```
上传镜像
```
$ sudo docker push ouruser/sinatra
```
保存镜像
```
$ sudo docker save -o ubuntu_14.04.tar ubuntu:14.04
```
加载镜像
```
$ sudo docker load --input ubuntu_14.04.tar   # 或者 sudo docker load < ubuntu_14.04.tar
```
删除镜像
```
sudo docker rmi training/sinatra
注：在删除镜像之前要先用 docker rm 删掉依赖于这个镜像的所有容器.
```
清理所有未打过标签的本地镜像
```
$ sudo docker rmi $(docker images -q -f "dangling=true")  #sudo docker rmi $(docker images --quiet --filter "dangling=true")
```
Dockerfile创建镜像
```
$ vim Dockerfile
# This is a comment
FROM ubuntu:14.04
MAINTAINER Docker Newbee <newbee@docker.com>
RUN apt-get -qq update
RUN apt-get -qqy install ruby ruby-dev
RUN gem install sinatra

$ sudo docker build -t="ouruser/sinatra:v2" .

注：其中 -t 标记来添加 tag，指定新的镜像的用户信息。 “.” 是 Dockerfile 所在的路径（当前目录），也可以替换为一个具体的 Dockerfile 的路径。

$ sudo docker run -t -i ouruser/sinatra:v2 /bin/bash
```
从本地文件系统导入
```
$ sudo cat ubuntu-14.04-x86_64-minimal.tar.gz  |docker import - ubuntu:14.04
```
### Docker 容器（Container）
Docker 利用容器（Container）来运行应用。

容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

*注：镜像是只读的，容器在启动的时候创建一层可写层作为最上层。

##### 新建并后台启动容器
```
$ sudo docker run -tid ubuntu /bin/bash
注：-t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， -i 则让容器的标准输入保持打开，-d 让容器进入后台运行.
$ sudo docker ps
CONTAINER ID        IMAGE                      COMMAND                  CREATED             STATUS                   PORTS                     NAMES
b548b2d4a537        ubuntu                     "/bin/bash"              11 seconds ago      Up 10 seconds                                   zen_engelbart
```
docker run 来创建容器时，Docker 在后台运行的标准操作包括：
检查本地是否存在指定的镜像，不存在就从公有仓库下载
利用镜像创建并启动一个容器
分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
从地址池配置一个 ip 地址给容器
执行用户指定的应用程序
执行完毕后容器被终止




查看docker容器
```
sudo docker ps
```
启动已经停止的容器
```
$ sudo docker start ubuntu:14.04
```
docker自带命令进入容器
```
$ sudo docker attach zen_engelbart #zen_engelbart是容器名当退出容器后，容器会关闭
$ docker exec -it zen_engelbart /bin/bash    #进入已经开启的容器，退出后容器能继续运行
```
第三方工具进入容器
```
$ cd /tmp; curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz | tar -zxf-; cd util-linux-2.24;
$ ./configure --without-ncurses
$ make nsenter && sudo cp nsenter /usr/local/bin
```
容器的第一个进程的 PID，可以通过下面的命令获取
```
$ PID=$(docker inspect --format "{{ .State.Pid }}" <container>)
$ nsenter --target $PID --mount --uts --ipc --net --pid
```
实例演示
```
$ sudo docker run -idt ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
$ sudo docker ps
CONTAINER ID    IMAGE        COMMAND          CREATED            STATUS          PORTS             NAMES
243c32535da7  ubuntu:latest "/bin/bash"    18 seconds ago     Up 17 seconds                  nostalgic_hypatia
$ PID=$(docker-pid 243c32535da7)
10981
$ sudo nsenter --target 10981 --mount --uts --ipc --net --pid
root@243c32535da7:/#
```
```
简单的方法是：下载 .bashrc_docker，并将内容放到 .bashrc 中。
$ wget -P ~ https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker;
$ echo "[ -f ~/.bashrc_docker ] && . ~/.bashrc_docker" >> ~/.bashrc; source ~/.bashrc

$ echo $(docker-pid <container>)
$ docker-enter <container> ls
```
获取容器日志
```
$ sudo docker logs ubuntu:14.04
```
导出容器
```
$ sudo docker ps -a  
CONTAINER ID   IMAGE       COMMAND           CREATED       STATUS     PORTS          NAMES
9c365aaa875f   mysql  "docker-entrypoint.sh"   9 days ago   Exited 8 minutes ago   0.0.0.0:3308->3306/tcp   mysql_3308
$ sudo docker export 9c365aaa875f > mysql.tar
```
导入容器快照
```
$ cat mysql.tar | sudo docker import - test/mysql:5.6
$sudo docker import http://example.com/exampleimage.tgz example/imagerepo
```


### Docker 仓库（Repository）

仓库（Repository）是集中存放镜像文件的场所。有时候会把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。

仓库分为公开仓库（Public）和私有仓库（Private）两种形式。

最大的公开仓库是 Docker Hub，存放了数量庞大的镜像供用户下载。
*注：Docker 仓库的概念跟 Git 类似，注册服务器可以理解为 GitHub 这样的托管服务。


### Dockerfile使用
指令

指令的一般格式为 INSTRUCTION arguments，指令包括 FROM、MAINTAINER、RUN 等。

##### FROM

格式为 FROM <image>或FROM <image>:<tag>。

第一条指令必须为 FROM 指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个 FROM 指令（每个镜像一次）。

##### MAINTAINER

格式为 MAINTAINER <name>，指定维护者信息。

##### RUN

格式为 RUN <command> 或 RUN ["executable", "param1", "param2"]。

前者将在 shell 终端中运行命令，即 /bin/sh -c；后者则使用 exec 执行。指定使用其它终端可以通过第二种方式实现，例如 RUN ["/bin/bash", "-c", "echo hello"]。

每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。

##### CMD

支持三种格式

CMD ["executable","param1","param2"] 使用 exec 执行，推荐方式；
CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用；
CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数；
指定启动容器时执行的命令，每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。

如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。

##### EXPOSE

格式为 EXPOSE <port> [<port>...]。

告诉 Docker 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 -P，Docker 主机会自动分配一个端口转发到指定的端口。

##### ENV

格式为 ENV <key> <value>。 指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持。

例如

ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
ADD

格式为 ADD <src> <dest>。

该命令将复制指定的 <src> 到容器中的 <dest>。 其中 <src> 可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）。

##### COPY

格式为 COPY <src> <dest>。

复制本地主机的 <src>（为 Dockerfile 所在目录的相对路径）到容器中的 <dest>。

当使用本地目录为源目录时，推荐使用 COPY。

##### ENTRYPOINT

两种格式：

ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2（shell中执行）。
配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。

每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。

##### VOLUME

格式为 VOLUME ["/data"]。

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

##### USER

格式为 USER daemon。

指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。

当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如：RUN groupadd -r postgres && useradd -r -g postgres postgres。要临时获取管理员权限可以使用 gosu，而不推荐 sudo。

##### WORKDIR

格式为 WORKDIR /path/to/workdir。

为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。

可以使用多个 WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如

WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
则最终路径为 /a/b/c。

##### ONBUILD

格式为 ONBUILD [INSTRUCTION]。

配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

例如，Dockerfile 使用如下的内容创建了镜像 image-A。

[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
如果基于 image-A 创建新的镜像时，新的Dockerfile中使用 FROM image-A指定基础镜像时，会自动执行 ONBUILD 指令内容，等价于在后面添加了两条指令。

##### FROM image-A
```
#Automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src
注：使用 ONBUILD 指令的镜像，推荐在标签中注明
```

### 实例演示(dockerfile创建镜像，运行Django+uwsgi+nginx+supervisor)
启动mysql容器
```
sudo docker run -d -e MYSQL_ROOT_PASSWORD=pinbot@123 --name mysql_3308 -v /data/mysql/data:/var/lib/mysql -p 3308:3306 mysql
注：用mysql镜像后台启动容器，并设置root用户初始密码为谁pinbot123，挂载本地目录/data/mysql/data到容器mysql_3308 的/var/lib/mysql目录，映射本地3308端口到容器的3306端口
```
用Dockerfile创建镜像
```
sudo docker build -t talentbi:1.0 .
注：根据Dockerfile创建镜像，并命名为talentbi:1.0；
```
后台启动容器
```
sudo docker run -d -p 8001:8080 -v /home/bigdata/github/TalentMiner/:/home/bigdata/github/TalentMiner --name talentbi1.0 talentbi:1.0
注：用talentbi:1.0镜像启动容器并后台运行，映射本地端口8001到容器内8080端口，挂载本地目录等
```
进入容器
```
sudo docker exec -ti talentbi1.1 /bin/bash
```
端口映射
```
sudo iptables -t nat -A  DOCKER -p tcp --dport 8080 -j DNAT --to-destination 172.17.0.3:8080
```
查看iptables列表
```
sudo iptables -t nat -nL
```
