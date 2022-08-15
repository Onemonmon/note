# 面试总结-Vue

## 1. Vue响应式数据原理

vue2通过Object.defineProperty来实现响应式数据，内部定义了一个defineReactive的函数来为对象上的属性动态添加getter和setter，然后通过Observer类来递归遍历对象的每一层属性，数组的响应式是通过重写部分数组的API实现的（push、pop、unshift、shift、splice、reverse、sort），为了复用数组原本的API，拷贝了一份数组的原型对象arrayMethods，然后重写该对象上的那几个API，在Observer中如果value是数组，则将value的原型指向arrayMethods，这样就能使用新的数组方法。

```javascript
// defineReactive.js
// 为对象上的某个属性添加响应式
function defineReactive(data, key, val) {
    if (arguments.length === 2) {
        val = data[key]
    }
    // 通过闭包的形式保存value
    let temp = val
    Object.defineProperty(data, key, {
        configurable: true,
        writable: true,
        enumerable: true,
        get() {
            return temp
        },
        set(newValue) {
            temp = newValue
        }
    })
}
```

```javascript
// observe.js
// 开始将对象转换成响应式
function observe(obj) {
    if (typeof obj !== 'object') {
        return
    }
    let ob
    if (obj.__ob__ !== undefined) {
        ob = obj.__ob__
    } else {
        ob = new Observer(obj)
    }
    return ob
}
```

```javascript
// Observer.js
// 递归遍历对象属性，进行响应式转换
class Observer {
    constructor(value) {
        Object.defineProperty(value, "__ob__", {
            value: this,
            enumerable: false
        })
        if (Array.isArray(value)) {
            Object.setPrototypeOf(value, arrayMethods)
        } else {
        	this.walk(value)
        }
    }
    // 遍历属性
    walk(value) {
        for(let key in value) {
            defineReactive(value, key)
        }
    }
}
```

```javascript
// array.js
// 重写数组的部分api
const arrayPrototype = Array.prototype
const arrayMethods = Object.create(arrayPrototype)
const arrayNeedChanges = ['pop', 'push', 'unshift', 'shift', 'splice', 'reverse', 'sort']
arrayNeedChanges.forEach(methodName => {
    const originalMethod = arrayPrototype[methodName]
    Object.defineProperty(arrayMethods, methodName, {
        value: function() {
            let insertVal = []
            const args = [...arguments]
            if (['push', 'unshift'].includes(methodName)) {
                insertVal = args
            } else if (['splice'].includes(methodName)) {
                insertVal = args.slice(2)
            }
            const res = originalMethod.call(this, args)
            return res
        }
    })
})
```

