# OS Interface 源码

## 源码索引

+ `sqlite.h.in`
  + `sqlite3_vfs`, `sqlite3_file`, `sqlite3_io_methods` 接口定义
+ `test_demovfs.c`
  + 一个 VFS 的示例实现

## `sqlite3_vfs`, `sqlite3_file`, `sqlite3_io_methods` 类

这三个类都定义在模块 `sqlite.h.in` 中，源码如下：

```C
typedef struct sqlite3_vfs sqlite3_vfs;
typedef void (*sqlite3_syscall_ptr)(void);
struct sqlite3_vfs {
    int iVersion;            /* Structure version number (currently 3) */
    int szOsFile;            /* Size of subclassed sqlite3_file */
    int mxPathname;          /* Maximum file pathname length */
    sqlite3_vfs *pNext;      /* Next registered VFS */
    const char *zName;       /* Name of this virtual file system */
    void *pAppData;          /* Pointer to application-specific data */
    int (*xOpen)(sqlite3_vfs*, const char *zName, sqlite3_file*,
               int flags, int *pOutFlags);
    int (*xDelete)(sqlite3_vfs*, const char *zName, int syncDir);
    int (*xAccess)(sqlite3_vfs*, const char *zName, int flags, int *pResOut);
    int (*xFullPathname)(sqlite3_vfs*, const char *zName, int nOut, char *zOut);
    void *(*xDlOpen)(sqlite3_vfs*, const char *zFilename);
    void (*xDlError)(sqlite3_vfs*, int nByte, char *zErrMsg);
    void (*(*xDlSym)(sqlite3_vfs*,void*, const char *zSymbol))(void);
    void (*xDlClose)(sqlite3_vfs*, void*);
    int (*xRandomness)(sqlite3_vfs*, int nByte, char *zOut);
    int (*xSleep)(sqlite3_vfs*, int microseconds);
    int (*xCurrentTime)(sqlite3_vfs*, double*);
    int (*xGetLastError)(sqlite3_vfs*, int, char *);
    /*
    ** The methods above are in version 1 of the sqlite_vfs object
    ** definition.  Those that follow are added in version 2 or later
    */
    int (*xCurrentTimeInt64)(sqlite3_vfs*, sqlite3_int64*);
    /*
    ** The methods above are in versions 1 and 2 of the sqlite_vfs object.
    ** Those below are for version 3 and greater.
    */
    int (*xSetSystemCall)(sqlite3_vfs*, const char *zName, sqlite3_syscall_ptr);
    sqlite3_syscall_ptr (*xGetSystemCall)(sqlite3_vfs*, const char *zName);
    const char *(*xNextSystemCall)(sqlite3_vfs*, const char *zName);
    /*
    ** The methods above are in versions 1 through 3 of the sqlite_vfs object.
    ** New fields may be appended in future versions.  The iVersion
    ** value will increment whenever this happens.
    */
};
```

```C
typedef struct sqlite3_file sqlite3_file;
struct sqlite3_file {
  const struct sqlite3_io_methods *pMethods;  /* Methods for an open file */
};
```

```C
typedef struct sqlite3_io_methods sqlite3_io_methods;
struct sqlite3_io_methods {
  int iVersion;
  int (*xClose)(sqlite3_file*);
  int (*xRead)(sqlite3_file*, void*, int iAmt, sqlite3_int64 iOfst);
  int (*xWrite)(sqlite3_file*, const void*, int iAmt, sqlite3_int64 iOfst);
  int (*xTruncate)(sqlite3_file*, sqlite3_int64 size);
  int (*xSync)(sqlite3_file*, int flags);
  int (*xFileSize)(sqlite3_file*, sqlite3_int64 *pSize);
  int (*xLock)(sqlite3_file*, int);
  int (*xUnlock)(sqlite3_file*, int);
  int (*xCheckReservedLock)(sqlite3_file*, int *pResOut);
  int (*xFileControl)(sqlite3_file*, int op, void *pArg);
  int (*xSectorSize)(sqlite3_file*);
  int (*xDeviceCharacteristics)(sqlite3_file*);
  /* Methods above are valid for version 1 */
  int (*xShmMap)(sqlite3_file*, int iPg, int pgsz, int, void volatile**);
  int (*xShmLock)(sqlite3_file*, int offset, int n, int flags);
  void (*xShmBarrier)(sqlite3_file*);
  int (*xShmUnmap)(sqlite3_file*, int deleteFlag);
  /* Methods above are valid for version 2 */
  int (*xFetch)(sqlite3_file*, sqlite3_int64 iOfst, int iAmt, void **pp);
  int (*xUnfetch)(sqlite3_file*, sqlite3_int64 iOfst, void *p);
  /* Methods above are valid for version 3 */
  /* Additional methods may be added in future releases */
};
```

其中，`sqlite3_vfs::xOpen` 包含 `sqlite3_file` 的指针；`sqlite3_file` 包含 `sqlite3_io_methods` 的指针。

要自己实现一个 `sqlite3_vfs` 类需要按照接口定义各种函数，例如 `test_demovfs.c` 中的一个示例（其中 `demoOpen`, `demoDelete` 等一系列都是前面自己定义的各种函数）：

```C
sqlite3_vfs *sqlite3_demovfs(void){
  static sqlite3_vfs demovfs = {
    1,                            /* iVersion */
    sizeof(DemoFile),             /* szOsFile */
    MAXPATHNAME,                  /* mxPathname */
    0,                            /* pNext */
    "demo",                       /* zName */
    0,                            /* pAppData */
    demoOpen,                     /* xOpen */
    demoDelete,                   /* xDelete */
    demoAccess,                   /* xAccess */
    demoFullPathname,             /* xFullPathname */
    demoDlOpen,                   /* xDlOpen */
    demoDlError,                  /* xDlError */
    demoDlSym,                    /* xDlSym */
    demoDlClose,                  /* xDlClose */
    demoRandomness,               /* xRandomness */
    demoSleep,                    /* xSleep */
    demoCurrentTime,              /* xCurrentTime */
  };
  return &demovfs;
}
```