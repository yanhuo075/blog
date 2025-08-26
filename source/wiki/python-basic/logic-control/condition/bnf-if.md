---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 5.1.6 以BNF为基础理解if语句
order: 5
---

# 以BNF为基础，理解python if 语句的语法

BNF是巴科斯范式， 是一种用递归的思想来表述计算机语言符号集的定义规范，在我的基础教程中，对if语句的语法进行了详细的讲解，虽然说了人话，但终究是缺少了技术研究所需要的严谨与规范。如果你认为自己理解能力很强，那么可以尝试阅读本文，将有更大的收获。

### 1. if_stmt

在[python官方文档](https://docs.python.org/3.6/reference/compound_stmts.html#if) 中，if语句的BNF描述如下：

```python
if_stmt ::=  "if" expression ":" suite
             ("elif" expression ":" suite)*
             ["else" ":" suite]
```

不了解BNF语法没有关系，这东西其实很简单，我来解读这段内容：

1. if语句必须以if开头
2. 紧跟在if语句后面的是表达式（expression）， 什么是表达式，可以阅读我的另一篇教程[表达式](http://www.coolpython.net/python_primary/expression/expression.html)， 表达式的后面必须是一个冒号，冒号后面是一个suite
3. suite 有自己的BNF描述，这里不展开讲，你可以把suite理解为一个全新的开始，既然是一个全新的开始，你在这里可以写任意的代码
4. elif 的语法与if是相同的，用小括号括起来并在后面追加一个*， 这个*表示0个或多个，因此，if语句后面可以没有elif， 也可以有多个elif
5. else 语句，用中括号括起来，表示可选，else可以有0个，或者1个

上面这段，采用文字描述，普通人大概率可以看得懂，但是不免繁琐冗长，BNF语法并不复杂，看过我的解释，你用这种元语言来理解python的if语句的语法，是不是更加清晰一些呢？

那么学习这个东西有什么用呢？

我的观点是，学习编程技术，如果你的目的性太强，总想着一步到位知晓答案，那么一定无法做到精纯。精纯的前提是博而广，打下这个基础，才能向精而专方向发展。

### 2. suite

关于这个suite，我考虑再三，还是在这篇教程里加以说明

```python
compound_stmt ::=  if_stmt
                   | while_stmt
                   | for_stmt
                   | try_stmt
                   | with_stmt
                   | funcdef
                   | classdef
                   | async_with_stmt
                   | async_for_stmt
                   | async_funcdef
suite         ::=  stmt_list NEWLINE | NEWLINE INDENT statement+ DEDENT
statement     ::=  stmt_list NEWLINE | compound_stmt
stmt_list     ::=  simple_stmt (";" simple_stmt)* [";"]
```

1. suite 可以是一个stmt_list， 也可以是一个statement或者多个statement， statement后面的+表示重复至少1次， 在开始这个statement之前，首先要开启新的一行，并且正确缩进。
2. statement 可以是一个stmt_list， 也可以是compound_stmt， 看compound_stmt的定义，它可以是if语句，while语句，for语句，好多好多，只看名字，你大概就可以猜测出它的含义
3. suite 和 statement 的定义里，都提到了stmt_list
4. stmt_list 可以是一个 simple_stmt， 它后面可以跟随0个或多个(";" simple_stmt)

分析到这里，simple_stmt是最后的关键

```python
simple_stmt ::=  expression_stmt
                 | assert_stmt
                 | assignment_stmt
                 | augmented_assignment_stmt
                 | annotated_assignment_stmt
                 | pass_stmt
                 | del_stmt
                 | return_stmt
                 | yield_stmt
                 | raise_stmt
                 | break_stmt
                 | continue_stmt
                 | import_stmt
                 | future_stmt
                 | global_stmt
                 | nonlocal_stmt
```

simple_stmt 都是最简单的语句，比如赋值语句，pass， break, import , continue

### 3. 对python语法的全新认识

if 语句的冒号后面，是一个suite， 看suite的定义，stmt_list 可以和if在同一行，而不需要重新起一行的，验证一下

```python
if 32 > 2: print('ok')
```

上面的代码虽然丑陋且不符合python的提倡，但它的确合法。

根据stmt_list 的定义，多条语句可以写在同一行，只需要用分号分隔开就好了，那么下面的代码也是合法的

```python
a = 3; b=4 ; print(a, b)
```
