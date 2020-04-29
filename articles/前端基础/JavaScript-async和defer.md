# async和defer

1. async作用
2. defer作用
3. async和defer哪个优先级更高


## async作用
async的js下载不会阻塞浏览器加载，但是一旦下载完成，就会开始执行，此时确实会阻塞渲染。

所以async的文件，加载顺序是**不能确定**的。


## defer作用
defer的js下载不会阻塞浏览器加载，直到页面中所有的内容都解析完成，才去执行defer的js。

加载顺序确定，在触发页面onload事件之前执行。



## async和defer哪个优先级更高
如果这两个同时使用的话，async会优先级更高一些。
但是如果不支持async的浏览器，会退回到支持defer。
