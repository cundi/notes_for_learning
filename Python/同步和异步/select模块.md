# select－I/O完成等待

该模块提供了三个函数：

1. select()
2. epoll()
3. devpoll()

适用场景：鼓励使用高级的selectors模块，除非你想精细化控制基本系统层面的使用。