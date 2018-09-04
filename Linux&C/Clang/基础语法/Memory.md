# Memory

三种内存模型：

1. Automatic
2. Static
3. Manual

- Automatic

默认所有函数内部的变量都是自动，而无需使用static关键字

- Static

在程序的生命周期中静态变量的内存地址是不变的。数组的大小在启动时是固定的，但是数组中的值是可变的，所以数组并不是完全静态的。

在函数外部（单文件范围之内）和内部使用了static关键字声明的变量都属于静态内存类型。

如果没有声明静态变量，那么所有变量都会被初始化为零或者NUll。

- Manual

手动分配的内存类型涉及malloc和free。而它也是数组在声明后唯一的可以改变大小的内存类型。

## 变量声明持久化

示例：状态机生成的斐波那契序列

```c
# include  <stdio.h>

long long int fibonacci(){
    static long long int first = 0;
    static long long int second = 1;
    long long int out = first+second;
    first=second;
    second=out;
    return out;
}

int main(){
    for (int i=0; i< 50; i++)
    printf("%lli\n", fibonacci());
}
```

静态变量拥有本地作用域。

### Frame

### Stack and Heap

