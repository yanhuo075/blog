---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 1.2 Python环境搭建
order: 1
---

# Python环境搭建



## Windows系统安装python

### 1 版本选择

2.x版本将慢慢退出历史的舞台，建议你从3.x开始学习，本教程所使用的python版本是3.6

### 2 下载安装包

进入官网下载页面 https://www.python.org/downloads/windows/
找到合适的安装包
![python的windows安装包](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143249779.jpeg)
本教程所提供的示例代码适用于3.0以上的版本，如果你喜欢更高的版本，可以自己选择，在下载安装包时，一定要选择x86-64 executable的安装包，64表示适用于64位的操作系统，executable表示是一个exe安装程序。

如果你选择安装和我一样的版本，我这里分享到了百度网盘 https://pan.baidu.com/s/1ztyH1DXjYJKkicBNI7kCCg

下载以后的程序是 python-3.6.8-amd64.exe

### 3 安装过程

#### 3.1 选择安装方式

双击安装程序python-3.6.8-amd64.exe，进入安装界面
![windows安装python步骤](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143257165.jpeg)
第一步，勾选Add Python 3.6 to PATH 这个选项

第二步，点击Customize installation，这种安装方式允许我们自由配置安装选项。

千万别忘记了勾选Add Python 3.6 to PATH，我后面会讲解勾选的原因。

#### 3.2 配置安装选项

![windows安装python步骤](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143223315.jpeg)
所有的勾选想默认是选中状态，不需要做任何操作，点击Next按钮，进入下一个配置界面
![windows安装python步骤](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143345841.jpeg)

第一步，勾选Install for all users， 勾上这个选项后，会在系统的环境变量里加入python，后面也会讲解环境变量。

第二步，修改python的安装位置，我这里选择安装在d盘下，你可以根据自己的电脑情况选择安装，最后点击install按钮

#### 3.3 等待安装结束

![windows安装python步骤](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143223281.jpeg)
这是安装的中间过程，需要等待几分钟的时间
![windows安装python步骤](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143223353.jpeg)
这是安装结束时的界面，出现了successful，关闭安装界面就可以了

### 4. 验证安装效果

#### 4.1 检查安装目录

进入安装目路 d:\Python36
![python的Scripts文件夹](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143227648.jpeg)
在这个安装目录里，我们注意Scripts文件夹和python.exe， 这个python.exe就是python解释器，你在配置pycharm的时候还会用到它。

Scripts文件夹里放的是pip和easy_install第三方库管理工具
![pip和easy_install第三方库管理工具](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143229884.jpeg)
关于pip的时候，本教程会有专门的篇章进行讲解

D:\Python36\Lib\site-packages目录，是安装存储第三方库的地方。

#### 4.2 进入python交互式解释器

先进入cmd命令窗口，然后执行python命令
![python交互式解释器](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143230025.jpeg)
在交互式解释器里，你可以验证一些简单的代码，所谓交互式，是指你写的代码，会立刻被执行并显示结果，这样及时反馈，有助于学习基础只是。

复杂的代码，例如函数，虽然也可以在这里编写，但写起来不方便，而且无法保存代码，不要在这里尝试写复杂代码。

你也可以通过IDLE进入交互式解释器，IDLE可以通过开始菜单找到，不同版本的windows电脑的开始菜单已经变得不一样了，如果你实在找不到，就算了，反正有cmd就足够了
![python-IDLE](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143230192.jpeg)

#### 4.3 检查环境变量

右键点击桌面上的我的电脑，然后选择属性，进入如下的界面
![设置python环境变量](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143350821.jpeg)

点击左侧的高级系统设置
![设置python环境变量](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143232790.png)

点击环境变量
![设置python环境变量](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143403816.jpeg)

在系统变量里找到Path变量，由于在3.2的步骤中勾选Install for all users，所以，有关python的环境变量会设置在系统变量中，你仔细观察Path的值，这里面一定有你刚刚安装好的python和python目录里的Scripts目录。

当你在cmd命令窗口执行python或者pip命令时，计算机会从Path所设置的文件夹中寻找python.exe文件或者pip.exe文件，如果找不到就会报出类似“xx不是内部或外部命令”的错误。

