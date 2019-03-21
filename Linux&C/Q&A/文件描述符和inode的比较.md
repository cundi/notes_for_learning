# 文件描述符和inode之间的区别

引用：`https://stackoverflow.com/questions/25819226/what-is-the-difference-between-inode-number-and-file-descriptor`

注：文件描述符以下简称为，FD。

An inode is an artifact of a particular file-system and how it manages indirection. A "traditional *ix" file-system uses this to link together files into directories, and even multiple parts of a file together. That is, an inode represents a physical manifestation of the file-system implementation.

On the other hand, a file descriptor is an opaque identifier to an open file by the Kernel. As long as the file remains open that identifier can be used to perform operations such as reading and writing. The usage of "file" here is not to be confused with a general "file on a disk" - rather a file in this context represents a stream and operations which can be performed upon it, regardless of the source.

A file descriptor is not related to an inode, except as such may be used internally by particular [file-system] driver.

File descriptors are also isolated across processes, but can be inherited by children/forks.

相同之处：

- 都可以抽象术语”文件“称之

区别：

- inode是表示文件系统的文件，FD是open函数返回的一个整数。

内核为每个进程使用一个数组来表示打开的文件