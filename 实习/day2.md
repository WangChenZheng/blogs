## shell编程

### shell概述

#### 什么是shell

shell 俗称壳，是一个用 C 语言编写的程序，它是用户和操作系统内核之间的桥梁，其主要作用是为用户提供一种可以和操作系统内核交互的接口。shell 既是一种命令语言，又是一种程序设计语言。使用shell 语法编写的程序我们称之为shell 脚本。

#### 什么是shell脚本

Shell Script，Shell脚本与Windows/Dos下的批处理相似，也就是用各类命令预先放入到一个文件中，方便一次性执行的一个程序文件，主要是方便管理员进行设置或者管理用的。但是它比Windows下的批处理更强大，比用其他编程程序编辑的程序效率更高，毕竟它使用了Linux/Unix下的命令。    

换一种说法也就是，shell script是利用shell的功能所写的一个程序，这个程序是使用纯文本文件，将一些shell的语法与指令写在里面，然后用正规表示法，管线命令以及数据流重导向等功能，以达到我们所想要的处理目的。

#### shell与shell脚本的区别

shell是什么呢？确切一点说，Shell就是一个命令行解释器，它的作用就是遵循一定的语法将输入的命令加以解释并传给系统。它为用户提供了一个向Linux发送请求以便运行程序的接口系统级程序，用户可以用Shell来启动、挂起、停止甚至是编写一些程序。 Shell本身是一个用C语言编写的程序，它是用户使用Linux的桥梁。Shell既是一种命令语言，又是一种程序设计语言(就是你所说的shell脚本)。作为命令语言，它互动式地解释和执行用户输入的命令；作为程序设计语言，它定义了各种变量和参数，并提供了许多在高阶语言中才具有的控制结构，包括循环和分支。它虽然不是 Linux系统内核的一部分，但它调用了系统内核的大部分功能来执行程序、创建文档并以并行的方式协调各个程序的运行。

#### 交互式shell和非交互式shell

交互式模式就是shell等待你的输入，并且执行你提交的命令。这种模式被称作交互式是因为shell与用户进行交互。这种模式也是大多数用户非常熟悉的：登录、执行一些命令、签退。当你签退后，shell也终止了。

shell也可以运行在另外一种模式：非交互式模式。在这种模式下，shell不与你进行交互，而是读取存放在文件中的命令，并且执行它们。当它读到文件的结尾，shell也就终止了。

### shell脚本

#### 输出命令（echo）

`echo -n 'hello world'`输出hello world且不换行

`echo "current network is: $(ifconfig)"`输出当前网络情况（必须使用双引号）

`echo -e 'hello\nworld\n'` 可以将\n解释为特殊字符

+ 不用引号：字符串数组原状输出，变量能被更换

+ 单引号：引号里边的内容会一成不变的显现出来

+ 双引号：里边特殊符号能被分析，变量就会被更换

![image-20230214161847048](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214161847048.png)

#### 多命令连接符（; && ||）

+ 分号 (;)：用于将多个命令放在同一行中执行，不考虑上一个命令的返回状态。
+ 逻辑与 (&&)：用于将多个命令连接起来，只有前一个命令成功执行，才会执行下一个命令。
+ 逻辑或 (||)：用于将多个命令连接起来，只有前一个命令执行失败，才会执行下一个命令。

```shell
# 显示或设置系统的主机名。
hostname
# 显示当前系统时间
date
# 显示当前用户的用户名
whoami
# 显示当前系统路径
pwd
# 同时执行多条命令
hostname;date;whoami;pwd
```

![image-20230214154255923](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214154255923.png)

```shell
# 依次执行，&&前命令执行成功才会执行后面的命令
echo true
echo true && echo 'hello world'
# 由于不存在echoecho命令，所以不会执行echo 'hello world'
echoecho true && echo 'hello world'
# ||前命令执行失败才会执行后面的命令，由于不存在echoecho命令，故可以输出hello world
echoecho true || echo 'hello world'
```

![image-20230214155756754](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214155756754.png)

在 Linux 中，`$?` 是一个特殊的变量，用于显示上一个命令的退出状态码。每个 Linux 命令在执行完成后都会返回一个状态码，用于表示命令是否执行成功。通常情况下，状态码为 0 表示命令成功执行，非 0 值则表示命令执行失败。通过 `$?` 变量可以获取上一个命令的状态码。

例如：

```shell
ls
echo $?  # ls命令执行成功，故此时$?值为0
echoecho 'hi'
echo $?  # 不存在命令echoecho，上一命令执行失败，此时$?值为非0
```

![image-20230214160121657](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214160121657.png)

多命令连接符使用场景举例：

```shell
make; make install  # 自动化构建程序
./configure && make && sudo make install  # 自动化构建
cd mydir || mkdir mydir  # 进入mydir文件夹，若不存在则创建
```

