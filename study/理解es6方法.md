# 理解es6方法

在class中定义的方法，不用写`function`的关键字。类的所有方法都定义在类的`prototype`属性上面。



判断一个变量是否是实例自身的属性（在this上）使用hasOwnProperty

获取实例对象的原型`Object.getPrototypeOf`。



class中不能添加私有属性和方法