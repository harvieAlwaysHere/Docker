[TOC]

## **容器与Docker**

#### **虚拟化技术**

一般指计算虚拟化(computing virtualization)，维基百科定义如下

![1-1-Docker与虚拟机比较](/1-2-虚拟化定义.png)

硬件模拟和传统操作系统软件实现的虚拟化限制过多，基于容器技术的操作系统虚拟化结合了两者的优点，而Docker正是容器技术中的代表

![1-1-Docker与虚拟机比较](/1-3-Docker虚拟化.png)

#### **Linux容器技术**

Linux容器(Linux Containers, LXC)技术，即有效地将由单个操作系统管理的资源划分到孤立的组中，以更好地在孤立的组之间平衡有冲突的资源使用需求

#### **Docker**

在LXC 的基础上，Docker提供了容器管理工具，如分发、版本、移植等，让用户无需关注底层操作，能够简单明了地管理和使用容器

Docker的核心思想是“Build, Ship and Run Any App, Anywhere”，即是通过对应用的封装(Packaging)、分发(Distribution)、部署(Deployment)、运行(Runtime)的生命周期进行管理，为应用的开发、运行、部署提供一站式的解决方案





## **Docker优点**

#### **容器虚拟化**

在云时代，Docker通过容器进行应用打包，解耦应用与运行平台之间的关系，为频繁迁移平台的分布式应用提供快速分发和部署的功能

#### **开发和运维**

* 高效的交付和部署

  Docker的镜像功能可用于快速构建标准开发环境

* 高效的资源利用

  Docker是内核级的虚拟化，性能高，资源需求低

* 轻松的迁移和拓展

  Docker容器的跨平台兼容性

* 简单的更新管理

  Dockerfile简化了先前的配置更新工作

#### **Docker与虚拟机**

![1-1-Docker与虚拟机比较](/1-1-Docker与虚拟机比较.png)





## **Docker概述**

#### **三大核心概念**

* **镜像(Image)**

  创建Docker容器的基础，类似虚拟机镜像

* **容器(Container)**

  运行和隔离应用的实例，类似轻量级沙箱

* **仓库(Repository)**

  存放镜像文件的场所，类似代码仓库

#### **三大官方生态**

* **Docker引擎**

  调用Dokcer容器的核心组件，提供简单安全的容器集群编排和管理功能

* **DockerHub**

  官方提供的托管公有或私有镜像的云仓库

* **DockerCloud**

  官方提供的部署与管理容器的云服务，完整地支持容器化项目

#### **安装Docker引擎(CentOS环境)**

按照Dokcer官方文档的指示安装Docker Engine-Community

一般有三种安装方式，本文采用Repository方式

- 通过Repository仓库在线安装
- 通过下载RPM软件包手动安装，一般用于无法访问互联网的系统
- 通过运行脚本安装，较为便利，但存在权限风险、无法定制等缺点

```shell
# 安装所需软件包
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# 添加Docker稳定版本的yum软件源
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# 安装最新版本的Docker Engine-Community和containerd
$ sudo yum install docker-ce docker-ce-cli containerd.io

# 启动Docker
$ sudo systemctl start docker
# 运行hello-world镜像验证Docker可用性，若可用会打印参考消息
$ sudo docker run hello-world
```

#### **查看与配置Docker服务**

查看Docker服务的配置

```shell
$ docker info
```

有多种方法可以为Docker配置服务和环境变量，大多采用两种

* **配置/etc/docker/daemon.json文件**

  官方推荐，配置文件是平台无关性的，以配置Docker运行工作目录为例

  ```
  # Take From /etc/docker/daemon.json
  {
      "data-root": "/data/docker"
  }
  ```

* **配置/usr/lib/systemd/system/docker.service文件**

  运用systemd进行服务管理的系统可用，如CentOS、RedHat系统，以配置Docker运行工作目录为例

  ```
  # Take From /usr/lib/systemd/system/docker.service
  
  # Old
  ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
  
  # New
  ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --data-root=/data/docker
  ```

