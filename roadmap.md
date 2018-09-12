# 学习路线

来源：[知乎 - 张原嘉](https://www.zhihu.com/question/22819578/answer/62544197)

先看 [Architecture of SQLite](https://link.zhihu.com/?target=http%3A//www.sqlite.org/arch.html)，了解 SQLite 的架构。总体上分为前端、中层、后端。建议从后端的 OS Interface, Pager, B-Tree 三个部分，按顺序学习。

## OS Interface

很薄的一层, 主要是为了提高可移植性而被设计出来的。

学习方法：

+ 看文档 [The OS Backend (VFS) To SQLite](https://link.zhihu.com/?target=http%3A//www.sqlite.org/vfs.html)
+ 看源码中的 `test_demovfs.c`，搞清楚 `sqlite3_vfs` `sqlite3_io_methods` `sqlite3_file` 三个结构体

## Pager

Pager 主要实现了三个功能：ACID, log, cache。

### ACID

+ 看文档 [Atomic Commit In SQLite](https://link.zhihu.com/?target=http%3A//www.sqlite.org/atomiccommit.html)
+ 看书 _SQLite Database System: Design and Implementation_ 中的第 3,4 章

### Cache

SQLite 使用插件式的 cache 结构。

+ 看书 _Inside SQLite_ 第 3 章
+ 看书 _SQLite Database System_ 第 5 章

### Log

略

## B-Tree

算法本身，理解 B 树和 B+ 树的算法。

+ 看书 _Inside SQLite_ 第 5 章
+ 看书 _SQLite Database System_ 第 6 章
