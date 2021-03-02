# 组件到diff渲染的实现

## 组件



### 组件的种类

1. **函数式组件(Functional component)**

   * 纯函数组件，没有自身状态，只接收外部数据。

   * 相同输入只返回同样的内容。

   * hooks是一种纯函数语法糖。

     

2. **有状态组件(Stateful component)**

   * class组件，有自身状态，生成instance。(通过this实现)
   * this是可变的，造成组件状态共享。

hooks纯函数例子([recompose](https://github.com/acdlite/recompose))：

```jsx 
// hooks
function Child() {
	const [state, setState] = useState('')
  return <div>{state}</div>
}

---------------------------------------------------------->
// pure
function Child({state, setState}) {
  return <div>{state}</div>
}

function recompose() {
  const [state, setState] = useState('')
  return <Child state={state} setState={setState} />
}
```



#### Hook(fc)和class组件的一些差异(非优缺点对比)

1. Hook:声明式(useEffect),声明所需要的依赖，依赖发生变化，执行副作用。(告诉需要什么，输出)

   class:命令式,  状态发生变化，运行某个副作用。(告诉怎么去做，输出)

2. 渲染一致性。hook可以通过ref"选择性退出"渲染一致性，而class却比较困难变为渲染一致性(闭包，但这不是hook的强项吗？)（CodeSendbox3个例子）

3. hook可能更加迎合未来的并发模式。（fc输出当前"帧"的状态，hook是个大闭包，state被闭包在了那一"帧"，而class的state,props修改会受到this的影响，而同步不同的输出状态）

   

> 正如hooks作者所说的，每一次渲染都有独立属于它自己的...所有。(props, state, useEffect, function...etc)
>
> [《Dan: useEffect完全指南》](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)



### 宫阙万间都做了Virtual DOM

组件的实质，是为了通过**JSX+数据**得到`vdom`：

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200903184929241.png" alt="image-20200903184929241" style="zoom:67%;" />



<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200903184909632.png" alt="image-20200903184909632" style="zoom: 33%;" />



1. `Virtual DOM`给前端带来了跨平台开发的能力，使得组件能够渲染在其它平台。(RN,Weex,WeApp,SSR)

   不仅提高了前端的作用地位，也极大的提高了其它平台的迭代速度。只需要一个请求一个js资源，就能更新原生内容UI。（ios端麻烦的发版流程）

2. aot编译，能够压缩代码体积。()

3. 函数式ui编程编为了可能。(函数里，如何进行真实dom的创建，优雅插入父组件，数据变化时，如何响应，移除，再更新)

---

## Virtual DOM



### vnode应该长什么样?

`vnode`是对`dom标签`的描述，通过js对象这样的一个描述，配合`DOM api `能够生成真实`DOM`, 配合`ios andrind api`能够被渲染为其它平台的ui界面，所以应该包含基本的标签名，各种属性，子节点等，`vnode`的设计可以非常灵活，这完全按照开发者的设计思路，一个基本的`div标签`可以被描述为这样:

```js
const eleVNode = {
  type: 'div', // 标签名
  props: {      // 属性
    style: {},
    class: {},
    event: {}
  },
  children: {  // 子节点
    type: 'p',
    props: null
  }
}
// 如果有多个子节点
const eleVNode = {
  ...,
  children: [
  ...
  ]
}

// 文本节点
const textVNode = {
  type: null,
  props: null,
  children: null, // 可以使用children存储文本内容，这样使得vnode更加简洁，复用性更高
  text: '文本'
}
```

除了普通的html标签，还会遇到组件类型，组件不是唯一的，并且需要有所产出vnode。**所以我们可以通过type是否是大写字母开头字符串来检测是否是组件类型**。用来作为type的值。

```js
const eleVNode = {
  type: 'div',
  props: null,
  children: [
    {   
      type: 'p',
      props: null
    },
    {
      type: `Component` // 用大小驼峰字符串分组件和html元素
      ...
    }
  ]
}
```

**React Fragments**

`React Fragments`，组件存在多个根节点情况。

```jsx
<>
<p />
<p />
...
</>
```



```js
const eleVNode = {
  type: 'Fragments', // Fragments
  props: {      
    style: {},
    class: {},
    event: {}
  },
  children: [
    {  
      type: 'p',
      props: null
  	},
    {  
      type: 'p',
      props: null
  	},
    ...
  ]
}

```

`Fragments`和`Component`判断似乎有些冲突，这块有很多区分方式，这取决于设计者的思路。

**ReactDOM.createPortal(child,container)**

`React Portal` (任意门)是用来创建一个节点，让它渲染到DOM的不同位置，常见`Overlay`。如果没有Portal,`z-index`的组件会被渲染在父级元素，这会导致蒙层失效。

```js
const eleVNode = {
  type: 'Portal',
  props: {
    position: '#id'
  },
  children: {
    type: 'div',
    props: {
      style: {zIndex: '999'}
    }
  }
}
```



#### vnode类型

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200904110230017.png" alt="image-20200904110230017" style="zoom: 50%;" />

---

### 这样就好了吗？

考虑这样一个场景：

```jsx
// 后端返回的JSON对象
message.text = JSON

-----
// react只会对标签自动转义，不会处理其它
<p>
  {message.text}
</p>

-------
```

在react里，防止xss注入，react会对`message.text`自动转义，然而这对**JSON对象无效**，根据上面的介绍，`p标签`会被解析生成`vnode`，所以通过这样的方式注入`vnode`，同样会被渲染。like this:

```jsx
// 后端返回的JSON对象xssCode
const xssCode = {
  type: 'div',
  props: {
    dangerouslySetInnerHTML: {
      __html: '/* xssCode */'
    },
  },
  // ...
};
// 传递给message
let message = { text: xssCode };

//  React 0.13版本 会很危险
<p>
  {message.text}
</p>
```

> [codesandbox:JSON对象造成的XSS](https://codesandbox.io/s/rough-star-ubht9?file=/src/App.js)

所以仅仅通过自动转义无法处理用户构造的**`vnode`XSS**。

#### $$typeof

于是`react`在0.14版本，加入了**$$typeof**，用以区别是否是react创建的，JSON不支持`symbol、undefined、function`，所以**$$typeof**的值用`symbol`来定义，在检测`eleVnode`是否存在**$$typeof**。

---

####  vnode"最终形态"

```js
const eleVNode = {
  type: vType,
  props: {
    position: '#id'
  },
  children: 
    [
			...
    ],
  $$typeof: Symbol.for("react.element") 
}
```

```js
export let REACT_ELEMENT_TYPE = 0xeac7;
export let REACT_PORTAL_TYPE = 0xeaca;
export let REACT_FRAGMENT_TYPE = 0xeacb;

if (typeof Symbol === 'function' && Symbol.for) {
  const symbolFor = Symbol.for;
  REACT_ELEMENT_TYPE = symbolFor('react.element');
  REACT_PORTAL_TYPE = symbolFor('react.portal');
  REACT_FRAGMENT_TYPE = symbolFor('react.fragment');
}
```





___

## diff渲染流程



### 实现渲染的约定（思路参考:reactDOM.render、vue2.render）

渲染:把`vnode`渲染到特定的平台(之前所说，有多个平台实现方案)

web平台基本思路：递归便利vnode，通过type判断，调用dom api，生成标签

渲染函数：webRender，接受两个参数vnode, container

渲染两个流程: mount(挂载)和patch(布丁布丁)

mount: 当不存在旧vnode，直接mount到容器里；

path：当存在旧的vnode，节约性能，做比较新旧vnode，更新

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200906225948756.png" alt="image-20200906225948756" style="zoom:50%;" />

根据上面的递归思路，可以基本实现webRender的基本形态：

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200907105618111.png" alt="image-20200907105618111" style="zoom:50%;" />

> 正常的思路是先把vnode渲染成真实dom，就是思考mount这个过程的实现，然后后续想更新就想到了patch过程，想移除就想到了removeChild，不断对webRender改造，这样比较平滑，时间原因，就简化这个心路历程。

### mount的设计

如果旧树上没有vnode，需要进行挂载处理(例如:新添节点，首次生成)。根据不同的节点类型，进行不同的挂载方式：

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200907111749278.png" alt="image-20200907111749278" style="zoom:50%;" />

代码实现:

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200907114421926.png" alt="image-20200907114421926" style="zoom:50%;" />

#### mountLaber

mountLaber用以把标签类型的vnode创建为真实dom，并添加到container中：

```js
const mountLabel(newNode, container) {
  // 创建
  const el = document.createElement(newNode.type)
  // 添加
  container.appendChild(el)  
}
```

目前实现的几个问题:

1. 后期比较更新如何删除、更新这个节点。
2. 没有对props属性进行渲染。
3. 没有处理子节点的mount。

第一个问题：

删除节点是对新旧节点更新时，调用原生的removeChild，需要获取真实dom作为参数，而removeChild的参数是container.vnode提供的，所以在mount的时候，要把通过vnode生成的真实dom绑定在vnode上。

```js
const mountLabel = (newNode, container) => {
  // 创建
  const el = document.createElement(newNode.type)
  // 把真实dom绑定在vnode上
  newNode.el = el
  // 添加
  container.appendChild(el)  
}
```

第二个问题：

props里描述着标签的各种属性包括:内联的style、class、event、自定义属性等,这里以style举例处理：

```js
const eleNode = {
  type: 'marquee',
  props: {
    style: {
      width: '66px',
      height: '66px',
      background: 'blue'
    }
  },
  $$typeof: Symbol.for("react.element")
}

const mountLabel = (newNode, container) => {
    // 创建
    const el = document.createElement(newNode.type)
    let {entries} = Object
    // 获取属性
    const {props} = newNode
    if (props) {
      for (let prop of props) {
        switch (prop) {
          case 'style':
            // 后期添加抛出报错,当乱写属性
            for (let [key, value] of entries(props[prop])) {
              el.style[key] = value
            }
          ...
        }
      }
    }
    // 把真实dom绑定在vnode上
    newNode.el = el
    // 添加
    container.appendChild(el)  
}

```

首先获取newNode的props属性，遍历props，当是'style'在遍历，最后添加到真实元素上。

第三个问题：(处理子节点的mount)

子节点，设计的数据类型为object、array、null。为了这里方便判断。就统一用arry包裹一层vnode（null除外，因为文本节点走不到这里）。包裹之后，就可以通过遍历children操作后续。因为子节点类型可能有很多种，所以这里不能调用mountLabel，要调用mount从走一遍。

```ts
// 挂载标签
const mountLabel = (newNode, container) => {
  // 创建
  const el = document.createElement(newNode.type);
  const { entries } = Object;
  const { props, children } = newNode;
	...
  // 遍历children，mount一下
  if (children) {
    Array.prototype.forEach.call(children, item => {
      mount(item, el);
    });
  }
  // 把真实dom绑定在vnode上
  newNode.el = el;
  // 添加
  container.appendChild(el);
};
```

#### mountText

在上面的演示，已经实现了mountText，文本节点没有props和children，直接创建元素，加入到container中。

#### mountFragment

有多个节点，没有el，所以直接把children传入mount去处理。

```ts
// 挂载fragment
const mountFragment = (newNode: VNode, container: vElement): void => {
  // 没有el，把父节点传入mount
  const { children } = newNode
  if (children) {
    Array.prototype.forEach.call(children, item => {
      mount(item, container)
    })
  }
   // 取到第一个子节点，方便后续移除更新
  newNode.el = (children[0] as VNode).el
}

// fragment vnode
const createFragment = {
  type: Symbol.for('react.fragment'),
  props: null,
  children: [
    {
      type: 'p',
      props: {
        style: {
          height: '66px',
          textAlign: 'center',
          background: 'green',
        },
      },
      children: [{ type: null, props: null, children: 'fragment' }],
      $$typeof: Symbol.for('react.element'),
    },
    createElement,
    createElement,
  ],
  $$typeof: Symbol.for('react.element'),
}

```

Fragment没有对应的真实dom，后续新旧节点比对结果的渲染怎么更删除呢？

```ts
function webRender(newNode: VNode, container: vElement): void {
  // 第一次之后会把vnode挂载到container上
  const oldNode = container.vnode
		...
      // 如果是fragment或者protal，获取父节点的所有子节点，然后移除
      if (
        oldNode.type === Symbol.for('react.fragment') ||
        Symbol.for('react.portal')
      ) {
        const children = oldNode.el.parentNode.children
        Array.prototype.forEach.call(children, el => {
          el && container.removeChild(el)
        })
      } else {
        container.removeChild(oldNode.el)
      }
    }
  }
  // 最后给container 绑定vnode
  container.vnode = newNode
}
```



#### mountPortal

和Fragment类似...

#### moutComponent

挂载组件，有两种情况，一种状态组件(class)，另外一种函数式组件。组件的实质是为了得到虚拟dom，所以分两种情况获取：mountClassComp、mountFuncComp。

##### 如何区分class组件和函数组件

##### 如何执行组件函数

如果是字符串，可以通过eval解析，但依旧无法执行函数，没有import导入。所以改写type类型

1. 改写vnode，type类型为函数，即是组件。
2. 修改webRender对组件的判断方式

##### mountClassComp

class组件，需要new一个实例，并调用这个实例的render来获取节点的信息。

```ts
// 挂载class组件
const mountClassComp = (newNode: VNode, container: vElement): void => {
  // 创建实例
  const instance = new newNode.type()
  // 调用render
  const vnode = instance.render()
  // 挂载vnode
  mount(vnode, container)
  // 组件没有真实el，但是vnode进去mount一定会被绑定
  newNode.el = vnode.el
}
```

Ps:这里生成了instance，可以绑定一些元素，生命周期什么的，内部this可以获取到。

##### mountFuncComp

函数和class类似... 调用返回即可。

```ts
// 挂载函数组件
const mountFuncComp = (newNode: VNode, container: vElement): void => {
  // 调用函数获取vnode
  const vnode = newNode.type()
  // 挂载vnode
  mount(vnode, container)
  // 组件没有真实el，但是vnode进去mount一定会被绑定
  newNode.el = vnode.el
}
```

### patch的设计

当newNode,container.vnode都存在时，就需要进行对比，而这样的对比过程就是diff算法的实现过程。状态的变化会引起ui更新，而实质还是生成的vdom进行比较的结果，把结果通过dom api 进行渲染。

#### Patch节点类型

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200912181815641.png" alt="image-20200912181815641" style="zoom:50%;" />

```tsx
// 补丁函数
export function patch(
  newNode: VNode,
  oldNode: VNode,
  container: vElement
): void {
  const newNodeType: any = newNode.type
  const oldNodeType: any = oldNode.type
  if (newNodeType !== oldNodeType) {
    // 节点类型不同调用
    patchElement(newNode, oldNode, container)
  } else if (typeof newNodeType === 'string') {
    patchLabel(newNode, oldNode, container)
  } else if (typeof newNodeType === 'function') {
    patchComponent(newNode, oldNode, container)
  } else if (typeof newNodeType === null) {
    patchText(newNode, oldNode, container)
  } else if (newNodeType === Symbol.for('react.portal')) {
    patchPortal(newNode, oldNode, container)
  } else if (newNodeType === Symbol.for('react.fragment')) {
    patchFragment(newNode, oldNode, container)
  }
}
```

##### patchElement

当节点类型不同，没有比较的意义

```tsx
const patchElement = (newNode: VNode, oldNode: VNode, container: vElement) => {
  // 移除旧节点，mount新节点
  container.removeChild(oldNode.el)
  mount(newNode, container)
}

```

所有的vnode挂载交给mount。（演示栗子🌰）

##### patchLabel

当节点类型为string，即认为比较标签元素。当同是标签元素时，根据vnode的设计，结果差异在props和children上。

但是react的vnode->key属性有更高更新权重，只要key不同，就会完全移除元素，再挂载新的元素,当然key的绑定不是必须的。重新设计vnode，添加key。

1. key差异

```ts
const patchLabel = (newNode: VNode, oldNode: VNode, container: vElement) => {
  // key不同移除旧节点，mount新节点
  if (newNode.key && newNode.key !== oldNode.key) {
    patchElement(newNode, oldNode, container)
  }
}
```

2. props差异

```ts
  // newNode要引用oldNode的el
  // el的获取要通过oldNode上引用的
  const el = (newNode.el = oldNode.el as HTMLElement)
  const { entries, keys } = Object
  const { props: newProps, children: newChildren } = newNode
  const { props: oldProps, children: oldChildren } = oldNode
  if (newProps) {
    for (const prop of keys(newProps)) {
      switch (prop) {
        case 'style':
          // 更新新style
          for (const [key, value] of entries(newProps[prop])) {
            el.style[key] = value
          }
          // 移除新style没有的
          for (const [key] of keys(oldProps[prop])) {
            if (!Object.prototype.hasOwnProperty.call(newProps[prop], key)) {
              el.style[key] = ''
            }
          }
      }
    }
  } else {
    // 如果没有新的，就全移除
    for (const prop of oldProps) {
      switch (prop) {
        case 'style':
          // 移除旧style
          for (const [key] of entries(oldProps[prop])) {
            el.style[key] = ''
          }
      }
    }
  }
```

3. children
   1. 旧节点为null
      1. 新节点为null
      2. 新节点不为null
   2. 旧节点不为null
      1. 新节点为null
      2. 新节点不为null

```ts
  // children
  if (!oldChildren) {
    if (!newChildren) {
      // 都为null没有操作
      console.log('done')
    } else {
      // 把新节点从新挂载, el扮演着container的作用
      for (let i = 0; i < newChildren.length; i++) {
        mount(newChildren[i] as VNode, el)
      }
    }
  } else {
    if (!newChildren) {
      // 旧节点不空，新节点空，全移除
      for (let i = 0; i < newChildren.length; i++) {
        el.removeChild((newChildren[i] as VNode).el)
      }
    } else {
      // 移除旧的
      for (let i = 0; i < oldChildren.length; i++) {
        container.removeChild((oldChildren[i] as VNode).el)
      }
      // 添加新的
      for (let i = 0; i < newChildren.length; i++) {
        mount(newChildren[i] as VNode, container)
      }
    }
  }
```

##### Full充分的复用

上面的更新过程没有重复子节点，直接把已经创建的真实dom移除，再创建。在React中，会对**同位置**的React尝试patch,不同位置哪怕一摸一样也不会复用(栗子🌰)。除非带着key，所以充分复用就是当新旧节点(组件)位置不同时，就带同样的key，这样类似v-if这种，也不会触发组件移除(unmounted的生命周期)。

不带key的实现：

```ts
  if (!newChildren) {
    ...
  } else {
    // 要得到新旧谁更长，共同部分patch，剩余部分mount或remove处理
    const newLen = newChildren.length,
          oldLen = oldChildren.length,
          commonLen = newLen > oldLen ? oldLen : newLen

    // 共有部分通过patch保证复用
    for (let i = 0; i < commonLen; i++) {
      patch(newChildren[i] as VNode, oldChildren[i] as VNode, el)
    }
    // mount新节点多出的
    if (newLen > commonLen) {
      for (let i = commonLen; i < newLen; i++) {
        mount(newChildren[i] as VNode, el)
      }
    }
    // 移除旧子节点多出的
    if (oldLen > commonLen) {
      for (let i = commonLen; i < oldLen; i++) {
        el.removeChild((oldChildren[i] as VNode).el)
      }
    }
  }
```

利用key尽可能地减少真实dom创建、移除达到优化目的,改写vnode，添加key(也可以不添加，但是要判断undefiend，比较麻烦)。优化策略就变成下述可能：

新节点有key和无key两种，无key不遍历，就和对应的元素对比，有key遍历旧节点找，如果找不到，就重新创建。

Ps：（为什么无key只和对应的对比呢，假设新1和旧4长一样，这样复用不是更好么，猜想patch时候无法知道这两个是长一样的）

1. 新节点无key （react策略：尝试和对应旧节点位置的元素对比）对比结果可能：
	1. 旧节点没有当前的index索引：mount(创建，插入到对应位置)
	2. 旧节点没有key：patch
	3. 旧节点有key：旧节点key存在，那新节点自己想办法创建吧，不能占用了key，mount(创建，插入到对应位置)
2. 新节点有key，遍历旧节点寻找：
	1. 旧节点找到了key：patch
	2. 旧节点没有找到key：mount(创建，插入到对应位置)
3. 旧节点没有用到的移除

> Ps: react, 这块的diff算法的时间复杂度应该是O(n),这实现的是O(n^2)

1.新节点无key :

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200915114645808.png" alt="image-20200915114645808" style="zoom:50%;" />

```ts
  for (let i = 0; i < newChildren.length; i++) {
    console.log(newChildren[i])
    // 新节点无key
    if ((newChildren[i] as VNode).key === null) {
      // 旧节点没有当前的index索引
      if (!oldChildren[i]) {
        mount(newChildren[i] as VNode, el)
      } else {
        // 旧节点有当前索引，且key也不存在，那就patch
        if ((oldChildren[i] as VNode).key === null) {
          patch(newChildren[i] as VNode, oldChildren[i] as VNode, el)
        } else {
          // 旧节点key存在，那新节点自己想办法创建吧
          mount(newChildren[i] as VNode, el)
        }
      }
    }
  }
```

2.新节点有key：

新节点有key，遍历旧节点寻找：

1. 旧节点找到了key：patch
2. 旧节点没有找到key：mount(创建，插入到对应位置)

```ts
  // 新节点有key 遍历旧节点
  for (let j = 0; j < oldChildren.length; j++) {
    // 旧节点key不为null，且等于新节点，patch
    if (
      (oldChildren[j] as VNode).key &&
      (newChildren[i] as VNode).key === (oldChildren[j] as VNode).key
    ) {
      patch(newChildren[i] as VNode, oldChildren[i] as VNode, el)
      break
    }
  }
  // 遍历完没找到key相同的，mount新节点
```

通过boolean元素，标记是否找到

```ts
  let isFind = false
  // 新节点有key 遍历旧节点
  for (let j = 0; j < oldChildren.length; j++) {
    // 旧节点key不为null，且等于新节点
    if (
      (oldChildren[j] as VNode).key &&
      (newChildren[i] as VNode).key === (oldChildren[j] as VNode).key
    ) {
      // 标记找到了，就不用mount
      isFind = true
      patch(newChildren[i] as VNode, oldChildren[j] as VNode, el)
      break
    }
  }
  // 遍历完没找到key相同的，mount新节点
  if (!isFind) {
    mount(newChildren[i] as VNode, el)
  }
```

3.旧节点没有用到的移除

移除节点，需要找到哪些节点没有被用到，所以这需要给已经用了的旧节点做下标记，标记为false的就移除。

```ts
      for (let i = 0; i < newChildren.length; i++) {
        // 新节点无key
            // 旧节点有当前索引，且key也为null，那就patch
            if ((oldChildren[i] as VNode).key === null) {
              patch(newChildren[i] as VNode, oldChildren[i] as VNode, el)
              // 动态添加'isUse',标记旧节点已用~
              oldChildren[i]['isUse'] = true
            } else {
							---
            }
          }
      }
      // 移除没有使用的旧节点
      for (let j = 0; j < oldChildren.length; j++) {
        // 没有使用切被已经被移除
        if (!oldChildren[j]['isUse']) {
          el.removeChild((oldChildren[j] as VNode).el)
        }
      }
```



目前的问题是，虽然通过key找到了对应的el元素，但是patch只会改变el元素的内容，不会改变它在父元素中的位置。所以要移动真实dom-》el元素的位置。

根据目前的逻辑，newChildren的创建顺序是不会变的，所以后一个元素总是在前一个的后面，由此，保存上一个元素的位置，新节点需要，就移动到它后面。

1. 旧节点位置和新节点位置相同，不用移动。
2. 新节点在第一位，el.insertAdjacentElement('afterbegin', oldChildren.el)
3. 不再第一个，每次存储前一个节点,(前一个节点的el).insertAdjacentElement('afterend', oldChildren.el)

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200915195645889.png" alt="image-20200915195645889" style="zoom:50%;" />



Tips: 避免用数组下标作为key,index在组件变化时不够稳定。

##### patchText

```tsx
// patchText
const patchText = (newNode: VNode, oldNode: VNode, container: vElement) => {
  // 移除旧节点，mount新节点
  container.removeChild(oldNode.el)
  mount(newNode, container)
}
```

patch的过程，是尽量对比新旧vnode的差异，然后把差异替换到已经存在的真实dom元素上的过程，减少dom创建是宗旨之一，每次更新文本，都是对新旧文本的内容更替，用nodeValue，且新旧文本不同才替换文本。

```tsx
// patchText
const patchText = (newNode: VNode, oldNode: VNode, container: vElement) => {
  const el = (newNode.el = oldNode.el)
  if (newNode.children !== oldNode.children) {
    el.nodeValue = newNode.Children
  }
}
```

##### patchFragment

---

##### patchPortal

---

##### patchComponent

更新组件的方式有两种：

1. class的setState和hook的setState
2. 组件上的props

###### patchClassComp

**class组件模拟this.setState来更新数据**

1. 修改class组件，添加this.state,再给父类，设置setState，触发时，调用webRender。

   _el 挂载的container，但是container,vnode保存着老的classnode，而this.render是classnode-》return的vnode,不是同一个类型，没法复用。

```ts
// 模拟 setstate
CuteComponent.prototype.setState = function (intialState) {
  // this 实例，给属性state负值
  this.state = intialState
  // 负值后，调用实例属性render重渲染
  webRender(this.render(), this._el)
}

// container
  const createClassComp = {
    type: ChildClass,
    props: null,
    children: null,
    key: null,
    $$typeof: Symbol.for('react.element'),
  }
// this.render
  
```

2. _el挂载classnode.el，但是classnode.el在mount的过程中没有挂载对应的vnode，所以会直接mount。

```tsx
...
webRender(this.render(), this._el(classnode.el))

// webRender
function webRender(newNode: VNode, container: vElement): void {
  // classnode.el没有vnode
  const oldNode = container.vnode
...
```

3. 不调用webRender,直接调用patch，patch需要newNode,oldNode,container,而这三个都会被后续更新替换掉，所以不能把patch放在，this.setState，而放在webRender某处。

   因为更新时，添加新真实dom总会触发mount，而mount内就可以获取到三个新的元素。所以在mountClassComp内的instance实例上绑定,\_renderComponent,闭包着三个新元素。再整个instance生命周期内都调用\_renderComponent，而按照patch里的逻辑，当instance的元素被替换了，就会触发mount，从新生成新的instance。

   <img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20200916154151179.png" alt="image-20200916154151179" style="zoom:100%;" />

   改写mountClassComp:

```tsx
// 挂载class组件
const mountClassComp = (newNode: VNode, container: vElement): void => {
  // 创建实例
  const instance = new newNode.type()
  // instance绑定_renderComponent
  instance._renderComponent = () => {
    // 挂载了，就执行patch的逻辑
    if (instance._ismounted) {
      const newNode = instance.render()
      const oldNode = instance.vnode
      patch(newNode, oldNode, oldNode.el.parentNode)
      // 让newNode成为下一次的旧节点
      instance.vnode = newNode
      instance._el = newNode.el
    } else {
      // 调用render
      const vnode = instance.render()
      // 挂载vnode
      mount(vnode, container)
      // 组件没有真实el，但是vnode进去mount一定会被绑定
      newNode.el = vnode.el
      // 给实例挂载上el
      instance._el = vnode.el
      // 给vnode赋值，后续patch需要
      instance.vnode = vnode
      instance._ismounted = true
    }
  }
  // 第一次正常挂载
  instance._renderComponent()
}
```

在patch的每一次更新中，都会对newNode绑定一次el，所以每一个mount/patch都需要注意el的绑定，这是贯穿所有逻辑的基准之一。

**class props更新**

为了在patch时能拿到instance，要在mount的时候就把instance先传给(当时的newNode，后来的oldNode)，这样在patch的时候，就能从oldNode上拿到instance。

###### patchFuncComp

##### 函数组件模拟hook更新数据

问题点如上setState时，需要调用webRender渲染，需要拿到2个参数，vnode，container。

1. 无法像class一样，通过instance.render获取，怎么调用函数自身呢
2. 没有intance，怎么绑定el

所以，hook组件是纯函数的无状态组件，保存状态需要借助第三方。这里模拟，hook为一个对象，用对象来收集依赖。

![image-20200913152828198](file:///Users/bjhl/Library/Application%20Support/typora-user-images/image-20200913152828198.png?lastModify=1600407220)

当调用了useState，就会把*hook.*isHookFun标记为true，代表为hook组件，这时在mount函数组件时，就把vnode，container赋值给_hook对象。这样在setState就能拿到webRender需要的两个参数。

![image-20200913153126592](file:///Users/bjhl/Library/Application%20Support/typora-user-images/image-20200913153126592.png?lastModify=1600407220)

mobx 利用双向绑定，当重新赋值，就调用类似setState，只要拿到webRender，就能把任意虚拟节点渲染挂载。

利用react暴露出来的Render，结合hook，应该能做出很多类似mobx这样的插件。

---

###### vue1，2和react

react无法精确捕捉到props的变化来更新子组件，所以每次父组件更新都会触发子组件运行。pureComponent和Memo浅比较、shouldUpdate，定点比较，由使用者决定。但通过patchClassComp里，instance.\_porps和newNode._props进行类似angular框架的脏检测(diff数据)应该也能减少子组件运行,然而比较带来的性能开销也会不少。

vue2通过响应式来精确定位，组件内部决定是否需要变化。在初始化需要消耗一部分开销处理data，增加白屏时间，但运行时，频繁变动时，vdom重建范围更小，减少运行开销。

react选择了比较dom节点的差异(节点发生变化反映到对应的ui)，而vue选择了比较数据本身的差异(data发生变化反映到ui，在vue1里没有用vdom，数据收集着很多真实dom，频繁操作带来了内存性能问题)。

<img src="/Users/bjhl/Library/Containers/com.tencent.WeWorkMac/Data/Library/Application Support/WXWork/Temp/ScreenCapture/企业微信截图_805651da-1bf1-410b-9e76-cbf38bfc6b15.png" alt="企业微信截图_805651da-1bf1-410b-9e76-cbf38bfc6b15" style="zoom:50%;" />

由于react会大范围的更新vdom，无法减少子组件rerender，就无法阻止diff，当数据变化时，react可能会占用主线程，导致无法响应用户的变动。比如input的渲染执行，react的相应会一直等到diff完成，然后回过头渲染input里的所有数据。当diff执行时间太长，会出现ui卡顿的情况(比如1、2、3会卡一下，突然一下渲染出来)。所以fiber的提出被尤小右吐槽是弥补缺陷。

比react更reactive。

hook的产生，会带来周边的升级，可以创造更多新的东西，但这需要对react不断深入理解。

### class组件一些生命周期的添加



---

## 实现createElement函数自动生成vnode



etc....

## AST

Etc...

