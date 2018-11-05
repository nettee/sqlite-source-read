# OS Interface

OS Interface 也叫 VFS （虚拟文件系统），使 SQLite 在不同操作系统间可移植。这一层位于 SQLite implementation stack 的最底层。SQLite 的其他模块都需要通过 VFS 来和操作系统交互。这样，要将 SQLite 移植到一个新的操作系统，只需要针对这个操作系统写一个新的 VFS，并注册到 SQLite 中即可。

## 注册与使用 VFS

SQLite 中内置了多个 VFS，包括 Unix 系的 `unix` 及其变种，Windows 系的 `win32` 及其变种。

使用 `sqlite3_vfs_register()` 接口既可以注册自己实现的新的 VFS，也可以重新注册已有的 VFS 并设置为默认。

## VFS 接口

VFS 主要使用三个类来抽象出文件系统接口：

+ `sqlite3_vfs`: 代表 VFS 实例。它实现了很多文件系统的操作接口，例如创建/删除文件、检查文件存在、打开文件等。这个对象里还有一些辅助的函数，如生成随机数、获取时间、sleep 等。
+ `sqlite3_file`: 代表一个 *已打开的* 文件。`sqlite3_vfs::xOpen` 函数会创建一个 `sqlite3_file` 对象以表示打开的文件。这个对象会记录文件的状态（如文件指针等）。
+ `sqlite3_io_methods`: 包括一些用于处理已打开文件的函数，包括读/写文件，截断文件，flush，关闭文件等。它也包括销毁 `sqlite3_file` 对象的函数。每个 `sqlite3_file` 对象都包含一个 `sqlite3_io_methods` 对象的指针。

要想实现自己的 VFS，只需要实例化以上三个类，实现对应的接口即可。

## 参考文档

+ [The OS Backend (VFS) To SQLite](https://link.zhihu.com/?target=http%3A//www.sqlite.org/vfs.html)
+ [源码分析](os_source.md)
