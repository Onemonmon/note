### 1. 生命周期

```
挂载阶段
constructor
static getDerivedStateFromProps
render
componentDidMount
```

```
更新阶段
static getDerivedStateFromProps
shouldComponentUpdate
render
static getSnapshotBeforeUpdate
componentDidUpdate
```

```
销毁阶段
componentWillUnmount
```

### 2. setState

```jsx
this.setState({ ... })
this.setState((state, props) => ({ ... }))
在同步代码中，setState会将多个state合并处理，useState不会进行合并处理，都只执行一次render
在异步代码中，setState、useState都各自执行一次render
```

```
setState并不是单纯同步或异步的，在生命周期函数、合成事件中表现为异步，在setTimeout、DOM原生事件函数中表现为同步，在V18版本中统一变成异步了。
React内部通过isBatchingUpdates变量判断是否开启批量更新，在执行生命周期、合成事件时先将isBatchingUpdates置为true，此时setState不会直接更新，而是先进入队列等待批量更新，当函数执行完毕后再将isBatchingUpdates置为false，因为isBatchingUpdates的值是在同步代码中修改的，所以在异步代码中使用setState时，由于isBatchingUpdates早已置为false，导致批量更新失效，而setTimeout、DOM原生事件本身就不受React控制，所以也不会进行批量更新。
```

### 3. 组件通信

```
1. 父组件通过props向子组件传递数据
2. 子组件通过调用父组件传入的函数，向父组件传递数据
3. 使用context跨层级传递数据
4. redux
```

##### context

```jsx
const MyContext = React.createContext()
   
class A extend React.Component {
    static contextType = MyContext
    state = {
        count: 1
    }
    render() {
        return <MyContext.Provider value={this.state.count}>
        	<B />
            <C />
            <D />
        </MyContext.Provider>
    }
}
// 通过props接收context的值
function B(props) {
    return <div>{props.count}</div>
}
// 使用useContext接收值
function C() {
    const count = useContext(MyContext)
    return <div>{count}</div>
}
// 使用Consumer消费值
function D() {
    return <MyContext.Consumer>
        {count => (
        	<div>{count}</div>
        )}
    </MyContext.Consumer>
}
```

##### Redux

```javascript
// actions/count-action-types.js
export const INCREMENT = 'INCREMENT';
export const ASYNC_INCREMENT = 'ASYNC_INCREMENT';
export const DECREMENT = 'DECREMENT';
```

```javascript
// actions/count-action.js
import { INCREMENT, DECREMENT, ASYNC_INCREMENT } from 'actions/count-action-types.js'
export const increment = { type: INCREMENT }
export const decrement = { type: DECREMENT }
// 异步action
export const asyncIncrement = async (dispatch) => {
  const res = await xxx()
  dispatch({
    type: ASYNC_INCREMENT,
    payload: res.data
  })
}
```

```javascript
// reducers/count-reducer.js
import { INCREMENT, DECREMENT, ASYNC_INCREMENT } from 'actions/count-action-types.js'
export default function(state = { count: 1 }, action) {
    switch(action) {
        case INCREMENT:
            return { ...state, count: state.count + 1 }
        case DECREMENT:
            return { ...state, count: state.count - 1 }
        case ASYNC_INCREMENT:
            return { ...state, count: state.count + action.payload }
        default:
            return state
    }
}
```

```javascript
// store.js
import { createStore, combineReducers, compose, applyMiddleware } from 'redux'
import countReducer from 'reducers/count-reducer.js'
import thunk from 'react-thunk'
const store = createStore(countReducer)
// 如果有多个reducer
const store = createStore(combineReducers({reducerA, reducerB}))
// 如果需要异步action
const store = createStore(
    combineReducers({reducerA, reducerB}), 
    compose(applyMiddleware(...[thunk]))
)
export default store
```

```jsx
// App.tsx
import store from 'store.js'
ReactDOM.render((
    <Provider store={store}>
        <UseRedux />
    </Provider>
), document.getElementById('root'));

// UseRedux.tsx
import { connect } from 'react-redux'
import { increment, decrement, asyncIncrement } 'actions/count-action.js'
const UseRedux = ({count, dispatch}) => (
	<>
    	<button onClick={() => dispatch(increment)}>自增</button>
    	<button onClick={() => dispatch(decrement)}>自减</button>
    	<button onClick={() => dispatch(asyncIncrement)}>异步增加一个值</button>
    	<div>{count}</div>
    </>
)
export default connect(
    (state) => ({
    	count: state.count
	}),
)(UseRedux)
```

### 4. 函数组件

```
为什么会出现hooks？
在类组件中，想用复用状态逻辑很难，复杂的类组件变得难以理解，还需要理解class面向对象的编程思维及其this的工作方式。
```

