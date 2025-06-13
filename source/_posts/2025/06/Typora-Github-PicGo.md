---
title: Typora+Github+PicGo搭建个人免费图床
categories: [装修日记]
keywords: 博客文章密码
password: pass
abstract: 私密文档
message:  输入密码，查看文章
wrong_pass_message: 密码有误，请重新输入
tags: [主题装修]
---

# 一、配置GitHub

## 1、新建公开仓库

### 1.1、New repository

[![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250609150012537.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220901738-2024171065.png)

### 1.2、定义仓库名称，设置公开属性

[![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250609150012570.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220901265-1393623121.png)

## 2、创建私人令牌（token）

### 2.1、个人头像 --> Settings

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220900607-256612593.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220900607-256612593.png)

### 2.2、左侧列底部点击 Developer settings

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220859820-723517043.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220859820-723517043.png)

### 2.3、选择 Generate new token (classic)

> 依此点击 Personal access tokens --> Tokens(classic) --> Generate new token --> Generate new token(classic)

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220859206-1834857940.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220859206-1834857940.png)

### 2.4、密码验证

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220858530-227799129.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220858530-227799129.png)

### 2.5、创建令牌

> Note：标记作用，任意写
>
> Token 过期时间：为了安全性，不建议选择永久
>
> 访问权限：选择repo，对私有库享有完全控制

[![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250609150013595.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220857739-1764979956.png)

[![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250609150013583.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220857204-874985880.png)

### 2.6、记事本记下token

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220856771-24639529.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220856771-24639529.png)

# 二、配置PicGo

## 1、PicGo下载地址

[Molunerfinn/PicGo](https://github.com/Molunerfinn/PicGo/releases)

> 依据自身的操作系统下载安装软件包

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220855596-495781787.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220855596-495781787.png)

## 2、图床设置

> 图床设置 --> GitHub

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220854595-1257469519.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220854595-1257469519.png)

## 3、具体图床参数配置

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220854045-451301020.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220854045-451301020.png)

### 3.1、图床配置名

当前图床的名称，自定义

### 3.2、设定仓库名

> 格式：用户名/仓库名
>
> 可以通过GitHub上创建仓库页的域名快速得到



bash

```bash
# 例如我这边的仓库名就是
misakivv/Cloud-Image-Hosting
```

[![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250609150014655.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220853562-1698621985.png)

### 3.3、设定分支名

> 默认情况下均为 main 分支

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220852995-684255350.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220852995-684255350.png)

### 3.4、设定Token

上面 一、2.6、保存的Token

### 3.5、设定存储路径

存放至 Github 指定仓库的哪个文件夹下

- 如果直接放到仓库的根目录下就不需要填写这一栏
- 如果需要放到某个目录下，格式：/
- 当填写的目录不存在时，Github会自动创建对应目录

### 3.6、设定自定义域名

> 这里使用免费的CDN：jsDelivr
>
> 官网链接：[jsDelivr - A free, fast, and reliable CDN for JS and open source](https://www.jsdelivr.com/)



bash

```bash
# https://cdn.jsdelivr.net/gh/：固定的前缀，相当于替换掉了Github地址中的https://github.com/
# user：Github上的用户名
# repo：仓库名
# @version：版本号（这里我们可以不管）
# file：文件名（这里我们也不需要加上，因为上传完图片后，它会自动将上传的图片的名字作为存储的文件名）
# 例如我这里的填写
https://cdn.jsdelivr.net/gh/misakivv/Cloud-Image-Hosting
```

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220852595-1340430072.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220852595-1340430072.png)

这里值得注意的是，如果需要指定上传到哪个分支，此时需要在自定义域名后面使用@ + 分支名，如果是仓库默认的分支，可以省略指定分支这一步。

eg：我需要上传到 k8s 分支上，此时自定义域名就变成了：https://cdn.jsdelivr.net/gh/misakivv/Cloud-Image-Hosting@k8s

# 三、测试

上述配置完成后即可上传本地图片生成对应云端平台链接使用

## 1、本地上传图片

> 上传区 --> 选择GitHub图床 --> 点击上传

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220851740-788124223.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220851740-788124223.png)

## 2、相册查看

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220850907-1584285037.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220850907-1584285037.png)

## 3、时间戳重命名

> 添加时间戳命名选项可以避免上传相同图片被覆盖的情况

[![img](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220850483-751662635.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220850483-751662635.png)

# 四、Typora插入图片实现自动上传至PicGo（可选）

> 需要将PicGo图床的链接格式换成Markdonw格式才能实现功能

[![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250609150017317.png)](https://gcore.jsdelivr.net/gh/misakivv/Cloud-Image/Default/20241231202037169.png)

## 1、打开 Typora 偏好设置

> ctrl + , 快捷键

[![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250609150015804.png)](https://img2023.cnblogs.com/blog/3332572/202412/3332572-20241208220850023-1681762472.png)

## 2、插入图片选项选择上传图片

[![img](https://cdn.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250609150016826.png)](https://gcore.jsdelivr.net/gh/misakivv/Cloud-Image/Default/20241231201935722.png)

## 3、上传服务设定

- 上传服务选择 PicGo（app）
- PicGo 路径为安装 PicGo 目录
