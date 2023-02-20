## Git

### 1. Git概述

Git 是一个免费的、开源的分布式版本控制系统，可以快速高效地处理从小型到大型的各种项目。Git 易于学习，占地面积小，性能极快。 它具有廉价的本地库，方便的暂存区域和多个工作流分支等特性。其性能优于 Subversion、CVS、Perforce 和 ClearCase 等版本控制工具。

### 2.  Git安装

官网地址：<https://git-scm.com/>

![image-20230218083734145](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218083734145.png)

![image-20230218083753226](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218083753226.png)

![image-20230218083804692](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218083804692.png)

一路Next即可。

![image-20230218084201254](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218084201254.png)

### 3. Git常用命令

| 命令名                                 | 作用           |
| -------------------------------------- | -------------- |
| `git config --global user.name 用户名` | 设置用户签名   |
| `git config --global user.email 邮箱 ` | 设置用户签名   |
| `git init`                             | 初始化本地库   |
| `git status `                          | 查看本地库状态 |
| `git add 文件名 `                      | 添加到暂存区   |
| `git commit -m "日志信息" 文件名`      | 提交到本地库   |
| `git reflog `                          | 查看历史记录   |
| `git reset --hard 版本号 `             | 版本穿梭       |

#### 3.1 设置用户签名

 基本语法

+ `git config --global user.name 用户名`
+ `git config --global user.email 邮箱`

![image-20230218085146411](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218085146411.png)

查看签名设置

`cat ~/.gitconfig`（注意要使用Git Bash，因为cmd并不支持cat命令）

![image-20230218085545193](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218085545193.png)

说明：签名的作用是区分不同操作者身份。用户的签名信息在每一个版本的提交信息中能够看到，以此确认本次提交是谁做的。Git 首次安装必须设置一下用户签名，否则无法提交代码。

**※注意：**这里设置用户签名和将来登录 GitHub（或其他代码托管中心）的账号没有任何关系。

#### 3.2 初始化本地库

```shell
mkdir test1
cd test1
git init
```

![image-20230218085823197](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218085823197.png)

可以看到test1文件夹下生成了.git文件夹

![image-20230218085945102](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218085945102.png)

#### 3.3 查看本地库命令

```shell
# 首次查看，工作区没有任何文件
git status
```

![image-20230218090031900](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218090031900.png)

```shell
# 创建hello.txt，并在文件内写入hello oworld
echo 'hello world' > hello.txt
# 查看文件
ls -la
# 查看本地库状态（检测到未追踪的文件）
git status
```

![image-20230218090149767](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218090149767.png)

#### 3.4 添加暂存区

`git add 文件名`将工作区的文件添加到暂存区

```shell
git add hello.txt
git status
```

![image-20230218090437988](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218090437988.png)

#### 3.5 提交本地库

##### 3.5.1 提交本地库

```shell
git commit -m "日志信息" 文件名
# 查看本地库状态
git status
```

![image-20230218090649923](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218090649923.png)

##### 3.5.2 修改文件

```shell
# 在hello.txt文件末尾追加ypdate file
echo 'update file' >> hello.txt
```

![image-20230218090900999](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218090900999.png)

##### 3.5.3 再次添加暂存区并提交

```shell
git add hello.txt
git status
git commit -m "update file" hello.txt
git status
```

![image-20230218091046360](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218091046360.png)

![image-20230218091222860](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218091222860.png)

#### 3.6 查看历史版本

+ `git reflog` 查看版本信息
+ `git log` 查看版本详细信息

![image-20230218091354473](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218091354473.png)

#### 3.7 版本穿梭

+ `git reset --hard 版本号`

```shell
# 查看当前版本hello.txt内容
cat hello.txt
# 查看版本日志
git reflog
# 版本穿梭
git reset --hard 版本号
# 再次查看hello.txt内容
cat hello.txt
```

![image-20230218091715929](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218091715929.png)

发现文件hello.txt的内容变成第一次提交时的内容。

### 4. Git分支操作

#### 4.1 Git分支概述

![image-20230218092107499](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218092107499.png)

在版本控制过程中，同时推进多个任务，对于每个任务，我们就可以创建每个任务的单独分支。使用分支意味着程序员可以把自己的工作从开发主线上分离开来，开发自己分支的时候，不会影响主线分支的运行。对于初学者而言，分支可以简单理解为副本，一个分支就是一个单独的副本。（分支底层其实也是指针的引用）

#### 4.2 Git分支操作

| 命令名称              | 作用                         |
| --------------------- | ---------------------------- |
| `git branch 分支名`   | 创建分支                     |
| `git branch -v`       | 查看分支                     |
| `git checkout 分支名` | 切换分支                     |
| `git merge 分支名`    | 把指定的分支合并到当前分支上 |

##### 4.2.1 查看分支

+ `git branch -v`

![image-20230218092535345](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218092535345.png)

