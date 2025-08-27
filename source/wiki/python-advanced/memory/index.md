---
layout: wiki  # 使用wiki布局模板
wiki: python-advanced # 这是项目id，对应 /_data/wiki/python-advanced.yml
title: 3.1 本章导读
order: 0
---

# python内存管理导读

python有自己的垃圾回收机制（GC）， 因此，你不必像C和C++程序员一样谨慎小心的使用内存，但这并不意味着你可以不关心内存。了解python对内存的管理，有助于我们优化程序性能。

了解了深拷贝与浅拷贝的区别，理解可变对象与不可变对象，才不会在写代码时留下难以发现的bug， 变量的引用是非常重要的概念，理解引用是理解python内存管理的关键。
