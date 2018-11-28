# ACID

## Atomic Commit In SQLite 文章

每次向磁盘写一定量的数据。用 Sector 表示每次 write 的最小数据量。典型的大小是 512B，这个大小在 3.3.14 之前是一个写死的值。

SQLite 假设磁盘 sector 的写操作是 _线性的_，即要么从前向后写，要么从后向前写，不会从中间写。

SQLite 假设操作系统会缓存和打乱写请求的顺序，所以在关键的地方 flush，保证写操作的关键顺序不会出错。

基础：锁的概念：shared lock (read lock) 和 exclusive lock (write lock)

SQLite 假设操作系统的“删除文件”操作是原子的（至少从用户角度而言）。SQLite 检查 rollback journal 文件是否存在来判断 transaction 是否完成。如果 rollback journal 文件存在，则进行回滚；如果不存在，则认为 transaction 完成。“删除文件”的原子性保证了 transaction 的原子性。

保证 rollback 能力的关键是：在向数据库文件写数据前，rollback journal 文件一定要 flush 到磁盘上。即使中途断电，rollback journal 文件还是完整保存的。