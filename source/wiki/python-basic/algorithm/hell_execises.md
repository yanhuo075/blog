---
layout: wiki  # 使用wiki布局模板
wiki: python-basic # 这是项目id，对应 /_data/wiki/python-basic.yml
title: 14.4 地狱练习题
order: 3
---

# 亲密字符串

## 题目要求

有两个字符串，A和B，如果A字符串内部交换任意两个位置的字符就能够让A和B相等，就认为两个字符串是亲密字符串。

```text
示例1
输入：A="cccb" ,B="cccb" 
输出：True
解释: 任意两个c交换位置，都可以让A=B,他们是亲密字符串

示例2
输入：A="ccbcab", B="cccbab"  
输出：True
解释：A内部交换A[2]和A[3]后，A=B，他们是亲密字符串

示例3
输入：A="ab", B="ab"  
输出：False
解释：A字符串内部不论怎样交换，都不可能与B相同
```

## 思路分析

如果A==B成立，那么就必须满足，A中有某个字符出现了两次或以上，只有这样，才能满足交换两个字符后与B相等，这是示例1的情况

如果A!=B，满足亲密字符串的条件就是A与B有两处不相同，而且交换这两处不相同的字符后A==B成立，这是示例2 的情况

如果A B两个字符串的长度不相等，必然不是亲密字符串。

遍历两个字符串，统计相同索引字符不相同的次数，如果次数是0，那么就必须满足A字符串有某个字符出现了两次或以上，可以用一个集合来保存A中的字符，如过有某个字符出现了两次或以上，那么最终，集合的大小一定小于A字符串的长度。

如果不相同的次数是2，那么交错对比这两处字符，如果不相同次数既不是0也不是2，则一定不是亲密字符串

## 示例代码

```python
# coding=utf-8


def is_close_str(str_1, str_2):
    # 长度要一致,且不能为0
    if len(str_1) != len(str_2) or len(str_1) < 2:
        return False

    index = 0
    diff_index_lst = []             # 记录不一致的索引位置
    char_set = set()
    while index < len(str_1):
        # 遇到相同位置字符不相同的情况
        if str_1[index] != str_2[index]:
            diff_index_lst.append(index)
            if len(diff_index_lst) > 2:
                return False
        char_set.add(str_1[index])
        index += 1

    if len(diff_index_lst) == 0:
        if len(str_1) != len(char_set):
            return True
    elif len(diff_index_lst) == 2:
        diff_index_1 = diff_index_lst[0]
        diff_index_2 = diff_index_lst[1]
        # 两处不一样的地方交错比较
        if str_1[diff_index_1] == str_2[diff_index_2] and \
                        str_1[diff_index_2] == str_2[diff_index_1]:
            return True

    return False

if __name__ == '__main__':
    print(is_close_str("", ""))
    print(is_close_str('abc', 'acba'))
    print(is_close_str('ab', 'ab'))
    print(is_close_str('cccb', 'cccb'))
    print(is_close_str('ccbcab', 'cccbab'))
    print(is_close_str('acbcad', 'dcbcaa'))
```



# 两个有序数组中找第k大的数

## 题目要求

已知有两个从小到大的有序数组，求两个数组的第k大的数。

```text
[1, 4, 6, 8, 12, 15, 18, 20, 28, 29]
[2, 5, 7, 10]
第8大的数是10
```

## 思路分析

两个数组都有序，那么就利用这个有序的特点来解决这个问题。假设数组分别是a b,令middle = k/2， middle_ex = k - middle。比较a[middle]和b[middle]的值。

- 如果a[middle - 1] == b[middle_ex - 1]，那么a[middle-1]不正好是第k大的数么，因为 k= middle_ex + middle,且两个数组都有序。
- 如果a[middle - 1] < b[middle_ex - 1]，让a = a[middle:],前面的那些元素都可以舍弃了， 问题转变成从a 和 b这两个数组里找到第 k - middle 大的值
- 如果a[middle - 1] > b[middle_ex - 1]，让b = b[middle_ex:]，前面那些元素都可以舍弃了，问题转变成从a 和 b这两个数组里找到第 k - middle_ex大的值

为什么在取值时用的索引是middle - 1呢，其实原因很简单，我们要找第k大的数，k最小是1，你不能说取第0大的数，我们日常是从1开始计数的，而计算机是从0开始计数的，middle = k/2， 从k计算而来，因此在使用索引时要减 1 。

有几个需要关注的地方

1. 要让数组长度更小的为a
2. 计算middle时，其实要考虑middle是否比a的长度小，不然取a[middle-1]时就出错了，计算middle和 middle_ex为的是从a， b 两个数组里各自找到第middle大和第middle_ex大的两个数，通过比较他们的大小，决定舍弃哪一部分
3. 不断的舍弃，不断的修改k的值，最后，一定会出现k==1的情况，这时，返回min(a[0], b[0])

## 示例代码

