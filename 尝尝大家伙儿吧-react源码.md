React.render() -> legacyRenderSubtreeIntoContainer()

legacyRenderSubtreeIntoContainer

1. 得到fiberRoot
2. 调用updateContainer传入fiberRoot，开始准备调度更新

->scheduleUpdateOnFiber()->performSyncWorkOnRoot()【开始执行同步work】

->renderRootSync() ->workLoopSync()->performUnitOfWork(workInProgress)

performUnitOfWork

1. 调用beginWork从workInProgress fiber开始fiber render，并且返回fiber.child，
2. 把fiber.child赋值给workInProgress开始下一次performUnitOfWork(workInProgress)

->beginWork()

beginWork

1. 首先判断是否bailout，是则不调用render，同时后续调用返回null，结束本分支的render。否则调用函数render（根据fiber.tag判断是calss/fun/Fragment/et...）,以hook为例：

updateFunctionComponent()->renderWithHooks()

renderWithHooks

1. 调用hook 函数，把结果返回【调用hook会触发setState修改fiber，返回的结果为vdom】

2. reconcileChildren(vdom),作用是把vdom的内容转化为一个fiber节点，同时挂载到workInProgress.children

3. 最后返回workInProgress.children 开始后续的递归调度

<img src="/Users/bjhl/Desktop/tiggaohwjd.jpeg" alt="tiggaohwjd" style="zoom: 25%;" />



<img src="https://upload-images.jianshu.io/upload_images/3649618-9e71c45599bb67f5.png" alt="img" style="zoom:25%;" />



### diff

广度优先会导致react生命周期乱套，父组件执行完，子组件才开始调用。

#### n(o)3 ->n(o)

自身从头深度遍历到尾：1层遍历

当前节点跨层级比较（react不跨层）：1层遍历

同一层的某一个元素遍历比较另一颗树的所有元素（key的作用）：1层遍历

### 为什么``setState``是异步的？

1. 保证内部一致性

```js
console.log(this.state.value) // 0
this.setState({ value: this.state.value + 1 });
console.log(this.state.value) // 1
this.setState({ value: this.state.value + 1 });
console.log(this.state.value) // 2
```

```js
console.log(this.props.value) // 0
this.props.onIncrement();
console.log(this.props.value) // 0
this.props.onIncrement();
console.log(this.props.value) // 0
```



即使state同步更新了，props也不会(假使state通过props传递给子组件)。除非放弃批处理，同步批处理，既是state变化了，但是此时由于没有触发render，所以state更新了，但是props没有。

#### unstable_batchedUpdates

```unstable_batchedUpdates```作用是保证在让react外部状态时，依旧能保持批处理能力。

