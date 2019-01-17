# CGI

CGI普通应用所具有的特性：

REQUEST_METHOD The specific request that the client used to fetch the page (that is, to invoke the CGI program): usually GET or POST.
PATH_INFO Any extra path information that appeared after the program name in the URL that invoked this CGI program.
PATH_TRANSLATED The path information from PATH_INFO, with any ``virtual-to-physical mapping'' performed on it.
SCRIPT_NAME The actual name of the CGI program that is running, as a URL, in case the generated page needs to reference itself.
QUERY_STRING The query being performed by the user. In the case of form processing, the QUERY_STRING variable contains encoded information about each field filled in by the user. (We'll have much more to say about QUERY_STRING in a bit.)
REMOTE_HOST The hostname of the client (if known).
CONTENT_TYPE The HTTP/MIME content type of the attached information, for queries (such as POST) with attached information.
CONTENT_LENGTH The size of the attached information, if any.

- HTTP_USER_AGENT An indication of the name and version of the client browser in use.

- HTTP_ACCEPT The specification by the client browser of which document types it will accept.

HTML页面

```html
```

## GET

cgi应用

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

extern char *cgigetval(char *);

main()
{
char *browser;
char *edittype;
char *text;

printf("Content-Type: text/html\n\n");

printf("<html>\n");
printf("<head>\n");
printf("<title>CGI test result</title>\n");
printf("</head>\n");
printf("<body>\n");

browser = getenv("HTTP_USER_AGENT");

printf("<p>\n");
printf("Hello, ");
if(browser != NULL && strstr(browser, "Lynx") != NULL)
	printf("Lynx");
else if(browser != NULL && strstr(browser, "Mosaic") != NULL)
	printf("Mosaic");
else if(browser != NULL && strstr(browser, "Mozilla") != NULL)
	printf("Netscape");
else	printf("unknown browser");
printf(" user!\n");

printf("<p>\n");

printf("Here is your modified text:\n");
printf("<br>\n");

text = cgigetval("textfield");
edittype = cgigetval("edittype");

if(text == NULL)
	{
	printf("You didn't enter any text!\n");
	}
else if(edittype != NULL && strcmp(edittype, "reverse") == 0)
	{
	reverse(text);
	printf("%s\n", text);
	}
else if(edittype != NULL && strcmp(edittype, "upper") == 0)
	{
	upperstring(text);
	printf("%s\n", text);
	}
else if(edittype != NULL && strcmp(edittype, "lower") == 0)
	{
	lowerstring(text);
	printf("%s\n", text);
	}
else	{
	printf("You didn't select a transformation!\n");
	}

printf("</body>\n");
printf("</html>\n");
}
```