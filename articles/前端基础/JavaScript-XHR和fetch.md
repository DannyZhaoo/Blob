# XHR与fetch


涉及内容：
1. XML与fetch异同
2. fetch的好处

### XML与fetch异同

- 相同点：都能发送异步ajax请求
- 不同点：
  1. 不能设置timeout
  2. 不能发送cookie
  3. fetch原生不支持404，500报错

### fetch的好处
1. fetch更加现代话，浏览器直接支持，直接返回promise，不用xhr再做一层封装。
2. fetch方法返回和使用的request，response都是原生对象，配置更加灵活，而xhr必须使用实例来进行处理。