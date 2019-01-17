# VM in c

引用：https://blog.felixangell.com/virtual-machine-in-c/

## 前置要求

编译器：gcc/clang
文本编辑器：
基本编程知识：

## 为什么要用c编写虚拟机

## 指令集

## 虚拟机的工作原理

“指令循环”，即，获取，解码，执行。

## 项目结构

```shell
$ cd ~/dev/
$ mkdir mac
$ cd mac
$ mkdir src
```

## Makefile

makefile相对的简单很多，不需要写到多个文件中，也不需要引用其他头文件。这里仅定义一些编译标识。

```makefile
SRC_FILES = main.c
CC_FLAGS = -Wall -Wextra -g -std=c11
CC = clang

all:
    ${CC} ${SRC_FILES} ${CC_FLAGS} -o mac
```

这个文件稍后还会继续改进的。

## 程序指令

```c
typedef enum {
    PSH,
    ADD,
    POP,
    SET,
    HLT
} InstructionSet;

const int program[] = {
    PSH, 5,
    PSH, 6,
    ADD,
    POP,
    HLT
};
```

### 获取当前指令

```c
#include <stdbool.h> 

bool running = true;

int main() {
    while (running) {
       int x = fetch();
       if (x == HLT) running = false;
       ip++;
    }
}
```

### 计算指令



