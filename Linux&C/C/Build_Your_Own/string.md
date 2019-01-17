# 字符串处理

- 判断大小写

```c
#include <ctype.h>

upperstring(char *str)
{
char *p;

for(p = str; *p != '\0'; p++)
    {
    if(islower(*p))
		*p = toupper(*p);
	}
}

lowerstring(char *str)
{
char *p;

for(p = str; *p != '\0'; p++)
	{
	if(isupper(*p))
		*p = tolower(*p);
	}
}

```