# ACID 源码

## 源码索引

+ `os.h`
  + Lock bytes 定义
+ `os.c`
  + `sqlite3OsLock`, `sqlite3OsUnlock` 定义
+ `os_unix.c`
  + `unix` VFS 的具体 `xLock`

## `sqlite3OsLock`, `sqlite3OsUnlock`

定义在 OS Interface 层。最终会调用 `sqlite3_io_methods::xLock` 和 `sqlite3_io_methods::xUnlock`。

```C
/* File: os.c */

int sqlite3OsLock(sqlite3_file *id, int lockType){
  DO_OS_MALLOC_TEST(id);
  return id->pMethods->xLock(id, lockType);
}

int sqlite3OsUnlock(sqlite3_file *id, int lockType){
  return id->pMethods->xUnlock(id, lockType);
}
```

具体的 `xLock()`, `xUnlock()` 如何实现取决于 VFS。我们参考一下 `unix` VFS 中的实现：`os_unix.c` 模块中的 `unixLock()`

### 示例：`unixLock()`

```C
/* File: os_unix.c */

static int unixLock(sqlite3_file *id, int eFileLock) {
    /* ... */
}
```

参数 `eFileLock` 表示 lock 类型，共有四种可能： `SHARED_LOCK`, `RESERVED_LOCK`, `PENDING_LOCK`, `EXCLUSIVE_LOCK`

可能出现的五种 lock 状态转移（括号中为自动插入的中间状态）：

+ UNLOCKED -> SHARED
+ SHARED -> RESERVED
+ SHARED -> (PENDING) -> EXCLUSIVE
+ RESERVED -> (PENDING) -> EXCLUSIVE
+ PENDING -> EXCLUSIVE

`unixLock()` 要利用 POSIX 系统的 lock primitives 来实现上述的四种 lock 类型和五种 lock 状态转移。我们知道，POSIX 提供 `fcntl()` 函数对文件进行 lock 操作。`fcntl()` 函数实际上提供的是 byte-range locking 功能，也就是可以对文件中的某些字节进行 locking。不过 byte-range locking 只支持两种类型的 lock: read lock 和 write lock（这样叫是为了不和 SQLite 中的 lock 名称混淆）。因此，SQLite 在数据库文件中定义了一些特殊的 bytes。通过这些 bytes 上 lock 的不同状态来表示四种 lock。

SQLite 定义了 1 字节的 pending byte，1 字节的 reserved byte，以及 510 字节的 shared bytes。简化版的定义如下：

```C
/* File: os.h */

#define PENDING_BYTE      (0x40000000)
#define RESERVED_BYTE     (PENDING_BYTE+1)
#define SHARED_FIRST      (PENDING_BYTE+2)
#define SHARED_SIZE       510
```

+ 获取 shared lock 时，首先尝试在 pending byte 上获取 read lock；如果成功，在 shared bytes 上加 read lock，并释放 pending byte 上的 lock
+ [要求已经获得了 shared lock] 获取 reserved lock 时，在 reserved byte 上加 write lock
+ [要求已经获得了 reserved lock] 获取 pending lock 时，在 pending byte 上加 write lock
  + 这样保证了有一个进程获得 pending lock 之后，其他进程无法获得新的 shared lock
+ [要求已经获得了 pending lock] 获取 exclusive lock 时，在 shared bytes 上加 write lock
  + 因为其他的 lock 都需要在 shared bytes 上获得 read lock，这样保证了有一个进程获得 exclusive lock 之后，其他进程无法获得任何 lock

以上功能其实只需要一个 shared byte 就可以完成。然而，在 SQLite 设计之初， Windows 95 系统只有 write lock 而没有 read lock。为了适配 Windows 95 系统，SQLite 定义了很多 shared bytes，每次随机选取一个 shared byte 并加锁来模拟 read lock。这一特性（510 个 shared bytes）为了向后兼容性被保留了下来。