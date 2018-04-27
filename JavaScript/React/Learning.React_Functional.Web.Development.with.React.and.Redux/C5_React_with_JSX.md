# CHAPTER 5 React with JSX

React元素：
JSX：即JavaScript extension

创建React元素的两种方法：

- React.createElement
- factories

本章目的：如何使用JSX构建包含React元素的虚拟DOM

## React元素即JSX

### JSX提示

#### 嵌套组件

JSX允许组件拥有子组件。

示例：在IngredientsList中多次渲染称作Ingredient的组件。

```jsx

<IngredientsList>
    <Ingredient />
    <Ingredient />
    <Ingredient />
</IngredientsList>
```

#### className

为了避免和JS的保留关键字`class`发生冲突，所以使用className作为CSS类名称。

```html
<h1 className="fancy">Baked Salmon</h1>
```

#### JS表达式

JS表达式：将变量包裹在大括号内并计算返回值。

#### 求值

两个大括号中间的JS会被求值，所以字符串拼接和加法会被运算，函数也会被调用。

示例：

```js
<h1>{"Hello" + this.props.title}</h1>
<h1>{this.props.title.toLowerCase().replace}</h1>
function appendTitle({this.props.title}) { console.log(`${this.props.title} is great!`)
}
```

#### 映射数组到JSX

因为JSX就是JS，所以可以在JS函数中直接使用JSX。

示例：讲一个数组映射为JSX元素

```js
<ul>
    {this.props.ingredients.map((ingredient, i) => 
        <li key={i}>{ingredient}</li>
    )}
</ul>
```

JSX虽然看起来整洁、可读，但是浏览器无法解释，所以需要Babel这样的工具。

## Babel

缘起：不是所有的浏览器都支持ES6、ES7语法，同时也没有任何浏览器支持JSX语法。

使用方法：

- 直接在浏览器中使用babel-core链接，并在脚本代码块中应用text/babel类型。不适用于生产环境。

示例：

```html
<!DOCTYPE html>
<html>
  <head>
<meta charset="utf-8">
<title>React Examples</title> </head>
    <body>
    <div class="react-container"></div>
        <!-- React Library & React DOM -->
        <script src="https://unpkg.com/react@15.4.2/dist/react.js"></script>
        <script src="https://unpkg.com/react-dom@15.4.2/dist/react-dom.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.29/browser.js"> </script>
        <script type="text/babel">
        // JSX code here. Or link to separate JavaScript file that contains JSX.
        </script>
    </body>
</html>

```

## Recipes as JSX

代码在被浏览器解释之前，需要从JSX转换为纯净的React。

示例：食谱数组

```js
var data=[ 
{
    "name": "Baked Salmon",
    "ingredients": [
      { "name": "Salmon", "amount": 1, "measurement": "l lb" },
      { "name": "Pine Nuts", "amount": 1, "measurement": "cup" },
      { "name": "Butter Lettuce", "amount": 2, "measurement": "cups" },
      { "name": "Yellow Squash", "amount": 1, "measurement": "med" },
      { "name": "Olive Oil", "amount": 0.5, "measurement": "cup" },
      { "name": "Garlic", "amount": 3, "measurement": "cloves" }
], "steps": [
      "Preheat the oven to 350 degrees.",
      "Spread the olive oil around a glass baking dish.",
      "Add the salmon, garlic, and pine nuts to the dish.",
      "Bake for 15 minutes.",
      "Add the yellow squash and put back in the oven for 30 mins.",
      "Remove from oven and let cool for 15 minutes. Add the lettuce and serve."
]}, {
    "name": "Fish Tacos",
    "ingredients": [
      { "name": "Whitefish", "amount": 1, "measurement": "l lb" },
      { "name": "Cheese", "amount": 1, "measurement": "cup" },
      { "name": "Iceberg Lettuce", "amount": 2, "measurement": "cups" },
      { "name": "Tomatoes", "amount": 2, "measurement": "large"},
      { "name": "Tortillas", "amount": 3, "measurement": "med" }
], "steps": [
      "Cook the fish on the grill until hot.",
      "Place the fish on the 3 tortillas.",
      "Top them with lettuce, tomatoes, and cheese."
] }
];
```

Menu组件：列出食谱
Recipe组件：描述每个食谱的UI


示例：食谱应用的代码结构

```js
// The data, an array of Recipe objects
vardata=[...];
// A stateless functional component for an individual Recipe
const Recipe = (props) => ( ...
)
// A stateless functional component for the Menu of Recipes
const Menu = (props) => ( ...
)
// A call to ReactDOM.render to render our Menu into the current DOM
ReactDOM.render(
    <Menu recipes={data} title="Delicious Recipes" />,
    document.getElementById("react-container")
)
```





## Webpack

