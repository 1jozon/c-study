# STL源码解析侯捷视频课

## allocator分配器

- ::oprator new和:: oprator delete完成allocator()和deallocator()

  new最终使用malloc delete使用free

![image-20250503170743755](./images/STL标准库与泛型编程/image-20250503170743755.png)

- 不同实现版本回不一样，G2.9实现方法不一样alloc（p69），G4.9又不是alloc默认了。

![image-20250503183433287](./images/STL标准库与泛型编程/image-20250503183433287.png)

## 不同容器之间的实现关系与分类

![image-20250503183658306](./images/STL标准库与泛型编程/image-20250503183658306.png)

缩排表示包含关系，如heap有一个vector

## 深度探索list（p128）

![image-20250504161813739](./images/STL标准库与泛型编程/image-20250504161813739.png)

### 所有容器的iterator都是class，要模拟指针

![image-20250504163939481](./images/STL标准库与泛型编程/image-20250504163939481.png)

###  iterator++函数![image-20250504164733708](./images/STL标准库与泛型编程/image-20250504164733708.png)

- self tmp = * this 先调用拷贝赋值，再用  *this的值赋值

- c++不允许后++两次，后置++返回的是self 而不是self&

 ![image-20250504165739255](./images/STL标准库与泛型编程/image-20250504165739255.png)

 G2.9 相对于 G4.9，优化了list_node指针

##  迭代器的设计原则

### 迭代器必须提供五种相关类型的type

![image-20250504171518526](./images/STL标准库与泛型编程/image-20250504171518526.png)

### 如果iterator仅仅只是c++普通指针，可将其看为退化的iterator，算法

![image-20250504172139239](./images/STL标准库与泛型编程/image-20250504172139239.png)

加一个中间层（iterator_traits），识别是常规指针还是iterator，重载函数模版，偏特化，因为iterator一般是类。

![image-20250504203420500](./images/STL标准库与泛型编程/image-20250504203420500.png)

## vector深度探索

![image-20250504211310115](./images/STL标准库与泛型编程/image-20250504211310115.png)

- 三个指针，在32位系统占12个字节。

-  插入的时候如果原大小为0， 则将大小变为1，后续增长都在原大小中扩充一倍，每次增长比较大的空间时，就有大量的拷贝构造，和析构函数
- G2.9 vector的迭代器就是内置类型的指针  G4.9版本是 _Tp*

![image-20250504213250114](./images/STL标准库与泛型编程/image-20250504213250114.png)

## array、forward_list深度探索

- G2.9 版本array不允许改变大小，指针当迭代器

  ![image-20250504221021718](./images/STL标准库与泛型编程/image-20250504221021718.png)

- forward_list就是和list差不多的

## deque、queue和stack深度探索

### deque

![image-20250504221549939](./images/STL标准库与泛型编程/image-20250504221549939.png)

五个缓冲区，deque中的 iterator中的node指针指向缓冲区，cur、first、last指向某个缓冲区的具体位置

- 插入时如果在头尾就操作头尾，如果是插入到中间则移动元素离头和尾较近的一端，以此来移动更少的元素
- 模拟连续空间全靠iterator，对一系列的运算符重载

![image-20250505131500412](./images/STL标准库与泛型编程/image-20250505131500412.png)

- deque缓冲区扩充的时候是将原来的数据复制到缓冲区中部，以便后续可以在deque两段插入数据

### queue和stack

![image-20250505134108605](./images/STL标准库与泛型编程/image-20250505134108605.png)

- queue和stack底层容器是deque，就是将部分deque的部分功能调用即可，所以称queue和stack不是容器，是两个适配器
- queue和stack都不提供遍历和iterator，不仅可以选择deque为底层容器，也能选择stack为底层容器，不可选择vector

![image-20250505134429537](./images/STL标准库与泛型编程/image-20250505134429537.png)

## RB-tree （红黑树）深度探索

- 构造需要指定五个模版参数，key，value，keyofvalue，compare，alloc
- handle（句柄） body结构，

## set、multiset深度探索

![image-20250505144129905](./images/STL标准库与泛型编程/image-20250505144129905.png)

- 其iterator为const修饰的红黑树的iterator，以此来保证set不能更改其元素值，
-  set和mutliset调用不同的insert()  

## map、multimap深度探索

 ![image-20250505171754554](./images/STL标准库与泛型编程/image-20250505171754554.png)

