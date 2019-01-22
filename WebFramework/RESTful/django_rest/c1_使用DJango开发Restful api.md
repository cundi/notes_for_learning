# Developing RESTful APIs with　Django

本章主线：使用Ｄjango和Django REST框架创建一个能够对SQLite数据库执行CRUD操作的REST风格的Web API。

小节分布：

- 设计和SQLite数据库交互的REST风格API
- 通过HTTP方法理解任务的执行
- 为Django REST设置虚拟环境
- 创建数据库模型
- 管理序列化和反序列化的数据
- 编写API视图
- 使用命令行工具对API发出HTTP请求
- 使用图形化工具合成并发送HTTP请求

## 设计和SQLite数据库交互的REST风格API

应用场景：

- 我们需要为一个手机应用提供REST风格的API进行游戏的CRUD操作
- 不想要花时间选择合适的ORM
- 只想尽可能快的完成交互API

需求实现：

- 游戏的属性:

整数标识符，名称，发布日期，分类描述，指明游戏是否被玩过的布尔值，新游戏加入数据的时间戳

- 第一版API:

|HTTP动词|作用域|语义描述|
|:-----:|:---:|:-----:|
|GET|游戏集合|重新取回所有集合中存储的游戏，并按照游戏名称升序排列|
|GET|游戏|重新取回单个游戏|
|POST|游戏集合|在集合中创建一个新游戏|
|PUT|游戏|更新一个已存在的游戏|
|DELETE|游戏|删除一个已存在的游戏|

## 通过HTTP方法理解任务的执行

## 为Django REST设置虚拟环境

- 安装基础模块

```shell
pip install django
pip install djangorestframework
```

- 创建新项目

```python
django-admin.py startproject gameapi
```

- 启动应用

```shell
cd gameapi && python manage.py startapp games
```

- 应用配置

```python
# gameapi/games/app.py
from django.apps import AppConfig


class GamesConfig(AppConfig):
    name = 'games'
```

- 引用Django REST框架和应用的配置

```python
# gamesapi/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 加入rest框架
    'rest_framework',
    # 应用game的配置
    'games.apps.GamesConfig',
]
```

## 创建数据库模型

- 在项目的`__init__.py`使用MySQLdb

```python
import pymysql
pymysql.install_as_MySQLdb()
```

- 连接数据库(Postgresql)

```shell
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'mytest',
        'USER': 'root',
        'PASSWORD': '',
        'HOST': '127.0.0.1',
        'PORT': '3306',
    }
}
```

- 创建模型Game来表示和持久化游戏属性。

```python
# games/models.py
from django.db import models


class Game(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    name = models.CharField(max_length=200, blank=True, default='')
    release_date = models.DateTimeField()
    game_category = models.CharField(max_length=200, blank=True, default='')
    played = models.BooleanField(default=False)

    class Meta:
        ordering = ('name',)
```

- Game模型的迁移初始化

```shell
python manage.py makemigrations games
```

- 执行模型执行

 ```shell
 python manage.py migrate
 ```

## 序列化和反序列化的管理

序列化：收到HTTP请求之时，Django REST框架在模型实例和Python原始数据类型之间起到了传递者的作用。

反序列化：返回HTTP响应时，

通过创建rest_framework.serializers.Serializer的子类来申明字段以及必要方法，以此来管理序列化和反序列化。

```python
# gamesapi/games/serializers.py
from rest_framework import serializers
from games.models import Game


class GameSerializer(serializers.Serializer):
    pk = serializers.IntegerField(read_only=True)
    name = serializers.CharField(max_length=200)
    release_date = serializers.DateTimeField()
    game_category = serializers.CharField(max_length=200)
    played = serializers.BooleanField(required=False)

    # 在调用从超类继承来的save方法时，使用者必须在子类中重写两个方法create和update
    # 否则在实例话时会抛出NotImplementedError异常

    def create(self, validated_data):
        return Game.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.name = validated_data.get('name', instance.name)
        instance.release_date = validated_data.get('release_date', instance.release_date)
        instance.game_category = validated_data.get('game_category', instance.game_category)
        instance.played = validated_data.get('played', instance.played)
        instance.save()
        return instance
```

