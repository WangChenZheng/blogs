## 轮询查询研招网何时可查考研成绩

### 准备

#### Python

官网下载即可，[官网](https://www.python.org/)

#### requests

进入项目所在文件夹，`cmd`执行`pip install requests`自动安装即可

### 找到查询接口

找到研招网查询成绩页面：https://yz.chsi.com.cn/apply/cjcx/t/xxxxx.dhtml, 其中xxxxx为院校代码。

如研招网南航成绩查询页面：https://yz.chsi.com.cn/apply/cjcx/t/10287.dhtml

![image-20230213190132087](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230213190132087.png)

F12打开开发者工具，输入信息点击查询后显示如下界面

![image-20230213190346957](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230213190346957.png)

同时监控到一条POST请求，请求地址为https://yz.chsi.com.cn/apply/cjcx/cjcx.do， 其中请求携带的载荷(data)即为刚才所填表单内容，如图

![image-20230213190534002](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230213190534002.png)

所以我们尝试使用Python，定时轮询该接口，若接口返回的结果不是上述“无查询结果”，则说明此时可在研招网查询成绩。

### 基本代码思路

进入项目地址，安装所需的包`requests`，执行如下命令`pip install requests`即可

```python
# -*- coding: utf-8 -*-
import requests

# 接口请求地址
URL = 'https://yz.chsi.com.cn/apply/cjcx/cjcx.do'
# 接口请求头
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"
}
# 接口所需载荷
DATA = {
    # 姓名
    'xm': '***',
    # 身份证号
    'zjhm': '***',
    # 考生号
    'ksbh': '***',
    # 报考院校代码
    'bkdwdm': '***',
    'checkcode': ''
}

session = requests.session()
# 关闭多余连接
session.keep_alive = False
resp = session.post(URL, DATA, headers=HEADERS)
if resp.status_code == 200:
    # 访问成功
    if '无查询结果' in resp.text:
        print('研招网目前无法查询成绩')
    else:
        print('出成绩了！！！')
```

为了能够每天定时查询，所以我们需要将代码上传至服务器执行并添加日志功能监控代码运行。添加日志功能以及轮询的代码为

```python
# -*- coding: utf-8 -*-
import time

import requests
from datetime import datetime


class Logger:

    # 日志等级
    INFO = 'INFO'
    WARNING = 'WARNING'
    ERROR = 'ERROR'


    def __init__(self, path):
        self.path = path

    def write2Log(self, status, content):
        f = open(self.path, mode='a', encoding='utf-8')
        f.seek(0, 2)  # 移至文件末尾
        # 东八区时间
        t = (datetime.utcnow()+timedelta(hours=8)).strftime('%y-%m-%d %H:%M:%S')
        s = f'[{t}] [{status}] {content}\n'
        f.write(s)
        f.close()

# 接口请求地址
URL = 'https://yz.chsi.com.cn/apply/cjcx/cjcx.do'
# 接口请求头
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"
}
# 接口所需载荷
DATA = {
    # 姓名
    'xm': '***',
    # 身份证号
    'zjhm': '***',
    # 考生号
    'ksbh': '***',
    # 报考院校代码
    'bkdwdm': '***',
    'checkcode': ''
}


if __name__ == '__main__':
    logger = Logger('./log.txt')
    while True:
        session = requests.session()
        # 关闭多余连接
        session.keep_alive = False
        resp = session.post(URL, DATA, headers=HEADERS)
        if resp.status_code == 200:
            # 访问成功
            logger.write2Log(Logger.INFO, '【研招网】访问成功。')
            if '无查询结果' in resp.text:
                logger.write2Log(Logger.INFO, '【研招网】目前无法查询成绩。')
            else:
                logger.write2Log(Logger.INFO, '【研招网】出成绩啦！！！')
                break
        else:
            logger.write2Log(Logger.ERROR, f'【研招网】访问失败。状态码：{resp.status_code}, 原因：{resp.reason}')
        session.close()
        time.sleep(60*30)  # 30分钟一次
```

事实上我们还有一个问题。就是如果可以查成绩了，如何第一时间通知自己呢，所以我们需要编写发送邮箱的服务或发送短信的服务，这里我选择短信代发平台 [短信宝](https://www.smsbao.com/)

### 短信发送

按照流程注册，登录即可。冲5块钱可以发送50条短信，自行购买套餐即可。

查看官网Api接口Python代码示例

![image-20230213194618371](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230213194618371.png)

添加短信服务后的代码为

```python
# -*- coding: utf-8 -*-
import time

import requests
import hashlib
from datetime import datetime

# 接口请求地址
URL = 'https://yz.chsi.com.cn/apply/cjcx/cjcx.do'
# 接口请求头
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"
}
# 接口所需载荷
DATA = {
    # 姓名
    'xm': '***',
    # 身份证号
    'zjhm': '***',
    # 考生号
    'ksbh': '***',
    # 报考院校代码
    'bkdwdm': '***',
    'checkcode': ''
}
ACCOUNT = {
    # 短信平台账号
    'user': '***',
    # 短信平台密码
    'password': '***',
}
# 接收短信的电话号码
PHONE = '***'


class Logger:

    # 日志等级
    INFO = 'INFO'
    WARNING = 'WARNING'
    ERROR = 'ERROR'


    def __init__(self, path):
        self.path = path

    def write2Log(self, status, content):
        f = open(self.path, mode='a', encoding='utf-8')
        f.seek(0, 2)  # 移至文件末尾
        # 东八区时间
        t = (datetime.utcnow()+timedelta(hours=8)).strftime('%y-%m-%d %H:%M:%S')
        s = f'[{t}] [{status}] {content}\n'
        f.write(s)
        f.close()


def md5(str):
    m = hashlib.md5()
    m.update(str.encode("utf8"))
    return m.hexdigest()


def url_encode(text):
    return requests.utils.quote(text)


def sendMessage(phone, content):
    statusStr = {
        '0': '短信发送成功',
        '-1': '参数不全',
        '-2': '服务器空间不支持,请确认支持curl或者fsocket,联系您的空间商解决或者更换空间',
        '30': '密码错误',
        '40': '账号不存在',
        '41': '余额不足',
        '42': '账户已过期',
        '43': 'IP地址限制',
        '50': '内容含有敏感词'
    }
    user = url_encode(ACCOUNT['user'])
    password = url_encode(md5(ACCOUNT['password']))
    content = url_encode(content)
    phone = url_encode(phone)

    smsapi = f'http://api.smsbao.com/sms?u={user}&p={password}&m={phone}&c={content}'
    response = requests.get(smsapi)
    statusCode = response.text

    if statusCode == '0':
        logger.write2Log(Logger.INFO, f'【短信宝】{statusStr[statusCode]}')
    else:
        logger.write2Log(Logger.WARNING, f'【短信宝】{statusStr[statusCode]}')


if __name__ == '__main__':
    logger = Logger('./log.txt')
    while True:
        session = requests.session()
        # 关闭多余连接
        session.keep_alive = False
        resp = session.post(URL, DATA, headers=HEADERS)
        if resp.status_code == 200:
            # 访问成功
            logger.write2Log(Logger.INFO, '【研招网】访问成功。')
            if '无查询结果' in resp.text:
                logger.write2Log(Logger.INFO, '【研招网】目前无法查询成绩。')
            else:
                sendMessage(PHONE, '【研招网】研招网出成绩啦！！！')
                break
        else:
            logger.write2Log(Logger.ERROR, f'【研招网】访问失败。状态码：{resp.status_code}, 原因：{resp.reason}')
        session.close()
        time.sleep(60*30)  # 30分钟一次
```

### 部署

当然我们需要先搞一台云服务器，自行去阿里云，华为云等购买即可。

为了不污染原服务器的环境，所以采用docker部署。

首先将python文件上传至云服务器，使用Xshell自带的xftp即可。

```shell
# docker安装python3.8镜像，等待下载完成即可。
docker pull python:3.8
# 查看docker镜像
docker images
# 为镜像重命名为python3.8
docker image tag docker.io/python:3.8 python3.8
```

![image-20230214084934026](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214084934026.png)

```shell
# 后台运行docker镜像
docker run -d -t python3.8 /bin/bash
# 查看正在运行的docker容器
docker ps
# 进入docker容器
docker exec -it 容器ID /bin/bash
```

![image-20230214085220444](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214085220444.png)

```shell
# 重新打开一个Shell
# 将文件拷贝至docker容器内    例如：docker cp /home/checkyzw.py 5ff7284f2403:/home
docker cp python文件地址  容器ID:容器内地址
# 然后进入容器
# 进入python文件存放路径
# 可以先试着执行以下python脚本
python ./python文件
# 为了能够使其后台运行，我们需要使用nohup
# 编写shell脚本，将上述  python ./python文件   命令写入shell脚本中
vim script.sh
```

![image-20230214085829414](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214085829414.png)

```shell
# 使用nohup实现后台运行
nohup ./script.sh &
# 查看运行日志
cat log.txt
```

![image-20230214090010491](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/image-20230214090010491.png)
