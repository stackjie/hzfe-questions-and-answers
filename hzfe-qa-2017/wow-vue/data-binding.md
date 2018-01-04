# Vue 双向绑定底层实现原理与源码解析

## 神奇的defineProperty

`defineProperty`是ES5新增的API也是 Vue 实现双向绑定的一个核心，`defineProperty`可以给对象的属性加上`setter`和`getter`访问器，这样就能感知属性的变化。
```js
var obj = {};
Object.defineProperty(obj, 'a',  {
   configurable: true,
   enumerable: true,
   get: function () {
     console.log('a 被get啦!');
   },
   set: function () {
      console.log('a 被set啦!');
   }
});

obj.a;
// 输出 a 被get啦!

obj.a = 'abc';
// 输出 a 被set啦!
```
## Observer
Vue数据双向绑定依赖于`defineProperty`并且基于观察者模式来实现，Vue中的`Observer`对象实际上是把数据属性遍历及递归转换成访问器属性(`defineProperty`)来检测数据的变更。

```js
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number;

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0

    // 在当前对象中加入`__ob__`隐式属性属性的值为自身
    def(value, '__ob__', this)

    // 判断当前对象是否为数组，数组做特殊处理
    if (Array.isArray(value)) {
      const augment = hasProto
        ? protoAugment
        : copyAugment
      augment(value, arrayMethods, arrayKeys)
      this.observeArray(value)
    } else {
      // 遍历数据添加访问器属性
      this.walk(value)
    }
  }

  
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      // Vue封装好的函数，用于给数据添加访问器属性如果数据为数组或者对象继续递归添加
      defineReactive(obj, keys[i], obj[keys[i]])
    }
  }

  // 遍历数组给数组元素也添加访问器属性
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```
以上代码摘自Vue 源码，简单来说Observer就是一个观察者，观察数据的变动然后通过依赖收集器触发订阅者的回调。

### 如何监测对象的变化
上面的代码有一个`defineReactive`函数，此函数就是用来处理`Object`给`Object`添加访问器属性来监测数据变动，接下来我们逐步解析下这个函数的代码。

为了逐步讲解，以下代码将精简过，目前只是讲解这个函数是如何给对象添加访问器属性的，后面随着内容逐步延伸。
```js
/**
 * Define a reactive property on an Object.
 */
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)
  if (property && property.configurable === false) {
    return
  }

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      /* eslint-enable no-self-compare */
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```

### 怪异的数组

## Watcher

## Dep