```python
# coding=utf-8


def find_kth(left_lst, left_len, right_lst, right_len, k):
    """
    从left_lst 和 right_lst中寻找第k大的数
    :param left_lst: 长度小的那个数组
    :param left_len:
    :param right_lst: 长度达的那个数组
    :param right_len:
    :param k:
    :return:
    """
    # 确保left_lst长度小于 right_lst 长度
    if left_len > right_len:
        return find_kth(right_lst, right_len, left_lst, left_len, k)

    # 长度小的数组已经没有值了,从right_lst找到第k大的数
    if left_len == 0:
        return right_lst[k-1]

    # 找到第1 大的数,比较两个列表的第一个元素,返回最小的那个
    if k == 1:
        return min(left_lst[0], right_lst[0])

    # k >> 1 ,其实就是k/2
    middle = min(k >> 1, left_len)
    middle_ex = k - middle
    # 舍弃left_lst的一部分
    if left_lst[middle-1] < right_lst[middle_ex-1]:
        return find_kth(left_lst[middle:], left_len-middle, right_lst, right_len, k-middle)
    # 舍弃right_lst 的一部分
    elif left_lst[middle-1] > right_lst[middle_ex-1]:
        return find_kth(left_lst, left_len, right_lst[middle_ex:], right_len-middle_ex, k-middle_ex)
    else:
        return left_lst[middle-1]


if __name__ == '__main__':
    left_lst = [1, 4, 6, 8, 12, 15, 18, 20, 28, 29]
    right_lst = [2, 5, 7, 10]
    k = 8
    print find_kth(left_lst, len(left_lst), right_lst, len(right_lst), k)

    # 合并后排序,找第k大的数
    lst = left_lst + right_lst
    lst.sort()
    print lst[k-1]
```



# 找到 K 个最接近的元素

## 题目要求

给定一个排序好的数组，两个整数 k 和 x，从数组中找到最靠近 x（两数之差最小）的 k 个数。返回的结果必须要是按升序排好的。如果有两个数与 x 的差值一样，优先选择数值较小的那个数

```text
输入: [1, 5, 8, 9, 17, 20], k=4, x=10
输出: [5, 8, 9, 17]
```

## 思路分析

### 第一个大于等于目标值的元素位置

题目要求找到K个最接近目标值的元素，我们先不考虑K个，而是考虑1个的情况，也就是最接近目标值的那个元素。

如果数组里有某个元素恰好和目标值相等，那么问题就转变成了二分查找法，在数组中查找目标值。如果数组里没有目标值呢，如何找到距离目标值最近的元素呢？

不妨先找到第一个大于等于目标值的元素，找到了这个元素，也就容易找到距离目标值最近的那个元素了，分析到这里，我们先要实现一个函数，该函数可以从数组中找到第一个大于等于目标值的元素位置，下面是这个函数的示例代码

```python
def find_eg_index(lst, start, end, target):
    """
    找到第一个大于等于target的元素的位置, 如果lst中最大的元素小于target
    则返回len(lst)
    :param lst:
    :param start:
    :param end:
    :param target:
    :return:
    """
    if start > end:
        return end + 1
    middle = (start + end)//2
    middle_value = lst[middle]
    if middle_value == target:
        return middle
    elif middle_value < target:
        return find_eg_index(lst, middle+1, end, target)
    else:
        return find_eg_index(lst, start, middle-1, target)
```

一定要记得考虑边界条件，如果目标值比数组最小值还小，那么函数返回0是应当的，但如果目标值比数组中最大值还大，该返回什么呢?为了概念的完整性，我这里返回len(lst), 这个索引是超出数组范围的，这表示无穷大，无穷大当然大于等于target

### 距离目标值最近的元素位置

前面的find_eg_index函数返回了第一个大于等于target的元素的位置eg_index，在此基础上，很容易就算出来距离target最近的元素的位置，所要考虑的逻辑分支如下

- eg_index == len(lst), target > max(lst), 数组最后一个元素最接近target
- eg_index == 0, 数组第一个元素最接近target
- target - lst[eg_index-1] <= lst[eg_index] - target, 说明lst[eg_index-1]更接近target, 反之lst[eg_index]更接近target

实现函数find_closest_index(lst, target) ，返回数组中距离target最近的元素的位置

```python
def find_closest_index(lst, target):
    """
    寻找距离target最近的位置
    :param lst:
    :param target:
    :return:
    """
    eg_index = find_eg_index(lst, 0, len(lst)-1, target)
    if eg_index == len(lst):
        return eg_index - 1
    elif eg_index == 0:
        return 0
    else:
        if target - lst[eg_index-1] <= lst[eg_index] - target:
            return eg_index-1
        else:
            return eg_index
```

### K个最接近的元素

在函数find_closest_index基础上继续思考，已经获得了closest_index， 只需要找到剩余的K-1个元素就可以了。

剩余的K-1个元素，一定分布在closest_index 的两侧，借鉴归并排序的思路，把closest_index 两侧的元素视为两个数组，分别从left_index(closest_index-1)和right_index(closest_index + 1)开始进行遍历比较，如果lst[left_index] <= lst[right_index]则lst[left_index] 就是要找的那K-1个元素里的一个，此时left_index -= 1，游标向左滑动，继续进行比较