```
函数组件与类组件的区别？
1. 类组件是面向对象编程，函数组件是函数式编程
2. 类组件需要创建并保存实例，占用内存
3. 函数组件有值捕获的特性
4. 类组件有自己的状态，函数组件要通过useState
5. 类组件有完整的生命周期，函数组件通过useEffect可模拟生命周期
6. 类组件的逻辑复用依靠继承，函数组件通过hooks
7. 类组件使用shouldComponentUpdate、PureComponent避免重新渲染，函数组件使用React.memo
```

```
函数组件值的捕获？
实现点击一个按钮，三秒后弹出一个受控文本值，但在三秒内对该文本进行修改后，函数组件得到的依然是此次渲染的值（修改前的值），而类组件得到的是最新的值（修改后的值）。
函数组件要获得修改后的值：使用useRef保存值
类组件要获得修改前的值：
  1. 点击时先获取值 const { value } = this.state 
  2. 写在render中（闭包）
```

```
useEffect(() => {
	...
	// 组件卸载时执行
	return ...
}, [...])
useEffect在渲染之后执行
useLayoutEffect在提交阶段执行，会阻塞渲染，但不会触发额外的回流、重绘
```

```
当某一个值的计算开销较大时使用useMemo
当一个组件的渲染开销较大时，可以对其函数props使用useCallback，对其引用类型props使用useMemo，因为当父组件更新后，该组件都会进行不必要的高消耗的渲染
```

```jsx
const myRef = useRef()
<div ref={myRef}></div>

this.myRef = React.createRef()
<div ref={myRef}></div>

<div ref={ref => this.myRef = ref}></div>
```

### 5. 事件合成

```
在JSX上绑定的事件如onClick，不会直接绑定在真实的dom上，而是使用事件委托的形式绑定在document（17版本绑定在container）上
在JSX上绑定的事件可能会对应多个原生事件，如：onChange对应blur、focus、change、input等
采用这种模式可以统一管理事件、抹平浏览器差异
```

### 6. DIFF算法

```
1. 在Web UI中跨层级操作dom节点的情况极少，可以忽略不计
2. 相同类的组件会生成相似的树结构，不同类的组件会生成不同的树结构
3. 针对同一层级下的子节点，可以通过添加唯一标识进行区分
针对以上三点，react分别对tree diff、component diff、element diff算法进行优化
> tree diff：只针对同层级的节点进行比较，如果节点不存在则直接删除该节点及其子节点，如果存在将节点移动到其他层级的节点下时，react不会进行移动，而是创建新节点，删除老节点，所以应该尽量保持dom结构稳定。
> component diff：对于不同类的组件，react会直接替换整个组件，对于相同类型则按原策略比较dom tree，还提供了shouldComponentUpdate。
> element diff：通过为同一层级下的子节点添加唯一标识，当节点不变，只是位置改变时，设置lastIndex=0，遍历新的子节点，index为节点原来所在的序号，如果lastIndex<index则节点位置不变，否则移动节点，然后lastIndex=index，所以尽量避免将节点从末尾移动到头部的操作，这样除了末尾节点位置不变，其他节点都需要进行移动。
```

### 7. React Fiber

```
16版本以前，React使用递归的方式创建虚拟dom，递归过程无法中断，如果组件树的结构复杂、层级较深，使得递归时间超过浏览器一帧的时间（16ms），则会使得用户交互时卡顿。
16版本后使用React Fiber，用链表模拟了原先递归的函数调用栈，将之前需要递归处理的事情分解成一个个执行单元，每个节点都是一个执行单元，用Fiber表示（return 指向父节点，child指向第一个子节点，sibling指向下一个兄弟节点）
```

```
协调阶段：会执行diff算法找出所有节点的变更（副作用），并为每个更新的节点生成对应的更新任务，将更新任务加入任务队列等待scheduler模块调度，scheduler模块实现了跨平台兼容的requestIdleCallback，该阶段可以中断、挂起、恢复，因此该阶段中如果存在副作用将可能重复触发。
constructor、getDerivedStateFromProps、shouldComponentUpdate、render

提交阶段：会将协调阶段处理的所有变更（副作用）一次性更新到dom上，同步执行，不可中断
componentDidMount、getSnapShotBeforeUpdate、componentDidUpdate
```

```
调度过程：
首先React向浏览器请求调度，浏览器在一帧的内容执行完成后还有剩余时间，会去判断是否有待执行任务，没有的话就继续执行下一帧，有则将控制权交给React，让其执行任务，当执行完一个任务后若还有剩余时间，且还有待执行任务，则会继续执行下一个任务，没有剩余时间了则会归还控制权，当协调阶段的任务没在当前时间片内执行完则会被中断，等待下一次调度时再执行。
```



