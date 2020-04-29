# BFC

1. 含义
2. BFC特点
3. BFC的用处
4. 触发条件


## 含义
一个BFC就是一个单独的block format content。BFC属于文档布局其中的一种。

> 文档布局分类：
> 1. Block Formatting Contexts
> 2. Inline Formatting Contexts
> 3. Flex Formatting Contexts
> 4. Grid Formatting Contexts


## 特点
那么一个BFC都有什么特点呢？
1. BFC不会相互重叠
2. 计算BFC的高度时，浮动子元素也参与计算
3. BFC是一个隔离的独立容器，容器里面的元素与外面元素互不影响


## 用处
拥有这些特点之后，BFC又有什么具体应用呢？
1. 阻止margin重叠
2. 可以包含浮动元素，防止高度塌陷
3. 自适应两栏布局
4. 可以阻止元素被浮动元素覆盖

## 触发条件
1. 根`<html>`元素
2. `float`不为`none`
3. `position`是`absolute`和`fixed`
4. `overflow`不是`visible`
5. `display`是`inline-block`、`flow-root`、`table-cell`
