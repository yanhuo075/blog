---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 1.3 安装配置pycharm
order: 2
---

# 安装配置pycharm

我只介绍windows的安装过程，因为mac的安装过程实在是过于简单了，一路继续就可以了。

## 1. windows安装过程

### 1.1 下载安装包

下载地址为

```text
https://www.jetbrains.com/pycharm/download/#section=windows
```

![pycharm安装包](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145417498.png)
根据自己的操作系统进行下载，左侧是Professional，收费专业版本，右侧是Community，免费社区版本。

你可以选择社区版本，功能基本够用，如果想用专业版本，就需要注册码，好在网络上很多人分享注册码，因此，专业版本也可以正常使用，只是需要经常更换注册码，本教程在后面会提供注册码。

如果你嫌麻烦，可以和我使用一样的安装包
网盘地址为: https://pan.baidu.com/s/1kOAWrynFlU5EqZ_EwOTzvw

### 1.2 安装过程

![pycharm安装过程](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145330932.png)
点击next
![pycharm安装过程](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145330269.png)
修改安装目录，不要安装在c盘就行，修改好以后点击next
![pycharm安装过程](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145330277.png)

除了Add launchers dir to the PATH ，其他都勾选，点击next
![pycharm安装过程](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145330264.png)
点击Install
![pycharm安装过程](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145417606.png)
安装过程大约3分钟，耐心等待
![pycharm安装过程](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145335473.png)

## 2. 启动，注册激活，配置

首次启动，需要对编辑器进行配置，注册激活，新建项目，设置项目的解释器，过程十分麻烦，本教程以windows为例，mac用户可以用作参考

### 2.1 启动，注册激活

在桌面上找到图标
![pycharm安装过程](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145440407.png)
双击启动
![pycharm安装过程](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145336611.png)
勾选Do not import settings，首次启动，没有历史配置可以导入
![pycharm如何使用](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145338689.png)
勾选 I confirm， 点击Continue按钮
![pycharm如何使用](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145342968.png)
点击Don't send按钮
![pycharm如何使用](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145344753.png)
点击左下角的skip 按钮
![pycharm如何使用](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145345541.png)
填写注册码

