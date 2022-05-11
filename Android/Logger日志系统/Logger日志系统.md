# Logger日志系统

## 1. 简介

Android提供的Logger日志系统是**基于内核中的Logger日志驱动程序**实现的，它将日志记录保存在内核空间中。为了有效的利用内存空间，Logger日志驱动程序内部使用一**个环形缓冲区**来保存日志。因此，当Logger日志驱动程序中的环形缓冲区写满之后，新的日志就会覆盖就的日志。

Logger日志驱动程序根据日志的类型以及日志的输出量来对日志进行分类，避免重要的日志被覆盖。日志一共有四种类型，其中**main是应用级别的日志**，而**system是系统级别的日志**。

- main - /dev/log/main 		应用级别日志
- system - /dev/log/system  系统级别日志
- radio - /dev/log/radio         无线设备相关日志
- events - /dev/log/events    诊断系统问题日志

Android系统在应用程序框架中提供了`android.util.Log`等接口来向Logger日志驱动程序中写入日志，Android系统在运行时库中也提供了三组C/C++的宏来向Logger日志驱动程序中写入日志，如`LOGV`等。无论是Java日志写入有接口还是native日志写入接口，它们最终都是通过运行时库层的日志库liblog来向Logger日志驱动程序中写入日志的。此外，系统还提供了一个Logcat工具来读取和显示Logger日志驱动程序中的日志。

<img src="F:\Projects\Markdowns\pngs\logger_arch.png" alt="Logger Architecture" style="zoom: 50%;" />



## 2. Logger日志格式

Logger日志一共划分为main/system/radio/events四种类型，其中，前三种类型的日志格式是相同的，而第四种类型events的日志格式稍有区别。

<img src="F:\Projects\Markdowns\pngs\logger_entry.png" style="zoom:50%;" />

其中，priority表示日志的优先级，一个整数。tag表示日志的标签，一个字符串。msg表示日志的内容。

## 3. Logger日志驱动程序

Logger日志驱动程序的实现在系统的内核空间中，主要有logger.h/logger.c两个源文件组成。

### 3.1 基础数据结构

Logger日志驱动程序主要使用到了logger_entry、logger_log和logger_reader三个结构体。

```c
# struct logger_entry
// kernel/goldfish/drivers/staging/android/logger.h
struct logger_entry 
{
	__u16	len;
     __u16	__pad;
     __s32	pid;
     __s32	tid;
     __s32	sec;
     __s32	nsec;
     char		msg[0];
};

#define LOGGER_ENTRY_MAX_LEN	(4*1024)
#define LOGGER_ENTRY_MAX_PAYLOAD	\
	(LOGGER_ENTRY_MAX_LEN - sizeof(struct logger_entry))
```

结构体**logger_entry用来描述一条日志记录**。每一个日志记录的最大程度为4K，其中，日志记录的有效负载长度为4K减去结构体logger_entry的大小。成员变量`len`表示实际的日志记录的有效负载长度。成员变量`msg`记录的是实际写入的日志记录内容，它的长度是可变的，由成员变量`len`决定。



```c
#struct loggger_log
// kernel/goldfish/drivers/staging/android/logger.c
/*
 * struct logger_log - represents a specific log, such as 'main' or 'radio'
 *
 * This structure lives from module insertion until module removal, so it does
 * not need additional reference counting. The structure is protected by the
 * mutex 'mutex'.
 */
struct logger_log 
{
	unsigned char 		*buffer;	/* the ring buffer itself */
	struct miscdevice	misc;	/* misc device representing the log */
	wait_queue_head_t	wq;		/* wait queue for readers */
	struct list_head	readers; 	/* this log's readers */
	struct mutex		mutex;	/* mutex protecting buffer */
	size_t			w_off;	/* current write head offset */
	size_t			head;	/* new readers start here */
	size_t			size;	/* size of the log */
};
```

结构体logger_log用来描述一个日志缓冲区。在Logger日志驱动程序中，每一种类型的日志都对应有一个单独的日志缓冲区。成员变量**`buffer`指向一个内核缓冲区，用来保存日志记录**，它是循环使用的，大小由成员变量`size`决定；成员变量`misc`的类型为miscdevice，用来描述一个日志设备。成员变量`wq`是一个等待队列，用来记录那些**等待读取日志记录的进程**；成员变量`readers`是一个列表，用来记录**正在读取日志记录的进程**，每一个进程都使用一个结构体logger_reader来描述；成员变量**mutex是一个互斥量**，**用来保护日志缓冲区的并发访问**。成员变量`w_off`表示下一条要写入的日志记录在缓冲区的位置；成员变量`head`表示当一个新的进程读取日志时，它应该从日志缓冲区中的什么位置开始读取。



```c
/*
 * struct logger_reader - a logging device open for reading
 *
 * This object lives from open to release, so we don't need additional
 * reference counting. The structure is protected by log->mutex.
 */
struct logger_reader 
{
	struct logger_log	*log;	/* associated log */
	struct list_head	list;	/* entry in logger_log's list */
	size_t			r_off;	/* current read head offset */
};
```

结构体logger_reader用来**描述一个正在读取某一个日志缓冲区的日志记录的进程**。变量`log`指向要读取的日志缓冲区结构体；成员变量`list`是一个列表项，用来**连接所有读取同一种类型日志记录的进程**；变量`r_off`表示当前进程要**读取的下一条日志记录在日志缓冲区的位置**。