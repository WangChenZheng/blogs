# Docker

## Docker基础

### Docker的安装与卸载（CentOS 7）

#### 配置镜像源并安装Docker

进入[阿里开源镜像站](https://developer.aliyun.com/mirror/)配置国内镜像源

```shell
# 配置CentOS 7镜像
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# 配置Epel镜像
wget -O /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
# 配置docker-ce镜像
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 使用阿里云的Docker-CE软件源
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
# Step 4: 更新并安装Docker-CE
sudo yum makecache
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo systemctl start docker
sudo systemctl enable docker
```

#### Docker的卸载

```shell
# Step 1: 停止Docker服务
sudo systemctl stop docker
# Step 2: 删除所有Docker容器
sudo docker rm -f $(sudo docker ps -aq)
# Step 3: 卸载Docker软件包
sudo yum remove docker-ce docker-ce-cli containerd.io
# Step 4: 删除Docker配置文件
sudo rm -rf /var/lib/docker
```

### Docker基础命令

#### 基础操作

```shell
# 启动docker
sudo systemctl start docker
# 重启docker
sudo systemctl restart docker
# 停止后台docker服务
sudo systemctl stop docker
# 查看docker运行状况
sudo systemctl status docker 
# 开机启动
sudo systemctl enable docker
# 查看docker概要信息
sudo docker info
# 查看docker全部帮助文档
sudo docker --help
# 查看docker某一命令帮助文档
sudo docker 具体命令 --help
# 查看docker版本
sudo docker version
# 查看镜像/容器/数据卷所占的空间
sudo docker system df
```

#### 镜像操作

```shell
# 列出本地主机上的所有镜像
sudo docker images [options]
# -a：列出本地所有的镜像（含历史镜像）
# -q：只显示镜像ID

# 到远程仓库中搜索xxx镜像
sudo docker search xxx
# sudo docker search xxx --limit 5  # 只显示5条命令

# 到远程仓库中下载xxx镜像
sudo docker pull xxx  # 默认版本为：lastest
sudo docker pull xxx:123  # 指定版本为123

# 删除镜像
sudo docker [options] rmi 镜像名 或者 镜像ID
# 例如 sudo docker rmi nginx
sudo docker [options] rmi  -f 镜像名 或者 镜像ID  # 强制删除
# 删除多个镜像
sudo docker rmi -f 镜像名字1:TAG 镜像名字2:TAG
# 删除全部镜像
sudo docker rmi -f $(docker images -aq)
```

#### 容器操作

```shell
# 启动容器
sudo docker run [options] IMAGE [COMMAND] [ARG...]
# sudo docker run -d --name httpd1 -p8080:80 httpd  # 启动httpd设置容器名为httpd1，端口映射为8080（将80端口映射为8080）
# sudo docker run -d 容器名或者容器ID
# [option]
# --name="容器的新名字"  为容器指定一个名称，没有的话随机分配；
# -d: 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)；
# -i：以交互模式运行容器，通常与 -t 同时使用；（interactive）
# -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；也即启动交互式容器(前台有伪终端，等待交互)；（tty）
 # -P: 随机端口映射，大写P
 # -p: 指定端口映射，小写p   
 # --restart=always:开机自启
# [COMMAND]
# /bin/bash:放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。
# 要退出终端，直接输入 exit:

# 列出当前正在运行的容器
sudo docker ps [OPTIONS]
# sudo docker ps -a
# [OPTIONS]
# -a: 列出当前所有正在运行的容器+历史上运行过的。
# -l: 显示最近创建的容器。
# -n: 显示最近n个创建的容器。
# -q: 静默模式，只显示容器编号。

# 退出容器（假设已经进入容器内）
exit  # 退出容器且容器停止
ctrl+p+q  # 退出容器但容器不停止

# 修改容器设置
sudo docekr update 容器ID或者容器名 [options]
# [options]同docker run

# 启动已停止运行的容器
sudo docker start 容器ID或者容器名
# 重启容器
sudo docker restart 容器ID或者容器名
# 停止容器
sudo docker stop 容器ID或者容器名
# 强制停止容器
sudo docker kill 容器ID或容器名
# 删除已停止的容器
sudo docker rm 容器ID
# 一次性删除多个容器实例
sudo docker rm -f $(docker ps -aq)
sudo docker ps -a -q | xargs docker rm
# 查看容器日志
sudo docker logs 容器ID
# 查看容器内运行的进程
sudo docker top 容器ID
# 查看容器内部细节
sudo docker inspect 容器ID
# 进入正在运行的容器并以命令行(/bin/bash)交互
sudo docker exec -it 容器ID /bin/bash  # exit不会导致容器的停止
sudo docker attach -it 容器ID /bin/bash  # exit会导致容器的停止
# 从容器内拷贝文件到主机上
sudo docker cp  容器ID:容器内路径 目的主机路径
# 导入和导出容器
# export: 导出容器的内容留作为一个tar归档文件[对应import命令]
# sudo docker export 容器ID > 文件名.tar  # 导出
# import: 从tar包中的内容创建一个新的文件系统再导入为镜像[对应export]
# sudo cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号  # 导入
```

#### 发布镜像

##### 发布镜像到云端（阿里云）

```shell
# docker commit提交容器副本使之成为一个新的镜像
sudo docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]
# 登录阿里云镜像仓库Docker Registry
sudo docker login --username=用户名 registry.cn-hongkong.aliyuncs.com
# 将镜像推送到Registry
sudo docker tag [镜像ID] registry.cn-hongkong.aliyuncs.com/仓库名/镜像名:[镜像版本号]
sudo docker push registry.cn-hongkong.aliyuncs.com/仓库名/镜像名:[镜像版本号]

# 从Registry中拉取镜像
sudo docker pull registry.cn-hongkong.aliyuncs.com/仓库名/镜像名:[镜像版本号]
```

##### 发布镜像到私有库

```shell
# 下载镜像Docker Registry
sudo docker pull registry
# 运行私有库Registry，相当于本地有个私有Docker hub
sudo docker run -d -p 5000:5000  -v /wangchen/myregistry/:/tmp/registry --privileged=true registry
# --privileged=true: 给容器授予所有的权限。
# -v /wangchen/myregistry/:/tmp/registry: 将主机上的 /wangchen/myregistry/ 目录挂载到容器中的 /tmp/registry 目录，以便将 Registry 数据存储在主机上。挂载目录可以确保 Registry 数据不会随着容器的销毁而丢失。

# docker commit提交容器副本使之成为一个新的镜像
sudo docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]

# 查看本地私服库上有什么镜像
sudo curl -XGET http://localhost:5000/v2/_catalog

# 将新镜像XXX修改符合私服规范的Tag
docker tag 镜像:Tag localhost:5000/镜像:Tag

# push推送到私服库
sudo docker push localhost:5000/镜像:tag

# 从私服库拉取镜像
sudo docker pull localhost:5000/镜像:tag
```

### Docker容器卷

Docker 容器卷（Volume）是 Docker 中一种用于数据持久化的机制。通过挂载卷，您可以将宿主机上的目录或文件夹挂载到容器中，并将容器中的数据持久化到宿主机上，从而实现容器数据的保存和共享。

使用 Docker 容器卷的好处包括：

- 数据持久化：容器卷可以将容器中的数据保存到宿主机上，从而实现数据持久化，即使容器被删除，数据仍然可以保留。
- 容器数据共享：多个容器可以共享同一个容器卷，这样就可以实现容器之间的数据共享。

以下是使用 Docker 容器卷的示例：

```shell
docker run -d -v /my/host/folder:/my/container/folder myimage
```

在这个示例中，`-v` 选项用于挂载容器卷。`/my/host/folder` 是宿主机上的目录，`/my/container/folder` 是容器中的目录，`myimage` 是要运行的 Docker 镜像。这条命令将宿主机上的 `/my/host/folder` 目录挂载到容器中的 `/my/container/folder` 目录，并将容器数据保存在宿主机上的 `/my/host/folder` 目录中。

还可以使用 Docker 卷容器来创建 Docker 容器卷。这种方法可以将数据卷与容器解耦，从而使得容器更易于管理和迁移。以下是使用 Docker 卷容器创建 Docker 容器卷的示例：

```shell
docker volume create myvolume
docker run -d -v myvolume:/my/container/folder myimage
```

在这个示例中，`docker volume create` 命令用于创建一个 Docker 卷，`myvolume` 是卷的名称。`-v` 选项用于挂载容器卷。`myvolume` 是 Docker 卷的名称，`/my/container/folder` 是容器中的目录，`myimage` 是要运行的 Docker 镜像。这条命令将 Docker 卷 `myvolume` 挂载到容器中的 `/my/container/folder` 目录。

## Docker高级

### MySQL主从复制

# Docker

## Docker基础

### Docker的安装与卸载（CentOS 7）

#### 配置镜像源并安装Docker

进入[阿里开源镜像站](https://developer.aliyun.com/mirror/)配置国内镜像源

```shell
# 配置CentOS 7镜像
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# 配置Epel镜像
wget -O /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
# 配置docker-ce镜像
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 使用阿里云的Docker-CE软件源
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
# Step 4: 更新并安装Docker-CE
sudo yum makecache
sudo yum -y install docker-ce
# Step 4: 开启Docker服务
sudo systemctl start docker
sudo systemctl enable docker

# 配置阿里云容器加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://v9ucwxai.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

#### Docker的卸载

```shell
# Step 1: 停止Docker服务
sudo systemctl stop docker
# Step 2: 删除所有Docker容器
sudo docker rm -f $(sudo docker ps -aq)
# Step 3: 卸载Docker软件包
sudo yum remove docker-ce docker-ce-cli containerd.io
# Step 4: 删除Docker配置文件
sudo rm -rf /var/lib/docker
```

### Docker基础命令

#### 基础操作

```shell
# 启动docker
sudo systemctl start docker
# 重启docker
sudo systemctl restart docker
# 停止后台docker服务
sudo systemctl stop docker
# 查看docker运行状况
sudo systemctl status docker 
# 开机启动
sudo systemctl enable docker
# 查看docker概要信息
sudo docker info
# 查看docker全部帮助文档
sudo docker --help
# 查看docker某一命令帮助文档
sudo docker 具体命令 --help
# 查看docker版本
sudo docker version
# 查看镜像/容器/数据卷所占的空间
sudo docker system df
```

#### 镜像操作

```shell
# 列出本地主机上的所有镜像
sudo docker images [options]
# -a：列出本地所有的镜像（含历史镜像）
# -q：只显示镜像ID

# 到远程仓库中搜索xxx镜像
sudo docker search xxx
# sudo docker search xxx --limit 5  # 只显示5条命令

# 到远程仓库中下载xxx镜像
sudo docker pull xxx  # 默认版本为：lastest
sudo docker pull xxx:123  # 指定版本为123

# 删除镜像
sudo docker [options] rmi 镜像名 或者 镜像ID
# 例如 sudo docker rmi nginx
sudo docker [options] rmi  -f 镜像名 或者 镜像ID  # 强制删除
# 删除多个镜像
sudo docker rmi -f 镜像名字1:TAG 镜像名字2:TAG
# 删除全部镜像
sudo docker rmi -f $(docker images -aq)
```

#### 容器操作

```shell
# 启动容器
sudo docker run [options] IMAGE [COMMAND] [ARG...]
# sudo docker run -d --name httpd1 -p8080:80 httpd  # 启动httpd设置容器名为httpd1，端口映射为8080（将80端口映射为8080）
# sudo docker run -d 容器名或者容器ID
# [option]
# --name="容器的新名字"  为容器指定一个名称，没有的话随机分配；
# -d: 后台运行容器并返回容器ID，也即启动守护式容器(后台运行)；
# -i：以交互模式运行容器，通常与 -t 同时使用；（interactive）
# -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用；也即启动交互式容器(前台有伪终端，等待交互)；（tty）
 # -P: 随机端口映射，大写P
 # -p: 指定端口映射，小写p   
 # --restart=always:开机自启
# [COMMAND]
# /bin/bash:放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。
# 要退出终端，直接输入 exit:

# 列出当前正在运行的容器
sudo docker ps [OPTIONS]
# sudo docker ps -a
# [OPTIONS]
# -a: 列出当前所有正在运行的容器+历史上运行过的。
# -l: 显示最近创建的容器。
# -n: 显示最近n个创建的容器。
# -q: 静默模式，只显示容器编号。

# 退出容器（假设已经进入容器内）
exit  # 退出容器且容器停止
ctrl+p+q  # 退出容器但容器不停止

# 修改容器设置
sudo docekr update 容器ID或者容器名 [options]
# [options]同docker run

# 启动已停止运行的容器
sudo docker start 容器ID或者容器名
# 重启容器
sudo docker restart 容器ID或者容器名
# 停止容器
sudo docker stop 容器ID或者容器名
# 强制停止容器
sudo docker kill 容器ID或容器名
# 删除已停止的容器
sudo docker rm 容器ID
# 一次性删除多个容器实例
sudo docker rm -f $(docker ps -aq)
sudo docker ps -a -q | xargs docker rm
# 查看容器日志
sudo docker logs 容器ID
# 查看容器内运行的进程
sudo docker top 容器ID
# 查看容器内部细节
sudo docker inspect 容器ID
# 进入正在运行的容器并以命令行(/bin/bash)交互
sudo docker exec -it 容器ID /bin/bash  # exit不会导致容器的停止
sudo docker attach -it 容器ID /bin/bash  # exit会导致容器的停止
# 从容器内拷贝文件到主机上
sudo docker cp  容器ID:容器内路径 目的主机路径
# 导入和导出容器
# export: 导出容器的内容留作为一个tar归档文件[对应import命令]
# sudo docker export 容器ID > 文件名.tar  # 导出
# import: 从tar包中的内容创建一个新的文件系统再导入为镜像[对应export]
# sudo cat 文件名.tar | docker import - 镜像用户/镜像名:镜像版本号  # 导入
```

#### 发布镜像

##### 发布镜像到云端（阿里云）

```shell
# docker commit提交容器副本使之成为一个新的镜像
sudo docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]
# 登录阿里云镜像仓库Docker Registry
sudo docker login --username=用户名 registry.cn-hongkong.aliyuncs.com
# 将镜像推送到Registry
sudo docker tag [镜像ID] registry.cn-hongkong.aliyuncs.com/仓库名/镜像名:[镜像版本号]
sudo docker push registry.cn-hongkong.aliyuncs.com/仓库名/镜像名:[镜像版本号]

# 从Registry中拉取镜像
sudo docker pull registry.cn-hongkong.aliyuncs.com/仓库名/镜像名:[镜像版本号]
```

##### 发布镜像到私有库

```shell
# 下载镜像Docker Registry
sudo docker pull registry
# 运行私有库Registry，相当于本地有个私有Docker hub
sudo docker run -d -p 5000:5000  -v /wangchen/myregistry/:/tmp/registry --privileged=true registry
# --privileged=true: 给容器授予所有的权限。
# -v /wangchen/myregistry/:/tmp/registry: 将主机上的 /wangchen/myregistry/ 目录挂载到容器中的 /tmp/registry 目录，以便将 Registry 数据存储在主机上。挂载目录可以确保 Registry 数据不会随着容器的销毁而丢失。

# docker commit提交容器副本使之成为一个新的镜像
sudo docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]

