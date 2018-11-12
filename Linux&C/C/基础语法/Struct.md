# Struct

1. 分号结尾
2. The tag name can be declared with the structure definition or even later in the main function.

声明：

```c
struct student   /*definition of structure*/
{
    int roll;                                          /*members*/
    char name [10];
    float per;
}s1,s2;                            /*objects or variables of the structure student*/
```

使用：

```c
struct student
{
    int roll;
    char grade;
    int marks;
};

main()
{
    struct student s1={2, 'a', 89};
    printf("Address of roll=%u", &s1.roll);
    printf("Address of grade=%u", &s1.name);
    printf("Address of marks=%u", &s1.marks);
}
```

声明：

```c
typedef struct {
  float x;
  float y;
} point;
```

使用：

```c
point p;
p.x = 0.1;
p.y = 10.0;

float length = sqrt(p.x * p.x + p.y * p.y);
```