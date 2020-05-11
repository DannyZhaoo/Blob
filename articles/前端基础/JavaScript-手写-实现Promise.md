# 实现Promise
之前一直觉得，promise是es6比较难的一部分，一直都不太能理解。然后，就有同事提醒我说，可以实现一个简单版本的promise，有助于理解和使用promise。

发现实现一下原生的promise确实加深理解能力。这一点还是非常nice的。特此记录一下。

```javascript
  // 2020.5版本
  // 判断变量否为function
  const isFunction = variable => typeof variable === 'function'
  // 定义Promise的三种状态常量
  const PENDING = 'PENDING'
  const FULFILLED = 'FULFILLED'
  const REJECTED = 'REJECTED'

  class MyPromise {
    constructor (handle) {
      if (!isFunction(handle)) {
        throw new Error('MyPromise must accept a function as a parameter')
      }
      // 添加状态
      this._status = PENDING
      // 添加状态
      this._value = undefined
      // 添加成功回调函数队列
      this._fulfilledQueues = []
      // 添加失败回调函数队列
      this._rejectedQueues = []
      // 执行handle
      try {
        handle(this._resolve.bind(this), this._reject.bind(this)) 
      } catch (err) {
        this._reject(err)
      }
    }
    // 添加resovle时执行的函数
    _resolve (val) {
      const run = () => {
        if (this._status !== PENDING) return
        // 依次执行成功队列中的函数，并清空队列
        const runFulfilled = (value) => {
          let cb;
          while (cb = this._fulfilledQueues.shift()) {
            cb(value)
          }
        }
        // 依次执行失败队列中的函数，并清空队列
        const runRejected = (error) => {
          let cb;
          while (cb = this._rejectedQueues.shift()) {
            cb(error)
          }
        }
        /* 如果resolve的参数为Promise对象，则必须等待该Promise对象状态改变后,
          当前Promsie的状态才会改变，且状态取决于参数Promsie对象的状态
        */
        if (val instanceof MyPromise) {
          val.then(value => {
            this._value = value
            this._status = FULFILLED
            runFulfilled(value)
          }, err => {
            this._value = err
            this._status = REJECTED
            runRejected(err)
          })
        } else {
          this._value = val
          this._status = FULFILLED
          runFulfilled(val)
        }
      }
      // 为了支持同步的Promise，这里采用异步调用
      setTimeout(run, 0)
    }
    // 添加reject时执行的函数
    _reject (err) { 
      if (this._status !== PENDING) return
      // 依次执行失败队列中的函数，并清空队列
      const run = () => {
        this._status = REJECTED
        this._value = err
        let cb;
        while (cb = this._rejectedQueues.shift()) {
          cb(err)
        }
      }
      // 为了支持同步的Promise，这里采用异步调用
      setTimeout(run, 0)
    }
    // 添加then方法
    then (onFulfilled, onRejected) {
      const { _value, _status } = this
      // 返回一个新的Promise对象
      return new MyPromise((onFulfilledNext, onRejectedNext) => {
        // 封装一个成功时执行的函数
        let fulfilled = value => {
          try {
            if (!isFunction(onFulfilled)) {
              onFulfilledNext(value)
            } else {
              let res =  onFulfilled(value);
              if (res instanceof MyPromise) {
                // 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调
                res.then(onFulfilledNext, onRejectedNext)
              } else {
                //否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
                onFulfilledNext(res)
              }
            }
          } catch (err) {
            // 如果函数执行出错，新的Promise对象的状态为失败
            onRejectedNext(err)
          }
        }
        // 封装一个失败时执行的函数
        let rejected = error => {
          try {
            if (!isFunction(onRejected)) {
              onRejectedNext(error)
            } else {
                let res = onRejected(error);
                if (res instanceof MyPromise) {
                  // 如果当前回调函数返回MyPromise对象，必须等待其状态改变后在执行下一个回调
                  res.then(onFulfilledNext, onRejectedNext)
                } else {
                  //否则会将返回结果直接作为参数，传入下一个then的回调函数，并立即执行下一个then的回调函数
                  onFulfilledNext(res)
                }
            }
          } catch (err) {
            // 如果函数执行出错，新的Promise对象的状态为失败
            onRejectedNext(err)
          }
        }
        switch (_status) {
          // 当状态为pending时，将then方法回调函数加入执行队列等待执行
          case PENDING:
            this._fulfilledQueues.push(fulfilled)
            this._rejectedQueues.push(rejected)
            break
          // 当状态已经改变时，立即执行对应的回调函数
          case FULFILLED:
            fulfilled(_value)
            break
          case REJECTED:
            rejected(_value)
            break
        }
      })
    }
    // 添加catch方法
    catch (onRejected) {
      return this.then(undefined, onRejected)
    }

    // 实例方法
    finally (cb) {
      return this.then(
        value  => MyPromise.resolve(cb()).then(() => value),
        reason => MyPromise.resolve(cb()).then(() => { throw reason })
      );
    }

    // 添加静态resolve方法
    static resolve (value) {
      // 如果参数是MyPromise实例，直接返回这个实例
      if (value instanceof MyPromise) return value
      return new MyPromise(resolve => resolve(value))
    }

    // 添加静态reject方法
    static reject (value) {
      return new MyPromise((resolve ,reject) => reject(value))
    }

    // 添加静态all方法
    static all (list) {
      return new MyPromise((resolve, reject) => {
        /**
         * 返回值的集合
         */
        let values = []
        let count = 0
        for (let [i, p] of list.entries()) {
          // 数组参数如果不是MyPromise实例，先调用MyPromise.resolve
          this.resolve(p).then(res => {
            values[i] = res
            count++
            // 所有状态都变成fulfilled时返回的MyPromise状态就变成fulfilled
            if (count === list.length) resolve(values)
          }, err => {
            // 有一个被rejected时返回的MyPromise状态就变成rejected
            reject(err)
          })
        }
      })
    }

    // 添加静态race方法
    static race (list) {
      return new MyPromise((resolve, reject) => {
        for (let p of list) {
          // 只要有一个实例率先改变状态，新的MyPromise的状态就跟着改变
          this.resolve(p).then(res => {
            resolve(res)
          }, err => {
            reject(err)
          })
        }
      })
    }

    // 添加静态allSettle方法
    static allSettle () {
      return new Promise((resolve, reject) => {  
        const res = []
        let num = 0
        for(let i = 0; i < array.length; i++) {
          array.then(data => {
            res[i] = data
          }, error => {
            res[i] = error
          }).finally(() => {
            num++
            if (num === array.length) {
              resolve(res)
            }
          })
        } 
      })
    }
  }

// test
new MyPromise (resolve => {
  console.log(1);
  resolve(3);
  // 4比3先输出，是因为then中的solve函数先注册
  MyPromise.resolve().then(()=> {
    console.log(4)
  })
  MyPromise.resolve().then(()=> {
    console.log(5)
  })
}).then(num => {
  console.log(num)
});
console.log(2);

```




