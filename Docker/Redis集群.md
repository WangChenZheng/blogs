## Docker高级篇-Redis集群

### 建立Redis集群

#### 1.安装Redis

```shell
docker pull redis
```

#### 2. 新建多个Redis实例（6个）

```shell
docker run -d --name redis-node-1 --net host --privileged=true -v /data/redis/share/redis-node-1:/data redis --cluster-enabled yes --appendonly yes --port 6381

docker run -d --name redis-node-2 --net host --privileged=true -v /data/redis/share/redis-node-2:/data redis --cluster-enabled yes --appendonly yes --port 6382

docker run -d --name redis-node-3 --net host --privileged=true -v /data/redis/share/redis-node-3:/data redis --cluster-enabled yes --appendonly yes --port 6383

docker run -d --name redis-node-4 --net host --privileged=true -v /data/redis/share/redis-node-4:/data redis --cluster-enabled yes --appendonly yes --port 6384

docker run -d --name redis-node-5 --net host --privileged=true -v /data/redis/share/redis-node-5:/data redis --cluster-enabled yes --appendonly yes --port 6385

docker run -d --name redis-node-6 --net host --privileged=true -v /data/redis/share/redis-node-6:/data redis --cluster-enabled yes --appendonly yes --port 6386

# 查看运行状况
docker ps
```

![image-20230220094512986](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220094512986.png)

- `-d`：在后台运行容器。
- `--name redis-node-6`：指定容器的名称为 `redis-node-6`。
- `--net host`：使用主机的网络命名空间，容器将使用主机的网络配置，包括 IP 地址和端口号等。
- `--privileged=true`：容器将拥有主机的全部特权，包括访问主机的设备和文件系统等。
- `-v /data/redis/share/redis-node-6:/data`：将主机上的目录 `/data/redis/share/redis-node-6` 挂载到容器内部的目录 `/data`，实现主机和容器之间的数据共享。
- `redis`：指定使用的镜像为 Redis。
- `--cluster-enabled yes`：开启 Redis 集群模式。
- `--appendonly yes`：开启 Redis 持久化模式，将写入的数据保存到磁盘上。
- `--port 6386`：将容器内部的 Redis 服务端口号设置为 `6381`。

#### 3. 建立集群关系（3主3从）

```shell
# 进入容器1
docker exec -it redis-node-1 /bin/bash
# 构建主从关系，注意要使用自己的IP
redis-cli --cluster create 192.168.111.133:6381 192.168.111.133:6382 192.168.111.133:6383 192.168.111.133:6384 192.168.111.133:6385 192.168.111.133:6386 --cluster-replicas 1
```

- `--cluster create`：表示创建 Redis 集群。
- `192.168.111.133:6381 192.168.111.133:6382 192.168.111.133:6383 192.168.111.133:6384 192.168.111.133:6385 192.168.111.133:6386`：指定 Redis 集群中的节点列表，这里指定了六个节点。
- `--cluster-replicas 1`：指定每个主节点的从节点个数为 1。

这个命令将会在当前 Redis 集群中创建六个节点，每个节点使用的是前面通过 Docker 启动的 Redis 容器，在这里分别使用了六个节点的 IP 地址和端口号。其中，前五个节点为主节点，最后一个节点为没有分配槽位的备用节点。每个主节点都会有一个从节点，因此集群中一共有十一个节点。从节点的数量可以通过 `--cluster-replicas` 参数指定，这里指定每个主节点有一个从节点。执行该命令后，`redis-cli` 会自动将节点之间的拓扑信息通过 Gossip 协议进行传播，建立起节点之间的连接关系。

哈希槽分配结果：

![image-20230220111926700](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220111926700.png)

主从关系：

![image-20230220112105125](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220112105125.png)

```shell
# 进入redis-node-1节点，查看集群状态
redis-cli -p 6381
# 查看集群信息
cluster info
# 查看集群节点信息
cluster nodes
```

![image-20230220112443485](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220112443485.png)

![image-20230220112516332](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220112516332.png)

根据上图即可得知：

+ 主节点node-1的从节点为node-4
+ 主节点node-2的从节点为node-5
+ 主节点node-3的从节点为node-6

### 主从容错切换迁移

#### 1. 数据读写存储

```shell
# 进入redis-node-1容器并连接redis服务器
docker exec -it redis-node-1 /bin/bash
redis-cli -p 6381
# 查看当前所有键值对
keys *
```

![image-20230220113343835](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220113343835.png)

```shell
# 插入数据
set k1 v1
set k2 v2
set k3 v3
set k4 v4
```

![image-20230220113608819](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220113608819.png)

由于k1与k4所计算的槽编号不在该redis服务器内（0-5460内），所以写入失败。

解决方法：不要使用单机模式连接redis服务器`redis-cli -p 6381`，要使用集群模式`redis-cli -o 6381 -c`

![image-20230220113911409](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220113911409.png)

```shell
# 检查集群信息
redis-cli --cluster check 192.168.111.133:6381
```

![image-20230220114306433](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220114306433.png)

#### 2. 容错切换迁移

```shell
# 连接redis-node-1的redis服务器
redis-cli -p 6381
# 查看redis集群节点信息
cluster nodes
```

