# `sqlite3_vfs` class

`sqlite.h.in`, Lines 1262-1364

The [`SQLITE_OPEN_EXCLUSIVE`] flag is always used in conjunction with the [`SQLITE_OPEN_CREATE`] flag, which are both directly analogous to the `O_EXCL` and `O_CREAT` flags of the POSIX open() API.  The `SQLITE_OPEN_EXCLUSIVE` flag, when paired with the `SQLITE_OPEN_CREATE`, is used to indicate that file should always be created, and that it is an error if it already exists.  It is <i>not</i> used to indicate the file should be opened for exclusive access.

At least szOsFile bytes of memory are allocated by SQLite to hold the  [`sqlite3_file`] structure passed as the third argument to xOpen.  The xOpen method does not have to allocate the structure; it should just fill it in.  Note that the xOpen method must set the `sqlite3_file`.pMethods to either a valid [`sqlite3_io_methods`] object or to NULL.  xOpen must do this even if the open fails.  SQLite expects that the `sqlite3_file.pMethods` element will be valid after xOpen returns regardless of the success or failure of the xOpen call.

[[`sqlite3_vfs.xAccess`]] ^The flags argument to xAccess() may be [`SQLITE_ACCESS_EXISTS`] to test for the existence of a file, or [`SQLITE_ACCESS_READWRITE`] to test whether a file is readable and writable, or [`SQLITE_ACCESS_READ`] to test whether a file is at least readable.   The file can be a directory.

SQLite will always allocate at least mxPathname+1 bytes for the output buffer xFullPathname.  The exact size of the output buffer is also passed as a parameter to both  methods. If the output buffer is not large enough, [`SQLITE_CANTOPEN`] should be returned. Since this is handled as a fatal error by SQLite, vfs implementations should endeavor to prevent this by setting mxPathname to a sufficiently large value.

The xRandomness(), xSleep(), xCurrentTime(), and xCurrentTimeInt64() interfaces are not strictly a part of the filesystem, but they are included in the VFS structure for completeness.  The xRandomness() function attempts to return nBytes bytes of good-quality randomness into zOut.  The return value is the actual number of bytes of randomness obtained.  The xSleep() method causes the calling thread to sleep for at least the number of microseconds given.  ^The xCurrentTime() method returns a Julian Day Number for the current date and time as a floating point value.
The xCurrentTimeInt64() method returns, as an integer, the Julian Day Number multiplied by 86400000 (the number of milliseconds in a 24-hour day).  
SQLite will use the xCurrentTimeInt64() method to get the current date and time if that method is available (if iVersion is 2 or greater and the function pointer is not NULL) and will fall back to xCurrentTime() if xCurrentTimeInt64() is unavailable.

The xSetSystemCall(), xGetSystemCall(), and xNestSystemCall() interfaces are not used by the SQLite core.  These optional interfaces are provided by some VFSes to facilitate testing of the VFS code. By overriding system calls with functions under its control, a test program can simulate faults and error conditions that would otherwise be difficult or impossible to induce.  The set of system calls that can be overridden varies from one VFS to another, and from one version of the same VFS to the next.  Applications that use these interfaces must be prepared for any or all of these interfaces to be NULL or for their behavior to change from one release to the next.  Applications must not attempt to access any of these methods if the iVersion of the VFS is less than 3.

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
