# Pointer

## 指针与简单变量

```c
/* ptr.c 指针的基本使用 */
#include <stdio.h>

int var = 1;

int *ptr;

int main(void)
{
    ptr = &var;

    /* 直接访问和间接访问 */
    printf("\nDirect access, var = %d", var);
    printf("\nIndirect access, var = %d", *ptr);

    /* 显示变量var内存地址的两种方法 */
    printf("\n\nThe address of var = %p", &var);
    printf("\nThe address of var = %p", ptr);
    
    return 0;
}
```

## 指针与变量类型

## 指针与数组

数组下标和指针的关系：

```c
*(array) == array[0] 
*(array + 1) == array[1] 
*(array + 2) == array[2]
```

```c
/* passing.c 将数组传递到函数 */
#include <stdio.h>

#define MAX 10

int array[MAX], count;

int largest(int num_array[], int length);

int main(void)
{
    for (count = 0; count < MAX; count++)
    {
        printf("Enter an integer value: ");
        scanf("%p", &array[count]);
    }

    printf("\n\nLargest value = %d\n", largest(array, MAX));

    return 0
}

int largest(int num_array[], int length)
{
    int count, biggest = -12000;
    for ()
}
```

## Demonstrating the & and * Operators

## 空指针

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

## 常见错误