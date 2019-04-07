# React Design Pattern

## Table of Contents

- [函数组件(Functional Component)](#函数组件functional-component) 
- [ Render Props](#render-props) 
- [高阶组件(HOC)](#高阶组件hoc) 
- [组合组件(Compound Components)](#组合组件compound-components) 
- [提供者模式(Provider Pattern)](#提供者模式provider-pattern) 
- [ State Reducer](#state-reducer) 
- [Controlled Components](#controlled-components) 
- [Hook](#hook) 



## 函数组件(Functional Component)

函数组件是纯 UI 组件，也称作傻瓜组件, 或者无状态组件。渲染所需要的数据只通过 props 传入, 不需要用 class 的方式来创建 React 组件, 也不需要用到 this 关键字，或者用到 state

函数组件具有以下优点

- 可读性好
- 逻辑简单
- 测试简单
- 代码量少
- 容易复用
- 解耦

```jsx
const Hello = props => <div>Hello {props.name}</div>;

const App = props => <Hello name={"world"} />;
```

什么情况下不使用函数组件？ 如果你需要用到 react 生命周期， 需要用到 state, 函数组件就没办法满足要求了。 但是新的 Hook 特性出来之后又大大提升了函数组件的应用范围， 所以没有最佳的设计模式，都是要因地制宜。

## Render Props

给某个组件通过 props 传递一个函数，并且函数会返回一个 React 组件，这就是 render props.

```jsx
const Hello = props => <div>{props.render({ name: "World" })}</div>;

const App = props => <Hello render={props => <h1>Hello {props.name}</h1>} />;
```

你也可以把函数放在组件 tag 的中间，组件可以通过`props.children` 获取

```jsx
const Hello = props => <div>{props.children({ name: "World" })}</div>;

const App = props => <Hello>{props => <h1>Hello {props.name}</h1>}</Hello>;

//复用
const App2 = props => <Hello>{props => <h1>Hey {props.name}</h1>}</Hello>;
```

render props 提高了组件的复用性和灵活性， 相比组件直接渲染特定模板，通过 render props，父组件自定义需要的模板，子组件只要调用父组件提供的函数， 传入需要的数据就可以了。

## 高阶组件(HOC)

高阶组件是一个接受 Component 并返回新的 Component 的函数。之所以叫高阶组件是因为它和函数式编程中的高阶函数类似， 一个接受函数为参数, 或者返回值为函数的函数便称作高阶函数.

```jsx
const Name = props => <span>{props.children}</span>;

const reverse = Component => {
  return ({ children, ...props }) => (
    <Component {...props}>
      {children
        .split("")
        .reverse()
        .join("")}
    </Component>
  );
};

const ReversedName = reverse(Name);

const App = props => <ReversedName>hello world</ReversedName>;
```

reverse 函数接受一个 Component 参数，返回一个可以 reverse 内容的新的函数式组件。这个高阶组件封装了 reverse 方法，以后每次需要 reverse 某些组件的内容就没必要重复写一下步骤：

```jsx
const Name = props => <span>{props.children}</span>;

const App = props => (
  <Name>
    {"hello world"
      .split("")
      .reverse()
      .join("")}
  </Name>
);
```

## 组合组件(Compound Components)

组合组件设计模式一般应用在一些共享组件上。 如 `<select>` 和`<option>`, `<Tab>` 和`<TabItem>`等，通过组合组件，使用者只需要传递子组件，子组件所需要的`props`在父组件会封装好，引用子组件的时候就没必要传递所有`props`了。 比如下面的例子，每个 ListItem 需要一个`index` 参数来显示第几项， 可以使用下面的方式渲染

```jsx
const List = ({ children }) => <ul>{children}</ul>;

const ListItem = ({ children, index }) => (
  <li>
    {index} {children}
  </li>
);

const App = props => (
  <List>
    <ListItem index={1}>apple</ListItem>
    <ListItem index={2}>banana</ListItem>
  </List>
);
```

这种方式存在一些问题， 每次新增加一个列表项都要手动传一个`index`值，如果以后要加其他的属性， 就需要每一项都修改，组合组件可以避免上述的缺陷。

```jsx
const List = ({children}) => (
    <ul>
        {React.Children.map(children, (child, index) => React.cloneElement(child, {
            index: index
        }))}
    </ul>
)

const ListItem = ({children, index}) => (
    <li>{index} {children}</li>
)

<List>
      <ListItem>apple</ListItem>
      <ListItem>banana</ListItem>
</List>
```

如果把 ListItem 通过`static`方式放在 List 组件里面，更具语义化。

```jsx
class List extends Component {
  static Item = ({ children, index }) => (
    <li>
      {index} {children}
    </li>
  );

  render() {
    return (
      <ul>
        {React.Children.map(this.props.children, (child, index) =>
          React.cloneElement(child, {
            index: index
          })
        )}
      </ul>
    );
  }
}

const App = props => (
  <List>
    <List.Item>apple</List.Item>
    <List.Item>banana</List.Item>
  </List>
);
```

## 提供者模式(Provider Pattern)

提供者模式可以解决非父子组件下的信息传递问题, 或者组件层级太深需要层层传递的问题

```jsx
const Child = ({ lastName }) => <p>{lastName}</p>;

const Mother = ({ lastName }) => <Child lastName={lastName} />;

const GrandMother = ({ lastName }) => <Mother lastName={lastName} />;

const App = props => <GrandMother lastName={"Kieffer"} />;
```

上面的例子`lastName`需要在每个组件都传递一次，提供者模式就可以避免这种 **Prop Drilling** 的写法

```jsx
const FamilyContext = React.createContext({});
const FamilyProvider = FamilyContext.Provider;
const FamilyConsumer = FamilyContext.Consumer;

const Child = ({ lastName }) => (
  <FamilyConsumer>{context => <p>{context}</p>}</FamilyConsumer>
);

const Mother = () => <Child />;

const GrandMother = () => <Mother />;

const App = props => (
  <FamilyProvider value={"Kieffer"}>
    <GrandMother />
  </FamilyProvider>
);
```



## State Reducer

State Reducer可以让父组件控制子组件state。render props 可以控制子组件的UI是如何渲染的，state reducer则可以控制子组件的state.

下面的例子，通过传入state reducer方法，父组件可以控制子组件最多只点击4次。

```jsx
class Counter extends Component{
  state = {
    count: 0
  }

  setInternalState = (stateOrFunc, callback) => {
    this.setState(state => {
      const changedState = typeof stateOrFunc === 'function'
          ? stateOrFunc(state)
          : stateOrFunc

      const reducedState = this.props.stateReducer(state, changedState) || {}

      // return null when reducedState is an empty object, prevent rerendering
      return Object.keys(reducedState).length > 0
          ? reducedState
          : null
    }, callback)
  }

  addCount = () => this.setInternalState(state => ({count: state.count + 1}))

  render() {
    return (
        <div>
          <p>You clicked {this.state.count} times</p>
          <button onClick={this.addCount}>
            Click me
          </button>
        </div>
    );
  }
}

const App = props => {
  const stateReducer = (state, change) => {
    if (state.count >= 4) {
      return {}
    }
    return change
  }
  return <Counter stateReducer={stateReducer}/>
}
```



## Controlled Components

Controlled Components将原来子组件改变state的逻辑移到父组件中，由父组件控制。一个运用Controlled Components最普遍的例子就是`<input/>`，不传任何`props`的情况下React可以直接用默认的`<input/>`组件，但是如果加了`value` 属性，如果不传`onChange`属性开始输入的话, input框不会有任何变化, 因为`<input/>`已经被父组件控制, 需要父组件指定一个方法, 告诉子组件`value`如何变化.

```jsx
class App extends Component{
  state = {
    value: '',
  }

  onChange = (e) => {
    this.setState({value: e.target.value})
  }

  render() {
    return (
          <input value={this.state.value} onChange={this.onChange}/>
    );
  }
}
```

下面是一个实际的例子, 如果给`Counter`组件传入`count`属性的话, `Counter`组件自己的`addCount`就不再起作用, 需要用户传入一个新的`addCount`方法来决定`count`如何变化.

```jsx
class Counter extends Component{
  state = {
    count: 0
  }

  isControlled(prop) {
    return this.props[prop] !== undefined
  }

  getState() {
    return {
      count: this.isControlled('count') ? this.props.count : this.state.count,
    }
  }

  addCount = () => {
    if (this.isControlled('count')) {
      this.props.addCount()
    } else {
      this.setState(state => ({count: state.count + 1}))
    }
  }

  render() {
    return (
        <div>
          <p>You clicked {this.getState().count} times</p>
          <button onClick={() => this.addCount()}>
            Click me
          </button>
        </div>
    );
  }
}

class App extends Component{
  state = {
    count: 0,
  }

  addCount = () => {
    this.setState(state => ({count: state.count + 2}))
  }

  render() {
    return (
        <Fragment>
          {/*this counter add 1 every time*/}
          <Counter/>
          {/*this counter add 2 every time*/}
          <Counter count={this.state.count} addCount={this.addCount}/>
        </Fragment>
    );
  }
}
```



## Hook

Hook 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数，用户可以在不使用`class`的情况下用一些 React 的特性，如`state`等等.

`useState` 就是一个 *Hook* 。`useState` 唯一的参数就是初始 state，通过在函数组件里调用它来给组件添加一些内部 state。React 会在重复渲染时保留这个 state。`useState` 会返回一对值：**当前**状态和一个让你更新它的函数，你可以在事件处理函数中或其他一些地方调用这个函数。它类似 class 组件的 `this.setState`，但是它不会把新的 state 和旧的 state 进行合并。



你之前可能已经在 React 组件中执行过数据获取、订阅或者手动修改过 DOM。我们统一把这些操作称为“副作用”，或者简称为“作用”。

`useEffect` 就是一个 Effect Hook，给函数组件增加了操作副作用的能力。它跟 class 组件中的 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 具有相同的用途，只不过被合并成了一个 API。



`useContext`则可以传入`Context`对象，获取当前的context value. 当相应的`Context.Provider`更新value, `useContext`会触发rerender, 并传入最新的context value

```jsx
import React, { useState, useEffect, useContext } from 'react';

const TitleContext = React.createContext({})
const TitleProvider = TitleContext.Provider

function Counter() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  useEffect(() => {
    // Update the document title using the browser API
    console.log(`You clicked ${count} times`)
  });

  const contextTitle = useContext(TitleContext)
  return (
      <div>
        <h1>{contextTitle}</h1>
        <p>You clicked {count} times</p>
        <button onClick={() => setCount(count + 1)}>
          Click me
        </button>
      </div>
  );
}

const App = props => (
    <TitleProvider value={'Counter'}>
      <Counter/>
    </TitleProvider>
)
```



以下是React所有的hook:

- [Basic Hooks](https://zh-hans.reactjs.org/docs/hooks-reference.html#basic-hooks)
  - [`useState`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usestate)
  - [`useEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useeffect)
  - [`useContext`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecontext)
- [Additional Hooks](https://zh-hans.reactjs.org/docs/hooks-reference.html#additional-hooks)
  - [`useReducer`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usereducer)
  - [`useCallback`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usecallback)
  - [`useMemo`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usememo)
  - [`useRef`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useref)
  - [`useImperativeHandle`](https://zh-hans.reactjs.org/docs/hooks-reference.html#useimperativehandle)
  - [`useLayoutEffect`](https://zh-hans.reactjs.org/docs/hooks-reference.html#uselayouteffect)
  - [`useDebugValue`](https://zh-hans.reactjs.org/docs/hooks-reference.html#usedebugvalue)



参考:

<https://medium.com/@soorajchandran/introduction-to-higher-order-components-hoc-in-react-383c9343a3aa>

https://blog.logrocket.com/guide-to-react-compound-components-9c4b3eb482e9

<https://engineering.dollarshaveclub.com/reacts-state-reducer-pattern-f66e82a82697>

<https://zh-hans.reactjs.org/docs/hooks-overview.html>

<https://medium.com/yazanaabed/advanced-react-patterns-7326f5a5ad1b>

