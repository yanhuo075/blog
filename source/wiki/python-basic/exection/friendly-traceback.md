---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 10.9 快速定位代码语法错误
order: 8
---

# friendly-traceback，帮你快速定位代码语法错误

初学者在编写代码时，时常会犯一些低级的语法错误，由于知识掌握的不够扎实，往往一时间无法找到是哪里出了问题。python提供了异常和错误信息，也就是traceback，对于有经验的人，可以根据这些提示信息很快便定位到问题只所在，但对于初学者来说，这些错误信息用处不大，一方面是因为初学者还不够熟练，更重要的是，这些提示信息所提供的有价值的信息太少了，比如下面的错误

```python
if len('hello') = 5:
    print('ok')
```

错误提示内容是

```text
  File "C:/Users/zhangdongsheng/PycharmProjects/liepin/test.py", line 1
    if len('hello') = 5:
                    ^
SyntaxError: invalid syntax
```

这段错误信息指出了错误的具体位置，可初学者正是因为基础不扎实才会犯错误，上面的信息对于他们来说，提示性仍然不够。

那么，有没有什么办法，可以获得更加具有提示性的错误信息呢？强烈建议安装使用friendly-traceback模块，一切语法错误都将不再是问题

```text
pip install friendly-traceback
```

如果你的某个脚本里有语法错误，依靠python原生提供的错误信息不足以定位问题，那么就可以使用该模块，上面的代码写在脚本test.py中，那么你可以这样来执行

```text
python -m friendly_traceback test.py
```

得到的错误提示为

```text
┌─────────────────────────────────────────────────────────────────── Python exception ───────────────────────────────────────────────────────────────────┐
│ Traceback (most recent call last):                                                                                                                     │
│   File "test.py", line 1                                                                                                                               │
│     if len('hello') = 5:                                                                                                                               │
│                     ^                                                                                                                                  │
│ SyntaxError: invalid syntax                                                                                                                            │
│                                                                                                                                                        │
│ Perhaps you needed == instead of =.                                                                                                                    │
│                                                                                                                                                        │
│ A SyntaxError occurs when Python cannot understand your code.                                                                                          │
│                                                                                                                                                        │
│ Python could not understand the code in the file 'test.py' beyond the location indicated by ^.                                                         │
│                                                                                                                                                        │
│     -->1: if len('hello') = 5:                                                                                                                         │
│                           ^                                                                                                                            │
│                                                                                                                                                        │
│ You used an assignment operator = instead of an equality operator ==.                                                                                  │
└────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

注意看最后一行

```text
You used an assignment operator = instead of an equality operator ==. 
```

几乎就等于告诉你应该怎样去修改，这就厉害了呀，不仅仅是之处问题在哪，而且还会给出非常准确的修改方案，即便是初学小白，也能快速修复问题了。

不只是语法错误，这个库对异常和运行时的错误都能给出非常具体的有参考意义的提示，如果python发行版能将这个模块内置其中就好了。