完成配置后需重启Docker服务生效配置

```shell
# 刷新更改
$ sudo systemctl daemon-reload
# 重启Docker
$ sudo systemctl restart docker
```





## **Docker镜像**

镜像是一个静态的只读文件，是容器运行的基础

#### **搜索镜像**

```shell
$ docker search KEYWORD
```

#### **获取镜像**

```shell
$ docker pull [Registry/]NAME[:TAG] [-a=] [--disable-content-trust=] [--registry-mirror=]
# Registry
  仓库地址，默认为DockerHub，即regisry.hub.docker.com
# NAME
  镜像仓库名称 
# TAG
  镜像标签(版本)，默认为latest
# -a
  是否获取仓库中所有镜像，默认为false
# --disable-content-trust
  是否取消镜像内容校验，默认为true 
# --registry-mirror
  指定镜像代理服务地址，如https://registry.docker-en.com
```

#### **查看镜像信息**

```shell
# 查看镜像列表 NAME-指定镜像仓库名称
$ docker image ls [NAME]

# 为本地镜像别名
$ docker image tag ubuntu:latest myubuntu:1

# 查看镜像详细信息 NAME-指定镜像仓库名称 -f 指定获取内容
$ docker image inspect NAME [-f {{".Metadata"}}]

# 查看镜像历史信息
$ docker image history NAME
```

#### **删除和清理镜像**

```shell
# 删除镜像 -f 强制删除有容器依赖的镜像
$ docker image rm [NAME:TAG] or [ID] [-f]
Untagged: hellojava:latest
Deleted: sha256:95e7f7f1990184267251b71e729d54f5e0bcfcb53fca3a1e21e007ed901a40f2
Deleted: sha256:d22e82fb1b1f39d3f3d72e58cedc0c7bb1f73356a8d4fd89800fe017aef5d41f
Deleted: sha256:c86daaa2f04bca2f6d695ac80ad0a66bb4931b3afa73e3ed21d7e4ac9a5bb343

# 删除容器 
$ docker container rm [ID]

# 清理镜像，删除临时镜像 -a 删除未使用镜像 -f 强制删除镜像
$ docker image prune [-a] [-f]
```

#### **创建镜像**

* 基于已有容器创建镜像

```shell
# 查看正在运行的容器 记录容器ID(cb879bb17584 )
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND        ...
cb879bb17584        ubuntu:18.04        "/bin/bash"    ...

# 基于已有容器ID创建镜像 -m 提交信息 -a 作者信息 -p 提交时暂停容器运行
$ docker container commit [-m] [-a] [-p] CONTAINER_ID NAME:TAG   
$ docker container commit -m "Test Build Image From Container" -a "Havie" cb879bb17584 test:0.1
sha256:3104701969a5849cd8c63f766c34c2a33bf59923b0a5189d9b3f5e2a71a58845

# 查看镜像
$ docker image ls
REPOSITORY     TAG      IMAGE ID          CREATED             SIZE
test           0.1      3104701969a5      10 seconds ago      64.2MB
```

* 基于Dockerfile创建

  Dockerfile包含指定命令描述基于某个父镜像创建新镜像的过程

```shell
# 创建Dockerfile
$ vim Dockerfile
FROM debian:stretch-slim

LABEL version="1.0" maintainer="docker user <docker_user@github>"

RUN apt-get update && \
	apt-get install -y python3 && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/*
	
# 根据Dockerfile创建镜像
$ docker image build -t python:3 .
...
Successfully built dc9502b41fc7
Successfully tagged python:3

# 查看镜像
$ docker image ls
REPOSITORY     TAG      IMAGE ID          CREATED             SIZE
python         3        dc9502b41fc7      About a minute ago  95.2MB
```

#### **导出与载入镜像**

```shell
# 导出镜像
$ docker image save python:3 -o python_3.tar
# 载入镜像
$ docker image load -i python_3.tar
```

#### **上传镜像**

