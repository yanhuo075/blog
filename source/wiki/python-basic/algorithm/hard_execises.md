---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 14.3 困难练习题
order: 2
---

# 二进制求和

## 题目要求

计算两个二进制字符串的和

```text
输入: 111  1110 
输出: 10101
```

## 思路分析

参与计算的二进制字符串长度可以不同，这样为计算带来麻烦，所以，首先要补齐那个较短的字符串。如果较短字符串长度比长字符串小3，就在较短字符串前面补3个0.

计算的过程，可以模拟手动计算的过程，从右向左逐位计算。反向遍历字符串，相同位上都是1，相加后，这一位就变成了0，而且还要向前进位，下一位的计算要把进位的数值考虑进来，计算右数第一位时，进位设置为0。

## 示例代码

```python
def binary_add(str_1, str_2):
    # 先补齐
    str_1_length = len(str_1)
    str_2_length = len(str_2)
    if str_1_length < str_2_length:
        str_1 = "0"*(str_2_length - str_1_length) + str_1
    else:
        str_2 = "0"*(str_1_length - str_2_length) + str_2

    # 进行计算
    index = len(str_1) - 1
    pre_num = 0         # 记录进位
    res_lst = []        # 记录结果

    # 方向遍历
    while index >= 0:
        item_1 = int(str_1[index])
        item_2 = int(str_2[index])
        item_sum = item_1 + item_2 + pre_num
        pre_num = item_sum // 2
        curr_num = item_sum % 2

        # 新的计算结果插入到结果的第一位
        res_lst.insert(0, str(curr_num))
        index -= 1

    if pre_num == 1:
        res_lst.insert(0, '1')

    return ''.join(res_lst)


if __name__ == '__main__':
    print(binary_add("111", '1110'))
    print(binary_add("11", '1'))
    print(binary_add("101", '1001'))
```



# 二进制中为1的位数

## 题目要求

给定一个整数，请计算二进制中为1的位数

```text
输入: 13
输出: 3
解释: 13的二进制表示是 1101，位为1的数量是3 
```

## 思路分析

如果一个数是奇数，那么它的二进制的最后一位一定是1，道理很简单，其他的位都表示2n 只有最后一位表示20 。我们可以利用最后一位是否为1来统计为1的位数，这就需要最后一位是变化的，还好，我们可以用位运算符 >> (右移位运算符)

13 的 二进制表示是 1101 ，13>>1 就表示二进制的每一位都向右移动一位，移动后为 110，最右边的1舍弃。如果二进制最后一位是1，那么一定是奇数。

## 示例代码

```python
# coding=utf-8


def count_one_bits(value):
    count = 0
    # 当value等于0时,二进制的每一位都是0
    while value:
        if value % 2 == 1:
            count += 1
        value = value >> 1
    return count


if __name__ == '__main__':
    print(count_one_bits(6))
    print(count_one_bits(13))
```

## 小小的升级

上面的分析中，利用奇偶数来判断最后一位是否为1，除此以外，还可以利用位运算符 &（按位与）来判断最后一位是否为1。

13 的二进制是 1101 ，1的二进制是1，13&1 等价于1101 & 0001， 相同位进行与运算，得到结果为0001。

示例代码

```python
# coding=utf-8


def count_one_bits(value):
    count = 0
    # 当value等于0时,二进制的每一位都是0
    while value:
        if value & 1 == 1:
            count += 1
        value = value >> 1
    return count


if __name__ == '__main__':
    print(count_one_bits(6))
    print(count_one_bits(13))
```



# 提取单词

## 题目要求

从字符串里提取单词，例如”this is a book“，将单词放到列表里，要求是不能使用split函数

## 思路分析

遍历字符串，空格的部分，一定不是单词，非空格的地方一定是单词。从空格到非空格是一次转变，从非空格到空格是一次转变。用两个变量start 和 end 分别记录这两次发生转变的索引位置。用b_start来标识当前索引是否在单词上。

## 示例代码

```python
string = " this is a book"
lst = []
# 记录单词开始和结束的位置
b_start = False
start = 0  # 单词开始的位置
end = 0    # 单词结束的位置

for index, item in enumerate(string):
    if item != " ":
        if b_start:
            continue
        else:
            b_start = True
            start = index
    else:
        if b_start:
            b_start = False
            end = index - 1
            lst.append(string[start:end+1])

if b_start:
    lst.append(string[start:])

print(lst)
```



