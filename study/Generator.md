# Generator
##  定义
### 语法
 Generator 函数是一个状态机，封装了多个内部状态。
### 使用
 可以从函数外部控制函数的执行顺序，改变函数的执行过程和结果。
## 关键词
### function  name() {   }
### yield
 是暂停执行的标记
 只能在Generator中使用
 本身没有返回值，所以可以返回值是undefined
 标注: 后面跟的是函数的状态值
### yield 
 在Generator函数内部调用另一个Generator函数
 标注: 后面是可以iterator的object
## prototype
### next
 参数
​     可以带一个参数，该参数就会被当作上一个yield表达式的返回值。
 注意
​     第一次调用next时输入的参数会被忽略，只有从第二次调用next方法时，参数才是有效的。
### return
 参数
​     返回给定的值，并且终结遍历Generator函数
 注意
​     如果函数内部有 try...finally代码块，那么return的方法会被推迟到finally代码块执行完再执行。
### throw
 参数
​     可以带一个参数，该参数会被catch语句接收
 注意
​     这个方法抛出的错误要被内部捕获，前提是至少执行过一次next方法，否则只能被外部的catch捕获。
​     g.throw这个方法在被内部捕获之后，会自动执行一次next方法
​     一旦Generator执行过程中抛出错误，且没有被内部捕获，就不会再执行下去了。
## 应用
### 给对象添加Iterator接口
```javascript
 function objectEntries() {
   let propKeys = Object.keys(this);
 
   for (let propKey of propKeys) {
       yield [propKey, this[propKey]];
   }
 }
 
 let jane = { first: 'Jane', last: 'Doe' };
 
 jane[Symbol.iterator] = objectEntries;
 
 for (let [key, value] of jane) {
     console.log(`${key}: ${value}`);
 }
```
### 取出嵌套数组的所有成员
```javascript
function iterTree(tree) {
   if (Array.isArray(tree)) {
     for(let i=0; i < tree.length; i++) {
       yield iterTree(tree[i]);
     }
   } else {
     yield tree;
   }
 }
 
 const tree = [ 'a', ['b', 'c'], ['d', 'e'] ];
 
 for(let x of iterTree(tree)) {
   console.log(x);
 }
```
### 遍历完全二叉树
 // 下面是二叉树的构造函数，
 // 三个参数分别是左树、当前节点和右树
 ```javascript
 function Tree(left, label, right) {
   this.left = left;
   this.label = label;
   this.right = right;
 }
 
 // 下面是中序（inorder）遍历函数。
 // 由于返回的是一个遍历器，所以要用generator函数。
 // 函数体内采用递归算法，所以左树和右树要用yield遍历
 function inorder(t) {
   if (t) {
       yield inorder(t.left);
       yield t.label;
       yield inorder(t.right);
   }
 }
 
 // 下面生成二叉树
 function make(array) {
   // 判断是否为叶节点
   if (array.length == 1) return new Tree(null, array[0], null);
   return new Tree(make(array[0]), array[1], make(array[2]));
 }
 let tree = make([[['a'], 'b', ['c']], 'd', [['e'], 'f', ['g']]]);
 
 // 遍历二叉树
 var result = [];
 for (let node of inorder(tree)) {
   result.push(node);
 }
 
 result
 // ['a', 'b', 'c', 'd', 'e', 'f', 'g']
 ```
### 实现状态机
```javascript
 var clock = function () {
   while (true) {
     console.log('Tick!');
     yield;
     console.log('Tock!');
     yield;
   }
 };

### 异步操作的同步化表达
 function loadUI() {
   showLoadingScreen();
   yield loadUIDataAsynchronously();
   hideLoadingScreen();
 }
 var loader = loadUI();
 // 加载UI
 loader.next()
 
 // 卸载UI
 loader.next()
```
 Ajax
 ```javascript
  function main() {
    var result = yield request("http://some.url");
    var resp = JSON.parse(result);
      console.log(resp.value);
  }

  function request(url) {
    makeAjaxCall(url, function(response){
      it.next(response);
    });
  }

  var it = main();
  it.next();
 ```

 逐行读取文本文件
 ```javascript
  function numbers() {
    let file = new FileReader("numbers.txt");
    try {
      while(!file.eof) {
        yield parseInt(file.readLine(), 10);
     }
    } finally {
      file.close();
    }
  }
 ```

### 控制流管理
```javascript
 function longRunningTask(value1) {
   try {
    var value2 = yield step1(value1);
    var value3 = yield step2(value2);
    var value4 = yield step3(value3);
    var value5 = yield step4(value4);
     // Do something with value4
   } catch (e) {
     // Handle any error from step1 through step4
   }
 }
 scheduler(longRunningTask(initialValue));
 
 function scheduler(task) {
   var taskObj = task.next(task.value);
   // 如果Generator函数未结束，就继续调用
   if (!taskObj.done) {
     task.value = taskObj.value
     scheduler(task);
   }
 }
```
 标注: 同步操作
## 用法
### 调用方法
 调用方法跟普通函数一样，函数名后面跟着一对圆括号，但是该函数并不执行，返回的结果是一个指向内部状态的对象指针。
### 与Iterator接口的关系
 任何一个对象都有Symbol.iterator的方法
 这个Symbol.iterator只能通过[]来调用
 普通对象这个值是undefined，但是本身就是Iterator对象的函数通过这个方法，可以得到这个对象的Generator函数。
 可以通过把Generator赋值给对象的Symbol.iterator属性，从而使得该对象具有Iterator接口。
### for...of循环
 可以自动遍历Generator函数，不用再调用next方法
### 其他
 扩展运算符（...）
 解构赋值
 Array.from
### 概要: 一旦next方法返回对象的done是true，那么循环就会中止，且不包含该返回对象。
## this
### Generator函数总是返回一个遍历器，ES6规定这个遍历器是Generator函数的实例，也继承Generator函数的prototype对象上的方法。
### 但是Generator函数不是构造函数

XMind: ZEN - Trial Version