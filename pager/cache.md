# Cache

Pager 模块的子模块之一提供 page 缓存功能，这种缓存称为 _page cache_。Page cache 放在内存空间中。操作系统可能会将内存的一部分内容放在 (CPU 的) cache 中，但我们不关注这个，SQLite 对 page cache 的管理和操作系统无关。

管理 page cache 主要是为了提升性能。为了提升搜索 cache 的速度，SQLite 将 page cache 组织为散列表。散列表中的每一项包括三个部分：

+ `PgHdr` 对象，pager 模块使用，对上层模块不可见
+ Page 的镜像
+ Private data，供 B+ tree 模块使用

`PgHdr` 中存储的元数据：

+ `pgno` -- page number
+ `injournal` -- 该 page 是否写入了 rollback journal
+ `needSync` -- 将此 page 写回数据库文件之前，journal 是否需要 flush
+ `dirty` -- 该 page 是否被修改过，并且修改没有写回数据库文件
+ `inStmt` -- 该 page 是否在当前的 statement journal 中
+ `nRef` -- 该 page 的引用计数，大于 0 表示 _pinned down_，等于 0 表示 _unpinned_ / _free_

`PgHdr` 中存储的指针：

+ `pNextHash`, `pPrevHash` -- 同一个 bucket 中的 pages 的链接指针
+ `pNextStmt`, `pPrevStmt` -- 在 statement journal 中的 pages 的链接指针
+ `pNextFree`, `pPrevFree` -- free pages 的链接指针

Pager object 中的指针：

+ `pNextAll` 指向所有 pages 的链表
+ `pFree` 指向所有 free pages 的链表
+ `pDirty` 指向所有 dirty pages 的链表

Free pages 不会从 bucket 中取出。

Pager 模块会将 page 的地址传给上层模块。但上层模块不能直接修改 page 的内容，而必须通知 pager 模块何时对 page 进行修改。上层模块在修改 page 内容之前，需要调用 `sqlite3_pager_write()` 函数

## Cache state

`Pager::eState` 表示了 `Pager` 对象可能处于的 7 种状态：

+ `PAGER_OPEN` -- 初始状态
+ `PAGER_READER` -- 开启了 read transactio，可以读 pages
+ `PAGER_WRITER_LOCKED` -- 开启了 write transaction，但还没有修改 cached pages
+ `PAGER_WRITER_CACHEMOD` -- 允许上层模块修改 cached pages
+ `PAGER_WRITER_DBMOD` -- 开始写数据库文件
+ `PAGER_WRITER_FINISHED` -- write transaction 即将 commit，所有修改已经结束
+ `PAGER_ERROR` -- 出错