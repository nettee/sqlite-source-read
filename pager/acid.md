# ACID

## 基础知识

锁的概念：shared lock (read lock) 和 exclusive lock (write lock)

## Atomic Commit In SQLite 文章

每次向磁盘写一定量的数据。用 Sector 表示每次 write 的最小数据量。典型的大小是 512B，这个大小在 3.3.14 之前是一个写死的值。

SQLite 假设磁盘 sector 的写操作是 _线性的_，即要么从前向后写，要么从后向前写，不会从中间写。

SQLite 假设操作系统会缓存和打乱写请求的顺序，所以在关键的地方 flush，保证写操作的关键顺序不会出错。

SQLite 假设操作系统的“删除文件”操作是原子的（至少从用户角度而言）。SQLite 检查 rollback journal 文件是否存在来判断 transaction 是否完成。如果 rollback journal 文件存在，则进行回滚；如果不存在，则认为 transaction 完成。“删除文件”的原子性保证了 transaction 的原子性。

保证 rollback 能力的关键是：在向数据库文件写数据前，rollback journal 文件一定要 flush 到磁盘上。即使中途断电，rollback journal 文件还是完整保存的。

## SQLite Database System 图书

SQLite 只支持 flat transaction，没有 nested transaction。（需要看书的前面章节）

DBMS 一般使用 lock 进行并发控制，使用 journal 存储 recovery 信息。

在写数据库文件之前，首先写 journal 文件，并保证 journal 保存在 stable storage （即磁盘，断电后可恢复）。如果 transaction 中间出错，可以使用 journal 恢复到之前的一致状态。

### Transaction

Transaction 分为 read-transaction 和 write-transaction。在 read-transaction 中只能读数据；在 write-transaction 中可以读写数据。同一时刻只能有一个 write-transaction，但可以有多个 read-transaction。

两类 Transaction：

+ _System transaction_ (or _auto transaction_, _implicit transaction_)
  + SQLite 在默认的 _autocommit_ 模式中
  + 自动把每一条 SQL 语句包裹在合适的 transaction 中：SELECT 语句对应 read-transaction，其他语句对应 write-transaction
+ _User transaction_ (or _explicit transaction_)
  + SQLite 退出 autocommit 模式
  + 通过 `begin ... commit` 或者 `begin ... rollback` 语句声明
  + 缓解对每个语句都自动添加 write-transaction 的开销 
  + 只会影响 write-transaction；read-transaction 仍然是自动管理的

在 transaction 中可以设置 _savepoint_。它表示一个良好的数据库状态，回滚时可以回滚到某个 savepoint。

### Locking

Locking 利用的是操作系统的 **file locking primitives**。SQLite 针对数据库文件上锁，而不会针对更细的粒度。因为限制了 read-transaction 和 write-transaction 的并发数量，SQLite 的 lock management 其实比较简单。

SQLite 中 lock 的类型：

+ Shared lock
  + 多个 transaction 可以同时拥有该 lock
  + 允许当前 transaction 读数据库文件
  + 不允许其他 transaction 写数据库文件
+ Exclusive lock
  + 只有一个 transaction 可以拥有该 lock
  + 允许当前 transaction 读/写数据库文件
  + 不允许其他任何 lock 存在
+ Reserved lock
  + 只有一个 transaction 可以拥有该 lock
  + 语义：计划写，但当前还只是读
  + 允许当前 transaction 读数据库文件
  + 允许任意多个 shared lock 存在，不允许除 shared lock 以外的任何 lock 存在
+ Pending lock (internal state)
  + 只有一个 transaction 可以拥有该 lock
  + 语义：马上要写
  + 允许当前 transaction 读数据库文件
  + 允许任意多个 shared lock 存在，但不允许新的 shared lock 创建，不允许除 shared lock 以外的任何 lock 存在

| Lock | 语义 | 允许读 | 允许写 | 是否允许多个该 lock | 是否允许其他 lock |
| :-: | :-: | :-: | :-: | :-: | :-: |
| Shared lock | 读数据库文件 | Y | N | Y | 不允许 exclusive lock，允许其他 |
| Exclusive lock | 写数据库文件 | Y | Y | N | 不允许任何 |
| Reserved lock | 计划写数据库文件 | Y | N | N | 允许多个 shared lock，不允许其他 |
| Pending lock | 马上要写数据库文件 | Y | N | N | Shared lock 不允许创建，其他不允许存在 |

Reserved lock 和 pending lock 有利于减少 deadlock 的可能性。

Lock 的获取顺序：

+ read-transaction: no lock -> shared -> no lock
+ write-transaction: no lock -> shared -> reserved -> pending -> exclusive -> no lock

获得 reserved lock 之后就可以向 cache 中的 page 写数据了；而获得 exclusive lock 之后才可以 write back。

Pager 模块负责获取适当的 lock。Pager 调用 `sqlite3OsLock` 来获取或者升级 lock；调用 `sqlite3OsUnlock` 来释放或者降级 lock。

大部分情况下，用户只需要声明 transaction，locking 的操作都是由系统完成的（更具体点，是 pager 模块完成的）。

Linux 系统只支持两种 lock 模式（read lock 和 write lock），而 SQLite 在其上建立了四种 lock 模式。SQLite 在不同的文件区域中使用 Linux lock。数据库文件中有一个 lock bytes 区域，有 pending byte, reserved byte, shared bytes (many) 等，SQLite 通过对不同的 bytes 加 lock 来实现四种 lock 模式。其中 exclusive lock 是对所有的 bytes 加上 write lock。

