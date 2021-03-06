## Week9: Index

之前讲了 MySQL数据库的部分内核原理, 执行流程, 事务和锁. 本节开始讲索引原理和查询流程.



### 64. 磁盘数据页的存储结构

1. **两两相邻数据页之间形成双向链表**

2. **数据页每行数据按照逐渐大小排序存储, 组成递增的单向链表.**

   ![image-20200914215547787](week9-%E7%B4%A2%E5%BC%95.assets/image-20200914215547787.png)



### 65. 如果没有索引, 如何搜索数据

1. **每个数据页都有一个页目录, 根据数据行的逐渐存放目录.** 

   <img src="week9-%E7%B4%A2%E5%BC%95.assets/image-20200914224441063.png" alt="image-20200914224441063" style="zoom: 50%;" />

2. 查找一条数据, 

   - 根据主键查: 根据数据页目录的主键进行二分查找.  定位到槽位. 
   - 根据非主键字段: 进入数据页, 按照单向链表遍历

3. 大量数据页的表如何查(全表扫描):
   - 所有数据页组成双向链表
   - 开始从第一个数据页里, 二分法/遍历查找.
   - 双向链表到下一个数据页里找. ...



### 66. 表内数据增加, 物理存储会页分裂

没有索引, 所有的查询都是全表扫描. 在一个表里不断的插入数据, 会有**页分裂**

1. **索引运行的核心基础**是, 数据页链表里面, **主键是递增的**, 后一个数据页的逐渐大于前一个数据页的主键.

2. 如果第一个数据页一条主键是10, 第二个**数据页居然有一个小于10, 此时就要页分裂**, 

   **把前一个数据页里的大主键挪到后面一个数据页, 小主键放到前面一个.** 





### 67. 基于主键的索引结构和使用

没有索引, 即使是主键也需要扫描所有的数据页. 

1. **主键索引:** 就是主键的目录, 把**[数据页页码+本页最小主键值] 组成索引目录.**

   ![image-20200915002502251](week9-%E7%B4%A2%E5%BC%95.assets/image-20200915002502251.png)

2. 基于主键索引的查询: 

   **在索引里搜索到页码(二分法), 然后去拿就好了.** 



### 68. 索引页 存储结构(B+ tree)

上面讲的hash索引, 如果表里数据很多, 那么hash索引里存放太多的k-v, 

1. **index的数据存储在数据页里面**

   ![image-20200915003739878](week9-%E7%B4%A2%E5%BC%95.assets/image-20200915003739878.png)

2. **索引页很多, 给索引页加一个层级[高级索引页], 保存[索引页页码, 页内最小主键值]**

   <img src="week9-%E7%B4%A2%E5%BC%95.assets/image-20200915005219502.png" alt="image-20200915005219502" style="zoom:50%;" />

   **组成了B+树**



### 69. 更新自维护的聚簇索引

主键索引就是聚簇索引嘛. 

1. **搜索主键Id对应的行, 从顶层的索引页开始, 按照二分法(**索引页里面是有序链表, 遍历, 索引页组成B+树, 按照二分法), 

2. 索引页组成B+树里面的node, 同一个级别的索引页也组成双向链表(和数据页一样)

   ![image-20200915005943175](week9-%E7%B4%A2%E5%BC%95.assets/image-20200915005943175.png)

3. 对数据增删改的时候, 直接把数据页放在聚簇索引里面. 

   - 如果发生**页分裂**, 就开始调整数据页, 保证双向链表数据页组成递增的主键id list.

   - 页分裂时候会维护上层索引结构, 上层索引页开始调整. 

   - 数据增多, 索引页也增加, 索引页node太多, 索引页上层的非叶子节点也增加. 



### 70. 非主键索引

**插入数据的时候:**

- 把完整数据放到聚簇索引的叶子节点数据页里, 维护好

- 为其他字段建立的索引创建一棵B+树(只放字段(k)和主键(v))

  这个B+树的叶子节点只存放id和字段, 数据页构成双向链表, 有序的.

![image-20200916223406998](week9-%E7%B4%A2%E5%BC%95.assets/image-20200916223406998.png)

- **非聚簇索引叶子节点: 存的都是<字段, id>. 如果字段相同, id增序排序.**
- 从非主键索引查到ID, 然后**回表**, 去聚簇索引查. 

**联合索引:** 

- 每个索引页里面放的<下面的索引页号, 最小的第一个字段, 最小的第二个字段>



### 71. 插入数据时如何维护所有的B+树

1. 表刚建, 只有一个数据页, 他就是聚簇索引的一部分(根页).
2. 插入数据, 直接插入. 直到需要第二个数据页
3. 组成双向链表, 根页把数据放给数据页. 
4. 根页变成了索引页. 放两个数据页的页号和最小主键值.
5. 数据增多, 数据页发生页分裂. 
6. 索引页也增多, 然后升级成一棵树. 
7. 树在生长. 



- **非聚簇索引叶子节点: 存的都是<字段, id>. 如果字段相同, id增序排序.**
- 非聚簇索引页, 放着最小的索引字段对应的id, 如果这个索引跨页了, 可以用这个id对比一下.



