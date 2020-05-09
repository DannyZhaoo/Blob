# 块级作用域

ES6中作用域分为三种
- 全局作用域
- 函数作用域
- 块级作用域

块级作用域主要就是{}

```javascript
// 块级作用域
let a = 1;
// 1
console.log(a)
{
    // Uncaught ReferenceError: Cannot access 'a' before initialization 
    // 暂时性死区
    console.log(a) 
    let a = 2;
    console.log(a)
}
// Uncaught SyntaxError: Identifier 'a' has already been declared
let a = 3;
// 下面不会执行
console.log(a)

```