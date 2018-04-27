# Component

实现组件的两种方法

- React.createClass
- React.Component

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