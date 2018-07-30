
# Redux-saga

示例：

```jsx
const { connect } = dva;
const { Router, Route } = dva.router;

const delay = timeout => {
  return new Promise(resolve => {
    setTimeout(resolve, timeout);
  });
};

// 1. Initialize
const app = dva();

// 2. Model
app.model({
  namespace: 'count',
  state: 0,
  effects: {
    *add(action, { put, call }) {
      yield call(delay, 1000);
      yield put({ type: 'minus' });
    },
  },
  reducers: {
    add(count) { return count + 1 },
    minus(count) { return count - 1 },
  },
});

// 3. View
const App = connect(({ count }) => ({
  count
}))(function(props) {
  return (
    <div>
      <h2>{ props.count }</h2>
      <button key="add" onClick={() => { props.dispatch({type: 'count/add'})}}>+</button>
      <button key="minus" onClick={() => { props.dispatch({type: 'count/minus'})}}>-</button>
    </div>
  );
});

// 4. Router
app.router(({ history }) =>
  <Router history={history}>
    <Route path="/" component={App} />
  </Router>
);

// 5. Start
app.start('#root');
```

核心概念
State：一个对象，保存整个应用状态
View：React 组件构成的视图层
Action：一个对象，描述事件
connect 方法：一个函数，绑定 State 到 View，第一个参数是 mapStateToProps 函数，mapStateToProps 函数会返回一个对象，用于建立 State 到 Props 的映射关系。
dispatch 方法：一个函数，发送 Action 到 State，被 connect 的 Component 会自动在 props 中拥有 dispatch 方法。

8个概念

- State
- Action: 操作事件，可以是同步，也可以是异步。Action 是一个普通 javascript 对象，它是改变 State 的唯一途径。无论是从 UI 事件、网络回调，还是 WebSocket 等数据源所获得的数据，最终都会通过 dispatch 函数调用一个 action，从而改变对应的数据。
- Model
- Reducer： 是唯一可以更新 state 的地方，接收参数 state 和 action，返回结果是当前 model 的 state 对象。注意：Reducer函数必须是纯函数。
- Effect
- Subscription
- Router
- RouteComponent

同步操作：Reducers --> State.
异步操作：Effects --> Reducers --> State

## dva-loading

loading的入口文件位于项目根目录下的index.js, 这样任何一个 dva 的 routes 组件中就都会有一个 loading 对象

```js
import createLoading from 'dva-loading';

const app = dva();

app.use(createLoading());
```

loading的使用：

```js
export default connect(({ app, loading }) => ({ app, loading }))(App);
```

loading对象的组成：

```js
loading: {
  global: false,
  models: {app: false},
  effects: {app: false}
}
```



## 视图

## 模型

1. State：整个应用的数据层。

2. namespace: model state在全局state中所用到的key。

- put(action):

创建一条 Effect 描述信息，指示 middleware 发起一个 action 到 Store。