```shell
# 规范上传镜像名称 Registry_Host[:Port]/NAME[:TAG]
$ docker image tag python:3 ryan333777/python:3
# 上传镜像
$ docker image push ryan333777/python:3
The push refers to repository [docker.io/ryan333777/python]
cbc339bfdf97: Layer already exists 
e0db3ba0aaea: Layer already exists 
3: digest: sha256:0531c29fd33120e55d4f3efa59099ed2e62d7a2a83c38e863185893d0de66083 size: 741
```





## **Docker容器**

容器是独立运行的一组应用和他们的运行环境，是镜像的一个运行实例

#### **运行容器**

```shell
# 根据镜像创建容器
$ docker container create IMAGE_NAME[:TAG] [OPTIONS]
# 查看所有容器(包括未运行的)
$ docker container ps -a
CONTAINER ID   IMAGE      COMMAND    CREATED            STATUS     ...
aa180b5190d4   python:3   "bash"     17 seconds ago     Created    ...

# 启动已创建的容器
$ docker container start CONTAINER_ID

# 根据镜像创建并启动容器
$ docker container run IMAGE_NAME[:TAG] [OPTIONS]

# 查看容器输出信息
$ docker container logs CONTAINER_ID [OPTIONS]
```

其中**create/run命令的[OPTIONS]参数**十分复杂对于设定容器的运行模式尤为重要

* 如**[-it]以互动模式**运行容器
* 如**[-d]以守护进程模式**运行容器

具体命令可参阅官方文档或参考以下表格

![](/容器运行参数1.png)

![](/容器运行参数2.png)

![](/容器运行参数3.png)

#### **停止容器**

```shell
# 暂停运行中的容器
$ docker container pause CONTAINER_ID
# 恢复暂停中的容器
$ docker container unpause CONTAINER_ID

# 终止运行中的容器
$ docker container stop CONTAINER_ID
# 启动终止的容器
$ docker container start CONTAINER_ID
# 清除所有终止状态的容器
$ docker container prune
# 强行终止运行中的容器
$ docker container kill CONTAINER_ID
# 终止运行中的容器并重新启动
$ docker container restart CONTAINER_ID
```

#### **进入容器**

后台运行的容器用户无法看到容器中的信息与操作容器，可进入容器进行查看与操作

```shell
# 进入容器(所有进入的线程同步信息)
$ docker container attach CONTAINER_ID or CONTAINER_NAME
# 进入容器(多线程进入)
$ docker container exec CONTAINER_ID or CONTAINER_NAME
```

#### **删除容器**

```shell
# 删除终止或退出的容器 [-f]强行终止并删除运行中的容器(终止+删除)
$ docker container rm [-f] CONTAINER_ID
```

#### **导出与载入容器**

可用于容器在系统间的迁移，但容器快照文件将丢失所有历史记录和元数据信息，而镜像存储文件将保存完整记录

```shell
# 导出容器
$ docker container export -o FILE_NAME.tar CONTAINER_ID
# 载入容器快照到本地镜像库
$ docker container import FILE_NAME.tar IMAGE_NAME[:TAG]
```

#### **查看容器**

```shell
# 查看容器具体信息，包括容器Id、创建时间、路径、状态、镜像、配置等
$ docker container inspect CONTAINER_ID

# 查看容器内进程，类似在容器内执行top命令
$ docker container top CONTAINER_ID

# 查看容器统计信息，包含CPU、内存、存储、网络等使用情况
$ docker container stats
```

#### **其他命令**

```shell
# 在容器和主机之间复制文件
$ docker container cp HOST_FILE_PATH CONTAINER_NAME:PATH
$ docker container cp data test:/temp/

# 查看容器内文件系统的变更
$ docker container diff CONTAINER_NAME

# 查看容器的端口映射
$ docker container port CONTAINER_NAME

# 更新容器运行时配置，如CPU配额等，具体[OPTIONS]配置可查看文档
$ docker container update [OPTIONS] CONTAINER_NAME
```





## **Docker仓库**

仓库(Repository)是集中存放镜像的地方，如Docker官方仓库DockerHub

