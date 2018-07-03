# Working with Class-Based Views and Hyperlinked APIs in Django

- 使用模型序列化器消除重复代码
- 使用包装器编写API视图
- 使用默认的解析和渲染选项
- 浏览API
- 设计和PostgreSQL数据库交互的复杂RESTful API
- 理解每个HTTP方法所执行任务
- 使用模型申明关系
- 使用关系和超链接来管理序列化和反序列化
- 使用通用类创建类视图
- 对API使用端点
- 创建和重新取回资源

## 使用模型序列化器消除重复代码

- 这里的Serializer和ModelSerializer相当于Form和ModelForm
- ModelSerializer自动生成默认字段集合，以及默认验证器集合，此外该类还提供了create和update方法的默认实现

```python
# gamesapi/games
from rest_framework import serializers
from games.models import Game


class GameSerializer(serializers.ModelSerializer):
    # Meta声明了两个属性，model和fields
    class Meta:
        model = Game
        fields = ('id',
                  'name',
                  'release_date',
                  'game_category',
                  'played')
```

## 使用包装器编写API视图

- 对函数视图使用rest_framwork.decorators中的api_view装饰器能够，在用户发出服务端未提供HTTP 动词时，返回错误代码`405 Mehtod Not Allowed`。
- api_view装饰器将函数视图包装到了rest_framework.view.APIView类的子类

## 使用默认的解析和渲染选项

- 在使用装饰器时，默认的解析器类和默认的渲染器类都会关联到函数视图。

```python
# gameapi/settings.py
# DEFAULT_PARSER_CLASSES的默认值
(
    'rest_framework.parsers.JSONParser',
    'rest_framework.parsers.FormParser',
    'rest_framework.parsers.MultiPartParser'
)
```

- api_view装饰器为request.data提供了下面的属性，以便通过合适的解析器处理以下内容类型的请求

```shell
application/json
application/x-www-form-urlencoded
multipart/form-data
```

- 编辑gamesapi/games/views.py文件，将JSONResponse类替换为api_view装饰器

```python
from rest_framework.parsers import JSONParser
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from games.models import Game
from games.serializers import GameSerializer


@api_view(['GET', 'POST']) # 修改后的结果
def game_list(request):
    if request.method == 'GET':
        games = Game.objects.all()
        games_serializer = GameSerializer(games, many=True)
        return Response(games_serializer.data)

    elif request.method == 'POST':
        game_serializer = GameSerializer(data=request.data) # 修改后的结果
        if game_serializer.is_valid():
            game_serializer.save()
            return Response(game_serializer.data, status=status.HTTP_201_CREATED) # 修改后的结果
        return Response(game_serializer.errors, status=status.HTTP_400_BAD_REQUEST) # 修改后的结果


@api_view(['GET', 'PUT', 'POST']) # 修改后的结果
def game_detail(request, pk):
    try:
        game = Game.objects.get(pk=pk)
    except Game.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND) # 修改后的结果

    if request.method == 'GET':
        game_serializer = GameSerializer(game)
        return Response(game_serializer.data) # 修改后的结果

    elif request.method == 'PUT':
        game_serializer = GameSerializer(game, data=request.data) # 修改后的结果
        if game_serializer.is_valid():
            game_serializer.save()
            return Response(game_serializer.data) # 修改后的结果
        return Response(game_serializer.errors, status=status.HTTP_400_BAD_REQUEST) # 修改后的结果

    elif request.method == 'DELETE':
        game.delete()
        return Response(status=status.HTTP_204_NO_CONTENT) # 修改后的结果
```

- 使用httpie进行接口测试

```shell
http OPTIONS :8000/games/
http OPTIONS :8000/games/3/
http -f POST :8000/games/ name='Toy Story 4' game_category='3D RPG' played=false release_date='2016-05-18T03:02:00.776594Z'
http PUT :8000/games/
```

## 浏览API

- Django REST框架提供了使用Bootstrap构建的HTML响应，即可浏览的API
- 该响应包括：以JSON显示的资源内容、执行不同请求的按钮、提交数据到资源的表单

```shell
http://localhost:8000/games/
```

## 设计和复杂PostgresSQL数据库交互的RESTful风格API

列举资源和模型名称：

- 游戏分类（GameCategory模型）
- 游戏（Game模型）
- 玩家（Player模型）
- 玩家分数（PlayerScore模型）

|模型|字段|
|:-:|:--:|
|游戏分类|名称|
|游戏|外键（指向游戏目录GameCategory）、名称、发布日期、游戏是否被玩过的布尔值标记、游戏插入数据库的时间戳|
|玩家|性别、名字、游戏插入数据库的时间戳|
|玩家分数|玩家外键（Player）、游戏外键（Game）、分数、玩家分数的生成日期|

## 理解HTTP方法所执行的任务

HTTP动词枚举表:

|HTTP动词|范围|语义|
|:-----:|:--:|:--:|
|GET|游戏分类的集合|以集合方式重新取回所有存储的游戏分类，并按名称升序排列。每个游戏分类必须包括游戏对应分类的URL列表|
|GET|游戏分类|重新取回单个游戏分类。游戏分类必须包含一个游戏资源对应分类的URL列表|
|POST|游戏分类的集合|在集合中创建一个新的游戏分类|
|PUT|游戏分类|更新一个已有游戏分类|
|PATCH|游戏分类|更新已有游戏分类的一个或者多个字段|
|DELETE|游戏分类|删除一个已有游戏分类|
|GET|游戏集合|以集合方式重新取回所有存储的游戏，按名称升序排列。每个游戏必须包括自己的游戏分类描述|

HTTP动词枚举表的对应URI:

- 游戏分类集合：/game-categories/
- 游戏分类：/game-category/{id}
- 游戏集合：/games/
- 游戏：/game/{id}
- 玩家集合：/players/
- 玩家：/player/{id}
- 分数集合：/player-scores/
- 分数：/player-score/{id}

## 声明模型间的关系

```python
# games/models.py
from django.db import models


class GameCategory(models.Model):
    name = models.CharField(max_length=200)

    class Meta:
        ordering = ('name',)

    def __str__(self):
        return self.name


class Game(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    name = models.CharField(max_length=200)
    # 游戏的外键
    game_category = models.ForeignKey(
        GameCategory,
        related_name='games',
        on_delete=models.CASCADE)
    release_date = models.DateTimeField()
    played = models.BooleanField(default=False)

    class Meta:
        ordering = ('name',)

    def __str__(self):
        return self.name


class Player(models.Model):
    MALE = 'M'
    FEMALE = 'F'
    GENDER_CHOICES = (
        (MALE, 'Male'),
        (FEMALE, 'Female'),
    )
    created = models.DateTimeField(auto_now_add=True)
    name = models.CharField(max_length=50, blank=False, default='')
    gender = models.CharField(
        max_length=2,
        choices=GENDER_CHOICES,
        default=MALE,
    )

    class Meta:
        ordering = ('name',)

    def __str__(self):
        return self.name


class PlayerScore(models.Model):
    # 关联到玩家模型的外键
    player = models.ForeignKey(
        Player,
        related_name='scores',
        on_delete=models.CASCADE)
    game = models.ForeignKey(
        Game,
        on_delete=models.CASCADE)
    score = models.IntegerField()
    score_date = models.DateTimeField()

    class Meta:
        # Order by score descending
        ordering = ('-score',)
```