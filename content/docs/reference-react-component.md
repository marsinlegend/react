---
id: react-component
title: React.Component
layout: docs
category: Reference
permalink: docs/react-component.html
redirect_from:
  - "docs/component-api.html"
  - "docs/component-specs.html"
  - "docs/component-specs-ko-KR.html"
  - "docs/component-specs-zh-CN.html"
  - "tips/UNSAFE_componentWillReceiveProps-not-triggered-after-mounting.html"
  - "tips/dom-event-listeners.html"
  - "tips/initial-ajax.html"
  - "tips/use-react-with-other-libraries.html"
---

本页面包含详细的React组件类定义的API引用。假定你熟悉基本React概念，such as [Components and Props](/docs/components-and-props.html), as well as [State and Lifecycle](/docs/state-and-lifecycle.html). If you're not, read them first.

## 概览

定义React组件可以以类或者函数的方式。定义为类的组件目前提供更多特征，这一页将详细描述这些特征。为定义一个React组件类，你需要继承`React.Component`：

```js
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

在`React.Component`子类中，你*必须*定义的唯一方法被叫做[`render()`](#render)。在本页面描述的所有其他方法都是可选的。

**我们强烈反对你自己创建组件的基类。** In React components, [代码重用主要通过组合而非继承达成](/docs/composition-vs-inheritance.html)。

> 注意：
>
>React doesn't force you to use the ES6 class syntax. If you prefer to avoid it, you may use the `create-react-class` module or a similar custom abstraction instead. Take a look at [Using React without ES6](/docs/react-without-es6.html) to learn more.

### 组件生命周期

每一个组件都有几个“生命周期方法”，你可以重写(override)他们，以在进程中的特定时期运行代码。**You can use [this lifecycle diagram](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/) as a cheat sheet.** In the list below, commonly used lifecycle methods are marked as **bold**. The rest of them exist for relatively rare use cases.

#### 装载

这些方法会按照下列顺序，在组件实例被创建并插入DOM中时被调用：

- [**`constructor()`**](#constructor)
- [`static getDerivedStateFromProps()`](#static-getderivedstatefromprops)
- [**`render()`**](#render)
- [**`componentDidMount()`**](#componentdidmount)

>Note:
>
>这些代码是遗留的，你应该 [避免他们](/blog/2018/03/27/update-on-async-rendering.html) 被用在新代码中：
>
>- [`UNSAFE_componentWillMount()`](#unsafe_componentwillmount)

#### 更新

一次更新是由改变属性或状态引起的。当一个组件在被重新渲染时，这些方法将按照下列顺序被调用：

- [`static getDerivedStateFromProps()`](#static-getderivedstatefromprops)
- [`shouldComponentUpdate()`](#shouldcomponentupdate)
- [**`render()`**](#render)
- [`getSnapshotBeforeUpdate()`](#getsnapshotbeforeupdate)
- [**`componentDidUpdate()`**](#componentdidupdate)

>Note:
>
>这些代码是遗留的， and you should [avoid them](/blog/2018/03/27/update-on-async-rendering.html) in new code:
>
>- [`UNSAFE_componentWillUpdate()`](#unsafe_componentwillupdate)
>- [`UNSAFE_componentWillReceiveProps()`](#unsafe_componentwillreceiveprops)

#### 卸载

当一个组件被从DOM中移除时，该方法被调用：

- [`componentWillUnmount()`](#componentwillunmount)

#### 错误处理

当任何一个子组件在渲染过程中、在一个生命周期方法中、或在构造函数中发生错误时，这些方法会被调用。

- [`static getDerivedStateFromError()`](#static-getderivedstatefromerror)
- [`componentDidCatch()`](#componentdidcatch)

### 其他API

每一个组件还提供了其他的API：

  - [`setState()`](#setstate)
  - [`forceUpdate()`](#forceupdate)

### 类属性

  - [`defaultProps`](#defaultprops)
  - [`displayName`](#displayname)

### 实例属性

  - [`props`](#props)
  - [`state`](#state)

* * *

## 参考

### 普遍被使用的生命周期方法

The methods in this section cover the vast majority of use cases you'll encounter creating React components. **For a visual reference, check out [this lifecycle diagram](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/).**

### `render()`

```javascript
render()
```

`render()`方法是类组件唯一必须的方法。

当被调用时，其应该检查`this.props` 和 `this.state`并返回以下类型之一：

- **React元素。**通常是由 [JSX](/docs/introducing-jsx.html) 创建。例如，`<div />` 和 `<MyComponent />` 是 React 元素，指示 React 渲染一个 DOM 节点，或是另一个用户定义的组件，各自分别地。 
- **数组和fragments。**让你从渲染中返回多个元素。See the documentation on [fragments](/docs/fragments.html) for more details.
- **Portals。**让你渲染孩子们到一个不同的DOM子树。See the documentation on [portals](/docs/portals.html) for more details.
- **字符串和数字。**这些将被渲染为 DOM 中的文本节点。
- **布尔或`null`。**什么都不渲染。（通常存在于 `return test && <Child />`模式，其中 `test` 是布尔值。）

`render()`函数应该是纯的，意味着不应该改变组件的状态，其每次调用都应返回相同的结果，同时它不会直接和浏览器交互。

若需要和浏览器交互，将任务放在`componentDidMount()`中或其他的生命周期方法中。保持`render()` 方法纯净使得组件更容易理解。

> 注意
>
> 若 [`shouldComponentUpdate()`](#shouldcomponentupdate)返回false，`render()`函数将不会被调用。

* * *

### `constructor()`

```javascript
constructor(props)
```

**如果你不初始化状态，也不绑定方法，那么你就不需要为React组件实现构造函数。**

React组件的构造函数将会在其装载之前被调用。当为一个`React.Component`子类定义构造函数时，你应该在任何其他的表达式之前调用`super(props)`。否则，`this.props`在构造函数中将是未定义的，这会导致臭虫。

典型地, 在 React 中构造函数只用于两个目的：

* 初始化[局部状态](/docs/state-and-lifecycle.html)，通过赋值一个对象到`this.state`。
* 绑定[事件处理器](/docs/handling-events.html)方法到一个实例。

在 `constructor()`中，你**不能调用 `setState()`** 。正确做法是，如果你的组件需要使用局部状态， 直接在构造函数中**将初始状态赋值给`this.state`**：

```js
constructor(props) {
  super(props);
  // Don't call this.setState() here!
  this.state = { counter: 0 };
  this.handleClick = this.handleClick.bind(this);
}
```

构造函数是你唯一可以直接赋值`this.state`的地方。在其他所有方法中，你需要使用`this.setState()`来代替。

避免引入任何副作用或者是订阅到构造函数中。对于这些用例，使用`componentDidMount()`来代替。

> 注意
>
> **避免拷贝属性(props)到状态！This is a common mistake:**
>
>```js
>constructor(props) {
>  super(props);
>  // Don't do this!
>  this.state = { color: props.color };
>}
>```
>
>这里的问题，一是不必要的(你可以直接使用 `this.props.color` 来代替)，二是创造臭虫 (更新属性 `color` 不会反映到状态中)。
>
>**此模式仅用于你希望故意忽略属性更新 。** 在此情况下， 比较合理的是重命名属性被叫作 `initialColor` 或是 `defaultColor`。然后你可以暴力地使组件“重置”它的初始状态 。通过[改变它的键`key`](/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key)当必要时。
>
>Read our [blog post on avoiding derived state](/blog/2018/06/07/you-probably-dont-need-derived-state.html) to learn about what to do if you think you need 一些依赖属性的状态。

* * *

### `componentDidMount()`

```javascript
componentDidMount()
```

`componentDidMount()`紧跟在组件装载后（被插入树中）调用。要求的DOM节点初始化应该放到这里。若你需要从远端加载数据，这是一个适合实例化网络请求的好地方。

这个方法是建立任何订阅的一个好地方。如果你那么做了，别忘了在`componentWillUnmount()`取消订阅。

在`componentDidMount()`中，你 **可以立即调用`setState()`**。它将会触发一次额外的渲染，但是它将在浏览器刷新屏幕之前发生。这保证了在此情况下即使`render()`将会调用两次，用户也不会看到中间状态。谨慎使用这一模式，因为它常导致性能问题。在大多数情况下，你可以 在`constructor()`中使用赋值初始状态来代替。然而，有些情况下必须这样，比如像模态框和工具提示框。这时，你需要先测量这些DOM节点，才能渲染依赖尺寸或者位置的某些东西。

* * *

### `componentDidUpdate()`

```javascript
componentDidUpdate(prevProps, prevState, snapshot)
```

`componentDidUpdate()`紧跟在更新发生后调用。对于初次的渲染，该方法并不会调用。

当组件被更新之后，使用此方法作为操作DOM的一次机会。这也是一个适合发送请求的地方，只要你对比了当前属性和前一次属性（例如，如果属性没有改变那么请求也就没必要了）。

```js
componentDidUpdate(prevProps) {
  // Typical usage (don't forget to compare props):
  if (this.props.userID !== prevProps.userID) {
    this.fetchData(this.props.userID);
  }
}
```

在`componentDidUpdate()`中，你 **可以立即调用`setState()`**。 但是要注意 **必须把它包裹在一个条件中** 就像前面的例子中那样。否则你将引发一个无限循环。也会引发额外的重新渲染，此时对于用户不可见，会影响组件的性能。如果你想“如实镜像”一些状态到上面来的属性中，考虑直接使用属性来代替。Read more about [why copying props into state causes bugs](/blog/2018/06/07/you-probably-dont-need-derived-state.html).

如果你的组件实现了 `getSnapshotBeforeUpdate()` 生命周期 (很罕见)，它的返回值将当作一个第三方快照参数被传递到 `componentDidUpdate()`。否则这个参数将是未定义undefined。

> 注意
>
> 若[`shouldComponentUpdate()`](#shouldcomponentupdate)返回false，`componentDidUpdate()`将不会被调用。

* * *

### `componentWillUnmount()`

```javascript
componentWillUnmount()
```

`componentWillUnmount()`紧挨着在组件被卸载和销毁之前调用。可以在该方法里处理任何必要的清理工作，例如解绑定时器，取消网络请求，清理任何在`componentDidMount`环节创建的订阅。

你 **不应该调用 `setState()`** 在 `componentWillUnmount()` 中，因为组件将永远不会被重新渲染了。一旦一个组件实例被卸载，它将永远不会再次装载。

* * *

### 很少使用的生命周期方法

The methods in this section correspond to uncommon use cases. They're handy once in a while, but most of your components probably don't need any of them. **You can see most of the methods below on [this lifecycle diagram](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/) if you click the "Show less common lifecycles" checkbox at the top of it.**


### `shouldComponentUpdate()`

```javascript
shouldComponentUpdate(nextProps, nextState)
```

使用`shouldComponentUpdate()`以让React知道是否组件的输出不受当前状态或属性影响。默认行为是在每一次状态的改变重新渲染，在大部分情况下你应该依赖于默认行为。

当接收到新属性或状态时，`shouldComponentUpdate()` 在渲染前被调用。默认为`true`。该方法在初始化渲染或当使用`forceUpdate()`时并不会被调用。

这个方法的存在是作为一种**[性能优化](/docs/optimizing-performance.html)。** 不要依赖它去阻止一次渲染，因为这会导致臭虫。**考虑使用内建的 [`PureComponent`](/docs/react-api.html#reactpurecomponent)** 代替手写`shouldComponentUpdate()`。`PureComponent` 对属性和状态执行浅比较，因而降低你略过必要更新的机会。

若你确信想要手写，你可能需要用`this.props`和`nextProps`以及`this.state` 和 `nextState`比较，并返回`false`以告诉React更新可以被忽略。注意，返回`false`不能阻止子组件当*他们的*状态改变时重新渲染。

我们不推荐做深相等检测，或使用`JSON.stringify()`在`shouldComponentUpdate()`中。这是非常无效率的会伤害性能。

当前，如果`shouldComponentUpdate()`返回`false`，那么[`UNSAFE_componentWillUpdate()`](#componentwillupdate)，[`render()`](#render)， 和 [`componentDidUpdate()`](#componentdidupdate)将不会被调用。注意，在未来React可能会将`shouldComponentUpdate()`作为一个线索而不是一个严格指令，返回`false`可能仍然使得组件重新渲染。

* * *

### `static getDerivedStateFromProps()`

```js
static getDerivedStateFromProps(nextProps, prevState)
```

组件实例化后和接受新属性时将会调用`getDerivedStateFromProps`。它应该返回一个对象来更新状态，或者返回null来表明新属性不需要更新任何状态。

注意，如果父组件导致了组件的重新渲染，即使属性没有更新，这一方法也会被调用。如果你只想处理变化，你可能想去比较新旧值。

调用`this.setState()` 通常不会触发 `getDerivedStateFromProps()`。

* * *

### `getSnapshotBeforeUpdate()`

`getSnapshotBeforeUpdate()`在最新的渲染输出提交给DOM前将会立即调用。它让你的组件能在当前的值可能要改变前获得它们。这一生命周期返回的任何值将会
作为参数被传递给`componentDidUpdate()`。

例如：

`embed:react-component-reference/get-snapshot-before-update.js`

在上面的例子中，为了支持异步渲染，在`getSnapshotBeforeUpdate` 中读取`scrollHeight`而不是`componentWillUpdate`，这点很重要。由于异步渲染，在“渲染”时期（如`componentWillUpdate`和`render`）和“提交”时期（如`getSnapshotBeforeUpdate`和`componentDidUpdate`）间可能会存在延迟。如果一个用户在这期间做了像改变浏览器尺寸的事，从`componentWillUpdate`中读出的`scrollHeight`值将是滞后的。

* * *

### 错误边界

[错误边界](/docs/error-boundaries.html)是一种React组件，能够捕捉在他们的子组件树中任意地方的JavaScript错误，记录这些错误，并且显示一个退路UI，而不是让组件树崩溃。错误边界捕捉的错误，位于其之下的整棵树中的在渲染期间、在生命周期方法中、和在构造函数中。

如果一个类组件定义了生命周期方法`static getDerivedStateFromError()`或`componentDidCatch()`的任何一个或两个，将变成一个错误边界。从这些生命周期中更新状态让你能捕捉到在其之下的树中、未处理的JavaScript错误，并显示退路UI。

只使用错误边界来恢复意外的异常；**不要尝试将它们用于控制流。**

详情请见[*React 16中的错误处理*](/blog/2017/07/26/error-handling-in-react-16.html)。

> 注意
> 
> 错误边界捕捉的错误只能是在树中**低于**它们的组件里。一个错误边界并不能捕捉它自己内部的错误。

* * *

### `static getDerivedStateFromError()`

```javascript
static getDerivedStateFromError(error)
```

这个生命周期被调用是在某个后代组件已经抛出一个错误之后。

它的一个参数接收被抛出的错误，并应该返回一个值去更新状态。

```js{7-10,13-16}
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```

> 注意
>
> `getDerivedStateFromError()` 被调用是在 "渲染" 阶段，所以不允许副作用。
对于这些用例，使用`componentDidCatch()` 来代替。

* * *

### `componentDidCatch()`

```javascript
componentDidCatch(error, info)
```

这个生命周期被调用是在某个后代组件已经抛出一个错误之后。

它收到两个参数：

1. `error`——被扔出的错误。
2. `info`——一个对象，带有一个`componentStack`键，详见[information about which component threw the error](/docs/error-boundaries.html#component-stack-traces)。

`componentDidCatch()`被调用是在"提交"期间阶段，所以允许副作用。

它应该用于的事情比如记录错误等：

```js{12-19}
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // Update state so the next render will show the fallback UI.
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // Example "componentStack":
    //   in ComponentThatThrows (created by App)
    //   in ErrorBoundary (created by App)
    //   in div (created by App)
    //   in App
    logComponentStackToMyService(info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children; 
  }
}
```

> 注意
> 
> In the event of an error, you can render a fallback UI with `componentDidCatch()` by calling `setState`, 但是这将在未来的版本中被废弃。
> 使用`static getDerivedStateFromError()`处理退路渲染来代替。

* * *

### Legacy Lifecycle Methods

The lifecycle methods below are marked as "legacy". They still work, but we don't recommend using them in the new code. You can learn more about migrating away from legacy lifecycle methods in [this blog post](/blog/2018/03/27/update-on-async-rendering.html).


### `UNSAFE_componentWillMount()`

```javascript
UNSAFE_componentWillMount()
```

`UNSAFE_componentWillMount()`在装载发生前被立刻调用。其在`render()`之前被调用，因此在这方法里同步地设置状态将不会触发重新渲染。

避免在该方法中引入任何的副作用或订阅。对于这些使用场景，我们推荐使用`constructor()`来替代。

这是唯一的会在服务端渲染调起的生命周期钩子函数。

> 注意
>
> 这一生命周期之前叫做`componentWillMount`。这一名字在17版前都有效。可以使用[`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles)来自动更新你的组件。

