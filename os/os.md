# OS Interface

OS Interface 也叫 VFS （虚拟文件系统），使 SQLite 在不同操作系统间可移植。这一层位于 SQLite implementation stack 的最底层。SQLite 的其他模块都需要通过 VFS 来和操作系统交互。这样，要将 SQLite 移植到一个新的操作系统，只需要针对这个操作系统写一个新的 VFS，并注册到 SQLite 中即可。

学习方法：

+ 看文档 [The OS Backend (VFS) To SQLite](https://link.zhihu.com/?target=http%3A//www.sqlite.org/vfs.html)
+ 看源码中的 `test_demovfs.c`（一个简单的示例 VFS），搞清楚 `sqlite3_vfs` `sqlite3_io_methods` `sqlite3_file` 三个结构体

## VFS implementations

VFS 中最重要的三个类 (struct)：

+ `sqlite3_vfs`: 代表 VFS 实例。它实现了很多文件系统的操作接口，例如创建/删除文件、检查文件存在、打开文件等。这个对象里还有一些辅助的函数，如生成随机数、获取时间、sleep等。
+ `sqlite3_file`: 代表一个 *已打开* 文件。`sqlite3_vfs` 中的 `xOpen` 函数会创建一个 `sqlite3_file` 对象以表示打开的文件。这个对象会记录文件的状态（如文件指针等）。
+ `sqlite3_io_methods`: 包括一些用于处理已打开文件的函数，包括读/写文件，截断文件，flush，关闭文件等。它也包括销毁 `sqlite3_file` 对象的函数。每个 `sqlite3_file` 对象都包含一个 `sqlite3_io_methods` 对象的指针。