#### 变量的定义和引用

```shell
# 变量定义
name=zhangsan
# 变量引用
echo $name
```

![image-20230214162129548](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214162129548.png)

#### 命令替换

命令替换（Command Substitution）是一种 Linux 命令行中的高级技巧，用于在命令中嵌入另一个命令的输出结果。命令替换可以通过反引号`$()` 来实现。

```shell
which cat
ls -l /usr/bin/cat
ls -l $(which cat)
```

![image-20230214162840333](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214162840333.png)

#### 第一个脚本

```shell
# 创建shell脚本
vim first.sh  # vim基本操作
# 下述为first.sh的内容
```

```shell
#!/bin/bash

# 下述四行(包括该行)为注释
# version: 1.0
# author: wangchen
# desc: print hello world
# date: 2023-2-24

echo 'hello world'
```

+ #!/bin/bash 表示该shell脚本由/bin/bash解释
+ echo 是输出语句

```shell
# 赋予所有用户first.sh运行权限
chmod +x first.sh
# 运行first.sh
./first.sh
```

![image-20230214094958160](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214094958160.png)

除此之外，还可以通过`bash first.sh`直接执行该文件，无需赋予运行权限。

#### 特殊变量

```shell
vim canshu.sh

# 以下为canshu.sh内容
#!/bin/bash

echo '$0: ' $0
echo '$1: ' $1
echo '$2: ' $2
echo '$3: ' $3
echo '$#: ' $#
echo '$?: ' $?
echo '$@: ' $@
echo '$$: ' $$
echo '$*: ' $*
# 以上为canshu.sh内容
chmod u+x canshu.sh
./canshu.sh 1 2 3 4 5
```

![image-20230214163435529](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214163435529.png)

```shell
解释：
内置或特殊变量：
$0 文件名称 文件本身
$1 第一个参数
$2 第二个参数
$3 第三个参数
$# 是参数个数
$?上一条命令的返回值
$@或$*  返回都是全部参数列表（有一定区别，主要区别是如何处理参数中的空格）
$$  进程ID 

$@与$*的区别
$@ 变量会将每个参数视为单独的字符串，因此它们之间的空格会被保留。
$* 变量会将所有参数视为单个字符串，因此它们之间的空格会被替换为第一个字符（通常是空格）的值。
区别演示：
vim qubie.sh
# 以下为qubie.sh内容
#!/bin/bash

for arg in "$@"
do
  echo $arg
done
# 以上为qubie.sh内容
chmod +x qubie.sh
./qubie.sh foo bar hello world
```

![image-20230214164741421](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214164741421.png)

将`$@`替换为`$*`后的输出结果为

![image-20230214164837754](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214164837754.png)

如果需要对每个参数进行单独处理，则应使用 `$@`，而如果需要将所有参数作为单个字符串进行处理，则应使用 `$*`。

#### 交互（read）

在 Linux 中，`read` 是一个用于读取用户输入的命令。`read` 命令会等待用户输入文本，然后将文本存储在一个变量中，以便稍后在脚本中使用。

以下是常见命令选项

- `-p`：在等待用户输入时，输出提示符。
- `-t`：指定超时时间，如果用户未在指定时间内输入，则退出。
- `-n`：指定输入字符数的最大值。
- `-s`：不回显输入字符，通常用于输入密码等敏感信息。

```shell
vim read.sh
# 以下为read.sh内容
#!/bin/bash
read -p 'Please input your name: ' u_name
echo ${u_name}
read -p 'Please input your pass: ' u_pass
echo ${u_pass}
# 以上为read.sh内容
chmod +x read.sh
./read.sh
```

![image-20230214165756036](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214165756036.png)

#### 条件判断

##### test命令

在 Linux 中，`test` 命令是一个用于测试文件和字符串的工具，可以用于条件判断和循环控制语句中。`test` 命令支持多种测试表达式，包括文件测试表达式、字符串测试表达式和数字测试表达式。以下是一些常见的测试表达式：

- 文件测试表达式：`-e`（文件或目录存在）、`-f`（文件存在且为普通文件）、`-d`（目录存在）、`-s`（文件大小大于0）、`-r`（文件可读）、`-w`（文件可写）和 `-x`（文件可执行）等。

- 字符串测试表达式：`-z`（字符串为空）、`-n`（字符串不为空）、`=`

- 数值测试表达式：`-eq`（大于）、`-gt`（大于）、`-lt`（小于）、`-ge`（大于等于）、

    `-le`（小于等于）、`-ne`（不等于）

```shell
test -e cunzai  # 测试当前目录是否存在cunzai文件
echo $?  # 输入上一个命令的状态码。
touch cunzai
test -e cunzai
echo $?

# test -e cunzai命令可用 [ -e cunzai ] 代替
[ -e cunzai ]  # 注意空格
echo $?
```

