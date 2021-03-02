**Observer**

---



`src/core/observer/index.js`

```ts
class Observer {
  value: any;
  dep: Dep;
  vmCount: number;
  
  constructor(value) {
    this.value = value
    // 为每个Observer对象创建依赖收集的实例
    this.dep = new Dep()
    ...
    // 遍历对象进行observer
    this.walk(value)
  }
  
  walk (obj) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      // 对对象进行getter、setter
      defineReactive(obj, keys[i])
    }
  }
}

/**
 * Define a reactive property on an Object.
 */
export function defineReactive (
	obj,
  key,
  val,
  shallow?: boolean
) {
  const dep = new Dep()
  ...
  Object.defineProperty(obj, key, {
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        // dep.target = new watcher()
        // dep.target.addDep(this)
        // dep收集watcher
        dep.depend()
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = val
      if (newVal = value) {
        return
      }
      // dep.notify() => subs[i].update()=> watcher[i].update()进入render.patch更新组件
      dep.notify()
    }
  })
}
```

**Dep**

---

```ts
// watcher均代表实例对象，非构造函数
class Dep {
  constructor () {
    // 收集watcher的数组
    this.subs = []
  }
  
  // 收集watcher的方法,在watcher内的addDep方法被触发执行。
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }
  
  // 
  depend () {
    if (Dep.target) {
      // watcher.addDep 内会执行 addSub添加依赖，this = watcher
      Dep.target.addDep(this)
    }
  }
  
  // 执行每一个sub.update，等于执行watcher.update更新dom
  notify () {
    subs.forEach((sub) => {
      sub.update()
      })
  }
}

// target 等于是定义在全局，在watcher内被修改指向当前watcher
Dep.target = null
```

**Watcher**

---



Vue2，用到了vdom，一个组件的vdom，对应一个watch，所以一个组件内用到的所有数据的dep会收集同一个watch，这样watch的数量变小了，同时更新的颗粒度就变大了。