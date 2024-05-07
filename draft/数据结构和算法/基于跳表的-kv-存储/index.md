# [未完成] 基于跳表的 KV 存储


## 1. 项目任务

- [ ] 学习跳表结构
- [ ] 使用跳表实现 KV 存储

> 实现接口：
>
> - insert_element 插入数据
> - delete_element 删除数据
> - search_element 查询数据
> - update_element 更新数据
> - display_list 打印跳跃表
> - dump_file 数据落盘
> - load_file 加载数据
> - size 返回数据规模
> - clear 清空跳表

**关键词：** 跳跃表、C++ 模板

## 2. 跳表

>跳表 ( SkipList，全称跳跃表) 是用于有序元素序列快速搜索查找的一个数据结构，跳表是一个随机化的数据结构，实质就是一种可以进行二分查找的有序链表。能提高搜索性能，同时也可以提高插入和删除操作的性能。

![skiplist](/SkipList/skiplist.png)

### **构造步骤**

1. 给定一个有序的链表。
2. 选择连表中最大和最小的元素，然后从其他元素中按照一定算法（随机）随即选出一些元素，将这些元素组成有序链表。这个新的链表称为一层，原链表称为其下一层。
3. 为刚选出的每个元素添加一个指针域，这个指针指向下一层中值同自己相等的元素。Top 指针指向该层首元素。
4. 重复 2、3 步，直到不再能选择出除最大最小元素以外的元素。

## 参考链接

1. [跳表实现 KV 存储](https://czgitaccount.github.io/Database/Skiplist-CPP/)
2. [Skip List（跳跃表）原理详解与实现](http://www.cppblog.com/mysileng/archive/2013/04/06/199159.html)
3. [跳表 (SkipList) 设计与实现 (Java)](https://www.cnblogs.com/bigsai/p/14193225.html)

