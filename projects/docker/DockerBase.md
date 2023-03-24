### Docker 镜像加载原理：联合文件系统

https://vuepress.mirror.docker-practice.com/ 官方文档

> docker的镜像实际上是由一层层的文件系统组成，这种层级的文件系统UnionFS

bootfs（最底层，主要引导加载kernel） + rootfs（在bootfs之上，就是各种不同的操作系统发行版，如centos ubuntu）

对于一个系统，bootfs基本是一致的，所以可以公用bootfs。镜像只需提供rootfs就可以了

* 启动docker ： systemctl start docker 

帮助命令：

* 版本 docker version

* 详情 docker info  
  类似Linux man命令   docker --help

 

更新yum软件包索引

yum makecache fast

 

镜像命令：

docker images 本地的镜像目录

 

REPOSITORY 镜像的仓库源

TAG 标签 同一个仓库源可以有多个TAG，代表这个仓库源的不同个版本。使用 REPOSITORY：TAG来定义不同镜像

IMAGE_ID 镜像ID  CREATED  创建时间  SIZE 大小

-a 代表All

-q 仅显示id 亦可 -aq

--digests 摘要

--no-trunc 完整镜像信息



* 查找镜像

docker search [image]

-s 

--no-trunc

 

拉取

doker pull [name:[tag]]

 

删除

docker rmi [PEPOSITORY]

docker rmi -f $(docker images -qa)

 

启动

docker run [OPTIONS]

docker run -it --name mycentos // 运行centos 虚拟机

此时可以用pwd和ls命令查看当前路径

 

-d 后台模式启动容器

docker run -d -p 8088:8080 hellozmg/tomcat:1.1

 

小p 是外部访问docker的端口

docker run -it -p 8080:8080 【容器id或REPOSITORY】

 

大P是随机分配

docker run -it -P 【容器id或REPOSITORY】

 

前台运行 docker run -it -p 7777:8080 tomcat

后台运行 docker run -d -p 6666:8080 tomcat

 

运行的容器

docker ps 【OPTIONS】不加参数为当前正在运行的

docker ps -n 3

-q 只显示容器编号（批量操作时可以用上）

-l 上次运行的容器

 

exit 停止容器，并退出

ctrl + p + q 不停止容器，并退出

 

docker ps -a 查看容器之后启动： 

docker start  [容器id] （or容器名称）

 

docker restart [容器id]

 

docker stop [容器id]

 

docker kill [容器id]

 

docker rm [容器id]

一次性删除多个容器 

docker rm -f $(docker ps -q)

docker rm -f $(docker ps -a -q)

docker ps -a -q | xargs docker rm

 

查看容器日志

docker logs -f -t -tail 【容器id】

 

这句话什么意思？

docker run -d centos /bin/sh -C "while true; do echo hello zzyy; sleep 2; done"

 

查看容器里的进程

docker top

 

全部的结构细节

docker inspect [容器id]

 

进入正在运行的容器并以命令交互

docker exec 在容器中打开新的终端，并且可以启动新的进程,直接运行命令返回结果

docker exec -it [容器id] bashShell 

如：docker exec -t 343432423 /bin/bash 或者 docker exec -it 89c5b9c81e74 bash 

docker attach 【容器id】直接进入容器启动命令的终端 

 

从容器内核拷贝文件到主机上

docker cp 容器id:/路径 /本机路径

 

 

提交容器副本使之成为一个新的镜像 

docker commit -m="提交的描述信息" -a="作者" 容器id 要创建的目标镜像名：【标签名】

docker commit -m="sucsess have html" -a="zmg" a98498d3f950 hellozmg/tomcat:1.1

 

容器和宿主机之间数据卷可以共享，容器停止退出后，主机修改的数据可以同步给容器

docker volume ls         查看所有数据卷

docker volume inspect 数据卷名  详情

数据卷添加

docker run -it -v /宿主机绝对路径目录:/容器内目录 镜像名

docker run -it -v /home/myDockerData:/home/myDockerData centos

 

docker inspect [容器ID]   查看数据卷是否挂载成功，显示的文件中有volumesRW: 属性 值为true 代表挂在成功

 

给加权限：

docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名

docker run -it -v /home/myDockerData:/home/myDockerData:ro centos

 

 

执行DockerFile 文件,构建新镜像 -f :指定要使用的Dockerfile路径  -t: 镜像的名字及标签

docker build -t 新镜像名字:tag .

docker build -f /myforder/myDockerfile -t zzxx/centos .

如果标准的文件名为Dockerfile 在文件当前目录下，可以直接执行 docker build -t tomcat .

 

 

Docker挂载主机目录 访问出现cannot open directory .:Permission denied

解决办法： 在挂载目录后多加一个--privileged=true参数即可

docker run -it -v /myDataVolume:/dataVolumeContainer --privileged=true

 

