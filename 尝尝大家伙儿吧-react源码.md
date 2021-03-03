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