# OS Interface

也叫 VFS （虚拟文件系统）。很薄的一层, 主要是为了提高可移植性而被设计出来的。

学习方法：

+ 看文档 [The OS Backend (VFS) To SQLite](https://link.zhihu.com/?target=http%3A//www.sqlite.org/vfs.html)
+ 看源码中的 `test_demovfs.c`（一个简单的示例 VFS），搞清楚 `sqlite3_vfs` `sqlite3_io_methods` `sqlite3_file` 三个结构体
