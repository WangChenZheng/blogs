# 实习记录

## 1 什么是云计算

## 2 Linux在云计算当中的地位

## 3 部署CentOS7基础环境

### 安装VMware

### 安装CentOS7虚拟机

## 4 Linux命令行管理

### 4.1 Shell

### 4.2 命令提示符

打开终端后会看到如下格式的提示

![QQ截图20230213105845.png](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/QQ%E6%88%AA%E5%9B%BE20230213105845.png)

其基本格式为：`[登陆系统用户名@主机名称 家目录]用户身份`

+ 用户身份
  
  + $ 表示普通用户
  
  + \# 表示root用户

### 4.3 基础管理命令

```shell
# 当前登录系统的用户
whoami
# 当前目录
pwd
# 显示目前登入系统的用户信息
w
# 列出当前目录文件列表
ls
# 列出当前目录文件列表详细信息
ll
ls -l
# 当前系统时间
date
# 显示当前终端下的所有程序，包括其他用户程序
ps -aux
# 等等
```

### 4.4 命令基本组成格式

`命令名称 选项 参数`

示例：

```shell
ls
ls -l
ls -i
ls -R
ls -l /etc  # 输出/etc文件夹下的文件详细信息
ls -l /mnt
```

## 5 Linux权限管理

### 5.1 基本权限

+ read --- r
+ write --- w
+ execute --- x

```shell
cd /tmp  # 进入/tmp 文件夹
rm -rf ./*  # 清空当前文件夹内文件
echo 'hello world' > myfile  # 将'hello world'存入myfile文件
```



![image-20230213142253274](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230213142253274.png)

文件myfile的文件权限为`-rw-r--r--`，其格式为所属用户权限(rwx)，所属用户组权限(rwx)，其他用户权限(rwx)

可知文件myfile文件所属用户（即root）的权限为rw，即可读可写；所属用户组以及其他用户的权限为r，即仅可读。



为更好理解不同用户的可读可写可执行权限，现添加两个用户进行测试。

```shell
useradd zhangsan  # 添加用户zhangsan
useradd lisi  # 添加用户lisi
su zhangsan  # su 用户名
```

![image-20230213143406300](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230213143406300.png)

```shell
# 使用zhangsan用户在文件myfile中追加内容'zhangsan'
echo 'zhangsan' >> myfile  # 从图中可知，权限不足
# 使用zhangsan用户查看文件内容
cat myfile
```

![image-20230213145606646](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230213145606646.png)



管理权限`chmod 用户+-=权限 文件` 

+ 其中用户可分为：所属用户(u)，所属用户组(g)，其他用户(o)

+ 权限可分为：rwx

示例：`chmod o+w myfile`为其他用户添加可写权限

![image-20230213151907706](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230213151907706.png)

其他示例：

```shell
chmod g=rw myfile  # 将所属用户组的权限设为rw
chmod g+x myfile  # 为所属用户组添加w权限
chmod ug=rwx myfile  # 将所属用户以及所属用户组权限设为rwx
```



注：这里还有一个我比较快捷的方法

```shell
# 将myfile文件权限修改为-rwxr-xr-x，即所属用户可读可写可执行，所属用户组以及其他用户可读可执行
chmod 755 myfile  # 755指的是二进制数111 101 101三位一组表示的十进制数，111即rwx，101即r-x
# 同理 1即--x等等，就是将rwx翻译为二进制，若有相应权限则为1否则为0
```

### 5.2 高级权限

* SUID

    * 作用：借用对应文件所属用户的身份来执行
    * 作用位置：**user**
    * 作用对象：**二进制可执行文件**

    ```shell
    # 在所属用户下执行该命令
    chmod u+s /usr/bin/cat
    
    # 添加用户zhangsan
    useradd zhangsan
    cd /usr/tmp
    ls
    rm -rf ./*
    touch test
    ls -l
    vim test
    # 以下为test内容
    hello world
    # 以上为test内容
    
    # 将test文件权限修改为rw-------
    chmod 600 test
    ls -l
    # 切换至zhangsan
    su zhangsan
    cat test
    # 输出test文件内容
    # 切换root用户
    su root
    # 在所属用户下执行该命令
    chmod u+s /usr/bin/cat
    # 切换zhangsan用户
    su zhangsan
    # 输出test文件内容
    cat test
    ```

    > - 只有可以执行的二进制程序才能设定 SUID 权限
    > - 命令执行者要对该程序拥有 x（执行）权限
    > - 命令执行者在执行该程序时获得该程序文件属主的身份（在执行程序的过程中灵魂附体为文件的属主）
    > - SetUID 权限只在该程序执行过程中有效，也就是说身份改变只在程序执行过程中有效

    ![image-20230215100757355](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230215100757355.png)

    ![image-20230215101120806](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230215101120806.png)

* SGID

    * 作用：共享目录
    * 作用位置：**group**

    ```shell
    # 新创建的文件划分到统一用户组root
    chmod g+s 目录
    ```

    > - 普通用户必须对此目录拥有 r 和 x 的权限，才能进入此目录
    > - 普通用户在此目录中的有效组会变成此目录的所属组
    > - 若普通用户对此目录拥有 w 权限时，新建的文件的默认所属组是这个目录的所属组

* STICKY

    * 作用：防止互殴
    * 位置：**other**
    * 作用对象：目录

    ```shell
    # 防止误删
    chmod o+t rootdir
    ```


### 5.3 特殊权限


