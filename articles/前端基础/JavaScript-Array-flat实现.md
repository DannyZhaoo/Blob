# flat实现
flat函数的主要目的就是把传入的参数**扁平化**。

```javascript
Array.prototype.flat2 = function () {
    const arr = this  
    for (let i = 0; i < arr.length; i++) {
        if (typeof arr[i] === 'object' ) {
            arr.splice(i, 1, ...arr[i])
            i--
        }
    }
    return arr
}

let arr = [1,2,3,[4,5,[6,7]]]
console.log(arr.flat2())
```