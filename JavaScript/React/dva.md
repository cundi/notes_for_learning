
# DVA

示例, DVA的5个API：


```jsx
import dva, { connect } from 'dva';

// 1. Create app
const app = dva();

// 2. Add plugins (optionally)
app.use(plugin);

// 3. Register models
app.model(model);

// 4. Connect components and models
const App = connect(mapStateToProps)(Component);

// 5. Config router with Components
app.router(routes);

// 6. Start app
app.start('#root');

```

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

官方推荐的项目目录结构：

```shell
├── /mock/           # 数据mock的接口文件
├── /src/            # 项目源码目录
│ ├── /components/   # 项目组件
│ ├── /routes/       # 路由组件（页面维度）
│ ├── /models/       # 数据模型
│ ├── /services/     # 数据接口
│ ├── /utils/        # 工具函数
│ ├── route.js       # 路由配置
│ ├── index.js       # 入口文件
│ ├── index.less    
│ └── index.html    
├── package.json     # 定义依赖的pkg文件
└── proxy.config.js  # 数据mock配置文件
```

核心概念
State：对象，保存整个应用状态
View：React 组件构成的视图层
Action：对象，描述事件
connect 方法：函数，绑定 State 到 View，第一个参数是 mapStateToProps 函数，mapStateToProps 函数会返回一个对象，用于建立 State 到 Props 的映射关系。
dispatch 方法：函数，发送 Action 到 State，被 connect 的 Component 会自动在 props 中拥有 dispatch 方法。

8个概念

- State
- Action: 操作事件，可以是同步，也可以是异步。Action 是一个普通 javascript 对象，它是改变 State 的唯一途径。无论是从 UI 事件、网络回调，还是 WebSocket 等数据源所获得的数据，最终都会通过 dispatch 函数调用一个 action，从而改变对应的数据。
- Model
- Reducer： 是唯一可以更新 state 的地方，接收参数 state 和 action，返回结果是当前 model 的 state 对象。注意：Reducer函数必须是纯函数。
- Effect: 异步操作, 不直接修改state，由actions触发，也可以调用actions。(action, { put, call, select })
- Subscription：异步的只读操作，并不直接修改state，可以调用actions。({ dispatch, history }：
- Router
- RouteComponent

同步操作：Reducers --> State.
异步操作：Effects --> Reducers --> State


架构：首先我们根据 url 访问相关的 Route-Component，在组件中我们通过 dispatch 发送 action 到 model 里面的 effect 或者直接 Reducer

当我们将action发送给Effect，基本上是取服务器上面请求数据的，服务器返回数据之后，effect 会发送相应的 action 给 reducer，由唯一能改变 state 的 reducer 改变 state ，然后通过connect重新渲染组件。

当我们将action发送给reducer，那直接由 reducer 改变 state，然后通过 connect 重新渲染组件。


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



## 路由
 
Router 表示路由配置信息，项目中的 router.js。

## 模型

Model:　dva 中最重要的概念，Model 非 MVC 中的 M，而是领域模型，用于把数据相关的逻辑聚合到一起，几乎所有的数据，逻辑都在这边进行处理分发。

1. State：整个应用的数据层。优先级比初始化的低

2. namespace: model 的命名空间，即全局state中所用到的key，只能用字符串，我们发送在发送 action 到相应的 reducer 时，就会需要用到 namespace 。

3. reducers：以key/value 格式定义 reducer，用于处理同步操作，唯一可以修改  state 的地方。由 action 触发。注意：Reducer函数必须是一个纯函数。通过 actions 中传入的值，与当前 reducers 中的值进行运算获得新的值（也就是新的 state）。

4. effects：处理异步操作和业务逻辑，不直接修改 state，简单的来说，就是获取从服务端获取数据，并且发起一个 action 交给 reducer 的地方。

- put(action):

创建一条 Effect 描述信息，指示 middleware 发起一个 action 到 Store。
effects中的put(action)等同于subscriptions中的dispatch(action)，它们的作用都是分发一个action。

```jsx
yield put({ type: actionType, payload: attachedData, error: errorIfHave});
dispatch({ type: actionType, payload: attachedData, error: errorIfHave});
```

- call(asyncFunction):

调用一个异步函数

```jsx
const result = yield call(api.fetch, { page: 1 });
```

- select(function):

从全局状态中选择数据

```jsx
const count = yield select(state => state.count);
```

5. Subscription: 用于订阅一个数据源，格式为 ({ dispatch, history }) => unsubscribe ，　会根据需要 dispatch 相应的 action。在 app.start() 时被执行，数据源可以是当前的时间、当前页面的url、服务器的 websocket 连接、history 路由变化等等。
