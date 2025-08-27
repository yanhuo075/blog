---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 13.8 开发简单的URL短链接
order: 7
---

# python实战练手项目---开发简单的URL短地址生成服务

## 1. 什么是URL短地址

第一次接触URL短地址是在微博上，很早以前，发一条微博只能使用140个字，不知道现在怎样，很久不玩微博了。当你发一条微博时，如果需要引用一个网址，那么这个网址URL就会挤占这条微博的内容，为了避免这种情况，微博会将你写的长URL转换为短URL。此外，我们现在经常收到一些手机短信里也会包含一些短URL，也是同样的道理，太长的URL写在短信里，会影响阅读和使用。

为了避免内容里出现较长的URL，很多公司提供了这种长URL转短URL的服务，比如百度的https://dwz.cn/
![短地址生成](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827104706553.jpeg)
这个服务可以免费使用，但单个网址一个月只能使用100次，我网站的URL是http://www.coolpython.net/， 利用百度的服务生成的短地址是 https://dwz.cn/Slo1bcwe

你应该注意到，这个短地址的域名仍然是dwz.cn， 和我的网站没有半点关系，当你在浏览器里输入这个短地址时，浏览器会向百度发起请求，百度收到请求后，根据Slo1bcwe找到所对应的真实地址 http://www.coolpython.net/， 然后返回302重定向响应，在响应头里，会有一个Location部首，这里记录了我的网站地址，浏览器根据这个信息向我的网站发起请求，由于这个过程极快，而且都是在浏览器内部处理，因此你完全察觉不到。

301和302都是重定向，301是永久的，302是临时的，如果使用301，那么就无法统计到短地址被点击的次数了，百度和谷歌甚至在搜索结果里甚至可能直接展示短地址所对应的长地址，因此建议使用302。

## 2. 短URL生成算法

### 2.1 不靠谱算法--长地址转短地址， 并且可逆

很多人第一时间想到的算法是将长地址转换为短地址，并且实现短地址到长地址的逆运算。这个算法听起来很玄妙，但根本无法实现。设想一下，将一个长度大约100个字符的url压缩到长度为10的字符串，并且过程可逆，这分明是在做无损压缩，哪里有这么好的压缩算法？

有人接着想了，既然可逆不行，那么干脆不可逆，只记录短地址和长地址的映射关系。可这样还是不行，问题出在你无法写出一个长地址到短地址的转换算法，这个算法必然会出现碰撞，也就是多个长地址对应到一个短地址。

### 2.2 不靠谱算法--- hash算法

有人想到用hash算法，当出现碰撞时，在后面加1， 2， 3.....这样听起来可行，但会极大的增加工程难度，因为你不得不在系统里记录这些已经发生碰撞的信息。

### 2.3 不靠谱算法---随机生成

既然写不出长地址转短地址的算法，那么干脆随机生成吧，生成以后，在系统里查找，这个随机生成的短地址是否已经使用过，如果已经用过，就再随机生成一次，直到生成一个没有使用过的短地址。这个方法的弊端在于，随着短地址生成量的增加，随机生成一个已经使用过的短地址的概率会逐渐变大，最后，为了生成一个新的短地址，系统可能需要随机很多次才行。

### 2.4 靠谱算法---拨号

正确的算法是使用拨号策略，给每一个长地址发一个号即可，小的系统直接使用mysql的主键自增就可以了，稍大点的系统可以考虑分布式key-value系统做发号器。

以百度提供的dwz.cn为例，第一个使用系统的人，给他的长地址分配1，短地址就是dwz.cn/1, 第二个为dwz.cn/2,为了更好的利用短地址，一般使用62进制，a-z, A-Z,0-9刚好是62个字符，第15个使用系统的人分配的是dwz.cn/f,在62进制中，f表示15。

使用这种算法生成短地址，可以避免前面3个算法所提到的各种问题，而你只需要实现一个10进制到62进制的转换即可。

## 3. python 实现URL短地址生成系统

接下来，和我一起逐步实现一个特别简单的URL短地址生成系统。

### 3.1 10进制与62进制互相转换

```python
ALPHABET = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"

def base62_encode(num, alphabet=ALPHABET):
    """
    10进制转62进制
    :param num:
    :param alphabet:
    :return:
    """
    if (num == 0):
        return alphabet[0]
    arr = []
    base = len(alphabet)
    while num:
        rem = num % base
        num = num // base
        arr.append(alphabet[rem])
    arr.reverse()
    return ''.join(arr)

def base62_decode(string, alphabet=ALPHABET):
    """
    62进制转10进制
    :param string:
    :param alphabet:
    :return:
    """
    base = len(alphabet)
    strlen = len(string)
    num = 0

    idx = 0
    for char in string:
        power = (strlen - (idx + 1))
        num += alphabet.index(char) * (base ** power)
        idx += 1

    return num

if __name__ == '__main__':
    print(base62_encode(4302842))
    print(base62_decode('i3mG'))
```

### 3.2 存储设计

使用sqlite作为存储长短地址映射关系的数据库, 我设计了两张表，url表用于存储短地址和长地址的映射关系， url_index表存储拨号数据，此外提供add_url， get_long_url， get_max_index三个函数

```python
import sqlite3

conn = sqlite3.connect('url.db')


def create_table():
    cursor = conn.cursor()
    table_sql = """
    create table url(
      id INTEGER PRIMARY KEY autoincrement NOT NULL ,
      short_url text NOT NULL,
      long_url text NOT NULL
    )
    """
    cursor.execute(table_sql)
    conn.commit()       # 一定要提交,否则不会执行sql

    table_sql = """
    create table url_index(
      id INTEGER PRIMARY KEY autoincrement NOT NULL,
      useless INTEGER
    )
    """
    cursor.execute(table_sql)
    conn.commit()       # 一定要提交,否则不会执行sql
    cursor.close()
```

### 3.3 使用flask实现简单的URL短地址生成服务

flask服务需要提供一个输入长地址的页面
![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250827104634324.jpeg)
同时提供一个生成短地址的接口, 这两部分用一个视图处理即可

```python
@app.route('/', methods=['POST', 'GET'])
def index():
    if request.method == 'GET':
        return render_template('index.html')
    else:
        url = request.form['url']
        index = get_max_index()
        token = base62_encode(index)
        add_url(url, token)
        short_url = 'http://127.0.0.1:8002/{token}'.format(token=token)
        return jsonify({'short_url': short_url})
```

