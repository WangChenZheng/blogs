## LAMP架构搭建农场

### 配置本地仓库

#### 挂在光盘

![image-20230216090511292](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216090511292.png)

![image-20230216090407367](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216090407367.png)

#### 建立本地软件仓库

```shell
# 备份原仓库
cd /etc/yum.repos.d/
mkdir /backup
mv ./* /backup
vim mycd.repo
# 以下为mycd.repo内容
[mycdrepo]
name=mycdrepo
baseurl=file:///mnt
enabled=1
gpgcheck=0
# 以上为mycd.repo内容

# 将ios光盘挂在到虚拟机中
mount /dev/cdrom /mnt
```

![image-20230216090851310](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216090851310.png)

### 安装Mariadb并启动

```shell
# 安装
yum install -y mariadb mariadb-server
systemctl start mariadb  # 启动mariadb
systemctl enable mariadb  # 开机启动mariadb
systemctl enable --now mariadb  # 等同上两步
# 启动
# 安全初始化，根据提示输入y/n即可
mysql_secure_installation
# 连接数据库  mysql -u用户名 -p密码
mysql -u用户名 -p密码
```

![image-20230216091202700](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216091202700.png)

![image-20230216091358795](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216091358795.png)

![image-20230216091459240](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216091459240.png)

### 关闭防火墙以及SELinux

```shell
# 关闭防火墙
systemctl stop firewalld  # 关闭防火墙
systemctl disable firewalld  # 禁用防火墙
systemctl mask firewalld  # 彻底禁用防火墙，不允许其他程序唤起

# 关闭selinux
vim /etc/selinux/config
# config文件内容修改为
SELINUX=disabled
# 以上config文件内容
setenforce 0

# 防火墙放行
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```



### 安装Apache HTTP服务器以及PHP相关依赖并启动

```shell
yum install -y httpd php php-mysql
systemctl enable --now httpd
```

![image-20230216091750859](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216091750859.png)

### 将农场程序包上传至Linux服务器并解压

```shell
# 百度网盘
链接：https://pan.baidu.com/s/1TFT-532vgtcKWSito-Meqw?pwd=75u2 
提取码：75u2
# 解压缩
unzip farm-ucenter1.5.zip
mv upload/ /var/www/html/farm  # 将程序移至/var/www/html下并重命名为farm
```

![image-20230216091939706](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216091939706.png)

### 创建农场程序数据库

```shell
# 登录数据库
mysql -u 用户名（root） -p密码
create database qqfarm;  # 创建数据库
 # 创建用户farmer并设置其密码为redhat，并将qqfarm库的所有权限赋予该用户
create user 'farmer'@'%' identified by 'redhat'; 
grant all on qqfarm.* to 'farmer'@'%';
# 该命令与前两个命令意思相同
grant all on qqfarm.* to 'farmer'@'%' identified by 'redhat';
# 刷新权限表
flush privileges;
mysql -u用户名 -p密码 qqfarm < /var/www/html/farm/qqfarm.sql
```

![image-20230216092103897](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216092103897.png)

![image-20230216092213084](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216092213084.png)

### 安装农场程序

```shell
# 在浏览器中输入以下地址 其中server_ip为服务器ip地址
http://server_ip/farm
```

![image-20230216092428614](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216092428614.png)

```shell
# 提示如上错误
vim /etc/php.ini
# 将php.ini内容修改为
short_open_tag = On
# 以上为php.ini内容
 # 重启http服务
systemctl restart httpd
# 刷新网站
```

![image-20230216092601119](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216092601119.png)

```shell
# 提示如上错误
chown -R apache:apache /var/www/html/farm/ 
# 刷新浏览器
```

![image-20230216092656735](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216092656735.png)

![image-20230216092851665](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216092851665.png)

点击安装即可，等待安装完成即可。

![image-20230216093536337](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216093536337.png)

![image-20230216093553437](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230216093553437.png)