注册服务器(Registry)是存放仓库的具体服务器地址，如private-docker.com

#### **DockerHub**

Docker官方的公共镜像仓库，地址为hub.docker.com

```shell
# 登录
$ docker login

# 搜索镜像  OFFICIAL-官方镜像或用户镜像 AUTOMATED-自动跟踪代码变更而重新构建的镜像
$ docker search centos
NAME                      DESCRIPTION                    STARS    OFFICIAL   AUTOMATED
centos                    The official build of CentOS.  5829     [OK]                
ansible/centos7-ansible   nsible on Centos7              128                  [OK]

# 下载镜像
$ docker pull centos

# 使用第三方镜像市场下载Docker官方镜像 <Registry_Url>/<Namespace>/<Repository>:<Tag>
$ docker pull index.tenxcloud.com/docker_library/node:latest
# 更新镜像标签与官方标签保持一致
$ docker tag index.tenxcloud.com/docker_library/node:latest node:latest
```





## **Docker数据管理**

容器的数据管理，如容器数据持久化、多个容器之间数据共享等

#### **数据卷(Data Volumes)**

数据卷是一个容器内可映射到主机目录的目录，有以下特点

* 容器间可共享和重用数据卷来数据传递
* 容器数据卷的数据与主机目录的数据实时同步
* 数据卷的更新对镜像无影响，解耦应用与数据
* 数据卷会持续到无容器使用

数据卷的相关操作

```shell
# 创建数据卷 [-d local]默认保存在主机/var/lib/docker/volumes路径下 [test] 数据卷名称
$ docker volume create -d local test 
# 查看数据卷列表
$ docker volume ls
DRIVER              VOLUME NAME
local               test
# 查看数据卷详情
$ docker volume inspect test
[
    {
        "CreatedAt": "2020-02-20T16:59:56+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/test/_data",
        "Name": "test",
        "Options": {},
        "Scope": "local"
    }
]

# 创建容器时将主机路径挂载到容器内作为数据卷
$ docker container run -mount type=[VOLUME_TYPE]，source=[HOST_PATH],destination=[CONTAINER_PATH] [IMAGE_NAME:TAG]
# VOLUME_TYPE
  volume-普通数据卷，映射到主机/var/lib/docker/volumes路径下
  bind-绑定数据卷，映射到主机指定绝对路径下
  tmpfs-临时数据卷，映射到主机内存中
$ docker container run -d -P --name web --mount type=bind,source=/webapp,destination=/opt/webapp training/webapp python app.py
```

#### **数据卷容器(Data Volume Cotainers)**

数据卷容器是一个专门提供数据卷给其他容器挂载的容器

```shell
# 创建数据卷容器 [-v /dbdata]在容器创建一个数据卷挂载到容器目录/dbdata下
$ docker run -it -v /dbdata --name dbdata ubuntu
# 查看/dbdata目录
root@cd147a1a5f49:/# ls
bin   dbdata  etc   lib    media  opt   root  sbin  sys  usr
boot  dev     home  lib64  mnt    proc  run   srv   tmp  var

# 创建测试容器 [--volumes-from dbdata]从其他容器挂载数据卷
$ docker run --volumes-from dbdata --name db1 ubuntu
$ docker run --volumes-from dbdata --name db2 ubuntu
```

#### **利用数据卷容器迁移数据**

```shell
# 备份数据卷容器(dbdata)内的数据卷
# 运行worker容器
# [--volumes-from dbdata] 挂载数据卷容器
# [-v ${pwd}:/backup] 挂载本地目录到worker容器的/backup目录
# [tar cvf /backup/backup.tar /dbdata] 将/dbdata(数据卷容器)文件备份到/backup/backup.tar(主机挂载目录)
$ docker run --volumes-from dbdata -v ${pwd}:/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata

# 恢复数据到容器
# 创建数据卷容器
$ docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
# 创建新容器 挂载数据卷容器 将备份文件恢复到所挂载的容器卷中
$ docker run --volumes-from dbdata2 -v ${pwd}:/backup busybox tar xvf /backup/backup.tar
```





