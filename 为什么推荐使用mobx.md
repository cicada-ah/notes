

常规用法的例子

**El:**

```tsx
// father.ts 
import store from './store'

// 我们有一个Observer生成的数据:value
// 当value更新时，不会触发Fatcher,re-run
// 而仅仅做到了更新被<Observer>包裹的<div>
function Father () {
  console.log('我没有被执行')
  return (
  <>
    <Observer>
      {
        () => {
          console.log('只有我被执行了')
          return <div>{stroe.value}<div>
        }
      }
    </Observer>
    <div>与我无关</div>
  </>
  )
}
```



**从表象的使用来看，我有什么样的思索过程:**

Mobx，在使用的表象上，就像是实现一个子组件的语法糖，而实际这个子组件内存在一个`forceUpdate`的hook，在初始化调用被`Proxy`代理的对象时，触发getter，就会收集这个`hook`，就能做到在对象被setter时，重新触发被收集的hook，那么对应的<Observer>子组件就会被触发。所以mobx做了三件事：

1. 和vue类似，通过响应式原理，利用Proxy代理对象，实现依赖收集/触发。

2. 在vue里是收集`new watch`，watch实例可以简单理解为`react`中的render。而

   mobx相当巧妙的，收集了各个用来强制更新组件的hook。

3. 巧妙形成子组件，通过<Observer>来创建属于自己的空间。

所以mobx有两个相当巧妙的神来之笔:

 **1. <Observer>形成子组件，为了创建一个forceUpdate；**

 **2. 创建了一个foceUpdate的hook，利用hook触发会更新组件的能力，来达到更新我们被<Observer>包裹的组件**



> 实现方式参考了vue，可能mobx在依赖方式可能有所不同。

60行具体实现一个mobx-react-lite:

```ts
// observer.js
import {useState, useCallback} from 'react'
import {Dep} from './observe'

// 每一个对象的dep中收集着update这个hook，触发对象的set时调用这个hook
// 剩下就是利用react自身的能力去更新
const useUpdate = () => {
    const [_, setState] = useState(0);
    return useCallback(() => {
      setState((num) => num + 1);
    }, []);
  }

// <Observer>组件内部就是为了让Proxy代理的对象收集到forceUpdate
// 这样当修改Proxy的时候，就可以把对象手机的forceUpdate全部触发，因为useUpdate实质是个useHook，所以Observer会重新执行
const Observer = (props) => {
    const forceUpdate = useUpdate()
    // 把一个全局对象Dep.target指向forceUpdate
    // 这样当对象的getter触发，就自然收集到了Dep.target
    Dep.target = forceUpdate
    // children就是我们传入
    const render = props.children()
    return render
}
export default Observer
```

```ts
// 利用vue的订阅收集和发布的Dep模型，实质创建一个管理数组的对象。
export class Dep {
    constructor () {
        this.subs = []
    }
    // 在getter时收集依赖
    // 在mobx里，addSub收集的<Observer>里的forceUpdate
    addSub (sub) {
        if (!this.subs.find((subItem) => subItem === sub)) {
            this.subs.push(sub)
        }
    }
    // 调用收集依赖的方法，之所以不直接用this.addSub收集，是为了更好的解藕，函数功能专一
    // 在vue中，depend触发是在watch对象里的，更需要功能点专一
    depend () {
        if(Dep.target) {
            this.addSub(Dep.target)
        }
    }
    // setter时触发所收集依赖
    notify () {
        this.subs.forEach((sub) => {
            sub()
        })
    }
}

// 挂载到构造函数上，其实充当的作用是全局变量的作用
Dep.target = null

// Proxy代理，修改getter\setter进行依赖收集
export const observe = (obj) => {
    // new一个实例，这样每一个对象都能收集一个Dep实例，或者说是一个数组用来管理属于自己的hook
    const dep = new Dep()
    const proxyObj = new Proxy(obj, {
        get(target, key) {
            // 收集forceUpdate
            dep.depend()
            return target[key]
        },
        set(target, key, newVal) {
            const oldVal = target[key]
            if (newVal === oldVal) {
                return false;
            }
            // 触发forceUpdate
            dep.notify()
            target[key] = newVal
            return true;
        }
    })
    return proxyObj
}
```

---

#### 二、为什么极力推荐mobx？（主观想法）

通过在猜想实现mobx的过程中，我体会到了mobx在状态管理方面的强大，并且理性上感受到了三点优点：

* ui和数据状态分离
* hooks闭包问题
* 更加精细的更新的机制



##### ui和数据状态分离

以往在使用`hook`的时候，假设有个loading，`[loading, setLoading]=useState(true)`

El:

```tsx
const Loading = () => {
  [loading, setLoading] = useState(true)
  return <spin loading={loading}></Spin>
}
```

