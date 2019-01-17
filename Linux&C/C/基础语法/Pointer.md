# Pointer

## Demonstrating the & and * Operators

## C中的空指针

定义：没有关联数据类型的指针就是空指针

```c
int a = 10;
char b = 'x';

void *p = &a;  // void pointer holds address of int 'a'
p = &b; // void pointer holds address of char 'b'
```

优点：

- malloc()\calloc()返回void *类型

```c
int main(void)
{
    // Note that malloc() returns void * which can be 
    // typecasted to any type like int *, char *, ..
    int *x = malloc(sizeof(int) * n);
}
```

- C的空指针可以用来实现通用函数

```c

```

相关事实：

- 空指针是不能够被引用的。

```c
#include<stdio.h>
int main()
{
    int a = 10;
    void *ptr = &a;
    printf("%d", *ptr);
    return 0;
}
```

输出，`Compiler Error: 'void*' is not a pointer-to-object type`。

然后下面的程序却可以编译和运行

```c
#include<stdio.h>
int main()
{
    int a = 10;
    void *ptr = &a;
    printf("%d", *(int *)ptr);
    return 0;
}
```

- 标准C中是不允许指针和空指针的数学计算。但是在GNU C中却是允许的，而且把

```c
#include<stdio.h>
int main()
{
    int a[2] = {1, 2};
    void *ptr = &a;
    ptr = ptr + sizeof(int);
    printf("%d", *(int *)ptr);
    return 0;
}
```