- 序列化测试脚本

```python
# gameapi/serializers_test_01.py
from datetime import datetime
from django.utils import timezone
from django.utils.six import BytesIO
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from games.models import Game
from games.serializers import GameSerializer


gamedatetime = timezone.make_aware(datetime.now(), timezone.get_current_timezone())
game1 = Game(name='Smurfs Jungle', release_date=gamedatetime, game_category='2D mobile arcade', played=False)
game1.save()
game2 = Game(name='Angry Birds RPG', release_date=gamedatetime, game_category='3D RPG', played=False)
game2.save()

print(game1.pk)
print(game1.name)
print(game1.created)
print(game2.pk)
print(game2.name)
print(game2.created)

#  生成Python原生数据类型字典
game_serializer1 = GameSerializer(game1)
print(game_serializer1.data)
game_serializer2 = GameSerializer(game2)
print(game_serializer2.data)

# 对Python数据做序列化
renderer = JSONRenderer()
rendered_game1 = renderer.render(game_serializer1.data)
rendered_game2 = renderer.render(game_serializer2.data)
print(rendered_game1)
print(rendered_game2)

# 下面是反序列化
json_string_for_new_game = '{"name":"Tomb Raider Extreme Edition","release_date":"2016-05-18T03:02:00.776594Z","game_category":"3D RPG","played":false}'
json_bytes_for_new_game = bytes(json_string_for_new_game , encoding="UTF-8")
stream_for_new_game = BytesIO(json_bytes_for_new_game)
parser = JSONParser()
parsed_new_game = parser.parse(stream_for_new_game)
print(parsed_new_game)

# 在使用data创建序列化器时，尝试访问序列化器必须调用 is_valid
new_game_serializer = GameSerializer(data=parsed_new_game)
if new_game_serializer.is_valid():
    new_game = new_game_serializer.save()
    print(new_game.name)
```

## 编写API视图

- 函数视图

```python
# games/views.py
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from rest_framework import status
from games.models import Game
from games.serializers import GameSerializer


class JSONResponse(HttpResponse):
    def __init__(self, data, **kwargs):
        content = JSONRenderer().render(data)
        kwargs['content_type'] = 'application/json'
        super(JSONResponse, self).__init__(content, **kwargs)


# 为视图设置CSRF cookie
@csrf_exempt
def game_list(request):
    if request.method == 'GET':
        games = Game.objects.all()
        games_serializer = GameSerializer(games, many=True)
        return JSONResponse(games_serializer.data)

    elif request.method == 'POST':
        game_data = JSONParser().parse(request)
        game_serializer = GameSerializer(data=game_data)
        if game_serializer.is_valid():
            game_serializer.save()
            return JSONResponse(game_serializer.data, status=status.HTTP_201_CREATED)
        return JSONResponse(game_serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@csrf_exempt
def game_detail(request, pk):
    try:
        game = Game.objects.get(pk=pk)
    except Game.DoesNotExist:
        return HttpResponse(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        game_serializer = GameSerializer(game)
        return JSONResponse(game_serializer.data)

    elif request.method == 'PUT':
        game_data = JSONParser().parse(request)
        game_serializer = GameSerializer(game, data=game_data)
        if game_serializer.is_valid():
            game_serializer.save()
            return JSONResponse(game_serializer.data)
        return JSONResponse(game_serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        game.delete()
        return HttpResponse(status=status.HTTP_204_NO_CONTENT)
```

- 项目的URL

```python
# games/urls.py
from django.conf.urls import url, include

urlpatterns = [
    url(r'^', include('games.urls')),
]
```

- 应用的URL

```python
#  gamesapi/urls.py
from django.conf.urls import url
from games import views

urlpatterns = [
    url(r'^games/$', views.game_list),
    url(r'^games/(?P<pk>[0-9]+)/$', views.game_detail),
]
```

- 启动开发服务器

```shell
python manage.py runserver
```

## 使用命令行工具对API发出HTTP请求

- curl和httpie

```shell
curl -X GET :8000/games/
```

- 安装httpie

```python
pip install httpie
```