安装网址 https://docs.docker.com/engine/install/centos/

官网：yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

 

（最新的20.10版本 有些软件找不到 后来我又实用的官网的）阿里云

um-config-manager –add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo2

 

 

配置镜像

vi /etc/docker/daemon.json

{"registry-mirrors":["http://hub-mirror.c.163.com"] }

systemctl restart docker

 

 

查看docker网桥命令

docker network ls

创建一个自己的网桥

docker network create [网桥名称] 或 docker network create -d [名称]

 

启动容器时指定网桥

docker run -d -p 8081:8080 --network [网桥名称] --name [名称] centos

 

注意：一旦在启动容器时指定了网桥之后，日后可以在任何这个网桥关联的容器中使用容器名字进行与其他容器通讯

 

删除网桥

docker network rm zmg

网桥信息 可以看到这个网桥上的所有容器

docker inspect zmg

 

注意 使用 docker run 指定 --network 网桥时，网桥必须存在

 

保存.war文件

docker save 

docker load 

 

docker cache 

docker build --no-cache 有缓存

 

 

 

 

 

 

 

练习：

\1. 拉取centos镜像： docker pull centos

2.新建并启动容器： docker run -it --name 666 300e315adb2f

3.列出当前正在运行的容器  docker ps 或 docker ps -l 或 docker ps -n 5

4.退出容器  ctrl + p + q 

5.启动容器  docker start 300e315adb2f

6.重启容器  docker restart 300e315adb2f

7.停止容器  docker stop 300e315adb2f

8.强制停止容器 docker kill 300e315adb2f

9.删除已停止的容器  docker rm -f 300e315adb2f

 

 

 

编写Dockerfile 

 

\# volume test

FROM centos

VOLUME ["/dataVolumeContainer1","/dataVolumeContainer2"]

CMD echo "finished,---------success1"

CMD /bin/bash

 

执行dockerfile文件，build镜像

docker build -f /docker/dockerfile -t test01/centos .

这里在build时，会将主机当前目录的所有文件进行打包，所以建议新建空文件夹作为当前目录

 

volumes-from 数据共享：

 

运行证明

docker run -it --name dc01 test01/centos /bin/bash

 

docker run -it --name dc02 --volumes-from dc01 test01/centos

 

docker run -it --name dc03 --volumes-from dc02 test01/centos

继承之后数据卷相互同步，

删除父容器后，子容器文件不受影响， 两个子容器数据卷仍然是共享的

 

 

这句话能读懂吗？

docker run -d -p 9080:8080 --name myt9 -v /home/tomcat/tomcat9/test:/usr/local/apache-tomcat-9.0.8/webapps/test -v /home/tomcat/tomcat9/tomcat9logs/:/usr/local/apache-tomcat-9.0.8/logs --privileged=true tomcat9

 

 

// 拉取mysql镜像

docker pull mysql:5.7

docker run -p 12345:3306 --name mysql5.7.1 -v /home/docker/mysql/conf:/etc/mysql/conf.d -v /home/docker/mysql/logs:/logs -v /home/docker/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

 

WARNING: IPv4 forwarding is disabled. Networking will not work.

问题原因是没有开启转发,docker网桥配置完后，需要开启转发，不然容器启动后，就会没有网络，要在配置/etc/sysctl.conf,添加net.ipv4.ip_forward=1 注：也可修改此文件：/usr/lib/sysctl.d/00-system.conf

重启network和docker服务 systemctl restart network && systemctl restart docker

查看是否修改成功  sysctl net.ipv4.ip_forward

 

// 拉取redis 镜像

docker pull redis

docker run -p 6379:6379 -v /home/docker/redis/data:/data -v /home/docker/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf -d redis redis-server /usr/local/etc/redis/redis.conf --appendonly yes

 

docker commit -a zmg -m "new emages" 容器id 新容器名称：版本号

 

 

 