# 寻找峰值

## 题目要求

峰值元素是指其值大于左右相邻值的元素，给定一个序列，请返回所有的峰值元素及其索引

示例1：

```text
输入: nums = [1,2,3,1]
输出: [(2, 3)]
```

以列表形式输出，元素为tuple，tuple[0]表示索引，tuple[1]表示元素

## 思路分析

遍历序列，对于每一个元素，都要和自己左右两侧的相邻值进行比较，如果大于两侧相邻值，那么这个元素就是峰值元素。

需要注意的是序列的两端元素，他们都只有一侧有相邻元素，因此在逻辑处理上要考虑这种边界情况。

## 示例代码

```python
def find_peak(lst):
    if len(lst) <= 2:
            return -1
    peak_lst = []
    for index, item in enumerate(lst):
        big_than_left = False       # 是否比左侧大
        big_than_right = False      # 是否比右侧大
        # 考虑边界情况
        if index == 0:
            big_than_left = True
            big_than_right = item > lst[index+1]
        elif index== len(lst) - 1:
            big_than_right = True
            big_than_left = item > lst[index-1]
        else:
            big_than_left = item > lst[index-1]
            big_than_right = item > lst[index+1]

        if big_than_left and big_than_right:
            peak_lst.append((index, item))

    return peak_lst


if __name__ == '__main__':
    lst = [1, 2, 1, 3, 5, 6, 4, 8]
    print(find_peak(lst))
```



# 第一个只出现一次的字符

## 题目要求

一个只包含小写字母的字符串，请找出第一个只出现一次的字符，并返回索引，如果这样的字符不存在返回-1，不允许使用字典

```text
输入： abcacd
输出： 1
解释： 出现一次的字符是 b 和 d，b是第一个出现的
```

## 思路分析

必须遍历字符串，统计每个字符出现的次数，然后再遍历一遍字符串，如果某个字符出现的次数是1，则返回这个字符的索引。

题目要求不允许使用字典，因此，需要换一个数据类型来存储字符串出现的次数，这里可以使用列表，创建一个长度为26的列表，元素初始化为0。小写字母一共有26个，字符a 的ascii码10进制是97，字符z的ascii码10进制是122，遍历过程中，先将字符转成ascii码十进制数值并减去97，就得到了在列表中的索引，列表索引位置元素加1，这样，就解决了字符出现次数的记录问题。

再次遍历字符串时，还是对字符进行转换，转换后的数值作为索引找到列表中的值，如果值为1，那么这个字符就是出现次数为1的字符。

## 示例代码

```python
# coding=utf-8


def find_first_single_char(string):
    """
    寻找字符串中第一个只出现一次的字符
    :param string:
    :return:
    """
    # 记录每个字符出现的次数
    lst = [0 for i in range(26)]
    for item in string:
        lst[ord(item) - 97] +=1

    for index, item in enumerate(string):
        if lst[ord(item) - 97] == 1:
            return index

    return -1


if __name__ == '__main__':
    print(find_first_single_char('abcacd'))
```



# 修改字典里的value (2)

## 1. 题目要求

有一个字典，保存的是学生各个编程语言的成绩，内容如下

```python
data = {
        'python': {'上学期': '90', '下学期': '95'},
        'c++': ['95', '96', '97'],
        'java': [{'月考':'90', '期中考试': '94', '期末考试': '98'}]
    }
```

各门课程的考试成绩存储方式并不相同，有的用字典，有的用列表，但是分数都是字符串类型，请实现函数transfer_score(score_dict)，将分数修改成int类型

## 2. 思路分析

不同于上一篇，本次字典的结构变得复杂了，但思路不变，仍然要遍历字典，只是在遍历时，要根据value的类型决定如何进行处理，如果value的类型是字典，那么则仍然按照上一篇的方法进行处理，如果value的类型是列表，则需要对列表里的元素类型进行判断，如果是字符串，则直接转换，如果是字典，则需要按照上一篇的方法进行处理，这次，我采用递归函数进行处理。

## 3. 示例代码

```python
import pprint


def transfer_score(data):
    # 如果data是字典
    if isinstance(data, dict):
        for key, value in data.items():
            data[key] = transfer_score(value)
        return data

    # 如果data 是列表
    if isinstance(data, list):
        data_lst = []
        for item in data:
            data_lst.append(transfer_score(item))

        return data_lst

    if isinstance(data, str):
        return int(data)


if __name__ == '__main__':
    data = {
        'python': {'上学期': '90', '下学期': '95'},
        'c++': ['95', '96', '97'],
        'java': [{'月考': '90', '期中考试': '94', '期末考试': '98'}]
    }

    data = transfer_score(data)
    pprint.pprint(data)
```



