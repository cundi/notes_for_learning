# Error

## TypeError 

### can't pickle memoryview objects

http://docs.celeryproject.org/en/latest/userguide/configuration.html#broker-url

uses librabbitmq if installed or falls back to pyamqp

如果没有安装rabbitmq的Clang客户端，则须将BROKER_URL的amqp改为pyamqp

## PENDING

be cautions, dont make a typo, 

Reason:
1. 调用的任务名称输入错误