---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 13.9 制作图片验证码
order: 8
---

# python实战练手项目---制作网站登录时的图片验证码

许多网站在注册时都要求输入验证码，这样做为了防止被程序恶意注册，本篇教程带你研究一下如何生成网站验证码

## 1. Pillow

PIL(Python Imaging Library)是一个强大的python图像处理库，只支持到python2.7, Pillow虽说是PIL的一个分支，但已经发展成比PIL本身更具活力的图像处理库，我们使用Pillow来生成验证码，安装方式为

```text
pip3 install Pillow
```

## 2. 生成一张指定大小随机颜色的图片

### 2.1 随机颜色

颜色的处理使用(r,g,b)格式，r, g, b 的范围是[0, 255]， 使用random模块的randint方法生成3个随机数

```python
def random_color():
    c1 = random.randint(0, 255)
    c2 = random.randint(0, 255)
    c3 = random.randint(0, 255)
    return c1, c2, c3
```

### 2.2 生成一张指定大小的图片

```python
def random_color():
    c1 = random.randint(0, 255)
    c2 = random.randint(0, 255)
    c3 = random.randint(0, 255)
    return c1, c2, c3


def generate_picture(width=120, height=35):
    image = Image.new('RGB', (width, height), random_color())
    return image

if __name__ == '__main__':
    image = generate_picture()
    image.save('test.png')
```

![Pillow生成图片](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827104757228.jpeg)
现在还只是生成了一张颜色随机的图片，接下来要在图片上面写上随机数字和字母

## 3. 在图片上写上随机数字和字母

### 3.1 生成随机数字和字母

```python
def get_random_str():
    '''
    获取一个随机字符, 数字或小写字母
    :return:
    '''
    random_num = str(random.randint(0, 9))
    random_low_alpha = chr(random.randint(97, 122))
    random_char = random.choice([random_num, random_low_alpha])
    return random_char
```

使用random模块提供的随机函数生成指定长度的字符串

### 3.2 在image对象上画数字和字母

```python
def draw_str(count, image, font_size):
    """
    在图片上写随机字符
    :param count: 字符数量
    :param image: 图片对象
    :param font_size: 字体大小
    :return:
    """
    draw = ImageDraw.Draw(image)
    # 获取一个font字体对象参数是ttf的字体文件的目录，以及字体的大小
    font_file = os.path.join('Andale Mono.ttf')
    font = ImageFont.truetype(font_file, size=font_size)
    temp = []
    for i in range(count):
        random_char = random_str()
        draw.text((10+i*30, -2), random_char, random_color(), font=font)
        temp.append(random_char)

    valid_str = "".join(temp)    # 验证码
    return valid_str, image

if __name__ == '__main__':
    image = generate_picture()
    valid_str, image = draw_str(4, image, 35)
    image.save('test.png')
```

![pillow生成验证码图片](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827104734763.jpeg)
创建一个ImageDraw.Draw对象，在image上画随机字符，你可以设置字体， 我使用了Andale Mono.ttf， mac电脑上在/System/Library/Fonts 目录下找到字体，其他系统也有各自的字体文件，将字体文件复制到与脚本相同的目录下。

## 4. 制造噪点

为了防止验证码被轻易的破解，还应该在图片上制造一些噪点，随机画几条线，随机画几个点

```python
def noise(image, width=120, height=35, line_count=3, point_count=20):
    '''

    :param image: 图片对象
    :param width: 图片宽度
    :param height: 图片高度
    :param line_count: 线条数量
    :param point_count: 点的数量
    :return:
    '''
    draw = ImageDraw.Draw(image)
    for i in range(line_count):
        x1 = random.randint(0, width)
        x2 = random.randint(0, width)
        y1 = random.randint(0, height)
        y2 = random.randint(0, height)
        draw.line((x1, y1, x2, y2), fill=random_color())

        # 画点
        for i in range(point_count):
            draw.point([random.randint(0, width), random.randint(0, height)], fill=random_color())
            x = random.randint(0, width)
            y = random.randint(0, height)
            draw.arc((x, y, x + 4, y + 4), 0, 90, fill=random_color())

    return image


if __name__ == '__main__':
    image = generate_picture()
    valid_str, image = draw_str(4, image, 35)
    image = noise(image)
    image.save('test.png')
```

