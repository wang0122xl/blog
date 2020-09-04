---
title: React组件设计
date: {{ date }}
updated: {{ date }}
tags:
  - react
  - 组件化
categories: 
  - 前端
---

组件（Component）是对数据和方法的简单封装，是为了减少代码冗余，提高代码复用度对代码进行合理拆分后的产物。
{% asset_img components.jpg components %}

<!-- more -->

## 为什么需要组件化

在开发过程中，我们常常会碰到以下情况：

* 页面逻辑越来越多，代码越写越庞大。随着项目的推进，类似的需求所需要的技术人力往往倍数提升，且质量难以得到保证。
* 面对新的需求束手束脚，生怕改动会影响别的代码的执行，产生不可预估的bug。
* debug时面对成百甚至上千行的代码不知所措。
* 同样的逻辑在多个地方重复拷贝代码，拷贝时看不出问题，一旦功能变更只能含泪挨个修改。效率低不说，还很容易遗漏。

> 为了解决以上问题，社区在前端组件化的尝试、演进从未停止

## 理想的组件

针对以上问题，理想的组件应该具有如下特点：

* ***高内聚***，组件通过某些特定的逻辑，对一部分功能（ability）进行组装，进而实现某个特定的独立功能（feature）。组件的基础功能应做到开箱即用，**需要一堆配置才可正常使用的组件有违组件化思想**。
* ***低耦合***，组件化开发往往伴随分层思想，我们将具有相同粒度的功能归于一层。同层组件之间不可相互依赖，避免出现“我修改了组件A，所以我还需要修改组件B”的情况。
* ***状态无关***，组件如有状态，应自己维护。业务代码只关心业务，不应该在使用组件时还需考虑组件的内部状态。

## React组件

组件化开发的本质是将代码拆分至更小的粒度已达到分治，复用的目的，组件本身其实是对这种拆分的抽象。
在 vue 中，我们有非常多的途径可以达到复用的目的， 比如，我们使用 directive 封装 dom 操作，用 mixin 封装组件间相同的逻辑
在 React 中，虽然语法和设计不尽相同，但不变的是 UI 的本质： **展示交互** 和 **数据流转**
那么由此我们先简单的把组件分为 2 大类别：**展示型组件** 和 **容器型组件**

## 展示型组件

展示型组件通常具有如下特点：

* 描述外观
* 通常具备 DOM 结构和样式，接收 props.children 作为子内容
* 不依赖于应用的其他部分，如 Redux
* 与数据的加载、流转无关
* 使用者通过 props 传入数据控制组件展示，传入回调函数获取事件通知
* 尽量不含 state （即便有，也只与该组件的 UI 相关）

展示型组件相对比较简单，以Antd为例，Antd中我们常用的Spin、Progress、Message等，他们大多只是承载组件的渲染，自身与数据无关，通过传入的 props 改变外观或相应用户的操作。
同样的，你也可以自由的在组件嵌套结构里插入组合别的自定义组件

```jsx
const styles = {
  backgroundColor: '#fff',
  padding: 20
}

// 简单的functional component
const FunctionalComponent = props => <div style={styles}>{props.children}</div>

// 或者你更倾向于 class
class ClassComponent extends React.PureComponent {
  render() {
    return <div style={styles}>{this.props.children}</div>
  }
}
```

## 容器型组件

容器型组件通常具有如下特点：

* 描述工作流程
* **没有 DOM 结构(除了包裹用的节点)和样式**
* 不依赖于应用的其他部分，如 Redux / Context
* 自身提供数据及表现函数
* 连接 redux / Context 等并向内传递数据和 action
* 通常都是有状态的，作为数据源存在
* 通常通过 HOC 生成，或使用 render props

通过这些定义，我们可以发现Antd中的Layout就是一个典型的容器型组件，他内部通过 Context.Provider，为内部组件提供数据和 actions

### HOC 高阶组件

**HOC（high order component）**：接收 component 为入参，并返回一个新的 component 的函数
HOC 被用来替代 createClass 时期的 mixin [Mixins Considered Harmful](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html)。解决了组件间复用公共数据逻辑的问题。
talk is cheap，我们来看一个简单的withUser的HOC实现：

```jsx
import hoistNonReactStatic from 'hoist-non-react-statics'
import { Page } from './components' // 一个展示型组件

const withUser = user => Component => {
  class ComponentWithUser extends React.Component {
    static displayName = `withUser(${Component.displayName || Component.name})`

    state = {
      user: user || { name: 'lily', logout() {}, getLocation() {} }
    }

    render() {
      return <Component {...this.props} user={this.state.user} />
    }
  }
  hoistNonReactStatic(ComponentWithUser, Component)
  return ComponentWithUser
}

const UserPage = withUser(Page) // 漂亮，你得到了一个容器型组件
```

这个函数接受一个组件作为入参，返回值是一个新的函数组件，原有组件内部数据功能不变的前提下，我们通过withUser为这个组件注入了新的数据及功能。

注意点：

