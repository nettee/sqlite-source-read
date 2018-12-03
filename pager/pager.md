# Pager

Pager 将操作系统的 IO API 封装，向上提供了易使用、与文件系统无关的访问数据库文件的接口。Pager 模块在以 byte-oriented 的文件上抽象出了 page-oriented 的数据库文件。上层模块可以将数据库文件看成是 page (页) 的数组。

Pager 模块可分为三个子模块：page cache，lock manager 和 log manager。三个子模块共同提供了 data/transaction manager 的功能。这三个子模块的功能同样对上层模块是不可见的，上层模块只需要将操作看成一个个的 transactions，而不需要考虑 transaction 是如何具体实施的。

Pager 模块实现了 `Pager` 数据结构。每一个 `Pager` 对象唯一对应一个打开的数据库文件。同一个数据库文件有多个 connection 时，每个 connection 都关联一个 `Pager` 对象。上层模块要使用数据库文件时，先创建一个 `Pager` 对象，然后通过这个 `Pager` 对象进行文件操作。

Pager 模块提供的一些接口函数：

+ `sqlite3PagerOpen` -- 打开数据库文件，创建 `Pager` 对象
+ `sqlite3PagerClose` -- 关闭数据库文件，销毁 `Pager` 对象
+ `sqlite3PagerGet` -- 通过 page number 获取一个页
+ `sqlite3PagerWrite` -- 在修改一个 page 之前发起写请求
+ `sqlite3PagerLookup` -- 查看某个 page 是否在 cache 中
+ `sqlite3PagerRef` -- 将某个 page 的引用计数加 1
+ `sqlite3PagerUnref` -- 将某个 page 的引用计数减 1
+ `sqlite3PagerBegin` -- 显式开启一个 write transaction
+ `sqlite3PagerCommitPhaseOne`
+ `sqlite3PagerCommitPhaseTwo`
+ `sqlite3PagerRollback`
+ `sqlite3PagerOpenSavepoint`
+ `sqlite3PagerSavepoint`