这里要考虑边界情况，如果left_index 超出了数组的索引范围，该如何比较呢，一侧超出了索引范围，剩余的最近元素一定集中在另一侧，此时，可以直接从另一侧数组里直接取剩余数量的元素，但我认为这样程序就会写的很复杂，要判断两侧的索引是否超出范围，要写很多if语句。

不弱这样处理，如果索引超出了范围，就认为这个索引上的元素是无穷大，这样就不需要判断左右两个索引是否超出范围以及取另一侧的剩余最近元素了。

示例代码

```python
def find_closet_ele(lst, target, k):
    closet_ele_lst = []
    closest_index = find_closest_index(lst, target)   # 距离target最近的元素的位置
    closet_ele_lst.append(lst[closest_index])

    ele_count = 1
    left_index = closest_index - 1
    right_index = closest_index + 1
    # 借鉴归并排序思路,向两侧滑动遍历
    while ele_count < k:
        left_ele = get_ele(lst, left_index)
        right_ele = get_ele(lst, right_index)
        # 哪边元素距离target更近,哪边就走一步
        if target - left_ele <= right_ele - target:
            closet_ele_lst.append(left_ele)
            left_index -= 1
        else:
            closet_ele_lst.append(right_ele)
            right_index += 1
        ele_count += 1

    closet_ele_lst.sort()
    return closet_ele_lst


def get_ele(lst, index):
    # 索引超出范围的,返回无穷大
    if index < 0 or index >= len(lst):
        return float("inf")
    return lst[index]
```

## 完整代码

```python
def find_closet_ele(lst, target, k):
    closet_ele_lst = []
    closest_index = find_closest_index(lst, target)   # 距离target最近的元素的位置
    closet_ele_lst.append(lst[closest_index])

    ele_count = 1
    left_index = closest_index - 1
    right_index = closest_index + 1
    # 借鉴归并排序思路,向两侧滑动遍历
    while ele_count < k:
        left_ele = get_ele(lst, left_index)
        right_ele = get_ele(lst, right_index)
        # 哪边元素距离target更近,哪边就走一步,这里注意取绝对值
        if abs(target - left_ele) <= abs(right_ele - target):
            closet_ele_lst.append(left_ele)
            left_index -= 1
        else:
            closet_ele_lst.append(right_ele)
            right_index += 1
        ele_count += 1

    closet_ele_lst.sort()
    return closet_ele_lst


def get_ele(lst, index):
    # 索引超出范围的,返回无穷大
    if index < 0 or index >= len(lst):
        return float("inf")
    return lst[index]


def find_closest_index(lst, target):
    """
    寻找距离target最近的位置
    :param lst:
    :param target:
    :return:
    """
    eg_index = find_eg_index(lst, 0, len(lst)-1, target)
    if eg_index == len(lst):
        return eg_index - 1
    elif eg_index == 0:
        return 0
    else:
        if target - lst[eg_index-1] <= lst[eg_index] - target:
            return eg_index-1
        else:
            return eg_index


def find_eg_index(lst, start, end, target):
    """
    找到第一个大于等于target的元素的位置, 如果lst中最大的元素小于target
    则返回len(lst)
    :param lst:
    :param start:
    :param end:
    :param target:
    :return:
    """
    if start > end:
        return end + 1
    middle = (start + end)//2
    middle_value = lst[middle]
    if middle_value == target:
        return middle
    elif middle_value < target:
        return find_eg_index(lst, middle+1, end, target)
    else:
        return find_eg_index(lst, start, middle-1, target)


if __name__ == '__main__':
    lst = [1, 5, 8, 9, 17, 20]
    print(find_closet_ele(lst, 10, 4))
```



# 最长公共前缀

## 题目要求

编写一个函数来查找字符串数组中的最长公共前缀，如果不存在公共前缀，返回空字符串 ""。
示例1：

```text
输入: ["flower","flow","flight"]
输出: "fl"
```

示例2：

```text
输入: ["dog","racecar","car"]
输出: ""
```

## 思路分析

以列表中的第一个单词作为基线，其余的单词逐个字符与第一个单词的字符进行比较，比如以flower做为基线，flower第一个字符是f，那么其余的单词的第一个字符也是f，就将这个f记录下来，逐一比较，直到某个位置上的字符出现不一致的情况。

在比较过程中，要注意单词的长度，如果某个单词已经比较到末尾，那么就可以停止了，因为这个单词的长度就是理论上可能的最大公共前缀

## 示例代码

```python
# coding=utf-8


def get_max_prefix(lst):
    if len(lst) == 0:
        return ""
    base_word = lst[0]
    prefix_lst = []
    for index, item in enumerate(base_word):
        b_common = False     # 标识在index位置上,所有单词的字符是否相同
        for word in lst:
            # index已经超过了单词的长度
            if index >= len(word):
                break
                
            # 在index这个位置上,字符不相同
            if word[index] != item:
                break
        else:
            # 如果for循环没有被break中断,就会进入到这个语句块
            b_common = True

        if not b_common:
            break
        prefix_lst.append(item)

    return "".join(prefix_lst)


if __name__ == '__main__':
    lst = ["flower", "flow", "flight"]
    print(get_max_prefix(lst))
```



