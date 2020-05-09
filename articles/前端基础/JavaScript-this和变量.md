# this和变量

```javascript
  // undefined
  console.log(a);
  var a = function() { alert('a'); };
```

```javascript
  
  // Uncaught ReferenceError: b is not defined
  console.log(b);
  let b = function() { alert('b'); };

  ```

```javascript
  
  // ƒ c() { alert('c'); }
  console.log(c);
  function c() { alert('c'); }

  ```

```javascript
  
  // Uncaught ReferenceError: d is not defined
  console.log(d);
  class d {}

```

```javascript
  // Uncaught ReferenceError: x is not defined
  console.log(x);
  // 此处如果定义的是 var x = 1, 那么x还是挂载在window上，但是这个并没有变量提升，所以在之前访问，会报错
  window.x = 1;
```

```javascript
window.x = 1
// 1
console.log(x)
```