![img](file:///C:/Users/zmg/AppData/Local/Temp/msohtmlclip1/01/clip_image002.jpg)

![img](file:///C:/Users/zmg/AppData/Local/Temp/msohtmlclip1/01/clip_image004.jpg)

 

![img](file:///C:/Users/zmg/AppData/Local/Temp/msohtmlclip1/01/clip_image006.jpg)

![img](file:///C:/Users/zmg/AppData/Local/Temp/msohtmlclip1/01/clip_image008.jpg)

![img](file:///C:/Users/zmg/AppData/Local/Temp/msohtmlclip1/01/clip_image010.jpg)

 

**Dockerfile**

1、基本结构

Dockerfile 由一行行命令语句组成，并且支持以 # 开头的注释行。

一般的，Dockerfile 分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

其中，一开始必须指明所基于的镜像名称，接下来推荐说明维护者信息。

后面则是镜像操作指令，例如 RUN 指令，RUN 指令将对镜像执行跟随的命令。每运行一条 RUN 指令，镜像添加新的一层，并提交。最后是 CMD 指令，来指定运行容器时的操作命令。

2、指令

2.1 FROM

格式为 FROM <image>或FROM <image>:<tag>。

第一条指令必须为 FROM 指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个 FROM 指令（每个镜像一次）。

2.2 MAINTAINER

格式为 MAINTAINER <name>，指定维护者信息。

2.3 RUN

格式为 RUN <command> 或 RUN ["executable", "param1", "param2"]。

前者将在 shell 终端中运行命令，即 /bin/sh -c；后者则使用 exec 执行。指定使用其它终端可以通过第二种方式实现，例如 RUN ["/bin/bash", "-c", "echo hello"]。

每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。

2.4 CMD

支持三种格式

·     CMD ["executable","param1","param2"] 使用 exec 执行，推荐方式；

·     CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用；

·     CMD ["param1","param2"] 提供给 ENTRYPOINT 的默认参数；

指定启动容器时执行的命令，每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条命令，只有最后一条会被执行。如果用户启动容器时候指定了运行的命令，则会覆盖掉 CMD 指定的命令。

2.5 EXPOSE

格式为 EXPOSE <port> [<port>...]。

告诉 Docker 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 -P，Docker 主机会自动分配一个端口转发到指定的端口。

2.6 ENV

格式为 ENV <key> <value>。 指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持。

2.7 ADD

格式为 ADD <src> <dest>。

该命令将复制指定的 <src> 到容器中的 <dest>。 其中 <src> 可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）。

2.8 COPY

格式为 COPY <src> <dest>。

复制本地主机的 <src>（为 Dockerfile 所在目录的相对路径）到容器中的 <dest>。当使用本地目录为源目录时，推荐使用 COPY。

ENTRYPOINT

两种格式：

·     ENTRYPOINT ["executable", "param1", "param2"]

·     ENTRYPOINT command param1 param2（shell中执行）。

配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。

每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。

![img](file:///C:/Users/zmg/AppData/Local/Temp/msohtmlclip1/01/clip_image012.jpg)

与cmd配合使用，可以用来做变量在运行时传送，切必须时用json格式。["executable", "param1", "param2"]

2.9 VOLUME

格式为 VOLUME ["/data"]。

创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。

2.10 USER

格式为 USER daemon。

指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。

当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如：RUN groupadd -r postgres && useradd -r -g postgres postgres。要临时获取管理员权限可以使用 gosu，而不推荐 sudo。

2.11 WORKDIR

格式为 WORKDIR /path/to/workdir。

为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。

可以使用多个 WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。

则最终路径为 /a/b/c。

2.12 ONBUILD

格式为 ONBUILD [INSTRUCTION]。

配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。

使用 ONBUILD 指令的镜像，推荐在标签中注明，例如 ruby:1.9-onbuild。

3、创建镜像

编写完成 Dockerfile 之后，可以通过 docker build 命令来创建镜像。

基本的格式为 docker build [选项] 路径，该命令将读取指定路径下（包括子目录）的 Dockerfile，并将该路径下所有内容发送给 Docker 服务端，由服务端来创建镜像。因此一般建议放置 Dockerfile 的目录为空目录。也可以通过 .dockerignore 文件（每一行添加一条匹配模式）来让 Docker 忽略路径下的目录和文件。

要指定镜像的标签信息，可以通过 -t 选项

 

 

 

 

 

Docker Compose 

定位是运行多个Docker容器的应用。同时可以对多个容器进行编排

核心概念：服务，一个应用的容器。服务可以存在多个

项目：由一个关联的应用容器组成的一个完整业务单元，在docker-compose.yml文件中定义

 

从github上下载docker-compose二进制文件安装  

curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

 

若是github访问太慢，可以用daocloud下载 

curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

添加可执行权限 

chmod +x /usr/local/bin/docker-compose 

验证

docker-compose --version

 

docker-compose up 启动docker-compose文件

 

docker-compose文件格式大概：

version: "3.0" # 版本是要与docker兼容的 可以在官网查看支持的docker 引擎

services: 用来书写当前项目中哪些容器 服务

  tomcatss: #服务名唯一

   image: tomcat  #创建当前这个服务使用镜像

   ports: #用来完成host与容器的端口映射关系

​     \- “8080:8080”

​      Volumes：# 完成宿主机与容器中目录数据卷共享

\-     Tomcatwebapps:/usr/local/tomcat/webapps    




portainer Docker可视化工具 官网:https://www.portainer.io/

下载命令： docker pull portainer/portainer

应该是创建数据卷 ： docker volume create portainer_data

启动： docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer

启动之后就可以ip:port访问了，首次进入需要设置8位密码，此次设置00000000

 

Stack概念： 一个docker-compose就是一个stack