# 最大正方形

## 题目要求

在一个由 0 和 1 组成的二维矩阵内，找到只包含 1 的最大正方形，并返回其面积

```text
输入: 

1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0

输出: 4
```

## 思路分析

只包含1的正方形可以出现在矩阵的任意位置，所以，不要妄想通过什么巧妙的办法快速的找到这个正方形的位置，正确的做法是遍历整个矩阵，尝试去找正方形，可能会找到多个，因此还要保留记录那个最大的正方形。

既然是遍历，那么就得从头到尾的进行遍历，假设矩阵matrix大小是M*N，那么就从matrix[0][0]开始进行遍历，一直遍历到matrix[M-1][N-1]。

当遍历到matrix[x][y]时，可以将这个点视为正方形的左上角，以此为基础，去判断matrix[x+1][y]， matrix[x][y+1]， matrix[x+1][y+1]这3个点是否也是1，如果成立，就找到了一个正方形，在此基础上，再向右下方扩展一层，判断能否以matrix[x][y]做正方形的左上角，以matrix[x+2][y+2]做正方形的右下角。

由于总是以某个点做正方形的左上角，因此，最后一行和最后一列是不需要遍历的，因为这些点无法构成正方形的左上角。

## 示例代码

```python
matrix = [
    [1, 0, 1, 0, 0],
    [1, 0, 1, 1, 1],
    [1, 1, 1, 1, 1],
    [1, 0, 1, 1, 1],
    [1, 0, 1, 1, 1],
]

matrix = [['1']]

def max_square(matrix):
    y = len(matrix)
    if y == 0:
        return 0

    x = len(matrix[0])
    if x == 0:
        return 0

    max_area = 0
    for i in range(0, y):
        for j in range(0, x):
            cell = matrix[i][j]
            if cell == '0':
                continue

            area = get_area(matrix, i, j)
            if area > max_area:
                max_area = area

    return max_area


def get_area(matrix, left_up_x, left_up_y):
    step = 1
    right_down_x = left_up_x + step
    right_down_y = left_up_y + step

    while right_down_x < len(matrix) and right_down_y < len(matrix[0]):
        b_square = True
        if matrix[right_down_x][right_down_y] == '0':
            break

        for i in range(left_up_x, right_down_x):
            if matrix[i][right_down_y] == '0':
                b_square = False
                break

        if not b_square:
            break

        for j in range(left_up_y, right_down_y):
            if matrix[right_down_x][j] == '0':
                b_square = False
                break

        if not b_square:
            break

        step += 1
        right_down_x = left_up_x + step
        right_down_y = left_up_y + step


    return step**2


if __name__ == '__main__':
    print(max_square(matrix))
```



# 还原ip地址

## 题目要求

一个字符串，里面存储的是ip地址，但是.（点）被删除了，只剩下数字，请编写算法，还原这个ip地址，如果可以还原成多个，请逐一列出

```text
输入: 192168119130
输出: ['192.168.119.130']

输入: 101224010
输出:['10.12.240.10', '10.122.40.10', '10.122.401.0', 
      '101.2.240.10', '101.22.40.10', '101.22.401.0', 
      '101.224.0.10']
      
输入: 101226010
输出: ['10.122.60.10', '101.22.60.10', '101.226.0.10']
```

## 思路分析

一个ip地址有四段，每段的范围是0~255，可以分四次从输入字符串中截取字符串，第一次截取时，必然是从字符串索引为0的位置开始，那么可以截取的长度是1， 2， 3。 这一段截取后，进行第二次截取，第二次截取的开始位置取决于第一次截取结束的位置，如果第一次截取的长度是2，那么第二次就要从索引为2的位置上开始截取，可以截取的长度是1， 2， 3， 第三次和第四次做同样的操作，这么一分析，你就应该想到用递归函数来做了。

需要考虑的情况是：

- 截取后的每一段，如果是以0开头，那么这一段只能是0，总不能出现10.21.01.23的ip地址吧，01是什么鬼
- 截取后的每一段，其值大小不能超过255
- 如果截取后剩余的部分长度超过剩余几段理论上的最大长度,那么这个截取就是不合理的，比如192168119130， 第一段只截取1，那么剩余部分92168119130 的长度是11，后面三段最长也就是9，所以这样截取是不合理的
- 如果截取后余下的部分长度是0，也是不合理的
- 如果截取后余下的部分长度比余下几段理论上的最小长度还小，也是不合理的，比如截取后余下部分长度是2，可是还有3段需要截取，这样就是不合理的

## 示例代码