```text
QYYBAC9D3J-eyJsaWNlbnNlSWQiOiJRWVlCQUM5RDNKIiwibGljZW5zZWVOYW1lIjoi6LaF57qnIOeoi+W6j+WRmCIsImFzc2lnbmVlTmFtZSI6IiIsImFzc2lnbmVlRW1haWwiOiIiLCJsaWNlbnNlUmVzdHJpY3Rpb24iOiIiLCJjaGVja0NvbmN1cnJlbnRVc2UiOmZhbHNlLCJwcm9kdWN0cyI6W3siY29kZSI6IklJIiwiZmFsbGJhY2tEYXRlIjoiMjAyMC0wMS0wNCIsInBhaWRVcFRvIjoiMjAyMS0wMS0wMyJ9LHsiY29kZSI6IkFDIiwiZmFsbGJhY2tEYXRlIjoiMjAyMC0wMS0wNCIsInBhaWRVcFRvIjoiMjAyMS0wMS0wMyJ9LHsiY29kZSI6IkRQTiIsImZhbGxiYWNrRGF0ZSI6IjIwMjAtMDEtMDQiLCJwYWlkVXBUbyI6IjIwMjEtMDEtMDMifSx7ImNvZGUiOiJQUyIsImZhbGxiYWNrRGF0ZSI6IjIwMjAtMDEtMDQiLCJwYWlkVXBUbyI6IjIwMjEtMDEtMDMifSx7ImNvZGUiOiJHTyIsImZhbGxiYWNrRGF0ZSI6IjIwMjAtMDEtMDQiLCJwYWlkVXBUbyI6IjIwMjEtMDEtMDMifSx7ImNvZGUiOiJETSIsImZhbGxiYWNrRGF0ZSI6IjIwMjAtMDEtMDQiLCJwYWlkVXBUbyI6IjIwMjEtMDEtMDMifSx7ImNvZGUiOiJDTCIsImZhbGxiYWNrRGF0ZSI6IjIwMjAtMDEtMDQiLCJwYWlkVXBUbyI6IjIwMjEtMDEtMDMifSx7ImNvZGUiOiJSUzAiLCJmYWxsYmFja0RhdGUiOiIyMDIwLTAxLTA0IiwicGFpZFVwVG8iOiIyMDIxLTAxLTAzIn0seyJjb2RlIjoiUkMiLCJmYWxsYmFja0RhdGUiOiIyMDIwLTAxLTA0IiwicGFpZFVwVG8iOiIyMDIxLTAxLTAzIn0seyJjb2RlIjoiUkQiLCJmYWxsYmFja0RhdGUiOiIyMDIwLTAxLTA0IiwicGFpZFVwVG8iOiIyMDIxLTAxLTAzIn0seyJjb2RlIjoiUEMiLCJmYWxsYmFja0RhdGUiOiIyMDIwLTAxLTA0IiwicGFpZFVwVG8iOiIyMDIxLTAxLTAzIn0seyJjb2RlIjoiUk0iLCJmYWxsYmFja0RhdGUiOiIyMDIwLTAxLTA0IiwicGFpZFVwVG8iOiIyMDIxLTAxLTAzIn0seyJjb2RlIjoiV1MiLCJmYWxsYmFja0RhdGUiOiIyMDIwLTAxLTA0IiwicGFpZFVwVG8iOiIyMDIxLTAxLTAzIn0seyJjb2RlIjoiREIiLCJmYWxsYmFja0RhdGUiOiIyMDIwLTAxLTA0IiwicGFpZFVwVG8iOiIyMDIxLTAxLTAzIn0seyJjb2RlIjoiREMiLCJmYWxsYmFja0RhdGUiOiIyMDIwLTAxLTA0IiwicGFpZFVwVG8iOiIyMDIxLTAxLTAzIn0seyJjb2RlIjoiUlNVIiwiZmFsbGJhY2tEYXRlIjoiMjAyMC0wMS0wNCIsInBhaWRVcFRvIjoiMjAyMS0wMS0wMyJ9XSwiaGFzaCI6IjE2MDgwOTA5LzAiLCJncmFjZVBlcmlvZERheXMiOjcsImF1dG9Qcm9sb25nYXRlZCI6ZmFsc2UsImlzQXV0b1Byb2xvbmdhdGVkIjpmYWxzZX0=-I7c5mu4hUCMxcldrwZEJMaT+qkrzrF1bjJi0i5QHcrRxk2LO0jqzUe2fBOUR4L+x+7n6kCwAoBBODm9wXst8dWLXdq179EtjU3rfJENr1wXGgtef//FNow+Id5iRufJ4W+p+3s5959GSFibl35YtbELELuCUH2IbCRly0PUBjitgA0r2y+9jV5YD/dmrd/p4C87MccC74NxtQfRdeUEGx87vnhsqTFH/sP4C2VljSo/F/Ft9JqsSlGfwSKjzU8BreYt1QleosdMnMK7a+fkfxh7n5zg4DskdVlNbfe6jvYgMVE16DMXd6F1Zhwq+lrmewJA2jPToc+H5304rcJfa9w==-MIIElTCCAn2gAwIBAgIBCTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTE4MTEwMTEyMjk0NloXDTIwMTEwMjEyMjk0NlowaDELMAkGA1UEBhMCQ1oxDjAMBgNVBAgMBU51c2xlMQ8wDQYDVQQHDAZQcmFndWUxGTAXBgNVBAoMEEpldEJyYWlucyBzLnIuby4xHTAbBgNVBAMMFHByb2QzeS1mcm9tLTIwMTgxMTAxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxcQkq+zdxlR2mmRYBPzGbUNdMN6OaXiXzxIWtMEkrJMO/5oUfQJbLLuMSMK0QHFmaI37WShyxZcfRCidwXjot4zmNBKnlyHodDij/78TmVqFl8nOeD5+07B8VEaIu7c3E1N+e1doC6wht4I4+IEmtsPAdoaj5WCQVQbrI8KeT8M9VcBIWX7fD0fhexfg3ZRt0xqwMcXGNp3DdJHiO0rCdU+Itv7EmtnSVq9jBG1usMSFvMowR25mju2JcPFp1+I4ZI+FqgR8gyG8oiNDyNEoAbsR3lOpI7grUYSvkB/xVy/VoklPCK2h0f0GJxFjnye8NT1PAywoyl7RmiAVRE/EKwIDAQABo4GZMIGWMAkGA1UdEwQCMAAwHQYDVR0OBBYEFGEpG9oZGcfLMGNBkY7SgHiMGgTcMEgGA1UdIwRBMD+AFKOetkhnQhI2Qb1t4Lm0oFKLl/GzoRykGjAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBggkA0myxg7KDeeEwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgWgMA0GCSqGSIb3DQEBCwUAA4ICAQAF8uc+YJOHHwOFcPzmbjcxNDuGoOUIP+2h1R75Lecswb7ru2LWWSUMtXVKQzChLNPn/72W0k+oI056tgiwuG7M49LXp4zQVlQnFmWU1wwGvVhq5R63Rpjx1zjGUhcXgayu7+9zMUW596Lbomsg8qVve6euqsrFicYkIIuUu4zYPndJwfe0YkS5nY72SHnNdbPhEnN8wcB2Kz+OIG0lih3yz5EqFhld03bGp222ZQCIghCTVL6QBNadGsiN/lWLl4JdR3lJkZzlpFdiHijoVRdWeSWqM4y0t23c92HXKrgppoSV18XMxrWVdoSM3nuMHwxGhFyde05OdDtLpCv+jlWf5REAHHA201pAU6bJSZINyHDUTB+Beo28rRXSwSh3OUIvYwKNVeoBY+KwOJ7WnuTCUq1meE6GkKc4D/cXmgpOyW/1SmBz3XjVIi/zprZ0zf3qH5mkphtg6ksjKgKjmx1cXfZAAX6wcDBNaCL+Ortep1Dh8xDUbqbBVNBL4jbiL3i3xsfNiyJgaZ5sX7i8tmStEpLbPwvHcByuf59qJhV/bZOl8KqJBETCDJcY6O2aqhTUy+9x93ThKs1GKrRPePrWPluud7ttlgtRveit/pcBrnQcXOl1rHq7ByB8CFAxNotRUYL9IF5n3wJOgkPojMy6jetQA5Ogc8Sm7RG6vg1yow==
```

