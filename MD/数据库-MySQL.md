
## MySQL
### 引擎对比
1. InnoDB支持事务
2. InnoDB支持外键
3. InnoDB是聚集索引，数据文件是和索引绑在一起的，必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而MyISAM是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。
4. InnoDB有行级锁

因为MyISAM相对简单所以在效率上要优于InnoDB。如果系统读多，写少，对原子性要求低。那么MyISAM最好的选择。  
如果系统读少，写多的时候，尤其是并发写入高的时候，还需要事务安全性。InnoDB就是首选了。

### SQL性能优化
* 对经常查询的列建立索引，为了避免全表扫描，建多了当数据改变时修改索引浪费资源
* 使用精确列名查询而不是*
* 减少嵌套查询
* 不用NOT IN,IS NULL,NOT IS NULL，无法使用索引

问：有个表特别大，字段是姓名、年龄、班级，如果调用`select * from table where name = xxx and age = xxx`该如何通过建立索引的方式优化查询速度？  
答：由于mysql查询每次只能使用一个索引，所以虽然这样已经相对不做索引时全表扫描提高了很多效率，但是如果在name、age两列上创建复合索引的话将带来更高的效率。如果我们创建了(name, age, class)的复合索引，那么其实相当于创建了(name, age, class)、(name, age)、(name)三个索引，这被称为最佳左前缀特性。因此我们在创建复合索引时应该将最常用作限制条件的列放在最左边，依次递减。

## 事务隔离级别
1. 原子性（Atomicity）：事务作为一个整体被执行 ，要么全部执行，要么全部不执行；
2. 一致性（Consistency）：保证数据库状态从一个一致状态转变为另一个一致状态；
3. 隔离性（Isolation）：多个事务并发执行时，一个事务的执行不应影响其他事务的执行；
4. 持久性（Durability）：一个事务一旦提交，对数据库的修改应该永久保存。

[https://www.jianshu.com/p/4e3edbedb9a8](https://www.jianshu.com/p/4e3edbedb9a8)

## 锁表、锁行
[https://segmentfault.com/a/1190000012773157](https://segmentfault.com/a/1190000012773157)

### 何时锁

### 悲观锁乐观锁、如何写对应的SQL
[https://www.jianshu.com/p/f5ff017db62a](https://www.jianshu.com/p/f5ff017db62a)

### 索引
#### 原理
我们拿出一本新华字典，它的目录实际上就是一种索引：非聚集索引。我们可以通过目录迅速定位我们要查的字。而字典的内容部分一般都是按照拼音排序的，这实际上又是一种索引：聚集索引。聚集索引这种实现方式使得按主键的搜索十分高效，但是辅助索引搜索需要检索两遍索引：首先检索辅助索引获得主键，然后用主键到主索引中检索获得记录。

主要使用[B+树](https://www.jianshu.com/p/3a1377883742)来构建索引，为什么不用二叉树是因为B+树是多叉的，可以减少树的高度，还有事索引本身较大，不会全部存储在内存中，会以索引文件的形式存储在磁盘上，然后是因为[局部性原理](https://www.cnblogs.com/xyxxs/p/4440187.html)，数据库系统巧妙利用了磁盘预读原理，将一个节点的大小设为等于一个页，这样每个节点只需要一次I/O就可以完全载入，(由于节点中有若干个数组，所以地址连续)。而红黑树这种结构，深度更深。由于逻辑上很近的节点（父子）物理上可能很远，无法利用局部性

#### 分析
好处：

1. 快速取数据
2. 唯一性索引能保证数据记录的唯一性
3. 实现表与表之间的参照完整性

缺点：

1. 索引需要占物理空间。
2. 当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，降低了数据的维护速度。

#### 使用场景
如果某个字段，或一组字段会出现在一个会被频繁调用的 WHERE 子句中，那么它们应该是被索引的，这样会更快的得到结果。为了避免意外的发生，需要恰当地使用唯一索引