```python
# coding=utf-8


def restore_ip(ip):
    retsection_lst = restore_ip_ex(ip, len(ip), 0, 4)
    return ['.'.join(item) for item in retsection_lst]


def restore_ip_ex(ip, ip_len, start_index, k):
    if k == 0:
        return [[]]

    if ip_len > 20 or ip_len < 4:
        return []

    section_lst = []
    for i in range(1, 4):
        # 如果截取后剩余的部分长度超过剩余几段理论上的最大长度,那么这个截取就是不合理的
        # 如果剩余长度为0也是不合理的
        rest_len = ip_len - i - start_index
        if (rest_len == 0 and k != 1) or rest_len > (k-1)*3 or rest_len < k-1:
            continue

        section = ip[start_index:start_index+i]
        # 如果某段以0开头,那么这一段就是只能是0,否则就会出现1.012.240.10这种情况,012显然是不合理的
        if section[0] == '0' and len(section) != 1:
            continue

        if int(section) > 255:
            continue

        # 截取下一段
        res_lst = restore_ip_ex(ip, ip_len, i+start_index, k-1)
        for item in res_lst:
            item.insert(0, section)

        section_lst.extend(res_lst)

    return section_lst


if __name__ == '__main__':
    print restore_ip("192168119130")
    print restore_ip("101224010")
    print restore_ip("1111")
    print restore_ip("101226010")
```



# 接雨水

## 题目要求

给定 n 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水

