# 原型链

涉及内容:
1. new的实现
2. Function和Object在原型链上的关系


## new的实现
```javascript
  function createObject (obj）{
    let o = Object.create()
    o.__proto__ = obj.prototype
    const res = obj.apply(o, [].slice.call(arguments, 1))
    return typeof res === 'object' ? res || o : o
  }
```

```javascript
  Function.prototype.a = () => console.log(1);
  Object.prototype.b = () => console.log(2);
  function A() {}
  // 1
  A.a()
  // 2
  A.b()
  const a = new A();
  // 报错，a不是一个function。说明A的prototype不包括Function.prototype
  a.a();
  // 2
  a.b();

  // false
  console.log(a instanceof Function)
  // true
  console.log(a instanceof Object)
  // true
  console.log(A instanceof Function)
  // true
  console.log(A instanceof Object)

  // undefined
  A();
  // Uncaught TypeError: a is not a function
  a();

  const B = () => {};
  // Uncaught TypeError: B is not a constructor
  const b = new B();
```