## **容器访问**

容器可通过**端口映射到主机**或者**互联机制**向外部或别的容器提供服务

#### **端口映射**

启动容器时指定端口映射，容器外部就可通过网络访问容器内的应用和服务

```shell
# 启动容器 -P 随机端口映射 
$ docker run -d -P training/webapp python app.py
CONTAINER ID        IMAGE             ...      PORTS                   
db9cd2d7a534        training/webapp   ...      0.0.0.0:32771->5000/tcp  

# 启动容器 -p 指定映射端口 [IP]:[HOST_PORT]:CONTAINER_PORT []表示可指定可忽略
$ docker run -d -p 127.0.0.1:5001:5000 training/webapp python app.py
CONTAINER ID        IMAGE             ...      PORTS                   
61403c1764f8        training/webapp   ...      127.0.0.1:5001->5000/tcp

# 查看容器映射端口 
$ docker port CONTAINER_ID or CONTAINER_NAME PORT
$ docker port 61403c 5000
127.0.0.1:5001
```

#### **互联机制(Linking)**

容器的互联机制在源容器与接收容器之间创建一种连接关系，容器间的应用可以通过容器名相互访问，而不用指定具体IP地址

Docker相当于在两个互联的容器之间建立了一个虚机通道，而且不用映射他们的端口到宿主主机上，从而避免暴露了一些服务(如数据库)端口到外部网络上

```shell
# 启动容器时自定义命名 如web容器、db容器 易于访问
$ docker run -d -P --name web training/webapp python app.py

# 容器互联
# 启动db容器
$ docker run -d --name db training/postgres
# 启动web容器 --link NAME:ALIAS 连接到的容器名:别名 测试跟db容器的连通
$ docker run -it --name web --link db:db training/webapp /bin/bash
root@268e4f37abfc:/opt/webapp# ping db
PING db (172.17.0.3) 56(84) bytes of data.
64 bytes from db (172.17.0.3): icmp_seq=1 ttl=64 time=0.101 ms
64 bytes from db (172.17.0.3): icmp_seq=2 ttl=64 time=0.074 ms
64 bytes from db (172.17.0.3): icmp_seq=3 ttl=64 time=0.083 ms
...
```







## **Dockerfile创建镜像**

Dockerfile是文本格式的配置文件可用于创建自定义镜像

#### **基本结构**

主体内容可分为四部分

```dockerfile
# 1.基础镜像信息
FROM ubuntu:xeniel

# 2.维护者信息
LABEL maintainer docker user<docker user@email.com>

# 3.镜像操作指令
# update the image
RUN echo "deb http://archive.ubuntu.com/ubuntu/xeniel main universe" >> /etc/apt/source.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/ngnix/nginx.conf

# 4.容器启动命令
CMD /usr/sbin/nginx
```

#### **Nginx镜像示例**

```dockerfile
FROM debian:jessie

MAINTAINER NGINX Docker Maintainers "docker-maint@nginx.com"

ENV NGINX_VERSION 1.10.1-1~jessie

RUN apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62 && \
echo "deb http://nginx.org/package/debian/ jessie nginx" >> /etc/apt/source.list && apt-get update && \
apt-get install --no-install-recommends --no-install-suggests -y ca-certificates nginx=$(NGINX_VERSION) \
nginx-module-xslt nginx-module-geoip nginx-module-image-filter nginx-module-perl nginx-module-njs gettext-base && \
rm -rf /var/lib/apt/lists/*

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log && ln -sf /dev/stderr /var/log/nginx/err.log

EXPOSE 80 443

CMD ["nginx","-g","daemon off;"]
```

#### **指令说明**

指令汇总

![](/Dockerfile指令.png)

配置指令

