---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 13.14 获取电脑cpu的速度
order: 13
---

# python实战练手项目---获取电脑cpu的速度

## 1. CPU的速度

cpu的速度用时钟频率来衡量，它是1秒内发生的同步脉冲数，单位是Hz（赫兹），通常来说，频率越大，cpu的性能越强劲，以3个最常用的系统为目标，研究一下，如何用python来获取电脑的cpu频率。

## 2.获取mac电脑cpu频率

cpu属于硬件，在终端上使用命令 system_profiler SPHardwareDataType ，可以获取电脑的硬件信息

```text
Hardware:

    Hardware Overview:

      Model Name: MacBook Air
      Model Identifier: MacBookAir7,2
      Processor Name: Intel Core i5
      Processor Speed: 1.6 GHz
      Number of Processors: 1
      Total Number of Cores: 2
      L2 Cache (per Core): 256 KB
      L3 Cache: 3 MB
      Memory: 8 GB
      Boot ROM Version: MBA71.0178.B00
      SMC Version (system): 2.27f2
      Serial Number (system): C1MR961GG943
      Hardware UUID: C42A211A-EFD1-55C3-9EEF-A1F3A19A618C
```

Processor Speed: 1.6 GHz 就是cpu的频率信息， 可以使用subprocess.Popen方法在程序里执行命令然后获取输出内容，对内容进行分析，获取频率。

## 3. 获取linux系统电脑的cpu频率

linux系统下，cpu的信息都存储在文件 /proc/cpuinfo 中，使用cat /proc/cpuinfo 就可以直接观察

```text
[root@sheng ~]# cat /proc/cpuinfo
processor   : 0
vendor_id   : GenuineIntel
cpu family  : 6
model       : 63
model name  : Intel(R) Xeon(R) CPU E5-2680 v3 @ 2.50GHz
stepping    : 2
microcode   : 0x1
cpu MHz     : 2494.224
cache size  : 30720 KB
physical id : 0
siblings    : 1
core id     : 0
cpu cores   : 1
```

cpu MHz 这一行就是我们要找的内容，由于信息存储在文件里，因此可以直接读取文件，来获取cpu 频率

## 4. 获取windows系统电脑cpu频率

windows系统电脑里，cpu的信息存储在注册表中，使用win+r 键组合，打开运行程序
![regedit](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827105042923.jpeg)
输入regedit，点击确定按钮
![windows查看cpu信息](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827105053725.jpeg)
python的标准库 winreg 可以用来操作注册表，只要知道路径，就可以准确获取cpu的频率

## 5. 识别操作系统

程序应当可以识别出不同的操作系统，并根据操作系统的不同来采用对应的方法获取cpu的速度，标准模块platform 可以获取平台的信息，platform.system() 可以获取操作系统的名称，下面是该方法在各个操作系统下执行时返回的名称

1. mac 上返回 Darwin
2. linux 上返回 Linux
3. windows 上返回 Windows 或者 Win32

## 6. 全部代码

```python
import platform
import subprocess
import fileinput


def get_mac_cpu_speed():
    commond = 'system_profiler SPHardwareDataType | grep "Processor Speed" | cut -d ":" -f2'
    proc = subprocess.Popen([commond], shell=True, stdout=subprocess.PIPE)
    output = proc.communicate()[0]
    output = output.decode()   # bytes 转str
    speed = output.lstrip().rstrip('\n')
    return speed


def get_linux_cpu_speed():
    for line in fileinput.input('/proc/cpuinfo'):
        if 'MHz' in line:
            value = line.split(':')[1].strip()
            value = float(value)
            speed = round(value / 1024, 1)
            return "{speed} GHz".format(speed=speed)


def get_windows_cpu_speed():
    import winreg
    key = winreg.OpenKey(winreg.HKEY_LOCAL_MACHINE, r"HARDWARE\DESCRIPTION\System\CentralProcessor\0")
    speed, type = winreg.QueryValueEx(key, "~MHz")
    speed = round(float(speed)/1024, 1)
    return "{speed} GHz".format(speed=speed)


def get_cpu_speed():
    osname = platform.system()  # 获取操作系统的名称
    speed = ''
    if osname == "Darwin":
        speed = get_mac_cpu_speed()
    if osname == "Linux":
        speed = get_linux_cpu_speed()
    if osname in ["Windows", "Win32"]:
        speed = get_windows_cpu_speed()

    return speed

print(get_cpu_speed())
```