- 通过定义pair将第一个元素定义为const类型，组合成value，来保证key不被改变
- find()对于key没找的，会插入key

## hashtable（散列表）深度探索

![image-20250505174221895](./images/STL标准库与泛型编程/image-20250505174221895.png)

- 如果元素比散列表的篮子（bucket）还多，则（不严谨地）扩充一倍bucket，一般采用质数为bucket的数量，buckets采用vector
- 使用hashtable容器需要7个模版，key，value，HashFcn(哈希函数)，ExtractKey，EqualKey，Alloc

![image-20250505174730660](./images/STL标准库与泛型编程/image-20250505174730660.png)

- iterator又两个数据成员组成，一个为指向buckets的指针，一个指向bucket中元素node的指针

![image-20250505175620799](./images/STL标准库与泛型编程/image-20250505175620799.png)

- HashFcn通过模版偏特化重载，如果传入数值，就把数值本身作为哈希值。字符串的哈希值基本上是尽量让哈希值分散。如果自定义HashFcn，需要偏特化hash函数，实现方法尽量靠经验

![image-20250505214401499](./images/STL标准库与泛型编程/image-20250505214401499.png)、![image-20250505214439711](./images/STL标准库与泛型编程/image-20250505214439711.png)



## （unordered_set）hash_set、hash_map、hash_multiset、hash_multimap概念

- hashtable的buckets一定大于元素个数

## 算法的形式

- 算法所需的所有信息全从iterator中获取

![image-20250505220815189](./images/STL标准库与泛型编程/image-20250505220815189.png)

## 迭代器的分类

- 五种迭代器，又继承关系。hashtable为底层的容器的迭代器为前向迭代器

![image-20250505221428727](./images/STL标准库与泛型编程/image-20250505221428727.png)

![image-20250505222231467](./images/STL标准库与泛型编程/image-20250505222231467.png)

## 迭代器的分类对算法的影响，算法函数通过迭代器类型进行重载

- distance函数，通过迭代器的类型进行模版重载，处理不同类型的迭代器

![image-20250510221825200](./images/STL标准库与泛型编程/image-20250510221825200.png)

- advance

![image-20250510222007349](./images/STL标准库与泛型编程/image-20250510222007349.png)

- copy算法，对不同参数类型都进行了不同处理，对于有重要的（trival）赋值操作的元素的类型，需要分开处理

![image-20250510223314957](./images/STL标准库与泛型编程/image-20250510223314957.png)

##   算法源码剖析

- accumulate 对每一个元素执行与init两个参数的函数

![image-20250510232420492](./images/STL标准库与泛型编程/image-20250510232420492.png)

- for_each 对每个元素执行一个操作

![image-20250510232506750](./images/STL标准库与泛型编程/image-20250510232506750.png)

- replace、replace_copy、replace_if

![image-20250510232717189](./images/STL标准库与泛型编程/image-20250510232717189.png)

- count 、count_if

![image-20250510232913613](./images/STL标准库与泛型编程/image-20250510232913613.png)

- find、find_if
- sort

## 仿函数functors

- 最适合绝大部分人拓展的STL部分
- 三大类：算数类、逻辑运算类、相对关系类
- 自定义函数要继承unary_function 和 binary_function,才能融入STL

## Adapters

- Container Adapter 、function Adapter、Iterator Adapter
- stack、queue （Container Adapter）

## 函数适配器

- Binder2nd(已过时)

![image-20250515230108584](./images/STL标准库与泛型编程/image-20250515230108584.png)

- not1

- bind

## 迭代器适配器

- reverse_iterator

![image-20250520230938635](./images/STL标准库与泛型编程/image-20250520230938635.png)

- inserter

![image-20250520231817636](./images/STL标准库与泛型编程/image-20250520231817636.png)

## X适配器（特殊的适配器）

- istream_iterator\ostream_iterator

## 一个万用的Hash Function

![image-20250522224933103](./images/STL标准库与泛型编程/image-20250522224933103.png)

- 利用可变参数模版实现

## Tuple用例

![image-20250525121300597](./images/STL标准库与泛型编程/image-20250525121300597.png)

- tuple使用模版可变参数递归构造，tuple递归继承自己 

![image-20250525121347363](./images/STL标准库与泛型编程/image-20250525121347363.png)

- tuple使用

## type traits

## cout

## movable元素对于STL容器速度的影响

- 右值，编译器知道将来不会再使用