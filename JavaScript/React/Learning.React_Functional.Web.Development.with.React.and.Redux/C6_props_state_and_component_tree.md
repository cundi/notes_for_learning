# Props

- 为什么要使用属性验证
- 如何使用`React.createClass`进行组件创建中实现它

组件树内的数据处理是使用React的一个关键优势。

## 属性验证

起因：
React组件提供了指定和验证属性类型的方法。

作用：
在提供不正确的属性类型时触发的告警信息极大的减少了调试应用所花费的时间。

React拥有6中内建的属性类型验证

|类型|验证器|
|:-:|:---:|
|数组|React.PropTypes.array|
|布尔值|React.PropTypes.bool|
|函数|React.PropTypes.func|
|数字|React.PropTypes.number|
|对象|React.PropTypes.object|
|字符串|React.PropTypes.string|

### 使用createClass验证Props

理解：
为什么组件属性类型验证如此重要

示例：
Summary组件从属性对象中解构ingredients,steps,以及title来构建需要显示的数据。
这里希望ingredients和steps为数组，所以使用Array.length计算数组的项。

```js
const Summary = createClass({
    displayname: "Summary",

    render() {
        const {ingredients, steps, title} = this.props

        return(
            <div className="summary">
                <h1>{title}</h1>
                <p>
                    <span>{ingredients.length} Ingredients</span> |
                    <span>{step.length} Steps</span>
                </p>
            </div>
        )
    }
})
```

如果我们给属性赋值为字符串，JS则会计算字符串中字符的长度。所以这里就体现出了组件属性类型验证重要性。

```js
render(
    <Summary title="Peanut Butter and Jelly"
               ingredients="peanut butter, jelly, bread"
               steps="spread peanut butter and jelly between bread" />,
      document.getElementById('react-container')
)
```

改进后的Summary组件，如下。

```js
const Summary = createClass({
    displayName: "Summary",
    propTypes: {
        ingredients: PropTypes.array,
        steps: PropTypes.array,
        title: PropTypes.string
    },

    render() {
        const {} = this.props;
        return (
                <div className="summary">
                    <h1>{title}</h1>
                    <p>
                        <span>{ingredients.length} Ingredients | </span>
                        <span>{steps.length} Steps</span>
                    </p>
                </div>
        )
    }
})
```

这样当我们传递不正确的属性类型时，Web控制台中可以看到错误。

渲染Summary组件不提供任何属性将所引发一个JS错误会将应用停止。

```js
render(
      <Summary />,
      document.getElementById('react-container')
)
```

避免这种错误发生，需要为React指定必传属性，以此便可以于应用停止之前在控制台显示所发生的错误。

```js
const Summary = createClass({
    displayName: "Summary", propTypes: {
    ingredients: PropTypes.array.isRequired,
    steps: PropTypes.array.isRequired,
    title: PropTypes.string
},
    render() {
        ...
    }
})
```

由于ingredients和steps仅用到了数组的length属性，而且这个组件实际上也不需要数组，所以这里可以重构为期望类型为数字。

```js
import { createClass, PropTypes } from 'react'

export const Summary = createClass({
    displayName: "Summary",
    propTypes: {
    ingredients: PropTypes.number.isRequired,
    steps: PropTypes.number.isRequired,
    title: PropTypes.string
}, render() {
    const {ingredients, steps, title} = this.props;
    return (
            <div className="summary">
                <h1>{title}</h1>
                <p>
                <span>{ingredients} Ingredients</span> | <span>{steps} Steps</span>
                </p>
            </div>
        );
    }
})
```



