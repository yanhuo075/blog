---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 13.7 发送网站注册激活邮件
order: 6
---

# python实战练手项目---发送网站注册后的激活邮件

## 1. 激活邮件

当你注册了一个网站，通常，网站会向你的邮箱里发一封激活邮件，邮件里会有一个url,只有当你点击这个url，才能激活你的用户身份。

本篇文章，尝试讨论激活邮件的实现方法

## 2. 身份确认

向你的邮箱里发送一封激活邮件，可以确认这个邮箱是不是真实可用的，可以防止恶意注册，不管你是手动恶意注册还是写程序恶意注册，这种方法都会增加恶意注册的成本。

如何完成身份确认呢？通常，邮箱里的那个url里会有一个token，这个token是所有问题的关键，当你点击url，网站后台会受到这个请求，得到这个token，网站必须通过这个token找到你是谁

### 2.1 随机token

当你注册以后，用户会为你分配一个唯一id，这时，可以生成一个随机token，然后在数据库里存储一条数据，记录这个token和id的映射关系。

当激活的url被点击后，后台得到这个token，拿着这个token去数据库里找到与之对应的id，那么，就知道你这个账号被激活了

### 2.2 token携带信息

还是注册以后，分配了唯一id,使用这个id生成一个token，最直接的办法就是对id进行加密生成token, 为了防止伪造，可以在生成token过程中使用私钥，这样，在没有私钥的情况，无法破解token得到id。

当激活的url被点击后，后台得到token，拿着私钥就可以解密这个token得到id，这样做省去了2.1中的token和id的映射关系存储

### 2.3 有效时间

激活也是有时间限制的，比如两天内激活有效，超过两天，你再来激活就无效了。

前面的随机token和token携带信息两种方法，都可以方便加入时间限制，对于随机token，只需要记录这个token的过期时间就可以了，对于token携带信息这种方法，在生成token时，将过期时间也加入其中就可以了，不管用哪种方法，得到token后对比时间就知道是不是已经过期了

## 3. python实现

使用itsdangerous 模块，可以轻松的生成临时身份令牌，也就是token

```text
pip3 install itsdangerous
```

### 3.1 生成token

```python
import time
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer

SECRET_KEY = 'owiernwertwet'
s = Serializer(SECRET_KEY, 3)
data = s.dumps({'user_id': 343})
token = data.decode()
print(token)
```

token的过期时间是3秒钟，设置为3秒是为了试验方便，生产环境下需要根据实际情况来设置，而且token携带了user_id这个重要的信息，不过不要怕，只要你不把SECRET_KEY告诉别人，那么这个token就是安全的

### 3.3 解析token

```python
import time
from itsdangerous import TimedJSONWebSignatureSerializer as Serializer

SECRET_KEY = 'owiernwertwet'
s = Serializer(SECRET_KEY, 3)
data = s.dumps({'user_id': 343})
token = data.decode()
print(token)


time.sleep(2)
serializer = Serializer(SECRET_KEY)
data = serializer.loads(token)
print(data)
```

解析token的时候也要使用SECRET_KEY，我在解析前sleep2秒钟，如果你改为4秒钟，那么由于token已经过期，解析就会失败