# 翻转字符串里的单词

## 题目要求

给定一个字符串，逐个翻转字符串中的每个单
示例:

```text
输入: " the sky is   blue",
输出: "blue   is sky the "
```

翻转后，空格不能减少，单词之间的空格数量不能发生变化

## 思路分析

如果只是单纯的翻转字符串，就过于简单了，因此要求翻转每一个单词，单词还是原来的样子，但是单词所在的位置却发生了翻转，第一个单词变成了倒数第一个单词。

可以先将整个字符串都翻转，这样单词的位置完成了翻转，结果如下：

```text
"eulb   si yks eht "
```

但这个时候，每一个单词的顺序还是颠倒的，因此需要对每一个单词进行一次翻转。

字符串是不可变对象，不能直接在字符串上进行翻转，要借助列表（list）进行翻转，翻转后在使用join方法将列表里的元素拼成字符串

## 示例代码

```python
# coding=utf-8


def reverse_word(string):
    # 先将整个句子翻转
    word_lst = list(string)
    for i in range(len(word_lst)/2):
        word_lst[i], word_lst[len(word_lst)-1-i] = word_lst[len(word_lst)-1-i], word_lst[i]

    print("".join(word_lst))
    # 翻转里面的单词
    start_index = None
    end_index = None
    b_word = False
    for index, item in enumerate(word_lst):
        if item.isalpha():
            if not b_word:
                start_index = index     # 单词开始的位置
                b_word = True
        else:
            if b_word:
                end_index = index -1   # 单词结束的位置
                b_word = False
                # 已经知道单词开始和结束的位置,翻转这个单词
                reverse_single_word(word_lst, start_index, end_index)

    return "".join(word_lst)


def reverse_single_word(lst, start_index, end_index):
    """
    将lst 中 start_index到end_index 这个区域翻转
    :param lst:
    :param start_index:
    :param end_index:
    :return:
    """
    while start_index < end_index:
        lst[start_index], lst[end_index] = lst[end_index], lst[start_index]
        start_index += 1
        end_index -= 1


if __name__ == '__main__':
    print(reverse_word(" the sky is   blue"))
```



# 1. 学生成绩分析

文件score.txt中存储了学生的考试信息，内容如下

```text
小明,98
小刚,90
小红,91
小王,98
小刘,80
```

请写代码，读取文件数据，并进行如下分析

1. 最高分和最低分分别是多少？
2. 得最高分的学生有几个？ 得最低分的学生有几个
3. 平均分是多少？

```python
def read_file(filename):
    """
    读取文件数据
    :param filename:
    :return:
    """
    f = open(filename, 'r', encoding='utf-8')
    datas = f.readlines()
    datas = [item.strip() for item in datas]
    f.close()

    return datas


def analse_score():
    datas = read_file('score.txt')
    score_lst = []

    for item in datas:
        score = int(item.split(',')[1])
        score_lst.append(score)

    max_score = max(score_lst)
    min_score = min(score_lst)

    max_score_count = score_lst.count(max_score)
    min_score_count = score_lst.count(min_score)

    avg = sum(score_lst)/len(score_lst)

    print(max_score, min_score)
    print(max_score_count, min_score_count)
    print(avg)

if __name__ == '__main__':
    analse_score()
```



# 字符串转整数

## 题目要求

实现函数atoi，将字符串转为整数，具体要求如下：

1. 字符串第一个非空字符如果不是数字，字符串无效，返回0
2. 字符串第一个非空字符如果是 + 或者 -，则表示正负，且该字符后面的第一个非空字符必须是数字，否则视为无效字符串，返回0
3. 遇到数字，将其与之后连续的数字字符组合起来形成整数
4. 有效数字范围是[−231, 231 − 1],数值超过可表示的范围，则返回231 − 1 或−231 。
   示例

```text
1、 "42" 转成 42
2、 "   -  42 43" 转成 -4243
3、 "4193 word" 转成4193
4、 "word 4193" 转成0
5、 "-91283472332" 转成-91283472332， 比−2^31还小，返回−2^31
```

## 思路分析

