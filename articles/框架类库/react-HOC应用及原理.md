# HOC应用及原理

涉及内容：
1. 设计方式
2. 设计原理
3. 具体应用
4. 具体使用

## 设计方式

HOC本质上是一个function。它接受一个组件作为参数，并且返回一个新的组件。

## 设计原理

### 属性代理
顾名思义，代理属性。就是把传入的属性先在这个组件中做一层处理，根据props判断是否需要渲染传入的组件。

```javascript
function proxyHOC(WrappedComponent) {
  return class extends Component {
    render() {
      return <WrappedComponent {...this.props} />;
    }
  }
}
```

### 反向代理
返回的class直接继承传入的参数组件。可以通过this，直接控制更多`生命周期函数、state、props、render`等。相比属性代理它能操作更多的属性。

```javascript
function proxyHOC(WrappedComponent) {
  return class extends ComWrappedComponentponent {
    render() {
      return super.render()
    }
  }
}

```

## 具体应用

- 组合渲染

  在渲染组件时，添加上相应的title或者其他的元素，一起展示。 
- 条件渲染
  
  根据props进行渲染。
- 操作props

  对传入的props进行增加、修改、删除或者根据特定的props进行特殊的操作。
- 状态管理
  
  通过反向代理，可以获取到当前的state和props，方便进行统一操作和管理。


## 具体使用
```javascript
// 通过继承传入的组件，给组件添加是否卸载的判断
// 目的是：组件卸载后，不再出现setState的情况
// wrappedComponent只能是class
export default function safeComponent (wrappedComponent) {
  function safe (setState, ctx) {
    return (...args) => {
      if (ctx._isMounted) {
        setState.call(ctx, ...args)
      }
    }
  }

  function did (didm, ctx) {
    return (...args) => {
      ctx._isMounted = true
      didm && didm.call(ctx, ...args)
    }
  }

  function will (willu, ctx) {
    return (...args) => {
      ctx._isMounted = false
      willu && willu.call(ctx, ...args)
    }
  }

  return class extends wrappedComponent {
    constructor (props) {
      super(props)
      this.setState = safe(this.setState, this)
      this.componentDidMount = did(this.componentDidMount, this)
      this.componentWillUnmount = will(this.componentWillUnmount, this)
    }
  }
}

```

## 存在问题
- HOC需要在原组件上进行包裹或者嵌套，如果大量使用HOC，将会产生非常多的嵌套，这让调试变得非常困难。
- HOC可以劫持props，在不遵守约定的情况下也可能造成冲突。



> 父子组件生命周期渲染顺序
> 
> 子组件挂载上后，触发`componentDidMount`，才能触发父组件的`componentDidMount`。



