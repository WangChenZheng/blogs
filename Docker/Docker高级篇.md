## Docker高级

### MySQL主从复制

#### 1. 准备主数据库容器

```shell
# 下载mysql 5.7
docker pull mysql:5.7
```

```shell
# 建立主服务器容器实例 端口3307
docker run -p 3307:3306 --name mysql-master \
-v /mydata/mysql-master/log:/var/log/mysql \
-v /mydata/mysql-master/data:/var/lib/mysql \
-v /mydata/mysql-master/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=数据库root用户密码 \
-d mysql:5.7
```

创建了一个名为 `mysql-master` 的容器，并使用了 MySQL 5.7镜像。`-v`选项用于挂在容器卷，`-e` 选项用于设置 MySQL 的 root 用户密码，`-d` 选项用于让容器在后台运行。

#### 2. 配置主数据库容器

```shell
# 修改配置文件，由于使用容器卷则可在容器外修改配置
cd /mydata/mysql-master/conf
vim my.cnf
```

以下为`my.cnf`文件内容

```
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=1
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能
log-bin=mall-mysql-bin
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
## 设置使用的二进制日志格式（mixed, statement, row）
binlog_format=mixed
## 二进制日志国企清理时间，默认值为0，表示不自动清理
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断
## 如：1062错误时指一些主键重复，1032错误时因为主从数据库数据不一致
slave_skip_errors=1062
```

重启主数据库容器

```shell
docker restart mysql-master
```

进入主数据库容器

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

![image-20230217210634328](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230217210634328.png)

若执行`SHOW MASTER STATUS;`结果为空,可能是因为 MySQL 没有正确配置二进制日志。

#### 3. 准备从数据库容器

```shell
# 建立主服务器容器实例 端口3308
docker run -p 3308:3306 --name mysql-slave \
-v /mydata/mysql-slave/log:/var/log/mysql \
-v /mydata/mysql-slave/data:/var/lib/mysql \
-v /mydata/mysql-slave/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=数据库root用户密码 \
-d mysql:5.7
```

创建了一个名为 `mysql-slave` 的容器，并使用了 MySQL 5.7镜像。`-v`选项用于挂在容器卷，`-e` 选项用于设置 MySQL 的 root 用户密码，`-d` 选项用于让容器在后台运行。

#### 4. 配置从数据库容器

```shell
# 修改配置文件，由于使用容器卷则可在容器外修改配置
cd /mydata/mysql-slave/conf
vim my.cnf
```

以下为`my.cnf`文件内容

```
[mysqld]
## 设置server_id，同一局域网中需要唯一
server_id=2
## 指定不需要同步的数据库名称
binlog-ignore-db=mysql
## 开启二进制日志功能
log-bin=mall-mysql-slave1-bin
## 设置二进制日志使用内存大小（事务）
binlog_cache_size=1M
## 设置使用的二进制日志格式（mixed, statement, row）
binlog_format=mixed
## 二进制日志国企清理时间，默认值为0，表示不自动清理
expire_logs_days=7
## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断
## 如：1062错误时指一些主键重复，1032错误时因为主从数据库数据不一致
slave_skip_errors=1062
## relay_log配置中继日志
relay_log=mall-mysql-relay-bin
## log_slave_updates表示slave讲复制事件写进自己的二进制日志
log_slave_updates=1
## slave设置为只读（具有super权限的用户除外）
read_only=1
```

重启主数据库容器

```shell
docker restart mysql-slave
```

进入从数据库容器前，需要先查看主数据库容器中二进制日志文件名和位置。

```shell
docker exec -it mysql-master mysql -u root -p -e "SHOW MASTER STATUS";
```

![image-20230217210634328](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230217210634328.png)

```shell
# 登录从数据库容器
docker exec -it mysql-slave mysql -uroot -p
CHANGE MASTER TO
    MASTER_HOST='192.168.111.133',
    MASTER_USER='slave',
    MASTER_PASSWORD='slave',
    MASTER_PORT=3307,
    MASTER_LOG_FILE='mall-mysql-bin.000001',
    MASTER_LOG_POS=749,
    MASTER_CONNECT_RETRY=30;
```

+ `MASTER_HOST`：主数据库IP地址；
+ `MASTER_USER`： 主数据库运行端口；
+ `MASTER_PASSWORD`： 在主数据库创建的用于同步数据的用户账号；
+ `MASTER_PORT`：在主数据库创建的用于同步数据的用户密码；
+ `MASTER_LOG_FILE`：指定从数据库要复制数据的日志文件，通过查看主数据库状态获取FILE参数；
+ `MASTER_LOG_POS`：指定从数据库从哪个位置开始复制数据，通过查看主数据库状态，获取POS参数；
+ `MASTER_CONNECT_RETRY`：连接失败重试的时间间隔

![image-20230217212507008](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230217212507008.png)

#### 5. 查看主从同步状态

```shell
# 进入从数据库容器
docker exec -it mysql-slave mysql -uroot -p
# 查看主从同步状态  \G是为了便于查看
SHOW SLAVE STATUS \G;
```

![image-20230217213427726](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230217213427726.png)

#### 6. 在从数据库中开启主从同步

```shell
# 进入从数据库容器
docker exec -it mysql-slave mysql -uroot -p
# 开启主从同步
START SLAVE;
```

![image-20230217213628505](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230217213628505.png)

```SQL
# 查看主从同步状态
SHOW SLAVE STATUS \G;
```

![image-20230217215546146](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230217215546146.png)

#### 7. 测试主从复制

![image-20230217215812735](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230217215812735.png)
