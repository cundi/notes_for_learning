# Array

An array is defined as finite ordered collection of homogenous data, stored in contiguous memory locations.

- finite: means data range must be defined.
- ordered: means data must be stored in continuous memory addresses.
- homogenous: means data must be of similar data type.

- 有限：数据组必须是定义过的。
- 有序：数据必须存储在一段连续的内存地址上。
- 同质：数据必须是相似数据类型。

## Declaring a Array 声明数组

```c
/* data-type variable-name[size]; */
int arr[10];
```

## Initialization of an Array 数组的初始化

After an array is declared it must be initialized. Otherwise, it will contain garbage value(any random value). An array can be initialized at either compile time or at runtime.

- 编译时：用户生命变量之后就赋值

```c
#include<stdio.h>

void main()
{
    int i;
    int arr[] = {2, 3, 4};      /*Compile time array initialization*/
    for(i = 0 ; i < 3 ; i++)
    {
        printf("%d\t",arr[i]);
    }
}
```

- 运行时：从scanf()获取用户输入。

```c
#include<stdio.h>

void main()
{
    int arr[4];
    int i, j;
    printf("Enter array element");
    for(i = 0; i < 4; i++)
    {
        scanf("%d", &arr[i]);    //Run time array initialization
    }
    for(j = 0; j < 4; j++)
    {
        printf("%d\n", arr[j]);
    }
}
```

不指定数组大小

```c
int array[] = { 100, 200, 300, 400 };
```

指定大小

```c
int array[4] = { 100, 200, 300, 400 };
```

部分初始化

```c
int array[10] = { 1, 2, 3 };
```

## Two dimensional Arrays 二位数组

C language supports multidimensional arrays also. The simplest form of a multidimensional array is the two-dimensional array. Both the row's and column's index begins from 0.

C语言支持多维数组，最简单的多维数组是二维数组。


```c
int array[4][3] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12 };
```

```c
int array[4][3] = { { 1, 2, 3 } , { 4, 5, 6 } , { 7, 8, 9 } , { 10, 11, 12 } };
```

```c
/*
arraysize.c 比较默认数据类型和所对应数组所占字节的不同 
*/
#include <stdio.h>

/* 声明多个含有100个元素对数组 */
int intarray[100];
float floatarray[100];
double doublearray[100];

int main(void)
{

}
```