注册码有效期至21年1月3日，点击按钮，至此，完成了初步配置与注册激活

### 2.2 创建新项目

![pycharm创建项目](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145348227.png)

点击create new project
![pycharm创建项目](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145515287.png)

左侧选择 pure python， 右侧Location设置项目的目录，你根据自己电脑的情况进行设置，最后点击create按钮
![pycharm创建项目](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145349848.png)

去掉show tips on startup ，点击close按钮，此时页面如下
![pycharm创建项目](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145351350.png)

### 2.3 新建脚本并执行

![pycharm创建项目](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145352664.png)

右键点击项目hello，选择New,选择Python File
![pycharm创建项目](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145354629.png)
输入脚本名称hello,点击回车
![pycharm创建项目](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145355238.png)
在新建脚本里写入代码

```text
print('hello world')
```

右键点击编辑区的任意位置，选择Run 'hello'， 编辑器里会显示执行结果
![pycharm创建项目](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145543369.png)
注意看黄色区域，hello项目使用的python解释器是虚拟环境下的，在使用上不会有任何问题，唯一的问题是，如果你在项目里使用第三方模块，那么你就需要通过pycharm进行安装，而且这个第三方模块只能用于这个项目，想要让第三方模块适用于所有项目，需要配置项目的解释器为你所安装的python解释器，而非虚拟环境下的python解释器

### 2.4 修改项目的解释器

我的python安装在D:\Python36 目录下，现在，我要讲hello项目的解释器修改为这里的解释器
![pycharm配置](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145537485.png)
点击左上角的File菜单，选择Settings
![pycharm配置](http://www.coolpython.net/pictures/python_primary/introduction/install_pycharm-1583236692-22.png)
找到左侧的hello项目，选择 project interpreter,可以看到右侧黄色框内的解释器是虚拟环境venv下的解释器
![pycharm配置](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145405769.png)
点击右侧的向下的小箭头，选择Show All
![pycharm配置](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145557295.png)
点击右侧的加号
![pycharm配置](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145411971.png)
勾选Existing environment，然后点击右侧红色框内的按钮
![pycharm配置](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145413353.png)
找到自己所安装的python解释器，点击ok按钮
![pycharm配置](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145413452.png)
勾选 make available to all projiects，点击ok按钮
![pycharm配置](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145413427.png)
选中自己安装的python解释器，点击ok
![pycharm配置](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822145416955.png)

再次执行程序，可以发现，python解释器已经发生改变。
