# Promise可能的问题

总结一下Promise，这个在我们开发中可能会遇到的问题。

1. 首先是then函数一定是异步的。不论定义promise的函数体是否是异步的，`then`函数都是异步的.

   ```javascript
   new Promise(resolve => {
       console.log(1);
       resolve(3);
   }).then(num => {
       console.log(num)
   });
   console.log(2);
   // 123
   ```

   Ps: 在源码中then的执行函数都是放在异步函数中执行

2. 由于then是异步调用的，调用顺序由注册`then`的顺序和`promise`的状态一起决定的。

   ```javascript
   // promise的执行主体是同步的，也就是说promise的状态是已完成后再注册的then中的参数函数也会被直接执行
   new Promise(resolve => {
       console.log(1);
       resolve(3);
       // 4比3先输出，是因为then中的solve函数先注册
       Promise.resolve().then(()=> {
         console.log(4)
       })
       Promise.resolve().then(()=> {
         console.log(5)
       })
   }).then(num => {
       console.log(num)
   });
   console.log(2);
   // 12453
   
   
   // promise的执行主体是异步的
   new Promise(resolve => {
       console.log(1);
       setTimeout(function() {
          resolve(3)       
       }, 1000);
       Promise.resolve().then(()=> {
         console.log(4)
       })
   }).then(num => {
       console.log(num)
   });
   Promise.resolve().then(()=> {
      console.log(5)
   });
   console.log(2);
   // 12453
   ```

3. 同一个promise可以被`then`多次调用，但这个then中的参数都是一样的，而且被依次调用。

   ```javascript
   var p = new Promise(function(resolve, reject) {
       resolve(3);
   });
   p.then(function(value){
   	value += 3; // 6
       console.log(value);
   });
   p.then(function(value){
   	value += 2; // 5
       console.log(value);
   });
   // 65
   ```

   Ps：在promise源码中可以看到promise有两个队列分别存放着then调用时注册的函数，当promise的status的状态变为`fulfilled`和`reject`后就依次调用。

4. then这个函数返回的是新的promise。

   ```javascript
   var p = new Promise(function(resolve, reject) {
       resolve(3);
   });
   var q = p.then(function(value){
   	value += 3; // 6
       console.log(value);
   }).then;
   p === q // false
   ```

5. 值可以透传，不论是resolve或是reject都可以透传。

   ```javascript
   var p = new Promise(function(resolve, reject) {
       resolve(3);
   });
   p.then()
    .then()
    .then(function(value) {
       console.log(value);
   })
   ```

6. 在链式调用中，then函数返回的promise的状态由then参数中的`resolved`函数或者`reject`函数的返回值决定。而且在then中的`reject`或`resolve`函数的返回值，会根据返回值得情况确定then返回的promise的状态。如果返回的数字，新的promise的状态就是fulfilled，如果返回的值时一个promise，则then返回的promise的状态跟参数函数返回的promise状态一致。

   ```javascript
   var p = new Promise(function(resolve, reject) {
       resolve(3);
   });
   p.then(function(value) {
       return Promise.reject(value)
   }).then(function(value){
       console.log("resolve", value);
   }, function(reason){
       console.log("reject", reason);
       
   });
   // reject 3
   
   var p = new Promise(function(resolve, reject) {
       resolve(3);
   });
   p.then(function(value) {
       return Promise.resolve(value)
   }).then(function(value){
       console.log("resolve", value);
   }, function(reason){
       console.log("reject", reason);
   });
   // resolve 3
   ```

7. 链式调用中then中`reject`函数只能捕获当前`then`之前的错误，不能获取当前then的`resolve`函数。

   ```javascript
   var p = new Promise(function(resolve, reject) {
       resolve(3);
   });
   p.then(function(value) {
       throw new Error(4);
   }, function(reason){
       console.log("reject", reason);
   }).catch(function(e){
       console.log("catch", e);
   });
   // catch Error: 4
   ```

8. then中链式调用要明确`catch`或者`reject`函数解决之后，返回的promise状态由`catch`中的函数或者`reject`函数的返回值决定。这中间的别的`resolve`函数等都忽略。

   ```javascript
   var p = new Promise(function(resolve, reject) {
       reject(3);
   });
   p.then(function(value) {
       throw new Error(4);
   }, function(reason){
       console.log("reject", reason);
       return 5;
   }).then(null，function(e){
       console.log("catch", e); // 忽略
   }).then(function(value){
   	console.log(value);
   });
   // reject 3
   // return 5
   ```

   ps: `catch`函数在源码中是调用了`then(null, onRejected)`

