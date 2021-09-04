# Docker

## 1. Docker的安装与启动

### 1.1 安装docker

​		这里安装在CentOS7.x以上版本，在CentOS6.x版本中，安装前需要安装其他许多环境而且Docker很多补丁不支持更新。

（1）yum包更新到最新

```
sudo yum update 
```

（2）安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

（3）设置yum源为阿里云或清华大学源

```
阿里云：sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
清华大学：sudo yum-config-manager --add-repo https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

（4）安装Docker社区版

```
sudo yum install docker-ce
```

（5）查看Docker版本

```
docker -v
```

### 1.2 设置ustc镜像

​		docker 从 docker hub 拉取镜像，因为是从国外获取，速度较慢。可以通过配置国内镜像源的方式，从国内获取镜像，提高拉取速度。这里介绍使用[中国科学技术大学](https://www.baidu.com/s?wd=中国科学技术大学&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)（LUG@USTC）的开源镜像 **https://docker.mirrors.ustc.edu.cn**

（1）编辑该文件，如果没有该文件请先创建

```
vi /etc/docker/daemon.json
```

（2）在该文件中输入如下内容

```
"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
```

![image-20210814191438191](https://i.loli.net/2021/08/14/g8rRB2UlWdAoc6k.png)

### 1.3 Docker的启动与停止

​		**systemctl**命令是系统服务管理命令。

（1）启动Docker

```
systemctl start docker
```

（2）停止Docker

```
systemctl stop docker
```

（3）重启Docker

```
systemctl restart docker
```

（4）查看Docker状态

```
systemctl status docker
```

（5）开机启动

```
systemctl enable docker
```

（6）查看Docker概要信息

```
docker info
```

（7）查看Docker帮助文档

```
docker --help
```

## 2. 常用命令

### 2.1 镜像相关命令

#### 2.1.1 查看镜像

（1）列出所有镜像

```
docker images
```

（2）列出指定镜像

```
docker image ls 镜像名
docker images 镜像名
```

REPOSITORY：仓库

TAGE：标签

IAMGE ID：镜像ID

CREATED：创建时间

SIZE：镜像大小

这些镜像都是存储在Docker宿主机的/var/lib/docker目录下。

#### 2.1.2 搜索镜像

```
docker search 镜像名称
```

NAME ：镜像名称

DESCRIPTION：镜像描述

STARS：用户评价，反应一个镜像的受欢迎度

OFFICIAL：是否官方

AUTOMATED：自动创建，表示该镜像有Docker Hub自动构建流程创建

#### 2.1.3  拉取镜像

（1）拉取镜像是从中央仓库下载镜像到本地

```
docker pull 镜像名称
```

（2）例如，下载centos7镜像

```
docker pull centos:7
```

如果不指定后面标签**（:7）**，其默认标签为**latest**，即为最新镜像。

#### 2.1.4 删除镜像

（1）删除指定镜像，按镜像ID删除

```
docker rmi 镜像ID
```

（2）删除所有镜像

```
docker rmi `docker images -q`
```

### 2.2 容器相关命令

#### 2.2.1 查看容器

（1）查看正在运行的容器

```
docker ps
```

（2）查看所有容器

```
docker ps -a
```

（3）查看最后一次运行的容器

```
docker ps -l
```

（4）查看停止的容器

```
docker ps -f status=exited
```

#### 2.2.2 创建和启动容器

创建容器常用的参数说明：

创建容器命令：**docker run**

- -i：表示运行容器

- -t：表示容器启动后会进入其命令行。加入这两个参数后，容器创建就能登录进去。即分配一个伪终端。
- –name ：为创建的容器命名。
- -v：表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个－v做多个目录或文件映射。注意：最好做目录映射，在宿主机上做修改，然后共享到容器上。
- -d：在run后面加上-d参数,则会创建一个守护式容器在后台运行（这样创建容器后不会自动登录容器，如果只加-i -t两个参数，创建后就会自动进去容器）。
- -p：表示端口映射，前者是宿主机端口，后者是容器内的映射端口。可以使用多个-p做多个端口映射。

（1）交互式方式创建容器。进入容器退出后，容器即运行停止。

```
docker run -it --name=容器名称 镜像名称:标签 /bin/bash
```

例如：

```
docker run -it --name=mycentos centos:7 /bin/bash
```

（2）退出当前容器

```
exit或CTRL+D
```

（3）守护式方式创建容器。进入容器在退出后，容器依旧运行。

```
docker run -di --name=容器名称 镜像名称:标签
```

（4）进入容器

```
docker exec -it 容器名称（或者容器ID）/bin/bash
```

#### 2.2.3 容器的启动与停止

（1）停止容器

```
docker stop 容器名称(或者容器ID)
```

（2）启动容器

```
docker start 容器名称(或者容器ID)
```

（3）创建容器时自启

```
docker run -d --restart=always --name=设置容器名 使用的镜像
```

（上面命令  --name后面两个参数根据实际情况自行修改）

 --restart具体参数值详细信息：

- no： 默认策略,容器退出时不重启容器；

- on-failure：在容器非正常退出时（退出状态非0）才重新启动容器；

- on-failure:3： 在容器非正常退出时重启容器，最多重启3次；

-  always：无论退出状态是如何，都重启容器；

- unless-stopped：在容器退出时总是重启容器，但是不考虑在 Docker 守护进程启动时就已经停止了的容器。

（4）修改已有容器，使用update

```
docker update --restart=always 容器ID(或者容器名)
```

（容器ID或者容器名根据实际情况修改）

#### 2.2.4 文件拷贝

（1）如果我们需要将文件拷贝到容器内可以使用cp命令

```
docker cp 需要拷贝的文件或目录 容器名称:容器目录
```

（2）也可以将文件从容器内拷贝出来

```
docker cp 容器名称:容器目录 需要拷贝的文件或目录
```

#### 2.2.5 目录挂载

在创建容器的时候，将宿主机的目录与容器内的目录进行映射，这样我们就可以通过修改宿主机某个目录的文件从而去影响容器。
创建容器 添加-v参数 后边为 宿主机目录:容器目录，例如：

```
docker run -di -v /usr/local/myhtml:/usr/local/myhtml --name=mycentos3 centos:7
```

如果你共享的是多级的目录，可能会出现权限不足的提示。

这是因为CentOS7中的安全模块selinux把权限禁掉了，我们需要添加参数 --privileged=true 来解决挂载的目录没有权限的问题。

#### 2.2.6 查看容器 IP 地址

（1）我们可以通过以下命令查看容器运行的各种数据

```
docker inspect 容器名称（容器ID） 
```

（2）也可以直接执行下面的命令直接输出IP地址

```
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称（容器ID）
```

#### 2.2.7 实时查看docker容器的日志

```
docker logs -f -t --tail -f 容器名称（或者容器ID）
```

#### 2.2.8 删除容器

删除容器前需要先停止要删除的容器。

```
docker rm 容器名称（容器ID）
```

## 3. 应用部署

### 3.1 MySQL部署

（1）拉取mysql镜像

```
docker pull centos/mysql-57-centos7
```

（2）创建容器

```
docker run -di --name=tensquare_mysql -p 33306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```

-p 代表端口映射，格式为 宿主机映射端口:容器运行端口

-e 代表添加环境变量 MYSQL_ROOT_PASSWORD 是root用户的登陆密码

（3）进入mysql容器

```
docker exec -it tensquare_mysql /bin/bash
```

（4）登录mysql并修改root密码

```
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

