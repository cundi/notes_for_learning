# 变量和数据类型

|类型|说明|示例|
|:-:|:--:|:--:|
void|Empty Type||
char|Single Character/Byte|char last_initial = 'H';|
int|Integer|int age = 23;|
long|Integer that can hold larger values|long age_of_universe = 13798000000;|
float|Decimal Number|float liters_per_pint = 0.568f;|
double|Decimal Number with more precision|double speed_of_swallow = 0.01072896;|

类型:

- 基本：整数型和浮点型
- 枚举：
- void：
- 衍生：指针，数组，Struct， Union， 函数

## void

定义：表示无值可用

- 函数返回void
- void作为函数参数
- 

## 示例

- 各数据类型所需的字节大小

```c
/*
sizeof.c
*/
#include <stdio.h>

int main(void)
{
    printf("\nA char is %d bytes", sizeof(char));
    printf("\nA int is %d bytes", sizeof(int));
    printf("\nA short is %d bytes", sizeof(short));

    return 0;
}
```