基本思路是遍历字符串，遍历过程中，关键点是找到第一个非空字符，这个非空字符决定了接下来程序的走向。

如果第一个非空字符是+或者-，就决定了数值的正负，后面的工作是提取数值

如果第一个非空字符是数字，事情变得简单了，直接提取数值。

如果第一个非空字符既不是数字，也不是+或-，就是无效字符串，返回0即可。

提取数值的过程，用一个列表保存单个数字，直到遇到一个既不是数字也不是空格的字符或者遇到字符串末尾，将列表里的数值连接起来并用int转成数值。

程序的最后，要和最大值和小值比较。

## 示例代码

```python
# coding=utf-8
MAX_INT = 2**31-1
MIN_INT = -2**31


def atoi(string):
    """
    将字符串转成int
    :param string:
    :return:
    """
    if not isinstance(string, basestring):
        return 0
    if string == u'':
        return 0

    tag = 1    # 标识正负, 赋值为1表示为正
    value = 0
    for index, item in enumerate(string):
        # 空字符不处理
        if item == u' ':
            continue
        elif item in ('+', '-'):
            if item == '-':
                tag = -1
            value = get_int(string, index)
            break
        elif item.isdigit():
            value = get_int(string, index)
            break
        else:
            return 0

    value = tag * value
    if value > MAX_INT:
        value = MAX_INT
    if value < MIN_INT:
        value = MIN_INT

    return value


def get_int(string, start_index):
    """
    提取数值
    :param string: 字符串
    :param start_index: 数值开始部分
    :return:
    """
    lst = []
    for index in range(start_index, len(string)):
        if string[index] == u' ':       # 空字符不用处理
            continue
        if string[index].isdigit():     # 遇到不是数字时结束
            lst.append(string[index])

    return int(''.join(lst))


if  __name__ == '__main__':
    print atoi("42")
    print atoi("   -  42 43 ")
    print atoi("4193 4 word")
    print atoi("word 4193")
    print atoi("-91283472332")
```



# 字符串相乘

## 题目要求

有两个字符串，str1,str2，内容为数值，数值均大于0请编写算法计算两个字符串相乘的结果，不可以使用大数据类型，不可以把字符串直接转成整数来处理。

```text
输入: str1='3'  str2='5'
输出: '15'

输入: str1='32'  str2='15'
输出: '480'

输入: str1='323434223434234242343'  
     str2='15492828424295835983529'
输出:
```

## 思路分析

题目要求的很明确，不可以直接把字符串转成整数然后相乘，所以int(str1)*int(str2)是不被允许的。

不止如此，第三个示例里，数值已经非常大，已经超出了264 ，在计算过程中，中间结果也非常的大，在大多数编程语言里，你找不到合适的数据类型来存储这样的数据，因此，只能用字符串来保存。

先来看第一个示例， 3 乘 5， 简单的乘法，在看第二个示例， 32 乘 15 ，小学的时候都学过如何计算乘法，32*5 + 32*10 =480。需要编写一个算法，来实现这个计算过程，但是要注意，32*5的结果必须保存成为字符串"160",为什么不能保存成数值型数据160呢？就32*15来说，保存成数值是没有问题的，可对于第三个示例，在计算中间结果时，得到的结果太大了，根本无法用数值型变量保存，因此必须保存成字符串。

### 最简单的乘法

我们需要先实现一个简单的字符串乘法，函数定义为

```python
def single_str_multi(str1, single):
    """
    single是一个小于10的数值, 例如 323354 * 4
    :param str1: 基础数值
    :param single: 单个数值
    :return:
    """
    pass
```

如果总是可以保证single是一个小于10的数值，那么计算起来就简单容易的多了，计算时，反向遍历str1，每个数值分别与single相乘，计算进位和当前位置留下的值，并存入一个列表中，遍历结束后，要注意检查最后一次进位的值，算法实现如下

```python
def single_str_multi(str1, single):
    """
    single是一个小于10的数值, 例如 323354 * 4
    :param str1: 基础数值
    :param single: 单个数值
    :return:
    """
    pre_sum = 0            # 进位
    single = int(single)
    res_lst = []
    for item in reversed(str1):
        # 每次计算都要考虑上一次计算结果的进位
        item_num = int(item) * single + pre_sum
        pre_sum = item_num / 10     # 计算进位
        curr_num = item_num % 10    # 计算当前位置的值
        res_lst.insert(0, str(curr_num))

    if pre_sum > 0:
        res_lst.insert(0, str(pre_sum))

    return ''.join(res_lst)
```