而在mobx里，不用在组件里声明变量，可以更舒适的处理ui和逻辑的结构：

El:

```tsx
import store from './mobx-store'

const Loading = () => {
  return <Observer>
  {() => {
      return <spin loading={store.loading}></Spin>
    }}
  </Observer>
}
```

而组件结构可以像抽离css一样（建议以后分为view和model文件，彻底分离）：

<img src="https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20201224132738998.png" alt="image-20201224132738998" style="zoom:50%;" />

以组件级别的去管理状态。

> 为什么要提数据状态和ui分离(这样玄之又玄的概念)？
>
> 数据状态和ui分离，就像是css样式和hooks分离，在多人协作开发时，单组件结构就相当庞杂，数据\数据处理\ui满天飞，复杂组件维护及其难受。useState的数据形式，相当离散。
>
> 提高聚合的好处，在于方便管理，例如子组件的方法在父组件里调用是相当费劲的，主要原因在于方法引用了子组件里的useState、function，所以没法提升到全局去，而交给mobx里管理，就能很轻松在组件间传递方法。(这只是很小的一部分好处)
>
> model和view的彻底分离：
>
> 1. 减少props传递，需要组件数据，通过import导入即可，相比hook函数中useState被限制在了函数体内，也导致依托useState的函数，被封闭在其中，难以周转复用到其它组件内。
>
>    (例子：当你在某个组件定义了一个state，在很多组件内到处用，某个定位bug的场景下，你想找这个数据哪里来的，真会得累死，想不到在哪来的。但是如果是import进的，1秒就能找到。而且彻底分离的一个好处，不用考虑这个数据是哪里来的：组件内？或者mobx定义里？这样的心智负担。会很容易发现是某个文件import进来的。哪怕是loading这样ui强相关的，因为你还是不清楚以后setloading会不会被传到子组件里，所以干脆早点放到mobx里管理)
>
> 2. 庞杂的状态数据，及可能来自于内部的useState也可能来自其它组件，业务代码就混乱揉杂在一起，业务逻辑都拥挤在狭小的hook函数里，这就造成有些逐渐复杂模块的难以维护，出现bug也不知道从何定位等问题。之所以出现这些问题，主要原因就是数据被定义在函数(或者叫view)内，非常不灵活。当data和action抽离出了组件函数，就能非常灵活。
>
> 雅典娜内的例子：
>
> El:
>
> <img src="https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20201224170030642.png" alt="image-20201224170030642" style="zoom: 25%;" />
>
> 

##### hooks闭包问题

常见hook内的闭包问题，就是在副作用里取不到异步期间被修改的最新值,解决方式通常是通过useRef.

El:

```tsx
const El () => {
  const [val,setVal] = useState(0)
  useEffect(() => {
    setTimeout(() => {
      console.log(val) // val: 0
    },1000)
  },[])
  
  return <div onclick={() => {setVal(10)}}></div>
}
```



而通过mobx创建的值，因为总是在stroe对象上，所以总能通过引用获取到最新的值，就像class组件，通过this实例对象去获取一样，就不会再有这样的心智负担。

El:

```tsx
const El () => {
  useEffect(() => {
    setTimeout(() => {
      console.log(stroe.val) // val: 10
    },1000)
  },[])
  
  return <div onclick={() => {stroe.val = 10}}></div>
}
```



##### 更加精细的更新的机制

mobx更新组件的范围是所包裹的内容，而如果期望通过hook、redux、context手段来达到同样效果，就需要把所包裹范围写在一个子组件里,like this:

El:

```tsx
// 因为只有div用到了context这个全局状态，所以只希望div被更新而Father不被执行，
const Father = () => {
  const store = useContext(context)
  return (
  <>
    <div>{store.context}</div>
    <div>123</div>
  </>
  )
}

----------------------------------
// div放在一个组件里
const Children = () => {
  const store = useContext(context)
  return <div>{store.context}</div>
}

const Father = () => {
  return (
  <>
    <Children/>
    <div>123</div>
  </>
  )
}
```

这样的问题有三点：

1. 不够优雅，需要为了`div`再新起一个file.ts文件，再import 进Facher文件。
2. 不太符合对组件复用性划分的定义，因为这个`div`可能只在Fatcher里使用一次。
3. redux/context需要做好之前所说的功能划分，就需要创建很多个其实意义上说父子逻辑相关的`React.context`实例，不然就有重渲染的性能问题，那么就需要创建很多redux/context去管理。



此时mobx数据监听模式(无改了某一个值导致其它地方被更新的，心智担心)，也只需要很简单的在`fatcher`组件里包裹一层即可实现局部精准更新的优势就体现出来了。