而之所以在你的电脑的系统变量里有python所在的安装文件夹，是因为你在3.1 步骤中勾选Add Python 3.6 to PATH，在安装python的过程中，自动的添加了环境变量，在3.0以前，环境变量都是手动添加的，很多人都在这里卡很久。

### 5. 可能会遇到的问题

windows电脑的环境搭建相比于其他系统要麻烦很多，一部分用户可能会遇到api-ms-win-crt-runtime-l1-1-0.dll丢失的问题，不用怕，解决方法非常简单

1. 到C:\Windows\SysWOW64 文件夹下，把api-ms-win-crt-runtime-l1-1-0.dll删除掉
2. 下载VC redit.exe ，重新安装，我这里提供了64位的下载地址，https://pan.baidu.com/s/1llx_QeoqeQRLA4ctUMj9dQ
3. 重启电脑

如果以上步骤还是无法解决问题，可以参考这篇文章 https://blog.csdn.net/lingaixuexi/article/details/80992542，可能你还需要下载一个补丁。



## Mac系统安装python

### 1 版本选择

mac电脑已经自带了python环境，版本是2.7, 卸载它既费力，也无必要，甚至会引来一些麻烦，最好的办法是让python3.6与python2.7 共存

### 2 下载安装包

进入官方安装包下载页面，https://www.python.org/downloads/mac-osx/

找到合适的安装包，基本上mac电脑都是64位的系统，因此选择64位的安装包进行下载
![mac电脑的python安装包](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143805333.jpeg)
如果你选择安装和我一样的版本，我这里提供了百度网盘下载地址 https://pan.baidu.com/s/1cj7s7zTOQ7ZipPwfU6AE2g

下载后的文件是pkg文件
![python的pkg安装文件](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143842027.jpeg)

### 3 安装过程

双击pkg文件，进入安装界面
![mac电脑python安装过程](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143805384.jpeg)
mac下的安装非常方便，一路继续即可，最后打开终端来验证安装是否正确
输入命令

```text
python3 --version
```

会得到python的版本号
![python版本号](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250822143808519.jpeg)

现在，你的mac电脑里有两份python，一份是python2.7，一份是python3.6，在终端里，当你使用python命令时，用的是python2.6，使用python3时用的是python3.6

### 4. python安装到了哪里

想要知道python3.6安装到了哪里，可以使用which命令

```text
which python3
```

在我的mac电脑上，安装目录是

```text
/Library/Frameworks/Python.framework/Versions/3.6/bin/python3
```

进入到/Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages目录，所有安装的第三方模块都在这里



## Linux系统安装Python

在Linux系统中运行Python3.6需要确保已正确安装Python3.6，并可以通过命令行或脚本执行。以下是详细步骤：

### 1.安装Python3.6

如果系统未安装Python3.6，可以通过以下方式安装：

#### CentOS/RHEL系统

```
sudo yum update

sudo yum install gcc zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel

wget https://www.python.org/ftp/python/3.6.1/Python-3.6.1.tgz

tar -zxvf Python-3.6.1.tgz

cd Python-3.6.1

/configure --prefix=/usr/local/python3

make && sudo make install

ln -s /usr/local/python3/bin/python3 /usr/bin/python3
```



#### Ubuntu/Debian系统

```
sudo apt update

sudo apt install python3.6 python3-pip
```



### 2.验证安装

运行以下命令检查Python和pip是否安装成功：

```
python3 --version

pip3 --version
```

输出应显示Python 3.6及pip版本。

### 3.运行Python代码

直接运行交互式解释器

在终端输入以下命令启动Python交互式环境：

```
python3
```

然后输入代码，例如：

```
print("Hello, World!")
```

运行Python脚本文件

将代码保存为*script.py*，然后通过以下命令运行：

```
python3 script.py
```

### 4.设置脚本可执行权限

如果希望直接运行脚本文件：

- 在脚本开头添加解释器声明：

```
#!/usr/bin/env python3
```

- 设置可执行权限：

```
chmod +x script.py
```

- 直接运行脚本：

```
./script.py
```

### 5.使用虚拟环境（可选）

为隔离项目依赖，建议使用虚拟环境：

```
python3 -m venv myenv

source myenv/bin/activate # 激活虚拟环境

pip install <package_name> # 安装依赖包

deactivate # 退出虚拟环境
```

通过以上步骤，您可以轻松在Linux上安装并运行Python3.6代码。