### 72. 索引带来的消耗

- 时间: 
  - 如果根据其他字段索引搜索, 只能基于其他字段的B+树查到ID, 然后会回表查询. 
  - 新增修改要维护多棵B+树, 树内的有序性. 

- 空间: 多棵B+树占据空间. 



### 73. 联合索引查询, 全值匹配规则

设计表的时候一般用联合索引: 很少单个字段做索引: 我们希望让索引数量尽量少.

- 如果一张学生成绩表: column: class, name, course, score 四个字段.

- 根据:<class, name, course> 建立联合索引.

- 索引B+树长这样: 按照class, name, course依次排序.

  ![image-20200917002311286](week9-%E7%B4%A2%E5%BC%95.assets/image-20200917002311286.png)

- 在全值匹配查询的过程中, 就在B+树里面左右逢源. 最终定位到一个. 二分法. **最后带着ID回表.**

- 查询条件有索引里所有字段, 然后用等号, 就是全值匹配



### 74. 基本的索引使用规则

- **等值匹配规则:** 建立联合索引之后, select索引字段, 而且是=等值匹配.  
- **最左侧列匹配:** 联合索引匹配最左侧字段
- **最左前缀匹配:** 使用like语句查字符串"abc%", 按照这个字段索引从abc开始匹配. 也用得上
- **范围查找规则:** where column1 < 'abc' and column1>'aba'; 也用得上索引, 回到叶子节点索引的**双向链表来回寻找.** 
- **等值匹配+范围匹配:** 联合索引多个字段, 前面几个等值, 后面的范围查询. 可以哦. **但也只能用一次范围查询.** 



### 75. 索引用于 排序SQL

`select * from table1 where column1='' order by column2`

- **普通执行:** 基于where语句[通过索引]筛选出数据, 在内存/临时磁盘文件里, 排序, 返回

  尽量避免

- **联合索引:** 联合索引<c1, c2, c3>, 本身就是排序好的. 

  **不过有一些限制:** 

  - 索引字段在索引树里从大到小排序, 所以**Order所有的字段要不就是ASC, 要不就是全部DESC.**
  - **order by的字段有不在联合索引里的, 或者有复杂函数**, 也不能利用联合索引



### 76. 索引用于 分组SQL

group by 把数据聚合统计, **字段最好是从联合索引的最左侧字段开始, 就可以匹配上联合索引的排序了.**

保证常用SQL语句都用上索引, 都不会太慢.  还有就要**按照执行计划, 进行SQL调优.**

更新语句的问题:

- 索引太多
- 锁等待和死锁
- MySQL连接池/redolog...之类问题. 





### 77. 回表查询的性能损害, 覆盖索引是什么

回表: 意味着要扫描两边树.

**覆盖索引:** 是基于索引的查询方式, 不是物理上的索引.  

- 索引上有自己想要的字段了, 就不用回表, 就是覆盖索引查询.





### 78-80. 设计索引考虑因素

- 表结构设计: 字段, 字段类型, 包含的数据是什么
- 索引设计: 根据查询的条件. 也可以等到开发时候使用的sql构建索引. 
  - 针对 where, order by, group by, 设计索引. 联合索引优先. 
  - **使用基数大的字段建索引:** 发挥B+树二分法的查询优势.
  - **前缀索引:**  如果有varchar(255)的大字段, 可以使用前20个字符创建索引. 但是只能等值搜索使用. 排序和group就用不上了. 
  - **查询字段不要搞函数:** 
  - **不要搞太多字段:** 索引的维护(页分裂)会耗费时间.
  - **主键用自增的:** 不要用UUID, **自增不会频繁的页分裂**





### 81-84. 案例: 社交App的MySQL索引设计

> 业务场景: 通过一系列条件去搜索和筛选好友(user_info). 
>
> 查询条件有: 性别年龄, 身高体重, 标签...最近在线时间...
>
> 还要排序, 分页查询. 
>
> SQL: select ** from user_info where sex='man' and age > 32 order by time limit 1, 20;

- **在业务SQL里, where筛选和order by排序很难用到索引.**  

  **where 和 order by出现索引设计冲突:** 是要where用索引, 拿到数据放在内存里做排序, 还是先用索引排序, 然后用where筛选. 

  - 一般是where用索引, 放在内存里排序. 

- **联合索引用哪些字段做? 字段顺序如何?**  
  - 把区分度低且必须用的放在联合索引左侧. 
  - 针对枚举类的字段, 也可以放进联合索引里(爱好标签之类的): 即使不筛选, 也可以把枚举值都放进去, 也能用到联合索引. 
  - **范围查询的字段尽量放在联合索引最后面.** 
  - 两个范围查询: 如果要查用户活跃时间, 'latest_login_time<=7day', 可以设计标志的冗余字段. `is_login_latest_7_days`.
  - **联合索引只用到了区分度很低的左侧字段:** 可以额外加一个索引, 包括**区分度低的字段+范围查询字段.** 

- **核心重点: 尽量用一两个复杂的多字段联合索引, 抵抗大多数查询, 然后添加一两个辅助索引**补充. 保证99%以上的SQL都用到索引. 



