# 查看本地私服库上有什么镜像
sudo curl -XGET http://localhost:5000/v2/_catalog

# 将新镜像XXX修改符合私服规范的Tag
docker tag 镜像:Tag localhost:5000/镜像:Tag

# push推送到私服库
sudo docker push localhost:5000/镜像:tag

# 从私服库拉取镜像
sudo docker pull localhost:5000/镜像:tag
```

### Docker容器卷

Docker 容器卷（Volume）是 Docker 中一种用于数据持久化的机制。通过挂载卷，您可以将宿主机上的目录或文件夹挂载到容器中，并将容器中的数据持久化到宿主机上，从而实现容器数据的保存和共享。

使用 Docker 容器卷的好处包括：

- 数据持久化：容器卷可以将容器中的数据保存到宿主机上，从而实现数据持久化，即使容器被删除，数据仍然可以保留。
- 容器数据共享：多个容器可以共享同一个容器卷，这样就可以实现容器之间的数据共享。

以下是使用 Docker 容器卷的示例：

```shell
docker run -d -v /my/host/folder:/my/container/folder myimage
```

在这个示例中，`-v` 选项用于挂载容器卷。`/my/host/folder` 是宿主机上的目录，`/my/container/folder` 是容器中的目录，`myimage` 是要运行的 Docker 镜像。这条命令将宿主机上的 `/my/host/folder` 目录挂载到容器中的 `/my/container/folder` 目录，并将容器数据保存在宿主机上的 `/my/host/folder` 目录中。

还可以使用 Docker 卷容器来创建 Docker 容器卷。这种方法可以将数据卷与容器解耦，从而使得容器更易于管理和迁移。以下是使用 Docker 卷容器创建 Docker 容器卷的示例：

```shell
docker volume create myvolume
docker run -d -v myvolume:/my/container/folder myimage
```

在这个示例中，`docker volume create` 命令用于创建一个 Docker 卷，`myvolume` 是卷的名称。`-v` 选项用于挂载容器卷。`myvolume` 是 Docker 卷的名称，`/my/container/folder` 是容器中的目录，`myimage` 是要运行的 Docker 镜像。这条命令将 Docker 卷 `myvolume` 挂载到容器中的 `/my/container/folder` 目录。

## Docker高级

### MySQL主从复制

#### 1. 准备主数据库容器

```shell
# 下载mysql 5.7
docker pull mysql:5.7
```

```shell
# 运行mysql
docker run --name mysql-master -e MYSQL_ROOT_PASSWORD=数据库root用户密码 -d mysql:5.7
docker run -p 3307:3306 --name mysql-master \
-v /mydata/mysql-master/log:/var/log/mysql \
-v /mydata/mysql-master/data:/var/lib/mysql \
-v /mydata/mysql-master/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root  \
-d mysql:5.7
```

创建了一个名为 `mysql-master` 的容器，并使用了 MySQL 5.7镜像。`-e` 选项用于设置 MySQL 的 root 用户密码，`-d` 选项用于让容器在后台运行。

#### 2. 从数据库容器

```shell
docker run --name mysql-slave -e MYSQL_ROOT_PASSWORD=数据库root用户密码 -d mysql:5.7
```

创建了一个名为 `mysql-slave` 的容器，并使用了 MySQL 5.7镜像。`-e` 选项用于设置 MySQL 的 root 用户密码，`-d` 选项用于让容器在后台运行。

#### 3. 配置主数据库容器

```shell
# 登录主数据库容器
docker exec -it mysql-master mysql -uroot -p
# SQL语句
# 创建用户slave
CREATE USER 'slave'@'%' IDENTIFIED BY '密码';
# 将从库的复制用户slave授权为复制主库数据的合法用户
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%';
FLUSH PRIVILEGES;
# 获取主数据库的二进制日志文件名和位置
SHOW MASTER STATUS;
```

若执行`SHOW MASTER STATUS;`结果为空,可能是因为 MySQL 没有正确配置二进制日志。在默认情况下，MySQL 是不启用二进制日志的。

进入容器内部查看MySQL配置文件位置：

```shell
docker exec -it mysql-master /bin/bash
mysql --help | grep my.cnf
```

![image-20230217194908684](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230217194908684.png)

默认情况下，MySQL 配置文件应该在 `/etc/mysql/my.cnf`。

在容器内部，可以使用 `cat` 命令查看配置文件的内容：

```shell
cat /etc/mysql/my.cnf
```

在配置文件中搜索 `log_bin` 配置项，确认是否已启用二进制日志。如果没有找到 `log_bin` 配置项，则需要在配置文件中添加以下内容：`log_bin=/var/lib/mysql/mysql-bin`

```shell
echo '[mysqld]' >> /etc/mysql/my.cnf
echo 'log_bin=/var/lib/mysql' >> /etc/mysql/my.cnf
echo 'server-id=1' >> /etc/mysql/my.cnf
```

然后，重启主数据库容器：

```shell
docker restart mysql-master
docker exec -it mysql-master 
```

#### 4. 配置从数据库容器

```shell
# 登录从数据库容器
docker exec -it mysql-slave mysql -uroot -p

CHANGE MASTER TO
    MASTER_HOST='mysql-master',
    MASTER_USER='slave',
    MASTER_PASSWORD='密码',
    MASTER_LOG_FILE='[File]',
    MASTER_LOG_POS=[Position];
```

在这些命令中，我们设置从数据库连接到主数据库的信息，包括主机名、用户名、密码、二进制日志文件名和位置。需要将 `[File]` 和 `[Position]` 替换为在主数据库容器中获取的的二进制日志文件名和位置。