（5）添加远程登录用户

```
CREATE USER 'tanghuan'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
GRANT ALL PRIVILEGES ON *.* TO 'tanghuan'@'%';
```

（6）远程登录mysql

```
mysql -h 连接宿主机的IP -u 数据库拥有的用户名 -p
```

### 3.2 tomcat部署

（1）拉取镜像

```
docker pull tomcat:7-jre7
```

（2）创建容器

-p表示地址映射端口，-v表示挂载目录

```
docker run -di --name=mytomcat -p 9000:8080 -v /usr/local/webapps:/usr/local/tomcat/webapps tomcat:7-jre7
```

（3）将本地war包拷贝到宿主机**/usr/local/webapps**路径下完成部署

### 3.3 Nginx部署

（1）拉取镜像

```
docker pull nginx
```

（2）创建Nginx容器

```
docker run -di --name=mynginx -p 80:80 nginx
```

（3）进入Nginx容器中

```
docker exec -it mynginx /bin/bash
```

（4）查看nginx.conf配置文件

```
cat /etc/nginx/nginx.conf
```

![image-20210815164254980](https://i.loli.net/2021/08/15/K74YAZoXLhUacus.png)

可以看到其配置是外部导入的，方便分模块化的管理。

（5）查看默认配置文件default.conf

```
cat /etc/nginx/conf.d/default.conf
```

<img src="https://i.loli.net/2021/08/15/d6ufP9OF3sEy7AI.png" alt="image-20210815164526860"  />

可以看到其欢迎页的目录是在**/usr/share/nginx/html**文件中。

（6）修改本地上传的文件名为**html**，并复制到Nginx容器的**/usr/share/nginx**目录中

```
mv 本地文件名 html
docker cp html mynginx:/usr/share/nginx
```

### 3.4 Redis部署

（1）拉取镜像

```
docker pull redis
```

（2）创建容器

```
docker run -di --name=myredis -p 6379:6379 redis
```

（3）进入容器并通过 redis-cli 连接测试使用 redis 服务。

```
docker exec -it myredis /bin/bash
```

![image-20210815170856069](https://i.loli.net/2021/08/15/MG1FKhnBEP23Qbq.png)

## 4. 备份与迁移

### 4.1 容器保存为镜像

```
docker commit 容器名称 镜像名称
例：docker commit mynginx mynginx_i
```

### 4.2 镜像备份

将镜像保存为文件

```
docker save -o 文件名 镜像名
例：docker save -o mynginx.tar mynginx_i
```

-o：output

### 4.3 镜像恢复与迁移

首先我们先删除掉自己创建的**mynginx_i**镜像，然后执行此命令进行恢复

```
docker load -i 文件名
例：docker load -i mynginx.tar
```

-i：输入的文件

## 5. Dockerfile

### 5.1 什么是Dockerfile

Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像。

1、对于开发人员：可以为开发团队提供一个完全一致的开发环境;
2、对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了;
3、对于运维人员：在部署时，可以实现应用的无缝移植。

### 5.2 常用命令

|              **命令**              |                           **作用**                           |
| :--------------------------------: | :----------------------------------------------------------: |
|        FROM image_name:tag         |              定义了使用哪个基础镜像启动构建流程              |
|        MAINTAINER user_name        |                       声明镜像的创建者                       |
|           ENV key value            |                  设置环境变量 (可以写多条)                   |
|            RUN command             |              是Dockerfile的核心部分(可以写多条)              |
| ADD source_dir/file dest_dir/file  | 将宿主机的文件复制到容器内，如果是一个压缩文件，将会在复制后自动解压 |
| COPY source_dir/file dest_dir/file |           和ADD相似，但是如果有压缩文件并不能解压            |
|          WORKDIR path_dir          |                         设置工作目录                         |

## 6. 使用脚本创建 JDK8 镜像

（1） 创建目录

```
mkdir –p /usr/local/dockerjdk8
```

（2）下载 jdk-8u301-linux-x64.tar.gz 并上传到服务器（虚拟机）中的 /usr/local/dockerjdk8 目录

（3）创建文件 Dockerfile (文件名必须是Dockerfile)

```
vi Dockerfile
```

（4）在Dockerfile 中编辑内容

```
# 依赖镜像名称和ID
FROM centos:7
# 指定镜像创建者信息
MAINTAINER tanghuan
# 切换工作目录
WORKDIR /usr
RUN mkdir /usr/local/java
#ADD 是相对路径jar,把java添加到容器中
ADD jdk-8u301-linux-x64.tar.gz /usr/local/java/

# 配置java环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_301
ENV JRE_HOME /usr/local/java/jdk1.8.0_301/jre
ENV PATH $JAVA_HOME/bin:$PATH
```

（5）执行命令构建镜像

```
docker build -t 'jdk:1.8' .
```

- -t：表示镜像名:标签
- .：表示在当前目录下找 Dockerfile 文件

（5）查看镜像是否构建完成

```
docker images
```

![image-20210815183955344](https://i.loli.net/2021/08/15/Pd2YVay4L5IhXDu.png)

## 7.Docker私有仓库

​		Docker私有仓库主要用于存放同一企业应用，在同一局域网下可以拉取镜像。Docker私有仓库本质上也是一个镜像。

### 7.1 私有仓库搭建与配置

（1）拉取私有仓库镜像

```
docker pull registry
```

（2）启动私有仓库容器

```
docker run -di --name=myregistry -p 5000:5000 registry
```

（3）访问 **http://虚拟机ip:5000/v2/_catalog** ，能看到 {“repositories”:[]} 表示创建成功，且仓库内容为空

（4）修改 daemon.json，用于让 Docker 信任私有仓库地址

```
vi /etc/docker/daemon.json
```

（5）daemon.json添加如下内容

```
{
    "insecure-registries":["192.168.190.139:5000"]
}
```

![image-20210815191549570](https://i.loli.net/2021/08/15/TiFkuoYq5avNO2G.png)

（6）修改docker配置，允许远程访问

```
vi /lib/systemd/system/docker.service
```

**其中ExecStart=后添加配置 -H tcp://0.0.0.0:2375 -H unix: ///var/run/docker.sock**

（7）刷新配置，重启服务

```
systemctl daemon-reload
systemctl restart docker
docker start myregistry
```

### 7.2 镜像上传至私有仓库

（1）标记此镜像为私有仓库的镜像

```
docker tag 标签名 IP:5000/镜像名
例如：
docker tag jdk:1.8 192.168.190.139:5000/jdk1.8
```

（2）再次启动私服容器

```
docker start myregistry
```

（3）上传标记的镜像

```
docker push IP:5000/标签名
例如：
docker push 192.168.190.139:5000/jdk1.8
```

（4）再次访问 **http://虚拟机ip:5000/v2/_catalog** ，能看到 {“repositories”:[jdk1.8]} 表示上传创建成功

（5）拉取标记的镜像

```
docker pull IP:5000/标签名
例如：
docker pull 192.168.190.139:5000/jdk1.8
```