![pillow生成验证码图片](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827104744104.jpeg)

## 5. 生成base64编码的图片

实践中，如果是生成网站注册使用的验证码图片，一般来说不会将其保存到图片文件中，因为这会生成大量的小图片，完全没必要。我们可以将图片的内容保存到BytesIO对象中，最终生成base64编码的图片，这样，向前端传回去的就是字符串，格式为

```text
data:image/jpeg;base64,iVBORw0KGgoAAAANSUhEUgAAAH...
```

data:image/jpeg;base64 这一段是固定写法，逗号剩余的部分是图片经过base64编码后的字符串

```python
def valid_code():
    """
    生成图片验证码,并对图片进行base64编码
    :return:
    """
    image = generate_picture()
    valid_str, image = draw_str(4, image, 35)
    image = noise(image)

    f = BytesIO()
    image.save(f, 'png')        # 保存到BytesIO对象中, 格式为png
    data = f.getvalue()
    f.close()

    encode_data = base64.b64encode(data)
    data = str(encode_data, encoding='utf-8')
    img_data = "data:image/jpeg;base64,{data}".format(data=data)
    return valid_str, img_data

if __name__ == '__main__':
    print(valid_code())
```

## 6. 全部代码

为了向你阐述生成验证码图片的过程，我将整个过程进行拆解，因此代码也被拆解的凌乱，你可以将这些代码整合,以便在实际应用中使用

```python
import os
import random
import base64
from io import BytesIO
from PIL import Image
from PIL import ImageDraw
from PIL import ImageFont


def random_color():
    c1 = random.randint(0, 255)
    c2 = random.randint(0, 255)
    c3 = random.randint(0, 255)
    return c1, c2, c3


def generate_picture(width=120, height=35):
    image = Image.new('RGB', (width, height), random_color())
    return image


def random_str():
    '''
    获取一个随机字符, 数字或小写字母
    :return:
    '''
    random_num = str(random.randint(0, 9))
    random_low_alpha = chr(random.randint(97, 122))
    random_char = random.choice([random_num, random_low_alpha])
    return random_char

def draw_str(count, image, font_size):
    """
    在图片上写随机字符
    :param count: 字符数量
    :param image: 图片对象
    :param font_size: 字体大小
    :return:
    """
    draw = ImageDraw.Draw(image)
    # 获取一个font字体对象参数是ttf的字体文件的目录，以及字体的大小
    font_file = os.path.join('Andale Mono.ttf')
    font = ImageFont.truetype(font_file, size=font_size)
    temp = []
    for i in range(count):
        random_char = random_str()
        draw.text((10+i*30, -2), random_char, random_color(), font=font)
        temp.append(random_char)

    valid_str = "".join(temp)    # 验证码
    return valid_str, image


def noise(image, width=120, height=35, line_count=3, point_count=20):
    '''

    :param image: 图片对象
    :param width: 图片宽度
    :param height: 图片高度
    :param line_count: 线条数量
    :param point_count: 点的数量
    :return:
    '''
    draw = ImageDraw.Draw(image)
    for i in range(line_count):
        x1 = random.randint(0, width)
        x2 = random.randint(0, width)
        y1 = random.randint(0, height)
        y2 = random.randint(0, height)
        draw.line((x1, y1, x2, y2), fill=random_color())

        # 画点
        for i in range(point_count):
            draw.point([random.randint(0, width), random.randint(0, height)], fill=random_color())
            x = random.randint(0, width)
            y = random.randint(0, height)
            draw.arc((x, y, x + 4, y + 4), 0, 90, fill=random_color())

    return image


def valid_code():
    """
    生成图片验证码,并对图片进行base64编码
    :return:
    """
    image = generate_picture()
    valid_str, image = draw_str(4, image, 35)
    image = noise(image)

    f = BytesIO()
    image.save(f, 'png')        # 保存到BytesIO对象中, 格式为png
    data = f.getvalue()
    f.close()

    encode_data = base64.b64encode(data)
    data = str(encode_data, encoding='utf-8')
    img_data = "data:image/jpeg;base64,{data}".format(data=data)
    return valid_str, img_data

if __name__ == '__main__':
    print(valid_code())
```