![image-20230214180845261](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214180845261.png)

![image-20230214181056233](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214181056233.png)

```shell
# 示例1
[ -e file1 ] && ech0 'file1 exist'
touch file1
[ -e file1 ] && ech0 'file1 exist'
```

![image-20230214181259573](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214181259573.png)

##### if命令

`if`命令的基本格式如下：

```shell
if command
then
    # Commands to execute if the command succeeds (returns 0)
else
    # Commands to execute if the command fails (returns non-zero)
fi
```

```shell
# 示例 文件判断
vim iftest.sh
# 以下为iftest.sh内容
#!/bin/bash
if [ -e file2 ]
then
    echo 'file2 exist'
else
    echo 'file2 not exist'
fi
# 以上为iftest.sh内容
chmod u+x iftest..sh
./iftest.sh
touch file2
./iftest.sh
```

![image-20230214181914493](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214181914493.png)

```shell
# 示例 数值判断
num=10
echo $num
[ $num -eq 11 ]
echo $?
[ $num -eq 10 ]
echo $?
# $num大于8且小于12
[ $num -gt 8 ] && [ $num -lt 12 ]
echo $?
# $num小于8或大于12
[ $num -lt 8 ] && [ $num -gt 12 ]
echo $?
```

![image-20230214182455301](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214182455301.png)

![image-20230214183402935](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214183402935.png)

```shell
# 示例 字符串判断
name=zhangsan
echo $name
[ $name == zhangsan ]
echo $?
[ -z name ]
echo $?
[ -n name ]
echo $?
```

![image-20230214184208935](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214184208935.png)

#### 多条件判断

`-a`（与）、`-o`（或）

```shell
num=100
[ $num -gt 90 -a $num -lt 110 ]
echo $?
[ $num -lt 90 -o $num -gt 110 ]
echo $?
```

![image-20230214231744209](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214231744209.png)

### 练习

#### 练习一：匹配用户名

提示用户输入用户名，若存在该用户则打印用户的UID、GID、家目录、shell；否则添加用户并将密码设为redhat。

```shell
#!/bin/bash
read -p "please inpurt your username: " username
if id ${username} &> /dev/null
then
    echo "${username} is exist."
    u_uid=$(grep ^${username} /etc/passwd|cut -d ":" -f 3)
    u_gid=$(grep ^${username} /etc/passwd|cut -d ":" -f 4)
    u_dir=$(grep ^${username} /etc/passwd|cut -d ":" -f 6)
    u_shell=$(grep ^${username} /etc/passwd|cut -d ":" -f 7)
    echo -e "${username} uid:\t ${u_uid}"
    echo -e "${username} gid:\t ${u_gid}"
    echo -e "${username} dir:\t ${u_dir}"
    echo -e "${username} shell:\t ${u_shell}"
else
	echo "${username} is not exist."
	useradd ${username}
	echo redhat|passwd --stdin ${username}
fi
```

![image-20230214232733395](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214232733395.png)

#### 练习二：查询文件信息

提示用户输入文件名，若存在且为普通文件则输出文件详细信息，否则创建文件。

```shell
#!/bin/bash
read -p "please input your filename: " filename
if [ -f ${filename} ]
then
	echo -e "File ${filename} is exist."
	ls -l ${filename}
else
	echo -e "File ${filename} is not exist."
	touch ${filename}
fi
```

![image-20230214233240491](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214233240491.png)

#### 练习三：批量创建文件

在文件夹files下批量创建文件file1-file7，修改权限依次为111-777

```shell
#!/bin/bash
for i in {1..7}
do
    if [ ! -e files ]
    then
    	mkdir files
    fi
    if [ ! -e ./files/$i ]
    then
    	touch ./files/$i
    	chmod $i$i$i ./files/$i
    fi
done
```

![image-20230214234028391](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214234028391.png)

#### 练习四：if练习

获取参数并判断是否为redhat或fedora。

```shell
#!/bin/bash
if [ $1 == redhat ]
then
    echo 'redhat'
elif [ $1 == fedora ]
then
	echo 'fedora'
else
	echo -e "Usage: $0 redhat|fedora"
fi
```

![image-20230215083429330](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230215083429330.png)

#### 练习五：猜数字游戏

随机初始化一个1-100的数，提示用户输入数字，若输入数字过大输出too large；若输入数字过小输出too samll；若输入数字等于生成的数字的输出you win。

```shell
#!/bin/bash
# 随机初始化1-100的数，
num=$((($RANDOM % 100) + 1))
while true
do
	read -p "Plese input your guess num: " Num
	if [ $Num -gt $num ]
	then
		echo 'too large......'
	elif [ $Num -lt $num ]
	then
		echo 'too small'
	else
		echo 'you win'
		break 
	fi
done
```

![image-20230215093831990](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230215093831990.png)