##### 4.2.2 创建分支

+ `git branch 分支名`

```shell
# 在当前分支上创建名为dev的分支
git branch dev
```

![image-20230218092658598](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218092658598.png)

##### 4.2.3 切换分支

+ `git checkout 分支名`

```shell
# 切换至dev分支
git checkout dev
# 在hello.txt文件添加dev branch 1内容
echo 'dev branch 1' >> hello.txt
# 查看hello.txt文件内容
cat hello.txt
# 提交
git add hello.txt
git commit -m "dev finish" hello.txt
```

![image-20230218093212353](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218093212353.png)

![image-20230218093230697](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218093230697.png)

```shell
# 切换会master分支
git checkout master
# 查看hello.txt文本内容
cat hello.txt
```

![image-20230218093404581](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218093404581.png)

发现master分支上的hello.txt文本内容并未改变

##### 4.2.4 合并分支

+ `git merge 分支名`

```shell
# 要将dev分支内容合并到master分支
# 切换至master分支
git checkout master
# 合并dev分支
git merge dev
# 查看hello.txt文本内容
cat hello.txt
```

![image-20230218093936286](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218093936286.png)

##### 4.2.5 冲突处理

```shell
# 创建分支fix
git branch fix
# 切换fix分支
git checkout fix
# 修改hello.txt文本内容，将dev修改为fix
vim hello.txt
```

![image-20230218095451376](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218095451376.png)

```shell
# 提交
git add hello.txt
git commit -m "fix branch update" hello.txt
# 切换至master分支
git checkout master
# 修改hello.txt文本内容，将dev修改为master
vim hello.txt
```

![image-20230218095606842](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218095606842.png)

```shell
# 提交
git add hello.txt
git commit -m "master branch update" hello.txt
# 合并fix分支
git merge fix
```

![image-20230218095727899](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218095727899.png)

发生冲突(MERGING)

冲突产生的原因：合并分支时，两个分支在**同一个文件的同一个位置**有两套完全不同的修改。Git 无法替我们决定使用哪一个。必须**人为决定**新代码内容。

```shell
# 查看状态（检测到有文件有两处修改）
git status
```

![image-20230218095900049](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218095900049.png)

**解决冲突**

```shell
# 进入发生冲突的文件
vim hello.txt
```

![image-20230218095957185](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218095957185.png)

删除特殊符号，决定要使用的内容

![image-20230218100041240](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218100041240.png)

```shell
# 提交，此时不能携带文件名
git commit -m "merge fix"
```

![image-20230218100245945](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218100245945.png)

发现冲突不再提示冲突。

### 5. Git团队协作机制

#### 5.1 团队内协作

![image-20230218100515837](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218100515837.png)

#### 5.2 跨团队协作

![image-20230218100536387](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218100536387.png)

### 6. Github操作(Gitee演示)

#### 6.1 远程仓库操作

| 命令名称                           | 作用                                                     |
| :--------------------------------- | -------------------------------------------------------- |
| git remote -v                      | 查看当前所有远程地址别名                                 |
| git remote add 别名 远程地址       | 起别名                                                   |
| git push 别名 分支                 | 推送本地分支上的内容到远程仓库                           |
| git clone 远程地址                 | 将远程仓库的内容克隆到本地                               |
| git pull 远程库地址别名 远程分支名 | 将远程仓库对于分支最新内容拉下来后与当前本地分支直接合并 |

![image-20230218135753757](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218135753757.png)

这里以仓库`https://gitee.com/wang-chen-zheng/gittest.git`为例

##### 6.1.1 查看远程仓库别名

`git remote -v` 

若结果为空则代表当前无远程仓库

##### 6.1.2 添加远程仓库

`git remote add 别名 远程地址`

```shell
git remote add origin https://gitee.com/wang-chen-zheng/gittest.git
```

![image-20230218140156771](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218140156771.png)

##### 6.1.3 将本地库推送至远程仓库

`git push 别名 分支`

```shell
git push origin master
```

##### 6.1.4 克隆远程仓库到本地

`git clone 远程地址`

![image-20230218180131880](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218180131880.png)

![image-20230218180145853](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218180145853.png)

![image-20230218180216832](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218180216832.png)

##### 6.1.5 拉取远程仓库内容

`git pull 远程地址别名 远程分支名`

```shell
# 修改本地仓库内容并上传至远程仓库
# 在hello.txt后添加一行pull test
git add .
git commit -m "pull test"
git push origin master
```

![image-20230218180406222](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218180406222.png)

![image-20230218180553172](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218180553172.png)

```shell
# 切换至刚才clone的文件夹
# 查看hello.txt内容
cat hello.txt
```

![image-20230218180725246](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218180725246.png)

```shell
# 拉取远程仓库内容
git pull origin master
# 再次查看hello.txt内容，发现已经更新了
cat hello.txt
```

![image-20230218180831428](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230218180831428.png)