* * *

### `UNSAFE_componentWillReceiveProps()`

```javascript
UNSAFE_componentWillReceiveProps(nextProps)
```

> 注意
>
> 推荐你使用[`getDerivedStateFromProps`](#static-getderivedstatefromprops)生命周期而不是`UNSAFE_componentWillReceiveProps`。[关于此建议在此了解详情。](/blog/2018/03/29/react-v-16-3.html#component-lifecycle-changes)

`UNSAFE_componentWillReceiveProps()`在装载了的组件接收到新属性前调用。若你需要更新状态响应属性改变（例如，重置它），你可能需对比`this.props`和`nextProps`并在该方法中使用`this.setState()`处理状态改变。

注意即使属性未有任何改变，React可能也会调用该方法，因此若你想要处理改变，请确保比较当前和之后的值。这可能会发生在当父组件引起你的组件重新渲染。

在 [装载](#mounting)期间，React并不会调用带有初始属性的`UNSAFE_componentWillReceiveProps`方法。其仅会调用该方法如果某些组件的属性可能更新。调用`this.setState`通常不会触发`UNSAFE_componentWillReceiveProps`。

> 注意
>
> 这一生命周期之前叫做`componentWillReceiveProps`。这一名字在17版前都有效。可以使用[`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles)来自动更新你的组件。

* * *

### `UNSAFE_componentWillUpdate()`

```javascript
UNSAFE_componentWillUpdate(nextProps, nextState)
```

当接收到新属性或状态时，`UNSAFE_componentWillUpdate()`为在渲染前被立即调用。在更新发生前，使用该方法是一次准备机会。该方法不会在初始化渲染时调用。

注意你不能在这调用`this.setState()`，若你需要更新状态响应属性的调整，使用[`getDerivedStateFromProps()`](#static-getderivedstatefromprops) 代替。

> 注意
>
> 这一生命周期之前叫做`componentWillUpdate`。这一名字在17版前都有效。可以使用[`rename-unsafe-lifecycles` codemod](https://github.com/reactjs/react-codemod#rename-unsafe-lifecycles)来自动更新你的组件。

> 注意
>
> 若[`shouldComponentUpdate()`](#shouldcomponentupdate)返回false，`UNSAFE_componentWillUpdate()`将不会被调用。

* * *

## 其他API

不同于上面的生命周期方法 (React 替你调用它)，下面的方法是*你*从你的组件中调用的方法。

只有两个方法：`setState()` 和 `forceUpdate()`。

### `setState()`

```javascript
setState(updater, [callback])
```

`setState()`将对组件状态的改变排队，并告诉该组件及其子组件需要用已经更新的状态来重新渲染。这个方法主要是用来更新用户界面以响应事件处理和服务器响应。

>（译者注：setState源码中将一个需要改变的变化存放到组件的state对象中，采用队列处理）

将`setState()`认为是一次*请求*而不是一次立即执行的命令来更新组件。为了更为可观的性能，React可能会推迟它，稍后会一次性更新这些组件。React不能保证状态改变被应用是立刻地。

`setState()`不总是立刻更新组件。其可能是成批处理或推迟更新。这使得在调用`setState()`后立刻读取`this.state`变成一个潜在陷阱。代替地，使用`componentDidUpdate`或一个`setState`回调（`setState(updater, callback)`），当中的每个方法都会保证在更新被应用之后触发。若你需要基于前一个状态来设置状态，阅读下面关于`updater`参数的介绍。

`setState()`永远都会导致重新渲染，除非`shouldComponentUpdate()` 返回`false`。如果使用了可变对象，并且在`shouldComponentUpdate()`中没有实现渲染条件逻辑，那么只有当新状态不同于前一个状态时才调用`setState()`，将会避免不必要的重新渲染。

第一个参数是一个`updater`函数，签名为：

```javascript
(state, props) => stateChange
```

`state`是一个引用，指向当改变正在被应用时的组件状态。它不应该被直接改变。代替地，应该构建一个来自于`state` 和 `props`输入的新对象来表示改变。例如，假设我们想通过`props.step`在状态中增量值：

```javascript
this.setState((state, props) => {
  return {counter: state.counter + props.step};
});
```

updater函数接收到的`state` 和 `props`保证都是最新的。updater的输出被浅合并到`state`中。

`setState()`的第二个参数是一个可选的回调函数，其执行将是在一旦`setState`完成，并且组件被重新渲染之后。通常，对于这类逻辑，我们推荐使用`componentDidUpdate`。

可选地，你可以传递一个对象作为 `setState()`的第一个参数代替一个函数：

```javascript
setState(stateChange, [callback])
```

其执行一次`stateChange`的浅合并到新状态中。例如，欲调整购物车中物品数量：

```javascript
this.setState({quantity: 2})
```

这一形式的`setState()`也是异步的，并在相同的周期中多次调用可能会被成批处理到一起。例如，若你在同一周期中尝试增加物品的数量多次，其等价于：

```javaScript
Object.assign(
  previousState,
  {quantity: state.quantity + 1},
  {quantity: state.quantity + 1},
  ...
)
```

之后的调用在同一周期中将会重写之前调用的值，因此数量仅会被加一次。若下一状态依赖于当前状态，我们推荐使用updater函数形式来代替：

```js
this.setState((state) => {
  return {counter: state.quantity + 1};
});
```

更多细节，查看：

* [State and Lifecycle guide](/docs/state-and-lifecycle.html)
* [In depth: When and why are `setState()` calls batched?](https://stackoverflow.com/a/48610973/458193)
* [In depth: Why isn't `this.state` updated immediately?](https://github.com/facebook/react/issues/11527#issuecomment-360199710)

* * *

### `forceUpdate()`

```javascript
component.forceUpdate(callback)
```

默认情况，当你的组件或状态发生改变，你的组件将会重新渲染。若你的`render()`方法依赖其他数据，你可以通过调用`forceUpdate()`来告诉React组件需要重新渲染。

调用`forceUpdate()`将会导致组件的 `render()`方法被调用，并忽略`shouldComponentUpdate()`。这将会触发每一个子组件的生命周期方法，涵盖，每个子组件的`shouldComponentUpdate()` 方法。若当标签改变，React仅会更新DOM。

通常你应该尝试避免所有`forceUpdate()` 的用法并仅在`render()`函数里从`this.props`和`this.state`读取数据。

* * *

## 类属性

### `defaultProps`

`defaultProps`可以被定义为在组件类本身上的一个属性，为该类设置默认属性。这对于未定义（undefined）的属性来说有用，而对于设为空（null）的属性并没用。例如：

```js
class CustomButton extends React.Component {
  // ...
}

CustomButton.defaultProps = {
  color: 'blue'
};
```

若未设置`props.color`，其将被设置默认为`'blue'`:

```js
  render() {
    return <CustomButton /> ; // props.color will be set to blue
  }
```

若`props.color`设为null，则保持为null：

```js
  render() {
    return <CustomButton color={null} /> ; // props.color will remain null
  }
```

* * *

### `displayName`

`displayName`字符串被用在调试信息中。Usually, you don't need to set it explicitly because it's inferred from the name of the function or class that defines the component. You might want to set it explicitly if you want to display a different name for debugging purposes or when you create a higher-order component, see [Wrap the Display Name for Easy Debugging](/docs/higher-order-components.html#convention-wrap-the-display-name-for-easy-debugging) for details.

* * *

## 实例属性

### `props`

`this.props`包含了这个组件调用者所定义的属性。查看[组件 & Props](/docs/components-and-props.html)关于属性的介绍。

特别地，`this.props.children`是一个特别属性，其通常由JSX表达式中的子标签定义，而不是在标签本身中定义。

### `state`

状态包含的数据专用于这个组件，组件可能随时间而改变。状态是用户定义的，且其应为普通JavaScript对象。

如果某些值不用于渲染或者数据流（例如，一个计时器ID），你不必将它放到状态中。诸如此类的值可以被定义为组件实例的字段。

查看[State & 生命周期](/docs/state-and-lifecycle.html)了解更多关于状态的信息。

永远不要直接改变`this.state`，因为后面对`setState()`的调用可能替换掉你所做的改变。将`this.state`当成是不可变的。
