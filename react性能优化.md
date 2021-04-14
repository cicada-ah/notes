https://github.com/brickspert/blog/issues/36

useState原理

https://juejin.cn/post/6844904152712085512#heading-13

<img src="https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20210217000811932.png" alt="image-20210217000811932" style="zoom:25%;" />

为什么setA之后，又setC，child只打印了一次，而不是两次呢？因为set操作是修改fiber上的memoizedState值，此时没有触发render，最后执行**scheduleWork，此时react才考虑触发render，从最顶级的fiber进行调用逐层便利触发**，因此，父子组件都修改值时，只会调用一次函数，而不是子函数hook触发一次，父函数更新时再触发一次子函数hook，这样的两次。

也因此，mobx没有管父子依赖触发会多次render子组件的问题，父子组件依赖foceupdate时，触发的两个setstate，最后交给了react来调度到底什么时候从**最顶层开始向下render一次，虽然依赖收集了父子两个组件，但是子组件也只会触发一次render**。

---

<img src="https://raw.githubusercontent.com/waiwai948/myTypora/main/img/image-20210217033511805.png" alt="image-20210217033511805" style="zoom: 33%;" />



如图，断点运行11时，打印Child的fiber，发现

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20210217033802537.png" alt="image-20210217033802537" style="zoom:33%;" />

此时fiber正常，alternate也是它本身，当从断点11运行到12时，fiber发生了变化，此时setC修改了fiber状态。setC->dispatchAction(fiber, queue, action),在此时，修改了fiber上的panding,以及expirationTime(用以合并25ms内的setstate为一个，后续批量更新的作用,react17移除了expirationTime)。set操作修改fiber后，调用scheduleWork(fiber, expirationTime)记录调度，带react合成事件/生命周期里会真正开始调度这是一个回调函数，整个过程在concourrent/suspense模式之前是同步过程。

<img src="/Users/bjhl/Library/Application Support/typora-user-images/image-20210217034537142.png" alt="image-20210217034537142" style="zoom:33%;" />



之后就交给schedulework进行调度，后续可以看作settimeout(expirationTime，25ms间隔内的都相同,25ms里的update都批量合并成一个update)，后等所有fiber修改完成后，通过root节点的fiber层层对比下去。

setstate流程：dispatchAction->scheduleUpdateOnFiber->performSyncWorkOnRoot

useState只修改了对应的fiber，经过scheduleWork调度（异步，等所有fiber修改完），经过renderRoot-》->prepareFreshStack-> createWorkInProgress->workloop-〉beginWork-》updateFunctionComponent-〉renderWithHooks

prepareFreshStack里会通过传入的fiber节点，通过createWorkInProgress创建workInProgress树（总是从root节点开始创建）,并让current.alternate<->workInProgress.alternate。

拿到workInProgress(root开始)后，开始workloop开始循环调度更新，每一次通过beginwork判断当前节点是否需要更新(1.props判断，2.updateExpirationTime < *renderExpirationTime*，updateExpirationTime是每次setstate修改fiber为最新的time，以此调和它的优先级，如果没有setstate，时间就会落后，就不会走后续调用当前组件的逻辑，而是判断子节点需不需要更新，判断子节点是不是要更新也是通过这两个time判断的，因为一次setstate会沿路自底向上的更新fiber的time，所以可以这样判断，如果没有需要更新的就让workInProgress = null，由此结束了render阶段），需要就调用当前组件updateFunctionComponent得到新的vdom，vdom会经过reconcileChildren生成fiber(fiber diff的过程)，并且这个fiber的effectTag属性会被标记(commit阶段通过遍历整个fiber树，将所有标记的进行更新)，之后把这个fiber赋值给workInProgress.child标记为需要更新的内容，之后beginwork结束，返回next，如果当前节点没有更新，就返回子节点，否则返回workInProgress.child，workInProgress，再进行beginwork判断。



每一次render，通过root的

其中beginWork会比对fiber.current和fiber.alternal(workInprogress)判断当前fiber是否需要更新，需要就调用当前组件对应的函数，否则跳过。

React 源码

https://github.com/y805939188/simple-react

https://www.jianshu.com/p/9c360697fe09

https://juejin.cn/user/2700056288311533/posts

https://zhuanlan.zhihu.com/p/103693631



