# Performance值与DOM树的关系

![timing-overview](/Users/danny/Desktop/timing-overview.png)

标注

**domLoading**：开始解析第一批收到的HTML文档字节。

**domInteractive**：完成对所有HTML文档的解析，并且完成DOM树的构建。

在domInteractive和domContentLoaded这两个时间节点中间，浏览器会执行带有defer属性的脚本文件。defer不会改变script的执行顺序。

**domContentLoaded**：CSSOM构建完成，可以开始构建渲染树了。

**domComplete**：全部内容，包括图片等文件，都加载完毕。



一般来说：

**dom processing**：domLoading - domContentLoaded

**page load**：domContentLoaded - onLoad



结论：

- JS文件会阻塞解析器，这个是说当HTML解析到**脚本文件**，会停止构建DOM树，但是浏览器会“偷看”DOM，预先下载相关的资源。

- CSS文件确定CSS不会阻塞DOM的解析，但是会阻塞DOM渲染，因为render树必须要DOM树和CSSOM都构建好才能render。

- 浏览器遇到 `<script>`且没有`defer`或`async`属性的 标签时，会触发页面渲染，因而如果前面`CSS`资源尚未加载完毕时，浏览器会等待它加载完毕再执行脚本。也就是说在执行脚本的时候，必然该脚本前面的DOM和CSSOM已经构建完了。这是因为要确保脚本能获取到最新的`DOM`元素信息。尽管脚本可能不需要这些信息。

