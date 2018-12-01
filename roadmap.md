# 学习路线

来源：[知乎 - 张原嘉](https://www.zhihu.com/question/22819578/answer/62544197)

先看 [Architecture of SQLite](https://link.zhihu.com/?target=http%3A//www.sqlite.org/arch.html)，了解 SQLite 的架构。总体上分为前端、中层、后端。建议从后端的 OS Interface, Pager, B-Tree 三个部分，按顺序学习。可以一边学习，一边写 demo。

## OS Interface

OS Interface 是很薄的一层, 主要是为了提高可移植性而被设计出来的。

+ [x] 看 [The SQLite OS Interface or "VFS"](https://www.sqlite.org/vfs.html)
+ [x] 看源码中的 `test_demovfs.c`
+ [x] 搞清三个结构体： `sqlite3_vfs`, `sqlite3_io_methods`, `sqlite3_file`

## Pager

Pager 主要实现了三个功能：ACID, log, cache。下面分别介绍。

### ACID

只看最基本的方法，然后跳过。

+ [x] 看 SQLite 是如何实现 ACID 的： [Atomic Commit In SQLite](https://www.sqlite.org/atomiccommit.html)
+ [x] 看书 _SQLite Database System: Design and Implementation_ 中的第 3,4 章，增进理解

### Cache

SQLite 使用插件式的 cache 结构，源码中同时有 `pcache.c` 和 `pcache1.c`。Cache 这个部分还是比较简单的。

+ [ ] 看书 _Inside SQLite_ 第 3 章
+ [ ] 看书 _SQLite Database System_ 第 5 章

### Log

目前先跳过吧。

## B-Tree

这一层就是用 B+ 数维护数据，提供高效的 CRUD。官方文档里没啥资料，不过算法也都是现成的。主要是要理解 SQLite 中怎么实现这些算法。

+ [ ] 看书 _Inside SQLite_ 第 5 章
+ [ ] 看书 _SQLite Database System_ 第 6 章，这里面讲得更详细一点

