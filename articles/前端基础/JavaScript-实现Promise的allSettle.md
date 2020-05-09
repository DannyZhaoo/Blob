# 实现Promise的allSettle方法
```JavaScript
Promise.allSettle2 = function (array) {  
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
```