# 理解Generator函数

## 定义

Generator函数是一个可以由外部控制和改变函数执行顺序和执行过程的一个函数。

## 应用

由于可以人为的控制函数执行，所以这个函数可以很方便的用来使异步操作同步化。这个也是我主要解释的内容。

还可以用来，例如：给对象添加Iterator接口，实现状态机以及控制流管理等等。

## 用法

具体用法，参考Generator.xmind文件，这个文件上组织的内容更方便记忆和分辨。

## 异步应用

这个部分是我之前看了很多遍都没看懂的部分，现在看的懂了，记录一下。

首先以前的异步编程有如下几个方法：

- 回调函数
- 事件监听
- 发布/订阅
- Promise事件

而Generator函数的概念类似于**协程**的概念，就是可以多个线程相互交互来完成一个任务。但是由于JavaScript是单线程的原因，Generator更类似于一个**半协程**的概念，它可以交出和收回执行权，但总归代码还是在一个线程上进行的。

Generator能封装异步的原因是：

- 可以暂停执行和恢复执行
- 函数体内外能数据交换
- 有错误处理机制

Generator虽然可以封装异步操作，但是如果异步操作很多的话，就不能一直人为控制Genrator的一步一步执行了。

所以就出现了如下**Thunk**和**Co**内容。这两个内容都是用来自动执行Generator函数的。

Thunk函数在JavaScript中是把多参数函数替换成一个接收回调函数的单参数函数，然后通过在回调函数中进一步执行Generator的next，这样就能自动执行下一步的内容。具体代码如下

```javascript
var fs = require('fs');
var readFileThunk = thunkify(fs.readFile);

function thunkify(fn) {
  return function() {
    var args = new Array(arguments.length);
    var ctx = this;

    for (var i = 0; i < args.length; ++i) {
      args[i] = arguments[i];
    }

    return function (done) {
      var called;

      args.push(function () {
        if (called) return;
        called = true;
        done.apply(null, arguments);
      });

      try {
        fn.apply(ctx, args);
      } catch (err) {
        done(err);
      }
    }
  }
};

var gen = function* (){
  // yield后函数必须是thunk函数
  var r1 = yield readFileThunk('./iterate.js');
  console.log(r1.toString());
  var r2 = yield readFileThunk('./promise.js');
  console.log(r2.toString());
};


// 手动用法
// var g = gen();

// var r1 = g.next();
// 返回的是trunk，接收回调函数
// r1.value(function (err, data) {
//   if (err) throw err;
//   var r2 = g.next(data);
//   r2.value(function (err, data) {
//     if (err) throw err;
//     g.next(data);
//   });
// });

function run(gen){
  var g = gen();

  function next(data){
    var result = g.next(data);
    if (result.done) return result.value;
    result.value(next);
  }

  next();
}

run(gen);
```



co模块是通过在Promise的回调中执行Generator的next函数，来完成一步一步自动执行下一步的操作。

```javascript
var fs = require('fs');

var readFile = function (fileName){
  return new Promise(function (resolve, reject){
    fs.readFile(fileName, function(error, data){
      if (error) return reject(error);
      resolve(data);
    });
  });
};

var gen = function* (){
  // yield后函数的返回值必须是promise
  var f1 = yield readFile('./iterate.js');
  var f2 = yield readFile('./promise.js');
  console.log(f1.toString());
  console.log(f2.toString());
};

// 手动用法
// var g = gen();

// g.next().value.then(function(data){
//   g.next(data).value.then(function(data){
//     g.next(data);
//   });
// });



function run(gen){
  var g = gen();

  function next(data){
    var result = g.next(data);
    if (result.done) return result.value;
    result.value.then(function(data){
      next(data);
    });
  }

  next();
}

run(gen);
```

最后再看Generator函数时，会发现去掉`yield`，代码看起来就非常像同步的代码。