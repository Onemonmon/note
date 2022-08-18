### 1. 数据类型

```
基本类型：String、Number、Boolean、Symbol、Null、Undefined、Bigint
变量名和值都保存在栈内存中

引用类型：Object、Array、Function、Date、RegExp
变量名和值的存储地址保存在栈内容中，值保存在堆内存中
```

```
Symbol用于生成一个唯一值，不能使用new Symbol()
Symbol() !== Symbol()
Symbol.for('name') === Symbol.for('name')
Symbol.for(key)方法会将创建的symbol放入全局的symbol注册表中，并且使用Symbol.for(key)时，不会每次都创建新的symbol，而是会先判断给定的key是否在注册表中，有则直接返回，否则再重新创建
```

```
Bigint用于表示一个任意精度的整数，不能使用new Bigint()
Bigint数据需添加后缀n（12n）
12n == 12
12n !== 12
如果与Number混合运算则需要类型转换
```

```
检测数据类型的方法：
> typeof
  能快速区分基本数据类型，但无法区分Object、Array、Null，都返回Object
> instanceof
  能区分Object、Array、Function，适用于判断自定义类实例，但无法判断基本数据类型
> Object.prototype.toString.call()
  精准判断
```

### 2. 变量声明

```
> var
  没有块级作用域，可以跨域访问
  可以先使用后声明（存在变量提升：JS执行前会将var、function的变量提前声明）
  可以重复声明，后面重复声明的变量会覆盖之前的
  var定义的全局变量会绑定在顶层对象window上
> let
  先定义后使用（存在暂时性死区：let的块级作用域开始，到初始化的位置），在暂时性死区中使用变量会报错
  不可重复声明
> const
  不可重复声明
  声明时必须初始化
  声明的常量不可重新赋值，但其只保证了变量名指向的地址不变，不能保证该地址保存的数据不变
```

### 3. 作用域与作用域链

```
作用域是代码运行过程中某些变量、函数的访问范围，用于隔离变量
> 全局作用域
  在代码的任何位置都能访问到的变量具有全局作用域
  包括：最外层函数、最外层变量、未定义就直接赋值的变量、window对象及属性
> 函数作用域
  在函数内部声明的变量拥有函数作用域，外层作用域无法访问内层作用域的变量
> 块级作用域
  ES6新增，通过let、const声明变量拥有块级作用域
```

```
作用域链：使用变量时现在当前函数作用域中取值，如果没有找到，则会向上级作用域（若是函数，则要到创建这个函数的作用域去找）查找，直到查到全局作用域
```

### 4. 闭包

```
闭包是指有权访问另一个函数中的变量的函数，因为变量被引用，所以另一个函数执行结束后，变量不会被回收，因此可以用来封装一个私有变量。
形成条件：函数的嵌套、内部函数使用了外部函数的局部变量
作用：保护函数的私有变量不受外部干扰，可以实现函数内部方法和变量的私有化
闭包函数使用后应该及时清除引用（置为null），不正当的使用闭包会造成内存泄漏
```

### 5. this指向

```
普通函数执行时，this指向window
当函数作为对象的方法执行时，this指向该对象
使用new调用时，this指向内部创建的对象
箭头函数的this会继承自定义时的外层函数的this，如果没有外层函数则指向window
DOM事件函数的this指向绑定事件的DOM元素
call、apply、bind动态绑定this
```

### 6. 原型与原型链

```
每个class和（非箭头）函数都有prototype原型对象，它有一个默认的constructor属性，用于记录实例是由哪个构造函数创建的
每个对象都有__proto__属性，它是一个对象，指向上层对象的原型对象，有constructor、__proto__两个属性
```

```javascript
Person.prototype.constructor === Person
p.__proto__ === Person.prototype
Person.prototype.__proto__ === Object.prototype
Object.prototype.__proto__ === null
```



### 9. EventLoop 事件循环

```
NodeJS中的事件循环分为6个阶段：
> timers阶段：执行setTimeout、setInterval回调
> pending callbacks阶段：执行上一轮循环未执行的I/O回调
> idle，prepare阶段
> poll阶段：执行大部分I/O回调，计算阻塞时间
  当poll阶段的队列不为空，则会遍历队列同步执行，直到清空队列或执行时间达到系统限制
  当poll阶段的队列为空，且有timers到期，则会进入到timers阶段
  当poll阶段的队列为空，且有setImmediate时，进入check阶段
  当poll阶段的队列为空，且没有setImmediate时，会阻塞在该阶段等待回调加入队列并执行
> check阶段：执行setImmediate回调
> close callback：执行一些关闭的回调

每个阶段结束后都会执行中间队列，先执行nextTick队列，再执行Promise队列
```