* 使用HOC时一大痛点时debug时定位困难，尤其是在HOC层层嵌套的情况下，定位异常变得尤为困难。因此我们需要为组件设定静态属性 displayName，这样有助于 debug 时可以更好的定位到目标组件
* HOC 组件接收到的 props 一定要传递给被包裹的组件
* 被包裹的组件的静态属性和方法需要使用 hoistNonReactStatic 提升至 HOC 组件（这也是为何不推荐继承而是使用组合）
* 需要额外接收参数请使用柯里化，这样符合 Rambda 演算 Unary 的特性，且可以转换为修饰器函数

### render props

起初它被大量运用在 chenlou 的 react-motion 库中，现在他正式出现在 React 官方文档的高级范式中。
他允许组件接收一个或多个函数类型的 props（返回值为组件），可以在容器 render 时，动态的将数据传递给这个函数，由此完成数据传递。
一般我们会使用 props.children 作为 render props。

```jsx
import { Page } from './components' // 一个展示型组件

class UserContainer extends React.Component {
  static propTypes = {
    children: PropTypes.func.isRequred
  }

  state = {
    user: { name: 'jason', logout() {}, getLocation() {} }
  }

  render() {
    return <React.Fragment>{this.props.children(this.state.user)}</React.Fragment>
  }
}

class UserPage extends React.Component {
  renderPage = user => <Page user={user} />

  render() {
    return <UserContainer>{this.renderPage}</UserContainer>
  }
}
```

注意点：

* 由于 render props 不寻常的设计，需要为组件声明 propTypes 的 children 属性为 PropTypes.func.isRequred（或在typescript中定义属性为required），以避免错误的使用
* 不要使用行内的函数声明，虽然他很简单，但在组件 rerender 时，会重新生成一个函数，造成资源浪费，可以把函数转换成组件的方法
* React.Fragment 为占位标记，不会生成任何结构，符合容器型组件尽量不含 dom 结构的目的

### 两者比较

> * 当使用 HOC 修饰 class 组件时，可以使用 es7 的修饰器语法简化代码

```js
@withUser({ name: 'lily' })
class Page extends React.Component {
  //...
}
```

> * 多个 HOC 同时作用于一个组件时，可以被组合（compose），这得益于函数式编程的特性
> * HOC 可以和函数组合一样，组合 N 个 hoc 函数至同一组件，但这样也会带来许多问题诸如 静态属性 的提升，props 的重名问题，同时也会造成 debug 的调用栈过深，以及不知道 props 是从哪一个 HOC 传入的。**所以需要谨慎设计你的 HOC**
> * render props 的特性使得他能够够好的支撑运行时，而不像 HOC 那样需要静态组合
> * 个人建议：HOC 可以作为顶层的数据和方法传入，而 render props 则适合局部的数据传递

### 案例

* Container with restful resource

```js
const withService = ({ url, params }) => BaseComponent => {
  class ComponentWithService extends React.Component {
    constructor(props) {
      this.initialSnapshot = { params }
      this.state = {
        data: null,
        params
      }
    }
    state = {
      data: null
    }
    componentDidMount() {
      this.fetchData()
    }

    componentDidUpdate(prevProps, prevState) {
      if (!_.isEqual(prevState.params, this.state.params)) {
        this.fetchData()
      }
    }

    fetchData = async () => {
      const data = await client.post(url, this.state.params)
      this.setState({ data })
    }

    setParams = newParams => {
      this.setState({
        params: { ...this.state.params, ...newParams }
      })
    }

    refresh = () => {
      this.setState({
        params: this.initialSnapshot.params
      })
    }

    render() {
      const serviceProps = { state: this.state, setParams: this.setParams, refresh: this.refresh }
      return <BaseComponent {...this.props} service={serviceProps} />
    }
  }

  return ComponentWithService
}

// 使用

const Page = props => <Page />

withService({
  url: '/api',
  params: {
    a: 1
  }
})(Page)

// or

@withService({
  url: '/api',
  params: {
    a: 1
  }
})
class Page extends Component {
  //...
}
```

### 好用的第三方库

> **[recompose](https://github.com/acdlite/recompose) 提供了一系列相当实用的 HOC 方法**

### 其他实现

当然出于 react props 的自由度，还有许多可作为实现组件复用的方法，比如 Component Inject, 或者 Nested Component。但在团队开发过程中，组件复用方式能满足开发需求即可。同样的目的，应尽量避免在项目中四处散落不同的实现。

## 总结

采取组件化开发，我们可以得到什么？

* 我们的展示型组件将非常容易测试，因为他们不依赖外部状态，只需要传入相应的 props，就可以验证展示的逻辑是否符合预期。
* 当我们熟练运用容器化组件，我们会很容易提取出行为层和数据层，使我们的代码分层更准确更清晰

参考资料：

* [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
* [A tale of Higher-Order Components & Render Props](https://medium.com/ingenious/a-tale-of-higher-order-components-render-props-a1ba47e8cfeb)
