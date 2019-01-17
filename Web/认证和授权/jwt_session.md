# jwt & session

引用：

http://blog.didispace.com/learn-how-to-use-jwt-xjf/
http://blog.didispace.com/user-authentication-with-jwt/
https://segmentfault.com/q/1010000002449556
https://zhuanlan.zhihu.com/p/22531819?refer=lsxiao
https://vimsky.com/article/3601.html
https://www.jianshu.com/p/78e15a1ac7f2

## 特性对比

|特性|Session|JWT|
|:-:|:----:|:--:|
|安全性|需要考虑CSRF攻击|无需考虑|
|存储|需要俩端都存储|客户端存储即可|
|可控性|服务端可随时修改权限....|只能等待Token过期|

## 存储方式比较

Session方式存储用户id的最大弊病在于要占用大量服务器内存，对于较大型应用而言可能还要保存许多的状态。一般而言，大型应用还需要借助一些KV数据库和一系列缓存机制来实现Session的存储。

而JWT方式将用户状态分散到了客户端中，可以明显减轻服务端的内存压力。除了用户id之外，还可以存储其他的和用户相关的信息，例如该用户是否是管理员、用户所在的分桶

## 应用场景

## 注销和修改密码

## 续签