# MVCCTree
MVCC技术

<pre>
MVCC(Multi-Version Concurrency Control)多版本并发控制。

      Mysql的大多数事务（如InnoDB, FaIcon）型存储引擎实现的都不是简单的行级锁，基于提升
   并发性能的考虑，他们一般都实现了MVCC。当前不仅仅是MYSQL，其它数据库系统如Oracle, 
   PostgreSQL也都实现了MVCC，MVCC并没有一个统一的标准，不同的数据库，不同的存储引擎的
   实现都不相同。
</pre>

<pre>
MVCC的优缺点
      MVCC在大多数情况下代替了行级锁，实现了对读的非阻塞，读不加锁，读写不冲突。缺点是没
   行记录都需要额外的存储空间，需要做更多的行维护和检查工作。
</pre>

![](https://i.imgur.com/4I4uUR9.png)

<pre>
MVCC的实现原理
    undo log
        为了便于理解MVCC的实现原理，简单介绍一下undo log的工作过程。
        1：开始事务
        2：记录数据行数据快照到undo log
        3: 更新数据
        5：将undo log写入磁盘
        6：将数据写入磁盘
        7：提交事务
    1）为了保证数据的持久性数据要在事务提交前持久化
    2）undo log的持久化必须在数据持久化之前，这样才能保证系统崩溃时，可以用undo log来
       回滚事务。

    InnoDB中的隐藏列
       InnoDB通过undo log保存了已更改行的旧版本的信息的快照。
       InnoDB的内部实现中为每一行数据增加了三个隐藏列用于实现MVCC

       列名         长度  作用
       DB_TRX_ID    6    插入或更新行的最后一个事务的事务 标识符（删除视作更新，标记为删除）
       DB_ROLL_PRT  7    写入回滚段的撤销日志记录（若行已更新，则撤销日志记录包含在更新行之前重
                             建行内容所需的信息）
       DB_ROW_ID    6    行标识（隐藏单调自增ID）
</pre>

<pre>
MVCC的工作过程
       MVCC只在READ COMMITED和REAPEATED READ两个隔离级别下工作。
       READ UNCOMMITED 总是读取最新的数据行，而不是符合当前事务版本的数据行，而SERIALIZABLE则会
    对所有的数据行加锁。
</pre>

<pre>
Select
      InnoDB会根据两个条件来检查每行记录：
          1：InnoDB只查找版本(DB_TRX_ID)早于当前事务版本的数据行（行的系统版本号 <= 事务的系统版本号，
             这样可以确保数据行要么是在开始之前已经存在的，要么是事务吱声插入或修改过的。
          2：行的删除版本号（DB_ROLL_PTR）要么未定义（未更新过），要么大于当前事务版本号（在当前事务开始
             之后更新的），这样可以确保事务读取到的行，在事务开始之前未被删除。
</pre>

<pre>
Insert
      InnoDB为新插入的每一行保存当前系统版本号作为行版本号
</pre>

<pre>
      InnoDB为删除的每一行保存当前系统的版本号作为删除标识
</pre>

<pre>
UPDATE
      InnoDB为插入一行新记录，保存当前系统版本号作为行版本号，同时保存当前系统版本号到原来的行作为删除标识。
</pre>
