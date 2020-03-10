---
title: STL与实现记录
date: 2020-03-01 10:26:55
tags:
categories:
  - CPP
  - CPP重点
img: https://lsmg-img.oss-cn-beijing.aliyuncs.com/STL%E8%AE%B0%E5%BD%95/%E5%B0%81%E9%9D%A2.jpg
---

# 容器库
| 顺序容器 | 按序访问 |
| --- | --- |
| array | 静态的连续数组 |
| vector | 动态的连续数组 |
| deque | 双端队列 |
| forward_list | 单链表 |
| list | 双链表 |


| 关联容器 | 能实现快速查找O(logn) |
| --- | --- |
| set | 唯一key的集合, 按照key排序 |
| map | k v集合, 按照k排序, k是唯一的 |
| multiset | key的集合 按照key排序 |
| multimap | k v的集合, 按照key排序 |

| 无序关联容器 | 快速查找均摊O(1)最坏O(n)的无序(哈希)数据结构 |
| --- | --- |
| unordered_set | 唯一key的集合, 按照key生成散列 |
| unordered_map | 唯一key, k v的集合, 按照key生成散列 |
| unordered_multiset | key的集合, 按照key生成散列 |
| unordered_multimap | k v的集合, 按照key生成散列 |

| 容器适配器 | 提供顺序容器的不同接口 |
| --- | --- |
| stack | 适配一个容器提供栈 |
| queue | 适配一个容器提供队列 |
| priority_queue | 适配一个容器提供优先级队列 |

# map
## lower_bound()和 upper_bound()
**lower_bound(const Key& key)**
https://zh.cppreference.com/w/cpp/container/map/lower_bound
返回指向首个不小于 key 的元素的迭代器。若找不到这种元素，则返回尾后迭代器



**upper_bound()**
返回指向首个大于 key 的元素的迭代器。

# vertor
| 函数名称 | 使用详解 |
| --- | --- |
| at [] | 获取指定位置的元素 有边界检查 |
| insert | 在指定位置的前面插入, 可以插入单项, vector和数组 |
| max_size | 理论最大容量 |
| capacity | 实际最大容量 |


# 图片Warn

![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/STL%E8%AE%B0%E5%BD%95/%E9%A1%BA%E5%BA%8F%E5%AE%B9%E5%99%A8.png)


![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/STL%E8%AE%B0%E5%BD%95/%E5%85%B3%E8%81%94%E5%AE%B9%E5%99%A8.png)


![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/STL%E8%AE%B0%E5%BD%95/%E6%97%A0%E5%BA%8F%E5%85%B3%E8%81%94%E5%AE%B9%E5%99%A8.png)


![](https://lsmg-img.oss-cn-beijing.aliyuncs.com/STL%E8%AE%B0%E5%BD%95/%E5%AE%B9%E5%99%A8%E9%80%82%E9%85%8D%E5%99%A8.png)