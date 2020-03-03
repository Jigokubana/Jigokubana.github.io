---
title: STL与实现记录
date: 2020-03-01 10:26:55
tags:
categories:
  - CPP
  - CPP重点
img: https://lsmg-img.oss-cn-beijing.aliyuncs.com/STL%E8%AE%B0%E5%BD%95/%E5%B0%81%E9%9D%A2.jpg
---

# Map

## lower_bound()和 upper_bound()
**lower_bound(const Key& key)**
https://zh.cppreference.com/w/cpp/container/map/lower_bound
返回指向首个不小于 key 的元素的迭代器。若找不到这种元素，则返回尾后迭代器



**upper_bound()**
返回指向首个大于 key 的元素的迭代器。

# std::vertor
| 函数名称 | 使用详解 |
| --- | --- |
| at [] | 获取指定位置的元素 有边界检查 |
| insert | 在指定位置的前面插入, 可以插入单项, vector和数组 |
| max_size | 理论最大容量 |
| capacity | 实际最大容量 |
