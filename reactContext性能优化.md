假设有个更新消息组件，和消息渲染组件，同时存在与通信无关的pure ui组件, 通信通过Context

```tsx
import React, { createContext, useState, useContext, useEffect } from 'react'
import ReactDOM from 'react-dom'

// create a context
const MsgContext = createContext({
  msgList: [] as string[],
  updateMsgList: (msgList: React.SetStateAction<string[]>) => {console.log(msgList)}
})

// updateMsg component
const UpdateMsgComp = () => {
  const { updateMsgList } = useContext(MsgContext)
  console.count('UpdateMsgComp is run')
  useEffect(() => {
    // imitate request
    window.setInterval(() => {
      Promise.resolve(['new msg1', 'new msg2']).then(msgList => {
        updateMsgList(msgList)
      })
    }, 10000)
  }, [updateMsgList])
  return <div>update</div>
}

// renderMsg component
const RenderMsgComp = () => {
  console.count('RenderMsgComp is run')
  const { msgList } = useContext(MsgContext)
  return (
    <ul>
      {msgList.map((item, index) => {
        return <li key={index}>{item}</li>
      })}
    </ul>
  )
}

// irrelevant component
const IrrelevantComp = () => {
  console.count('IrrelevantComp is run')
  return <div>不要渲染我这个无关组件QAQ</div>
}

const MsgProvider = () => {
  const [msgList, setMsgList] = useState<Array<string>>(['msg1', 'msg2'])
  // update func
  const updateMsgList = (newMsgList: React.SetStateAction<string[]>) => {
    setMsgList(newMsgList)
  }
  const msgContext = {
    msgList,
    updateMsgList
  }
  return (
    <MsgContext.Provider value={msgContext}>
      <UpdateMsgComp />
      <RenderMsgComp />
      <IrrelevantComp />
    </MsgContext.Provider>
  )
}

ReactDOM.render(<MsgProvider />, document.getElementById('root'))



```

![image-20201203143914284](https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20201203143914284.png)

---



期望是当消息更新时，只有`RenderMsgComp`触发调用，然后结果是都触发了，同时当useContext的返回值作为`useEffect`之类的依赖时，更会触发意料之外的事，context造成这样的原因比较多:

第一点，jsx的实质是被解析为`React.createElement(xxx)`,所有当`setMsgList`执行的时候，`MsgProvider`又调用了一遍内部的`createElement`,从新跑了一遍hook组件。避免的方法就是通过children把组件传入给``MsgProvider``。

```tsx
const MsgProvider = ({ children }) => {
  ...
  return     
    <MsgContext.Provider value={msgContext}>
				{children}
    </MsgContext.Provider>
}
const App = () => (
    <MsgProvider>
        <UpdateMsgComp /> -->vnode
        <RenderMsgComp /> -->vnode
        <IrrelevantComp /> -->vnode
    </MsgProvider>
)
ReactDOM.render(<App />, document.getElementById('root'))
```

这样children就是`React.createElement(xxx)`的vdom结果，就不会重复出发无关组件`IrrelevantComp`的调用。总结就是所有状态无管的子组件，都可以提取作为children传入。(ps: 或者通过memo控制是否渲染)

结果： ![image-20201203150325494](https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20201203150325494.png)



原因二：![image-20201203153311707](https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20201203153311707.png)

当 `MsgProvider` 中的 `updateMsgList` 被子组件调用，导致 `MsgProvider`重渲染之后，必然会导致传递给 Provider 的 value 发生改变，由于 value 包含了 `updateMsgList` 和 `msgList` 属性，所以两者中任意一个发生变化，都会导致所有的订阅了 `MsgProvider` 的子组件重新渲染。

解决办法就是，让value的状态分离至不同的context下，这样通过不同的`Provider`去维护相关的「状态组件」rerun。

```tsx
// create msgrender context
const MsgContext = createContext({
  msgList: [] as string[],
})
// create update context
const UpdateMsgContext = createContext({
  updateMsgList: (msgList: React.SetStateAction<string[]>) => {console.log(msgList)}
})
const MsgProvider = ({children}) => {
  ...
  const msgContext = {
    msgList
  }
  const updateContext = {
    updateMsgList
  }
  return (
        <UpdateMsgContext.Provider value={updateContext}>
          <MsgContext.Provider value={msgContext}>
          {children}
          </MsgContext.Provider>
      </UpdateMsgContext.Provider>
  )
}
```

最后，保证MsgProvider更新时,`msgcontext、updateContext、updateMsgContext`引用地址不变，用`useMemo、useCallback`包裹一下。

```tsx
const MsgProvider = ({children}: any) => {
  const [msgList, setMsgList] = useState(['msg1', 'msg2'])
  // update func
  const updateMsgList = useCallback((newMsgList: React.SetStateAction<string[]>) => {
    setMsgList(newMsgList)
  }, [])
  const msgContext = useMemo(() => ({
    msgList
  }), [msgList])
  const updateContext = useMemo(() => ({
    updateMsgList
  }), [updateMsgList])

  return (
      <UpdateMsgContext.Provider value={updateContext}>
        <MsgContext.Provider value={msgContext}>
        {/* <UpdateMsgComp />
        <RenderMsgComp />
        <IrrelevantComp /> */}
        {children}
        </MsgContext.Provider>
      </UpdateMsgContext.Provider>

  )
}
```

![image-20201203161455940](https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20201203161455940.png)

原因:

```
const A = React.createContext({})
const B = React.createContext({})
<A.Consumer>A comp</A.Consumer>
<B.Consumer>B comp<B.Consumer>

```



---

### 总结

1. 所有「状态无关的子组件」，都可以提取出，然后作为children传入，避免creactElement重执行。(ps: 或者通过memo控制是否渲染)
2. context内部要做状态分离，具体哪些value聚集在一起，依据引用的组件？。
3. 用useMemo和useCallBack，弥补hook不能像class组件那样引用this上挂在的不变方法和属性。

