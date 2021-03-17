# yum stuck forever
https://unix.stackexchange.com/questions/77061/yum-update-stuck-forever

yum命令调不起，只能kill -9结束。

solution is:

```
yum clean all
rpm --rebuilddb
```
