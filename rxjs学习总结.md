优点：

rxjs通过观察者模式使得数据处理思维成为了线性的(和promise的线性思考量类似)，

又因为函数式编程能力，使得逻辑高内聚、耦合低，数据流动的过程是一个个强功能、单一的函数处理。

其它优点来自函数式编程，具有这个编程模式所有优点。

- 同步与异步的统一
- 获取和订阅的统一
- 现在与未来的统一
- 可组合的数据变更过程

缺点：

学习有成本，不一定适合团队协作。

语义化差，因为功能单一，所以api多。

其它缺点待定。

---

ReactiveX 结合了 [观察者模式](https://en.wikipedia.org/wiki/Observer_pattern)、[迭代器模式](https://en.wikipedia.org/wiki/Iterator_pattern) 和 [使用集合的函数式编程](http://martinfowler.com/articles/collection-pipeline/#NestedOperatorExpressions)，以满足以一种理想方式来管理事件序列所需要的一切。

在 RxJS 中用来解决异步事件管理的的基本概念是：

- **Observable (可观察对象):** 表示一个概念，这个概念是一个可调用的未来值或事件的集合。
- **Observer (观察者):** 一个回调函数的集合，它知道如何去监听由 Observable 提供的值。
- **Subscription (订阅):** 表示 Observable 的执行，主要用于取消 Observable 的执行。
- **Operators (操作符):** 采用函数式编程风格的纯函数 (pure function)，使用像 `map`、`filter`、`concat`、`flatMap` 等这样的操作符来处理集合。
- **Subject (主体):** 相当于 EventEmitter，并且是将值或事件多路推送给多个 Observer 的唯一方式。
- **Schedulers (调度器):** 用来控制并发并且是中央集权的调度员，允许我们在发生计算时进行协调，例如 `setTimeout` 或 `requestAnimationFrame` 或其他。



```ts
class Observable {
    // 变量存储可观察对象，在后续手动调用subscribe，触发存储的可观察对象
    _subscribe = null
    constructor (subscribe) {
        this._subscribe = subscribe
    }
    // 订阅，observer(观察者),去可观察对象内消费next。
    subscribe (observer) {
        if (typeof observer === 'function') {
            let _observer = {
                next: observer,
            }
            this._subscribe(_observer)
        } else {
            this._subscribe(observer)
        }
    }
}
// 简单理解：传入Obervable的构造参数(是一个含有next的方法)，会暂时被构造函数保存到Observable.prototype._subscribe这个变量上
// 等到后续调用testOberve.subscribe(val)时，subscribe内部会调用原型上的属性testOberve._subscribe(val)，并把val作为参数传递给_subscribe
// 而val.next就可以拿到一开始传入进去的next(123)，123这个值,开发人员就可以在val.next这个方法中各种处理123。
let testOberve = new Observable((oberver) => {
    oberver.next(123)
    oberver.next(321)
    setTimeout(() => {
        oberver.next(213)
    }, 1000);
})
(oberver) => {
    oberver.next(123)
    oberver.next(321)
    setTimeout(() => {
        oberver.next(213)
    }, 1000);
}
testOberve.subscribe(function (number) {
    console.log(number)
})
```

---

## 拉取 (Pull) vs. 推送 (Push)

|          | 生产者                             | 消费者                             |
| -------- | ---------------------------------- | ---------------------------------- |
| **拉取** | **被动的:** 当被请求时产生数据。   | **主动的:** 决定何时请求数据。     |
| **推送** | **主动的:** 按自己的节奏产生数据。 | **被动的:** 对收到的数据做出反应。 |

**拉取**:

1. 每个Jvascript函数都是拉取体系,函数是数据的生产者，调用函数return一个值来消费函数。

2. Es2015 generator和iterators(function*),是另一种拉取体系，调用iterators.next消费

   iterator中的值，决定数据获取权在消费者。

**推送**:

1. RxJS，Observable 是多个值的生产者，并将值“推送”给观察者(消费者)。*（实际是Observable内实现多个next(1,2,3)等待观察者进入，并一次性传递给观察者）



### rxjs中的单播和多播

单播，一个observable只能被一个观察者消费。例子：

```ts
let observable = Rx.Observable.create(function subscribe(obsever) {
  observer.next(1)
  observer.next(2)
})
observable.subscribe(v => console.log(v))
observable.subscribe(v => console.log(v))
// 1,2,1,2
```

虽然observer,订阅了同一个observable，但是处理数据都是独立的，他们不会在同时处理next中的数值，可以理解为，传入subscribe的每一个(v => console.log(v))观察者，都在本次运行完，下一个观察者才继续传入消耗next的值。



多播，就是可以被多个观察者同时订阅，在同一时刻消费next的值。可以理解为：

```ts
// 单播
observer.next(1)
observer.next(2)
// 多播
[observer1,observer2].foreach((observer) => {observer.next(1)})
[observer1,observer2].foreach((observer) => {observer.next(2)})
```

所以当传入两个观察者时，他们会被存入一个数组里，然后在碰见next时，遍历数组，获取next的值。

```ts
let observable = Rx.Observable.create(function subscribe(obsever) {
  // 此时的observer是subscribe时存入的一个数组，[v => console.log(v), v => console.log(v)]
  observer.next(1)
  observer.next(2)
})
// 多播
let subject = new Rx.Subject()
// 当碰见观察者,先把观察者存入一个数组中
subject.subscribe(v => console.log(v))
subject.subscribe(v => console.log(v))
observable.subscribe(subject)

// 1，1，2，2
```

所以打印时，`1,1,2,2`，同时把next中的val，传给两个观察者[v => console.log(v), v => console.log(v)]



**Subject是消费主体，subScribe存入观察者集合等待调用next消费数据，next可以存入被消费的数据(数据)，Observable是创造被观察对象(数据)，subScribe调用单个观察者直接消费next，如果要多个观察者同时消费，需要Subject。**



所以Subject有next方法，可以作为观察者集合，传入到Obsrvable.subsrcibe(Subject)里面进行观察。

Observable、Subject实现:

```ts
class Observable {
    // 变量存储可观察对象，在后续手动调用subscribe，触发存储的可观察对象
    _subscribe = null
    constructor (subscribe) {
        this._subscribe = subscribe
    }
    // 订阅，observer(观察者),去可观察对象内消费next。
    subscribe (observer) {
        if (typeof observer === 'function') {
            let _observer = {
                next: observer,
            }
            // 直接消费
            this._subscribe(_observer)
        } else {
            this._subscribe(observer)
        }
    }
}

// Subject
class Subject {
  observers = []
  constructor () {
    
  }
  subscribe (observer) {
    if (typeof observer === 'function') {
      let _observer = {
        next: observer,
      }
      // 观察者存到数组，等调用next时，开始消费
      this.observers.push(observer)
    } else {
      this.observers.push(observer)
    }
  }
  next (val) {
    this.observers.forEach((observer) => {
      observer.next(val)
    })
  }
}
```





