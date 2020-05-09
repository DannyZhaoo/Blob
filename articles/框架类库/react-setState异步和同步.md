# setState异步和同步

涉及内容：
1. 什么情况下异步触发
2. 什么情况下同步触发
3. 为什么会有这种差异

## 什么情况下异步触发

其实大部分正常使用的时候，都会异步调用，主要使用场景如下：
1. 生命周期函数中使用`setState`
2. 使用react的dom绑定事件，触发`setState`
  
大部分使用场景中，都是异步调用。

## 什么情况下同步触发

但是也有**同步触发**的情况，在调用后，就可以直接得到`this.state`中的值。

1. 在`setTimeout`和`setInterval`中调用，就会直接触发渲染。
2. 使用`addEventListener`添加的绑定函数调用`setState`会直接调用。
3. 在`async`函数中是同步更新的。

## 为什么会有这种差异


## 使用建议
如果newState内容与render有依赖关系，就不建议**同步更新**。

*因为每次render都会完整的执行一次批量更新流程（只是dirtyComponents长度为1，stateQueue也只有该组件的newState），会调用一次diff算法，这样会影响react性能。*