## Docker

### Docker基础

#### Docker的安装与卸载（CentOS 7）

##### 配置镜像源并安装Docker

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

##### Docker的卸载

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

#### Docker基础命令

##### 基础操作

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

##### 镜像操作

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

##### 容器操作

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

##### 发布镜像

```shell
```





```shell
cd /etc/yum.repos.d/
ls
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
wget -O /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
yum makecache fast
yum -y install docker-ce
systemctl start docker
systemctl enable docker
docker version
systemctl status docker
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
systemctl restart docker
cat /etc/docker/daemon.json 
```



 