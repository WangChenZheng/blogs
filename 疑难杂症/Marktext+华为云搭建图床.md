## Marktext + 华为云搭建图床

### 1. 下载Marktext

[Marktext·github](https://github.com/marktext/marktext/releases)

```
百度网盘下载

链接：https://pan.baidu.com/s/14a2ojJoNOJdNZEZdV_w-kQ?pwd=erda 
提取码：erda
```

### 2. 下载Nodejs

[Node.js官网](https://nodejs.org/en/)

### 3. 安装picgo-core



- WIN+R 呼出运行，输入 cmd 打开命令窗口
  
  ![20230205123318imagepng](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/2023-02-05-12-33-18-image.png)

- 输入命令 npm install picgo -g 等待安装完成即可
  
  ![20230205123711imagepng](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/2023-02-05-12-37-11-image.png)

### 4. 安装华为云OBS插件

| 参数名称            | 类型       | 描述             | 是否必须  |
|:--------------- |:--------:|:--------------:|:-----:|
| AccessKeyId     | input    | 用户-我的凭证-访问密钥   | true  |
| AccessKeySecret | password | 用户-我的凭证-访问密钥   | true  |
| Bucket          | input    | OBS桶名称         | true  |
| EndPoint        | input    | 在桶-概览-基本信息 中查看 | true  |
| 存储路径            | input    | 自定义在桶中的存储路径    | false |
| 自定义域名           | input    | 自定义域名          | false |

```
# 插件安装
picgo install picgo-plugin-huawei-uploader
# 设置插件参数
picgo set uploader
# 应用插件
picgo use uploader



# 卸载插件
picgo uninstall picgo-plugin-huawei-uploader
```

- 控制台-我的-我的凭证-访问密钥-新增访问密钥
  
  即可获得AccessKeyId, AccessKeySecret.
  
  ![Snipaste20230205183926jpg](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/Snipaste_2023-02-05_18-39-26.jpg)
  
  ![Snipaste20230205184009jpg](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/Snipaste_2023-02-05_18-40-09.jpg)

- 进入OBS对象存储服务-创建桶-概览
  
  即可查看Bucket, Endpoint.
  
  ![Snipaste_2023-02-05_18-44-38.jpg](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/Snipaste_2023-02-05_18-44-38.jpg)

### 5. 测试Picgo是否可用

```
picgo upload image_path
```

![Snipaste_2023-02-05_18-47-34.jpg](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/Snipaste_2023-02-05_18-47-34.jpg)

### 6. 开启Marktext的图片自动上传

![Snipaste_2023-02-05_18-48-30.jpg](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/Snipaste_2023-02-05_18-48-30.jpg)

![Snipaste_2023-02-05_18-49-01.jpg](https://image-bed-693a.obs.cn-north-4.myhuaweicloud.com/imgbed/Snipaste_2023-02-05_18-49-01.jpg)

### 7. 测试Marktext图片自动上传

将图片拖入Marktext中，若成功显示图片且图片link为HuaweiCloud地址即可
