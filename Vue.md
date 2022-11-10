#### Vue2 Diff算法

![](C:\Users\one\Pictures\Saved Pictures\9e159cbc836446c083f316baebf9c31c_tplv-k3u1fbpfcp-zoom-in-crop-mark_3024_0_0_0.webp)

```
1. 判断当前节点是否为静态节点isStatic，是则直接跳过
2. 更新当前节点的属性（直接替换为新属性）
3. 获取当前节点的新旧子节点，并判断子节点的类型
   新   空 数组 文本
   旧   空 数组 文本
3. 当新旧子节点都是数组的时候，进入diff算法，使用双指针对新旧子节点进行首尾对比，双指针分别向中间靠拢
```

```javascript
while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    // 先从左往右比较开始节点
    if (sameVnode(oldStartVnode, newStartVnode)) {
        // 进行节点复用，开始指针往右移动
        oldStartIdx ++
        newStartIdx ++
    }
    // 再从右往左比较结尾节点
    else if (sameVnode(oldEndVnode, newEndVnode)) {
        // 进行节点复用，结尾指针往左移动
        oldEndIdx --
        newEndIdx --
    }
    // 再分别比较开始节点和结尾节点
    else if (sameVnode(oldStartVnode, newEndVnode)) {
        // 进行节点复用，老开始指针往右移动，新结尾指针往左移动
        oldStartIdx ++
        newEndIdx --
    }
    else if (sameVnode(oldEndVnode, newStartVnode)) {
        // 进行节点复用，老结尾指针往左移动，新开始指针往右移动
        oldEndIdx --
        newStartIdx ++
    }
    // 经过以上处理还没有找到可复用的节点
    else {
        // 遍历新子节点列表，从老子节点列表中找到可复用的节点
        // 会为老子节点列表创建一个oldKeyToIdxMap
        // 这样新节点可以通过key快速找到可复用的老节点
        idxInOld = oldKeyToIdx.get(newStartVnode.key)
        if (!idxInOld) {
            // 没找到则创建新节点
        } else {
            // 找到了则进行节点复用，并移动
        }
    }
}
```

#### Vue2响应式

使用Object.defineProperty时，需要在外部单独声明一个变量来保存属性值，defineReactive利用闭包原理对其进行封装

```javascript
let bValue = 100
const obj = {}
Object.defineProperty(obj, 'age', {
    get() { return bValue },
    set(newValue) { bValue = newValue }
})
```

```javascript
// 将对象中的某个属性变成响应式
function defineReactive(data, key, val) {
    // 每个属性会记录自己的依赖(watcher)
    const dep = new Dep()
    if (arguments.length === 2) {
        val = data[key]
    }
    // 进行递归
    observe(val)
    Object.defineProperty(data, key, {
        enumerable: true, // 可配置
        configurable: true, // 可枚举
        get() {
            // 处于依赖收集阶段
            if (Dep.target) {
                // Dep.target.addDep(this)
                dep.depend()
            }
            return val
        },
        set(newValue) {
            if (val === newValue) { return }
        	val = newValue
        }
    })
}
```

```javascript
// 获取数组的原型对象
const arrayPrototype = Array.prototype
// 以arrayPrototype为原型创建一个新对象，这样访问如push时，会先访问自己的push，再访问arrayPrototype的push
const arrayMethods = Object.create(arrayPrototype)
// 改写数组的方法
const methodsNeedChange = ['push', 'pop', 'unshift', 'shift', 'splice', 'sort', 'reserve']
methodsNeedChange.forEach((methodName) => {
    // 先备份原来的方法
    const original = arrayPrototype[methodName]
    // 更改arrayMethods的数组方法
    def(arrayMethods, methodName, function() {
        // 这里能直接拿到__ob__，因为数组肯定是对象上的某个属性，之前已经被observe过了
        const ob = this.__ob__
        // 调用原方法实现原本的功能
        const result = original.call(this, arguments)
        // push\unshift\splice会往数组中插入新项，插入的项也要变成响应式
        let inserted = []
        switch(methodName) {
            case 'push':
            case 'unshifr':
                inserted = [...arguments]
                break
            case 'splice':
                inserted = [...arguments].slice(2)
                break
        }
        // 将插入的项变成响应式
        if (inserted.length) {
            ob.observeArray(inserted)
        }
        return result
    }, false)
})
```

```javascript
// 将一个对象的所有属性变为响应式
class Observer {
	constructor(value) {
    	// 为对象添加__ob__
        def(value, '__ob__', this, false)
        // 判断是否为数组，是数组则将其原型指向arrayMethods，这样数组就能访问改写后的方法了
        if (Array.isArray(value)) {
            Object.setProperyOf(value, arrayMethods)
            // 遍历数组的每一项，将其变成响应式
            this.observeArray(value)
        } else {
            // 遍历对象的属性，将其变成响应式
            this.walk(value)
        }
    }
    // 遍历对象的属性，将其变成响应式
    walk(value) {
        for (const key in value) {
            defineReactive(value, key)
        }
    }
    // 遍历数组的每一项，将其变成响应式
    observeArray(arr) {
        for (let i = 0, len = arr.length; i < len; i ++) {
            observe(arr[i])
        }
    }
}
```

```javascript
// 为需要响应式的对象添加Observer类的实例(__ob__)
function observe(value) {
    // 只处理对象
    if (typeof value !== 'object' && value !== null) return
    // 开始侦听对象
    const ob = value.__ob__ || new Observer(value)
    return ob
}
```

##### 响应式流程

```
1. 调用observe(value)，为value添加Observer类实例__ob__
2. Observer类执行构造函数new Observer(value)
2.1 如果value是普通对象
	2.1.1 调用walk方法，遍历对象的每个属性并调用defineReactive将其变成响应式
	2.1.2 defineReactive中会继续调用observe，递归将当前属性的子属性（如果有的话）变成响应式
2.2 如果value是数组
	2.2.1 生成arrayMethods对象，它的原型指向Array.prototype，它将改写数组的几个方法
	2.2.2 将value的原型指向arrayMethods，这样value就可以访问改写后的方法了，此时如果访问的是push\unshift\splice时，需要将插入的新项也变成响应式
	2.2.3 调用observeArray方法，遍历数组的每一项，并调用observe将其变成响应式
```

##### 依赖收集触发

```
1. 在组件挂载阶段会new Watcher()初始化一个watcher，Watcher类对应了vue3的ReactiveEffect类
2. 调用pushTarget，将当前Watcher入栈Dep.target = watcher
3. 执行render的过程中会访问响应式数据，响应式数据中的每个属性有自己的Dep实例dep，Dep类专门用于管理依赖，类似于vue3的targetMap，触发getter会dep.depend()收集当前组件的watcher作为依赖，当前的watcher会记录被哪些属性收集
4. 收集完依赖后，调用popTarget将当前Watcher出栈，返回父级Watcher
5. 触发setter时，会dep.notify()触发当前属性收集的所有watcher，watcher会重新执行updateComponent更新组件
```