![img](https://fastly.jsdelivr.net/gh/yanhuo075/images-repo/upload/20250824001209783.jpeg)

上面是由数组 [0,1,0,2,1,0,1,3,2,1,2,1] 表示的高度图，在这种情况下，可以接 6 个单位的雨水（蓝色部分表示雨水）

## 思路分析

### 思考才是最重要的

这个题目非常好的诠释了一句话----“技术，只是实现目标的方式和手段”。

很多人面对问题时，总是有一种较劲脑汁写算法的冲动，还有人，以为每一种问题都有一个成熟的解决方案，似乎接触过这个方案并记忆下来，今后再遇到时就可以迎刃而解了。殊不知，编程是一个创造性的思考过程，很多人不愿意思考，更喜欢走马观花般的背答案。短时间内的确可以提升算法能力，但长期来看，你的逻辑能力才是解决问题的根本，思考才是第一位的。

### 两个基本判断

我们先不去想怎么写算法，而是先分析，抽丝剥茧的对这个问题进行分析。

首先，最左侧的柱子和最右侧的柱子上面不可能存储雨水，这个判断很简单，不做讨论。

其次，每个柱子上能存储多少水，取决于它左右两侧最高柱子的高度，以索引为5的柱子为例，这个柱子的高度是0，它左侧最高的柱子是索引为3的柱子，高度为2， 它右侧最高的柱子是索引为7的柱子，高度为3，这两个柱子的高度决定了，索引为5的柱子上面可以存储水2个单位。

### 算法思路

经过上面的两点分析，解题的思路就非常清晰了。

遍历数组，计算每个柱子左右两侧的最高柱子的高度，这两个高度中选最小的，如果这个最小高度大于当前柱子的高度，那么差值就是当前这个柱子能够存储的水的量。

由于遍历的过程是从左向右，因此，左侧最高柱子的高度可以用一个变量保存起来，而右侧最高柱子的高度则需要每次都去计算,但是经过一次寻找后，这个最高柱子的位置就确定了，假设位置为K，对于这个柱子左侧的所有柱子来说，他们右侧的最高柱子都是固定的，因此，也可以存储起来，直到遍历过程中当前柱子的索引大于等于K，则需要重新寻找。

## 实例代码

```python
def trap(lst):
    left_height = 0
    right_index = 0
    rain = 0
    for index, item in enumerate(lst):
        # 第一个和最后一个不用考虑，因为柱子上方无法存储水
        if index == len(lst) - 1 or index == 0:
            left_height = item
            continue

        # 如果左侧最高为0或者和当前高度相同，当前这个柱子的上方无法存储水
        if left_height == 0 or left_height == item:
            left_height = item
            continue

        # 获取右侧最高的柱子高度， 取左右两侧较矮的，如果当前柱子高度小于它，则可以存储水
        if index >= right_index:
            right_index, right_heigth = find_right_heigth(index, lst)

        min_heigth = min([left_height, right_heigth])
        if min_heigth > item:
            rain += min_heigth - item

        # 更新左侧最高柱子的高度
        if item > left_height:
            left_height = item

    return rain


def find_right_heigth(index, lst):
    """
    返回index右侧最高柱子的位置和高度
    :param index:
    :param lst:
    :return:
    """
    right_heigth = 0
    right_index = index
    for i in range(index, len(lst)):
        if lst[i] > right_heigth:
            right_index = i
            right_heigth = lst[i]

    return right_index, right_heigth


if __name__ == '__main__':
    lst = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]

    print(trap(lst))
```



# 逆波兰表达式求值

## 题目要求

根据逆波兰表示法，求表达式的值。
比如 ["2", "1", "+", "3", "*"]，对应到小学所学的四则运算表达式就是((2 + 1) * 3) 计算结果为9

["4", "13", "5", "/", "+"] 所对应的四则运算表达式是 (4 + (13 / 5)) = 6

现在请计算["10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"] 的结果

## 思路分析

题目所要求计算的逆波兰表达式对应的四则运算表达式是((10 * (6 / ((9 + 3) * -11))) + 17) + 5

逆波兰表达式的计算，其实是栈这种数据结构的经典应用，将逆波兰表达式中的元素逐一放入一个栈中，当遇到运算符时，则从栈顶连续弹出两个元素进行计算，然后把计算结果放入到栈中，最后，栈里只有保存一个元素，这个元素就是整个表达式的计算结果。

因此这道题目，你需要实现一个简单的栈，好在有列表这种神器，栈的实现非常容易。

有一个比较坑的地方是python的除法，python中，采用的是向下取整的办法，比如

- 5/3 = 1.666 向下取整得到1
- 5/-3 = -1.6666 向下取整得到-2

其他语言中，5/-3会向上取整，就会得到-1， 这个是符合我们日常判断的，但python里却偏偏向下取整得到了-2，为了使结果符合人们的常识，需要用到operator 模块的truediv 方法。

## 示例代码

```python
# coding=utf-8
import operator

class Stack(object):
    def __init__(self):
        self.items = []
        self.count = 0

    def push(self, item):
        """
        放入一个新的元素
        :param item:
        :return:
        """
        self.items.append(item)
        self.count += 1

    def top(self):
        # 获得栈顶的元素
        return self.items[self.count-1]

    def size(self):
        # 返回栈的大小
        return self.count

    def pop(self):
        # 从栈顶移除一个元素
        item = self.top()
        del self.items[self.count-1]
        self.count -= 1
        return item

def cal_exp(expression):
    stack = Stack()
    for item in expression:
        if item in "+-*/":
            # 遇到运算符就从栈里弹出两个元素进行计算
            value_1 = stack.pop()
            value_2 = stack.pop()
            if item == '/':
                res = int(operator.truediv(int(value_2), int(value_1)))
            else:
                res = eval(value_2 + item + value_1)
            # 计算结果最后放回栈,参与下面的计算
            stack.push(str(res))
        else:
            stack.push(item)

    res = stack.pop()
    return res

if __name__ == '__main__':
    print(cal_exp(["10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"]))
```



# 简单的计算器

## 题目要求

实现一个简单的计算器，能够计算简单字符串表达式，表达式可以包括小括号，运算符包含+ - * /, 表达式允许有空格,参与运算的数值都是正整数。
示例1

```text
输入: "1 + 2"
输出: 3
```

示例2

```text
输入:" 2 - 3 + 2 "
输出: 1
```

示例3

```text
输入:"(1+(4+5+3)-3)+(9+8)"
输出: 27
```

示例4

```text
输入:"(1+(4+5+3)/4-3)+(6+8)*3"
输出: 43
```

## 思路分析

### 1 表达式预处理

万事开头难，先不考虑如何计算，我们应该先对表达式进行处理，处理以后，只有数值和运算符，这样才能对他们进行具体的操作，比如"1 + 2" 处理后得到['1', '+', '2']， 运算符和数值都分离开了。

这样的解析并不复杂，只需要遍历字符串，解析出数值部分放入到列表中，遇到小括号或者运算符则直接放入列表中，代码如下：

```python
def exp_to_lst(exp):
    lst = []
    start_index = 0    # 数值部分开始位置
    end_index = 0      # 数值部分结束位置
    b_start = False
    for index, item in enumerate(exp):
        # 是数字
        if item.isdigit():
            if not b_start:  # 如果数值部分还没有开始
                start_index = index    # 记录数值部分开始位置
                b_start = True         # 标识数值部分已经开始
        else:
            if b_start:  # 如果数值部分已经开始
                end_index = index     # 标识数值部分结束位置
                b_start = False       # 标识数值部分已经结束
                lst.append(exp[start_index:end_index])   # 提取数值放入列表

            if item in ('+', '-', '*', '/', '(', ')'):    # 运算符直接放入列表
                lst.append(item)

    if b_start:    # 数值部分开始了,但是没有结束,说明字符串最后一位是数字,
        lst.append(exp[start_index:])
    return lst
    
def test_exp_to_lst():
    print exp_to_lst("1 + 2")
    print exp_to_lst(" 2 - 3 + 2 ")
    print exp_to_lst("(1+(4+5+3)-3)+(9+8)")
    print exp_to_lst("(1+(4+5+3)/4-3)+(6+8)*3")

test_exp_to_lst()
```

程序输出结果为

```python
['1', '+', '2']
['2', '-', '3', '+', '2']
['(', '1', '+', '(', '4', '+', '5', '+', '3', ')', '-', '3', ')', '+', '(', '9', '+', '8', ')']
['(', '1', '+', '(', '4', '+', '5', '+', '3', ')', '/', '4', '-', '3', ')', '+', '(', '6', '+', '8', ')', '*', '3']
```

## 2 中序转后序

1+2 这种表达式是中序表达式，运算符在运算对象的中间，还有一种表达式，运算符在运算对象的后面，称之为后序表达式，也叫逆波兰表达式。1+2 转成后序表达式后是 1 2 +， +号作用于它前面的两个运算对象。

后序表达式相比于中序表达式更容易计算，因此，这一小节，我要把中序表达式转换成后序表达式。

转换时要借助栈这个数据结构，变量postfix_lst 表示后序表达式，遍历中序表达式，根据当前值进行处理：

- 如果遇到数值，则放入到postfix_lst中
- 遇到（ 压如栈中
- 遇到 ），将栈顶元素弹出并放入到postfix_lst中，直到遇到（，最后弹出（
- 遇到运算符，把栈顶元素弹出并放入到postfix_lst中，直到栈顶操作符的运算优先级小于当前运算符，最后将当前运算符压入栈

代码如下：

```python
# 运算优先级
priority_map = {
    '+': 1,
    '-': 1,
    '*': 2,
    '/': 2
}

class Stack(object):
    def __init__(self):
        self.items = []
        self.count = 0

    def push(self, item):
        """
        放入一个新的元素
        :param item:
        :return:
        """
        self.items.append(item)
        self.count += 1

    def top(self):
        # 获得栈顶的元素
        return self.items[self.count-1]

    def size(self):
        # 返回栈的大小
        return self.count

    def pop(self):
        # 从栈顶移除一个元素
        item = self.top()
        del self.items[self.count-1]
        self.count -= 1
        return item
        
def infix_exp_2_postfix_exp(exp):
    """
    中序表达式转后序表达式
    """
    stack = Stack()
    exp_lst = exp_to_lst(exp)
    postfix_lst = []
    for item in exp_lst:
        # 如果是数值,直接放入到postfix_lst 中
        if item.isdigit():
            postfix_lst.append(item)
        elif item == '(':
            # 左括号要压栈
            stack.push(item)
        elif item == ')':
            # 遇到右括号了,整个括号里的运算符都输出到postfix_lst
            while stack.top() != '(':
                postfix_lst.append(stack.pop())
            # 弹出左括号
            stack.pop()
        else:
            # 遇到运算符,把栈顶输出,直到栈顶的运算符优先级小于当前运算符
            while stack.size() != 0 and stack.top() in ("+-*/")\
                    and priority_map[stack.top()] >= priority_map[item]:
                postfix_lst.append(stack.pop())
            # 当前运算符优先级更高,压栈
            stack.push(item)

    # 最后栈里可能还会有运算符
    while stack.size() != 0:
        postfix_lst.append(stack.pop())

    return postfix_lst
    
print infix_exp_2_postfix_exp("1 + 2")
print infix_exp_2_postfix_exp(" 2 - 3 + 2 ")
print infix_exp_2_postfix_exp("(1+(4+5+3)-3)+(9+8)")
print infix_exp_2_postfix_exp("(1+(4+5+3)/4-3)+(6+8)*3")
```

程序输出结果为

```python
['1', '2', '+']
['2', '3', '-', '2', '+']
['1', '4', '5', '+', '3', '+', '+', '3', '-', '9', '8', '+', '+']
['1', '4', '5', '+', '3', '+', '4', '/', '+', '3', '-', '6', '8', '+', '3', '*', '+']
```

## 3 后序表达式计算

后续表达式计算同样需要用到栈，这个算法在逆波兰表达式计算的练习题中已经有讲解，直接复用代码

```python
def cal_exp(expression):
    stack = Stack()
    for item in expression:
        if item in "+-*/":
            # 遇到运算符就从栈里弹出两个元素进行计算
            value_1 = stack.pop()
            value_2 = stack.pop()
            if item == '/':
                res = int(operator.truediv(int(value_2), int(value_1)))
            else:
                res = eval(value_2 + item + value_1)
            # 计算结果最后放回栈,参与下面的计算
            stack.push(str(res))
        else:
            stack.push(item)

    res = stack.pop()
    return res
```

## 全部代码

```python
# coding=utf-8
import operator

# 运算优先级
priority_map = {
    '+': 1,
    '-': 1,
    '*': 2,
    '/': 2
}

class Stack(object):
    def __init__(self):
        self.items = []
        self.count = 0

    def push(self, item):
        """
        放入一个新的元素
        :param item:
        :return:
        """
        self.items.append(item)
        self.count += 1

    def top(self):
        # 获得栈顶的元素
        return self.items[self.count-1]

    def size(self):
        # 返回栈的大小
        return self.count

    def pop(self):
        # 从栈顶移除一个元素
        item = self.top()
        del self.items[self.count-1]
        self.count -= 1
        return item


# 表达式中序转后序
def infix_exp_2_postfix_exp(exp):
    """
    中序表达式转后序表达式
    """
    stack = Stack()
    exp_lst = exp_to_lst(exp)
    postfix_lst = []
    for item in exp_lst:
        # 如果是数值,直接放入到postfix_lst 中
        if item.isdigit():
            postfix_lst.append(item)
        elif item == '(':
            # 左括号要压栈
            stack.push(item)
        elif item == ')':
            # 遇到右括号了,整个括号里的运算符都输出到postfix_lst
            while stack.top() != '(':
                postfix_lst.append(stack.pop())
            # 弹出左括号
            stack.pop()
        else:
            # 遇到运算符,把栈顶输出,直到栈顶的运算符优先级小于当前运算符
            while stack.size() != 0 and stack.top() in ("+-*/")\
                    and priority_map[stack.top()] >= priority_map[item]:
                postfix_lst.append(stack.pop())
            # 当前运算符优先级更高,压栈
            stack.push(item)

    # 最后栈里可能还会有运算符
    while stack.size() != 0:
        postfix_lst.append(stack.pop())

    return postfix_lst




def exp_to_lst(exp):
    """
    表达式预处理，转成列表形式
    """
    lst = []
    start_index = 0    # 数值部分开始位置
    end_index = 0      # 数值部分结束位置
    b_start = False
    for index, item in enumerate(exp):
        # 是数字
        if item.isdigit():
            if not b_start:  # 如果数值部分还没有开始
                start_index = index    # 记录数值部分开始位置
                b_start = True         # 标识数值部分已经开始
        else:
            if b_start:  # 如果数值部分已经开始
                end_index = index     # 标识数值部分结束位置
                b_start = False       # 标识数值部分已经结束
                lst.append(exp[start_index:end_index])   # 提取数值放入列表

            if item in ('+', '-', '*', '/', '(', ')'):    # 运算符直接放入列表
                lst.append(item)

    if b_start:    # 数值部分开始了,但是没有结束,说明字符串最后一位是数字,
        lst.append(exp[start_index:])
    return lst


def cal_exp(expression):
    """
    计算后续表达式
    """
    stack = Stack()
    for item in expression:
        if item in "+-*/":
            # 遇到运算符就从栈里弹出两个元素进行计算
            value_1 = stack.pop()
            value_2 = stack.pop()
            if item == '/':
                res = int(operator.truediv(int(value_2), int(value_1)))
            else:
                res = eval(value_2 + item + value_1)
            # 计算结果最后放回栈,参与下面的计算
            stack.push(str(res))
        else:
            stack.push(item)

    res = stack.pop()
    return res


def test_exp_to_lst():
    print exp_to_lst("1 + 2")
    print exp_to_lst(" 2 - 3 + 2 ")
    print exp_to_lst("(1+(4+5+3)-3)+(9+8)")
    print exp_to_lst("(1+(4+5+3)/4-3)+(6+8)*3")


def test_infix_exp_2_postfix_exp():
    print cal_exp(infix_exp_2_postfix_exp("1 + 2"))
    print cal_exp(infix_exp_2_postfix_exp(" 2 - 3 + 2 "))
    print cal_exp(infix_exp_2_postfix_exp("(1+(4+5+3)-3)+(9+8)"))
    print cal_exp(infix_exp_2_postfix_exp("(1+(4+5+3)/4-3)+(6+8)*3"))


if __name__ == '__main__':
    test_infix_exp_2_postfix_exp()
```



# 判断子序列

## 题目要求

给定字符串 s 和 t ，判断 s 是否为 t 的子序列

你可以认为 s 和 t 中仅包含英文小写字母。字符串 t 可能会很长（长度 ~= 500,000），而 s 是个短字符串（长度 <=100）

字符串的一个子序列是原始字符串删除一些（也可以不删除）字符而不改变剩余字符相对位置形成的新字符串。（例如，"ace"是"abcde"的一个子序列，而"aec"不是）

示例 1:
s = "abc", t = "ahbgdc"
返回 true.

示例 2:
s = "axc", t = "ahbgdc"
返回 false

## 思路分析

一种思路是从t中寻找长度和s相同的子序列，然后判断这些子序列中有没有和s相等的子序列。这虽然是一个可行的方案，但显然效率很差。

另一个思路是先对t进行分析，遍历t获得每一个字符的出现位置，最终将得到一个字符分布的信息，例如“ahbgdc”, 它的字符分布情况是

```python
{
'a': [0], 
'b': [2], 
'c': [5], 
'd': [4], 
'g': [3], 
'h': [1]
}
```

得到这些字符分布情况后，就可以根据每个字符的所在位置来判断字符串s是否是t的子序列，假设a出现的位置都大于b出现的位置，那么字符串s一定不是字符串t的子序列，每个字符可以出现在很多个位置上，我们只要找到一个位置满足s是t的子序列就可以了。

以"abc" 为例，先找到a出现的位置，不论有多少个，只取第一个，记为pre_index，因为a的位置越靠前，留给bc的空间就越大，那么接下来从出现的位置中寻找第一个比pre_index大的位置并赋值给pre_index，为什么要找第一个，因为b的位置越靠前，留给c的空间就越大，最后从c的位置找到第一个比pre_index大的位置

## 示例代码

```python
def is_sub_seq(s, t):
    char_info = get_char_info(t)
    pre_index = -1
    for item in s:
        # 如果t中不包含item这个字符
        index_lst = char_info.get(item)
        if index_lst is None:
            return False
        for index in index_lst:
            # pre_index是上一个字符可以出现的位置
            # 当前字符出现的位置中,找第一个比pre_index大的位置
            if index > pre_index:
                pre_index = index
                break
        else:
            # 如果循环正常退出
            return False

    return True


def get_char_info(t):
    info = {}
    for index, item in enumerate(t):
        info.setdefault(item, [])
        info[item].append(index)
    return info


if __name__ == '__main__':
    print(is_sub_seq("abc", "ahbgdc"))
```
