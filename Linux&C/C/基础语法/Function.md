# 函数

```c
int add_together(int x, int y) {
  int result = x + y;
  return result;
}
```

```c
int added = add_together(10, 18);
```

## 静态函数

```c
static int fun(void)
{
  printf("I am a static function ");
}
```

使用场景：

- 需要限制对函数的访问，因为静态函数仅可在其被声明的文件中可以使用
- 需要在其他文件中防止使用重复相同名称的函数

```c
/* file1.c */
#include <stdio.h>

static void staticFunc(void)
{
  printf("Inside the static function staticFunc() ");
}

int main()
{
  staticFunc();
  return 0;
}
```

```c
/* file2.c */
#include <stdio.h>
int main()
{
  staticFunc();
  return 0;
}
```

如果编译`file2.c`，则得到错误，`undefined reference to staticFunc()`.
