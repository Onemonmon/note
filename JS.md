### 数据类型

#### 基本类型

String、Number、Boolean、Null、Undefined、Symbol、BigInt

##### Symbol

```javascript
Symbol用于生成一个唯一值，不能使用new Symbol()
Symbol() !== Symbol()
Symbol.for('name') === Symbol.for('name')
Symbol.for(key)方法会将创建的symbol放入全局的symbol注册表中，并且使用Symbol.for(key)时，不会每次都创建新的symbol，而是会先判断给定的key是否在注册表中，有则直接返回，否则再重新创建
```

##### BigInt

```
Bigint用于表示一个任意精度的整数，不能使用new Bigint()
Bigint数据需添加后缀n（BigInt(12) => 12n）
12n == 12
12n !== 12
如果与Number混合运算则需要类型转换
```

#### 引用类型

Object、Array、Function、Date、RegExp、Map、WeakMap、Set、WeakSet

##### Map

```javascript
const map = new Map()
// new Map([[key1, value1], [key2, value2]])
// Map的键key可以是任意类型的数据
map.set(key, value)
map.has(key)
map.get(key)
map.delete(key)
// 获取数据个数
map.size
// 遍历Map
const keys = map.keys() // [key1, key2]
const values = map.values() // [value1, value2]
const entries = map.entries() // [[key1, value1], [key2, value2]]
map.forEach((value, key) => {})
// 清空集合
map.clear()
```

##### WeakMap

```javascript
const wmap = new WeakMap()
// new WeakMap([[key1, value1], [key2, value2]])
// WeakMap的键名key只能是引用类型
let objectKey = { name: "lala", age: 18 }
wmap.set(objectKey, value)
wmap.has(objectKey)
wmap.get(objectKey)
wmap.delete(objectKey)
// key对应的引用被释放后，WeakMap中的键值对也会删除
objectKey = null
wmap.has(objectKey) // false
// WeakMap不可枚举，不存在遍历方法，不能获取size，不能clear
```

##### Set

```javascript
const set = new Set()
// new Set([value1, value2])
// Set的值可以是任意类型的数据
set.add(value)
set.has(value)
set.delete(value)
// 获取数据个数
set.size
// 遍历Set
set.forEach((value) => {})
// 清空集合
set.clear()
```

##### WeakSet

```javascript
const wset = new WeakSet()
// new WeakSet([value1, value2])
// WeakSet的值只能是引用类型
let objectValue = { name: "lala", age: 18 }
wset.add(objectValue)
wset.has(objectValue)
wset.delete(objectValue)
// WeakSet不可枚举，不存在遍历方法，不能获取size，不能clear
```

#### 检测数据类型

##### typeof

能快速区分基本数据类型，但无法区分Object、Array、Null，都返回object

```javascript
typeof "str" === 'string'
typeof 100 === 'number'
typeof true === 'boolean'
typeof undefined === 'undefined'
typeof Symbol() === 'symbol'
typeof BigInt() === 'bigint'
typeof function(){} === 'function'
typeof null === 'object'
typeof Object、Array、Date、RegExp、Map、WeakMap、Set、WeakSet === 'object'
```

##### instanceof

适用于判断自定义类的实例，但无法判断基本数据类型

```javascript
// 引用类型 instanceof Object === true 所以无法判断是否是普通对象
({}) instanceof Object === true
([]) instanceof Object === true
([]) instanceof Array === true
```

##### Object.prototype.toString.call

可以精准判断所有类型，返回`"[object Xxx]"`

### 变量声明

##### var

```
1. 没有块级作用域，可以跨域访问
2. 可以先使用后声明（存在变量提升：JS执行前会将var、function的变量提前声明）
3. 可以重复声明，后面重复声明的变量会覆盖之前的
4. var定义的全局变量会绑定在顶层对象window上
```

##### let

```
1. 不可重复声明
2. 先定义后使用（存在暂时性死区：let的块级作用域开始，到初始化的位置），在暂时性死区中使用变量会报错
```

##### const

```
1. 不可重复声明
2. 声明时必须初始化 const name = ''
3. 声明的常量不可重新赋值，值如果是对象，对象中的属性值可以改变
```

### 作用域

代码运行过程中对变量、函数的访问范围。

##### 全局作用域

在代码的任何位置都能访问到的变量具有全局作用域，包括：最外层函数、最外层变量、未定义就直接赋值的变量、window对象及属性。

##### 函数作用域

在函数内部声明的变量拥有函数作用域。

##### 块级作用域

通过let、const声明的变量拥有块级作用域。

##### 作用域链

使用变量时，先在当前函数作用域中取值，如果没有找到，则会向上级作用域（若是函数，则要到创建这个函数的作用域去找）查找，直到查到全局作用域。

### 闭包

```
定义：在函数A中定义并返回了函数B，使得函数B能访问到函数A中声明的变量，函数B执行时就会形成闭包，闭包函数使用后应该及时清除引用（置为null），否则造成内存泄漏。

作用：实现函数内部方法和变量的私有化

应用场景：节流、防抖、函数柯里化、vue3生命周期的定义
```

### this指向

```
1. 普通函数执行时fn()，this指向window
2. 当函数作为对象的方法执行时，this指向该对象
3. 使用new调用时，this指向构造函数内部创建的对象
4. 箭头函数的this指向定义时的外层函数的this，如果没有外层函数则指向window
5. DOM事件函数的this指向绑定事件的DOM元素
6. call、apply、bind动态绑定this
```

### 原型与原型链

![](C:\Users\one\Pictures\Saved Pictures\assets\images\prototype.webp)

```javascript
function Foo() {}
const f1 = new Foo()
// 原型对象Foo.prototype的constructor指向构造函数本身
Foo.prototype.constructor === Foo
// 实例f1的原型__proto__指向原型对象Foo.prototype
f1.__proto__ === Foo.prototype
// 原型对象Foo.prototype本身也是个对象
Foo.prototype.__proto__ === Object.prototype
// 原型链的尽头
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







