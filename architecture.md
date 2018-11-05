# SQLite 总体架构

SQLite 使用分层式架构，称为 SQLite implementation stack。由下自上分为七个部分：

+ OS Interface
+ Pager
+ B-Tree
+ Virtual machine
+ Code generator
+ Parser
+ Tokenizer

## OS Interface 

这个模块也叫 VFS （虚拟文件系统）。SQLite 使用抽象的 VFS 对象来提供在操作系统间的可移植性。每个 VFS 都会提供自己的读写文件的一套方法。

## Pager

Pager 模块提供了按页缓存的功能。上层的 B-tree 模块需要以页为单位从磁盘读写数据。Pager 负责这些固定大小的页的读、写、缓存操作。Pager 还提供了回滚、原子型提交、数据库文件锁等功能。B-tree 模块的很多功能都需要下层的 pager 的封装和支持。