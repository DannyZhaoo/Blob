# 连等

```javascript
  var a = { x: 1 };
  var b = a;
  a.x = a = { n: 1 };
  // {n: 1}
  console.log(a);
  //{x: {n: 1}}
  console.log(b);
```