![image-20230220114613620](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220114613620.png)

```shell
# 人为停掉redis-node-1，以便查看主从容错切换
docker stop redis-node-1
```

![image-20230220114723769](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220114723769.png)

```shell
# 进入redis-node-2并连接redis服务器
docker exec -it redis-node-2 /bin/bash
redis-cli -p 6382 -c
# 查看集群节点信息
cluster nodes
```

![image-20230220115008303](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220115008303.png)

可以看到redis-node-1的redis服务器已经宕机，它的从节点redis-node-4变为master(主节点)。

```shell
# 检查数据是否仍然存在
get k1
get k2
get k3
get k4
```

![image-20230220115217710](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220115217710.png)

还原之前的三主三从：

```shell
# 启动redis-node-1容器
docker start redis-node-1
# 查看主从关系
docker exec -it redis-node-1 /bin/bash
redis-cli -p 6381 -c
clster nodes
```

![image-20230220115412974](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220115412974.png)

![image-20230220115643142](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220115643142.png)

发现redis-node-4仍然是主节点，而redis-node-1变为从节点。

```shell
# 恢复redis-node-1主节点的身份
# 先停掉redis-node-4在启动
docker stop redis-node-4
docker exec -it redis-node-1 /bin/bash
redis-cli -p 6381 -c
clster nodes
# 给redis-node-1重新上位的机会
```

![image-20230220120108657](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220120108657.png)

```shell
# 重新启动redis-node-4
docker start redis-node-4
```

![image-20230220120219705](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220120219705.png)

### 主从扩容

需求：将redis-node-7（端口6387）与redis-node-8（端口6388）加入集群，并将redis-node-8作为redis-node-7的从节点。

```shell
# 再启动两台redis服务器
docker run -d --name redis-node-7 --net host --privileged=true -v /data/redis/share/redis-node-7:/data redis --cluster-enabled yes --appendonly yes --port 6387
docker run -d --name redis-node-8 --net host --privileged=true -v /data/redis/share/redis-node-8:/data redis --cluster-enabled yes --appendonly yes --port 6388
# 查看是否启动成功
docker ps
```

![image-20230220120817315](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220120817315.png)

```shell
# 进入7号机
docker exec -it redis-node-7 /bin/bash
# 将本节点加入集群
# redis-cli --cluster add-node 本机IP:端口 集群中某主机IP:端口
redis-cli --cluster add-node 192.168.111.133:6387 192.168.111.133:6381
```

![image-20230220121238388](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220121238388.png)

```shell
# 检查集群情况
redis-cli --cluster check 192.168.111.133:6381
```

![image-20230220121415284](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220121415284.png)

发现7号机已经成功加入集群但并未被分配槽位。

```shell
# 重新分配槽号
# redis-cli --cluster reshard 集群某主机IP:端口
redis-cli --cluster reshard 192.168.111.133:6381
```

![image-20230220121846020](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220121846020.png)

What is the receiving node ID？应输入7号机的ID: xxxx666cb

Source node: all

```shell
# 查看分配结果
redis-cli --cluster check 192.168.111.133:6381
```

![image-20230220122349593](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220122349593.png)

发现分配方式是将原来三台主机的哈希槽各分出一部分来给7号机（第四台主机）。

```shell
# 为7号机分配从节点redis-node-8
# redis-cli --cluster add-node 从机IP:端口 主机IP:端口 --cluster-slave --cluster-master-id 主机ID
redis-cli --cluster add-node 192.168.111.133:6388 192.168.111.133:6387 --cluster-slave --cluster-master-id 8c04742e8589c814f0e6487100483e596ba666cb
# 查看集群节点信息
redis-cli -p 6387 -c
cluster nodes
```

![image-20230220140654374](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220140654374.png)

![image-20230220140725352](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220140725352.png)

### 主从缩容

需求：流量降低，删除7号机和8号机，恢复三主三从

```shell
# 从集群中删除8号机（端口6388）
# redis-cli --cluster del-node ip:端口 节点ID
redis-cli --cluster del-node 192.168.111.133:6388 1bfec5063bcf0af1e07bbc283cb2521eb6ec7090
# 查看集群节点信息
redis-cli --cluster check 192.168.111.133:6381
```

![image-20230220141845093](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220141845093.png)

![image-20230220141900477](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220141900477.png)

发现端口号为6388的机器已经不在集群中了。

```shell
# 将6387的哈希槽清空，重新分配，本例将请出来的槽号都给端口为6381的机器
# 重新分配槽号
redis-cli --cluster reshard 192.168.111.133:6381
```

![image-20230220142601858](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220142601858.png)

```shell
# 查看集群主机信息
redis-cli --cluster check 192.168.111.133:6381
```

![image-20230220143022551](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220143022551.png)

发现7号机的槽被分配给了1号机。

```shell
# 删除7号机
redis-cli --cluster del-node 192.168.111.133:6381 8c04742e8589c814f0e6487100483e596ba666cb
# 再次查看集群
redis-cli --cluster check 192.168.111.133:6381
```

![image-20230220143222487](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230220143222487.png)

发现恢复到了原来三主三从的情况。
