# increment and decrement

|运算符|示例表达式|说明|
|:--:|:-------:|:—:|
|++|++a|加1，然后使用新的值
|++|a++|

示例一：

```c
#include <stdio.h>

int main(void)
{
    int c;

    // postincrement
    c = 5;
    printf("%d\n", c); // print 5
    printf("%d\n", c++); // print 5 then postincrement
    printf("%d\n\n", c); // print 6

    // preincrement
    c = 5;
    printf("%d\n", c); // print 5
    printf("%d\n", ++c);  //preincrement then print 6
    printf("%d\n", c); // print 6

}
```

常见错误：

对表达式进行自增或者自减，例如，++(x+1)