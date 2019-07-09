# Pointer

任何类型的变量都由内存地址和值组成，
所以指针型变量的值是另外一个变量/常量的内存地址。

```c
/* pointer_how_to.c */
int var = 10;       /* 假设其内存地址为 0x0012308 */

int *ptr = &var; 
    *ptr = 20;      /*  将变量 var 的值替换为 20 */

int **ptr = &ptr;   /* 双重指针 */
    **ptr = 30;     /* 将变量 var 的值替换为 30 */
```

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

## 指针的数学运算

```c
#include <stdio.h>

int main(void) 
{ 
	// Declare an array 
	int v[3] = {10, 100, 200}; 

	// Declare pointer variable 
	int *ptr; 

	// Assign the address of v[0] to ptr 
	ptr = v; 

	for (int i = 0; i < 3; i++) 
	{ 
		printf("Value of *ptr = %d\n", *ptr); 
		printf("Value of ptr = %p\n\n", ptr); 

		// Increment pointer ptr by 1 
		ptr++; 
        printf("after plus ptr is : %p\n\n", ptr);
	} 

    return 0;
}
```

输出结果为：

```shell
Value of *ptr = 10
Value of ptr = 0x7ffeec1ab90c

after plus ptr is : 0x7ffeec1ab910

Value of *ptr = 100
Value of ptr = 0x7ffeec1ab910

after plus ptr is : 0x7ffeec1ab914

Value of *ptr = 200
Value of ptr = 0x7ffeec1ab914

after plus ptr is : 0x7ffeec1ab918
```

## 双重指针（指针的指针）


## Demonstrating the & and * Operators

## 空(Void)指针

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

## Null指针

```c
#include <stdio.h> 
int main() 
{ 
int *i, *j; 
int *ii = NULL, *jj = NULL; 
if(i == j) 
{ 
printf("This might get printed if both i and j are same by chance."); 
} 
if(ii == jj) 
{ 
printf("This is always printed coz ii and jj are same."); 
} 
return 0; 
} 
``

## 函数指针

与普通指针相比较：

1. 函数指针指向的是代码而不是数值。通常函数指针存储可执行代码的开始部分。
2. 函数指针不会“分配”和“重新分配”内存。
3. 函数的名称也能用来获取函数的内存地址。
4. 函数指针可以拥有数组
5. 函数指针能够用于循环

示例1: `函数的名称也能用来获取函数的内存地址`

```c
// 移除 & 内存地址运算符，然后调用fun_ptr也掉了 * ，程序也正常运行
#include <stdio.h>

void fun(int a)
{
	printf("Value of a is %d\n", a);
}

int main()
{
	void (*fun_ptr)(int) = fun; // & removed

	fun_ptr(10); // * removed

	return 0;
}
```

示例2: `函数指针能够用于循环`

```c
// 选择0到2来执行不同的运算函数
#include <stdio.h>

void add(int a, int b)
{
	printf("Addition is %d\n", a+b);
}
void subtract(int a, int b)
{
	printf("Subtraction is %d\n", a-b);
}
void multiply(int a, int b)
{
	printf("Multiplication is %d\n", a*b);
}

int main()
{
	// fun_ptr_arr 是一个由函数指针构成的数组
	void (*fun_ptr_arr[])(int, int) = {add, subtract, multiply};
	unsigned int ch, a = 15, b = 10;

	printf("Enter Choice: 0 for add, 1 for subtract and 2 "
			"for multiply\n");
	scanf("%d", &ch);

	if (ch > 2) return 0;

	(*fun_ptr_arr[ch])(a, b);

	return 0;
}
```

## 申明指向函数的指针

错误的申明方式：

```c
/*
该语句表示，接受一个整数类型参数的函数foo，返回一个整数类型的指针变量。
因为 () 的运算优先级高于 *
 */
int * foo(int);
```

正确的声明方式：

```c
int (*foo)(int);
```

### 删除指向函数的指针

```c
#include <stdio.h>
// A normal function with an int parameter
// and void return type
void fun(int a)
{
	printf("Value of a is %d\n", a);
}

int main()
{
	// fun_ptr is a pointer to function fun() 
	void (*fun_ptr)(int) = &fun; 

	/* The above line is equivalent of following two 
	void (*fun_ptr)(int); 
	fun_ptr = &fun; 
	*/

	// Invoking fun() using fun_ptr 
	(*fun_ptr)(10); 

	return 0; 
} 
```

## 常见错误