有了single_str_multi函数做基础，就可以实现最终的字符串相乘算法了，定义函数

```python
def str_multi(str1, str2):
    pass
```

这一次，反向遍历str2，每次遍历都得到一个单一数值single，这样恰好可以调用single_str_multi 函数，但是需要注意的是每次得到的结果都要根据single的位置补0，如果single是str2的百位，那么计算结果就要乘100。

### 字符串相加

每次调用single_str_multi函数，得到的都是中间结果，这些结果必须加在一起才能得到乘法的结果，因此，我们还需要一个计算字符串加法的函数，前面的计算二进制加法的练习题已经有过讲解，代码稍作修改即可

```python
def str_sum(str1, str2):
    """
    计算两个字符串的加法
    :param str1:
    :param str2:
    :return:
    """
    # 先补齐
    str_1_length = len(str1)
    str_2_length = len(str2)
    if str_1_length < str_2_length:
        str1 = "0"*(str_2_length - str_1_length) + str1
    else:
        str2 = "0"*(str_1_length - str_2_length) + str2

    # 进行计算
    index = len(str1) - 1
    pre_num = 0         # 记录进位
    res_lst = []        # 记录结果

    # 反向遍历
    while index >= 0:
        item_1 = int(str1[index])
        item_2 = int(str2[index])
        item_sum = item_1 + item_2 + pre_num
        pre_num = item_sum/10
        curr_num = item_sum % 10

        # 新的计算结果插入到结果的第一位
        res_lst.insert(0, str(curr_num))
        index -= 1

    if pre_num == 1:
        res_lst.insert(0, '1')

    return ''.join(res_lst)
```

### 字符串相乘

万事具备，之前东风，最后来实现str_multi函数

```python
def str_multi(str1, str2):
    """
    字符串相乘
    :param str1:
    :param str2:
    :return:
    """
    res = '0'
    for index, item in enumerate(reversed(str2)):
        if item == '0':   # 为0时不用计算
            continue
        # 一定要补0
        single_res = single_str_multi(str1, item) + '0'*index
        # 每次相乘结果要相加
        res = str_sum(res, single_res)
    return res
```

## 全部代码

```python
def str_sum(str1, str2):
    """
    计算两个字符串的加法
    :param str1:
    :param str2:
    :return:
    """
    # 先补齐
    str_1_length = len(str1)
    str_2_length = len(str2)
    if str_1_length < str_2_length:
        str1 = "0"*(str_2_length - str_1_length) + str1
    else:
        str2 = "0"*(str_1_length - str_2_length) + str2

    # 进行计算
    index = len(str1) - 1
    pre_num = 0         # 记录进位
    res_lst = []        # 记录结果

    # 方向遍历
    while index >= 0:
        item_1 = int(str1[index])
        item_2 = int(str2[index])
        item_sum = item_1 + item_2 + pre_num
        pre_num = item_sum//10
        curr_num = item_sum % 10

        # 新的计算结果插入到结果的第一位
        res_lst.insert(0, str(curr_num))
        index -= 1

    if pre_num == 1:
        res_lst.insert(0, '1')

    return ''.join(res_lst)


def single_str_multi(str1, single):
    """
    single是一个小于10的数值, 例如 323354 * 4
    :param str1: 基础数值
    :param single: 单个数值
    :return:
    """
    pre_sum = 0            # 进位
    single = int(single)
    res_lst = []
    for item in reversed(str1):
        # 每次计算都要考虑上一次计算结果的进位
        item_num = int(item) * single + pre_sum
        pre_sum = item_num // 10     # 计算进位
        curr_num = item_num % 10    # 计算当前位置的值
        res_lst.insert(0, str(curr_num))

    if pre_sum > 0:
        res_lst.insert(0, str(pre_sum))

    return ''.join(res_lst)


def str_multi(str1, str2):
    """
    字符串相乘
    :param str1:
    :param str2:
    :return:
    """
    res = '0'
    for index, item in enumerate(reversed(str2)):
        if item == '0':   # 为0时不用计算
            continue
        # 一定要补0
        single_res = single_str_multi(str1, item)
        if single_res != '0':
            single_res += '0'*index
        # 每次相乘结果要相加
        res = str_sum(res, single_res)
    return res


if __name__ == '__main__':
    print(str_multi('323434223434234242343', '15492828424295835983529'))
```