之前版本
```javascript
/**
 * 实现一个promise
 */

function Promise(executor) {
  var self = this;
  self.status = 'pending'; // promise的状态不能被外部更改
  self.data = undefined; // 执行函数的返回值
  self.onResolvedCallback = []; // promise的then函数可能被调用多次
  self.onRejectCallback = [];

  // 内部封装的resolve，通过这个resolve传递给then函数，具体再执行内部的函数，触发改变Promise状态
  function resolve(value) {
    // 不论value的值是什么，都是希望通过resolve和reject来处理数据
    if (value instanceof Promise) {
      // 执行完value再辞调用这个resolve和reject
      return value.then(resolve, reject);
    }
    setTimeout(function() {
      if (self.status !== 'pending') {
        return
      } else {
        self.status = "resolved";
        self.data = value;
        for(var i = 0; i < self.onResolvedCallback.length; i++) {
          self.onResolvedCallback[i](value);
        }
      }  
    });
  }

  function reject(reason) {
    setTimeout(() => {
      if(self.status !== 'pending') {
        return
      } else {
        self.status = 'rejected';
        self.data = reason;
        for(var i = 0; i < self.onRejectCallback.length; i++) {
          self.onRejectCallback[i](reason);
        }
      }
    });
  }

  try {
    // 所有promise的参数函数都执行在try-catch中
    executor(resolve, reject); 
  } catch(e) {
    reject(e);
  }
}

Promise.prototype.then = function(onResolved, onRejected) {
  var self = this;
  var promise2;

  // 值的透传
  onResolved = typeof onResolved === 'function' ? onResolved : function(value) { return value; };
  onRejected = typeof onRejected === 'function' ? onRejected : function(reason) { throw reason;};

  if(self.status === 'resolved') {
    return promise2 = new Promise(function(resolve, reject) {
      // 所有then的部分异步回调
      setTimeout(function() {
        try {
          // x可能是一个Promise对象，即thenable，要以最保险的方式调用x上的then方法
          var x = onResolved(self.data);
          resolvePromise(promise2, x, resolve, reject);
        } catch(e) {
          reject(e);
        }
      });
    });
  }

  if(self.status === 'reject') {
    return promise2 = new Promise(function(resolve, reject) {
      // then函数一般是异步调用
      setTimeout(() => {
        try {
          var x = onRejected(self.data);
          resolvePromise(promise2, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      });
    });
  }
  
  if(self.status === 'pending') {
    return promise2 = new Promise(function(resolve, reject) {
      // 在resolve中已经是异步调用
      self.onResolvedCallback.push(function(value) {
        try {
          var x = onResolved(value);
          resolvePromise(promise2, x, resolve, reject);
        } catch(e) {
          reject(e);
        }
      });
      
      self.onRejectCallback.push(function(reason) {
        try {
          var x = onRejected(value);
          resolvePromise(promise2, x, resolve, reject);
        } catch(e) {
          reject(e);
        }
      })
    });
  }
}

Promise.prototype.valueof = function() {
  return this.data;
}

Promise.prototype.catch = function(onRejected) {
  return this.then(null, onRejected);
}

function resolvePromise(promise2, returnValue, resolve, reject) {
  var then;
  var thenCalledOrThrow = false;

  if(promise2 === returnValue) {
    return reject(new TypeError('chaining cycle detected for primise'));
  }

  if(returnValue instanceof Promise) {
    if(returnValue.status === 'pending') {
      // 确定一下返回值 
      returnValue.then(function(value) {
        // If/when x is fulfilled, fulfill promise with the same value；If/when x is rejected, reject promise with the same reason.
        resolvePromise(promise2, value, resolve, reject);
      }, reject);
    } else {
      // 返回值的状态同步
      returnValue.then(resolve, reject);
    }
    return;
  }

  if((returnValue !== null) && ((typeof returnValue === 'object') || (typeof returnValue === 'function'))) {
    try {
      // 兼容返回的是thenable，根据Promises/A+中规定
      then = returnValue.then;
      if(typeof then === 'function') {
        then.call(returnValue, function rs(y) {
          // the first call takes precedence
          if(thenCalledOrThrow) return;
          thenCalledOrThrow = true;
          resolvePromise(promise2, y, resolve, reject);
        }, function rj(r) {
          if(thenCalledOrThrow) return;
          thenCalledOrThrow = true;
          reject(r)
        });
      } else {
        resolve(returnValue);
      }
    } catch (e) {
      if(thenCalledOrThrow) return;
      thenCalledOrThrow = true;
      reject(e);
    }
  } else {
    resolve(returnValue);
  }
}

Promise.prototype.finally = function(fn) {
  return this.then(function(v){
    setTimeout(fn);
    return v;
  }, function(r){
    setTimeout(fn);
    throw r;
  })
}

Promise.all = function(promises) {
  return new Promise(function(resolve, reject){
    var resolvedCounter = 0;
    var promiseNum = promises.length;
    var resolvedValues = new Array(promiseNum);
    for(var i = 0; i < promiseNum; i++){
      (function(i){
        Promise.resolve(promise[i]).then(function(value) {
          resolvedCounter++;
          resolvedValues[i] = value;
          if(resolvedCounter === promiseNum) {
            return resolve(resolvedValues);
          }
        }, function(reason) {
          return reject(reason);
        })
      })(i)
    }
  })
}

Promise.race = function(promises) {
  return new Promise(function(resolve, reject) {
    for (var i = 0; i < promises.length; i++) {
      Promise.resolve(promises[i]).then(function(value) {
        return resolve(value)
      }, function(reason) {
        return reject(reason)
      })
    }
  })
}

Promise.resolve = function(value) {
  var promise = new Promise(function(resolve, reject) {
    resolvePromise(promise, value, resolve, reject)
  })
  return promise
}

Promise.reject = function(reason) {
  return new Promise(function(resolve, reject) {
    reject(reason)
  })
}


new Promise(resolve => {
  console.log(1);
  resolve(3);
  Promise.resolve().then(()=> console.log(4))
}).then(num => {
  console.log(num)
});
console.log(2)

// 答案是1243
```