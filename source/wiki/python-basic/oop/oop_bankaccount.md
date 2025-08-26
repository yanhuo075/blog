---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 11.6 面向对象编程示例
order: 5
---

# python面向对象编程示例---银行账号

在最初接触面向对象编程时，你会感到有些不习惯，但这种编程范式却有助于我们思考问题，前提是你准确的理解面向对象这四个字的含义。今天，我以银行账户为例向你展示如何利用面向对象的编程方法来编写一个银行账户，这个过程中，你将感受到面向对象编程与面向过程编程的不同。

## 1. 属性与方法

我需要一个银行账户的类，我将其命名为BankAccount, 它应该有如下的属性：

1. 用户姓名（username）
2. 账号(card_no)
3. 余额(balance)

这个类还应该有几个方法：

1. 存钱（deposit）
2. 取钱（withdrawal）
3. 转账（transfer）
4. 查看操作记录（history）

## 2. 业务分析

在取钱时，如果账户余额小于所取金额，那么要提示用户余额不足，在转账的时候同样如此。在存钱，取钱，转账时，都必须将业务操作记录保存到一个列表中，查看历史记录时，遍历这个列表，这里我不考虑查询历史记录的时间范围。

## 3. 代码实现

### 3.1 定义类

```python
    from datetime import datetime


    class BankAccount(object):

    def __init__(self, username, card_no, balance):
        self.username = username     # 用户姓名
        self.card_no = card_no       # 账号
        self.balance = balance       # 余额
        self.history_lst = []        # 历史操作记录

    def deposit(self, amount):
        '''
        存钱
        :param amount:
        :return:
        '''
        pass

    def withdrawal(self, amount):
        '''
        取钱
        :param amount:
        :return:
        '''
        pass

    def transfer(self, another, amount):
        '''
        转账
        :param another:
        :param amount:
        :return:
        '''
        pass

    def history(self):
        '''
        历史操作记录
        :return:
        '''
        pass
```

如果业务分析已经很透彻，你可以像我这样一次性将类的全部属性和方法都写好，之后逐步完善，这样做事井井有条，不至于丢三落四。不过你很难一次性想清楚所有细节，因此在编码过程中，还会逐渐补充。

### 3.2 实现存钱方法

不论进行何种操作，金额都必须是大于0的，因此，我需要一个判断金额是否合法的方法

```python
    @classmethod
    def is_amount_legitimate(cls, amount):
        '''
        判断金额是否合法
        :return:
        '''
        if not isinstance(amount, (float, int)):
            return False

        if amount <= 0:
            return False

        return True
```

由于这个方法无需访问实例属性，因此我把它定义为类方法，每一次操作，都需要记录，记录里必然包括操作发生的时间，因此还需要一个获取当前时间的方法

```python
    @classmethod
    def _get_current_time(cls):
        now = datetime.now()
        current_time = now.strftime('%Y-%m-%d %H:%M:%S')
        return current_time
```

现在，来实现存钱方法

```python
    def deposit(self, amount):
        '''
        存钱
        :param amount:
        :return:
        '''
        if not BankAccount.is_amount_legitimate(amount):
            print('金额不合法')
            return

        self.balance += amount
```

### 3.3 实现取钱

转账，都需要判断操作的金额是否小于等于账户余额，看来，我需要一个方法来完成这件事情

```python
    def withdrawal(self, amount):
        '''
        取钱
        :param amount:
        :return:
        '''
        if not self.is_amount_legitimate(amount):
            print('金额不合法')
            return

        self.balance -= amount
        log = "{operate_time}取出金额{amount}".format(operate_time=self._get_current_time(), amount=amount)
        self.history_lst.append(log)
```

### 3.4 实现转账

此前，我只定义了转账函数，还缺少一个接收转账的函数，这两个函数一起实现

```python
    def transfer(self, another, amount):
        '''
        转账
        :param another:
        :param amount:
        :return:
        '''
        self.balance -= amount
        another.accept_transfer(self, amount)
        log = '{operate_time}向{username}转账{amount}'.format(operate_time=self._get_current_time(),
                                                           username=another.username,
                                                           amount=amount)
        self.history_lst.append(log)

    def accept_transfer(self, another, amount):
        '''
        接收转账
        :param another:
        :param amount:
        :return:
        '''
        self.balance += amount
        log = '{operate_time}收到{username}转来的{amount}'.format(operate_time=self._get_current_time(),
                                                           username=another.username,
                                                           amount=amount)
        self.history_lst.append(log)
```

### 3.5 查看历史操作记录

```python
    def history(self):
        '''
        历史操作记录
        :return:
        '''
        for log in self.history_lst:
            print(log)
```

### 3.6 测试

```python
def test():
    account_1 = BankAccount('小明', '248252543', 4932.13)
    account_2 = BankAccount('小红', '429845363', 3211.9)

    account_1.deposit(400)      # 存钱
    account_1.withdrawal(300)   # 取钱

    account_1.transfer(account_2, 1024)     # 转账
    account_1.history()                     # 查询历史操作记录

    account_2.history()

if __name__ == '__main__':
    test()
```

测试输出结果

```text
2020-01-09 19:48:01 存入金额400
2020-01-09 19:48:01取出金额300
2020-01-09 19:48:01向小红转账1024
2020-01-09 19:48:01收到小明转来的1024
```