```dockerfile
# ARG [KEY]=[VALUE]
# 定义Dockerfile中的变量 创建镜像完成后失效 
ARG VERSION=9.3
FROM debian:${VERSION}

# FROM [IMAGE]:[TAG]
# 指定建立镜像的基础镜像 
FROM ubuntu:18.04

# LABEL [KEY]=[VALUE] [KEY]=[VALUE] ...
# 未生成的镜像添加元数据标签信息 可用于辅助过滤镜像 
LABEL version="1.0.0-rc3" author="havie@github.com" date="2020-02-22"

# EXPOSE [PORT] [PORT] ...
# 声明镜像内服务监听端口 具体端口映射需在启动容器时指定
EXPOSE 22 80 8443

# ENV [KEY]=[VALUE] or [KEY] [VALUE]
# 指定环境变量 可用于镜像生成、RUN指令、容器启动等 可被覆盖
ENV APP_HOME=/usr/local/app
ENV PATH $PATH:/usr/local/bin

# ENTRYPOINT ["executable","PARAM1","PARAM2"] - exec调用执行
# ENTRYPOINT command PARAM1 PARAM2 - shell中执行  
# 指定镜像的默认入口命令 该命令在启动容器时会执行 可被覆盖

# VOLUME ["PATH"]
# 创建一个数据卷挂载点
VOLUME ["/data"]

# USER USER_NAME 
# 指定运行容器时的用户名 后续RUN命令也会使用指定的用户身份
USER daemon

# WORKDIR /PATH/TO/WORIDIR 
# 为后续的RUN、CMD、ENTRYPOINT命令配置工作目录
WORKDIR /a/b/c

# ONBUILD [COMMAND]
# 指定镜像的子镜像自动自动执行的操作命令
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python build --dir / app/src 

# STOPSIGNAL
# 指定创建镜像启动的容器接收退出的信号值
STOPSIGNAL signal

# HEALTHCHECK [OPTIONS] CMD COMMAND or NONE
# 配置启动容器进行健康检查的方式

# SHELL ["executable","PARAMS"]
# 指定其他命令使用shell时默认shell类型
SHELL ["/bin/sh","-c"]
```

操作指令

```dockerfile
# RUN ["executable","PARAM1","PARAM2"] - exec调用执行
# RUN command PARAM1 PARAM2 - shell中执行(/bin/sh -c)
# 运行指定命令 可用\换行
RUN apt-get update \
	&& apt-get install -y libsnappy-dev zliblg-dev libbz2-dev \
	&& rm -rf /var/cache/apt
	
# CMD ["executable","PARAM1","PARAM2"] - exec调用执行
# CMD command PARAM1 PARAM2 - shell中执行
# CMD [PARAM1","PARAM2"] - 提供给ENTRYPOINT的默认参数
# 指定启动容器时默认指定的命令 只能有一条 可被手动覆盖
CMD ["nginx","-g","daemon off;"]

# ADD [HOST_PATH] [CONTAINER_PATH]
# 添加 Dockfile所在目录的相对路径/URL 的内容到容器内 绝对路径 下
ADD /*.java /code/

# COPY
# 复制 Dockerfile所在目录的相对路径 的内容到容器内 绝对路径 下
COPY /*.java /code/
```

#### **创建镜像**

```shell
# 创建镜像 [-t 指定生成镜像的标签信息] [-f 指定非上下文路径下的Dockerfile]
$ docker build -t builder/image_name:1.0.0 /temp/docker_builder/
```

具体可用的命令选项

![](/创建镜像命令选项.png)

#### **镜像编写技巧**

* **精简镜像用途**

  让每个镜像用途单一，避免构建复杂、多功能的镜像

* **选择合适的基础镜像**

  容器的核心是应用，选择过大的基础镜像会造成生成应用镜像臃肿

* **减少镜像层数**

  尽量合并RUN/ADD/COPY指令

* **恰当使用多步骤创建(Multi-stage)**

  多步骤创建将编译和运行等过程分开，保证最终生成的镜像只包含运行应用所需的最小化环境

* **运用.dockerignore文件**

  避免发送不必要的数据内容加快镜像创建过程

* **减少外部源干扰**

  外部指定地址需稳定且可以复用