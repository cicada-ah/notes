

Vue & react

一种是以 Vue 为代表的 **mutable + change tracking**。即可变的数据结构，配合变更追踪，触发更新函数。

另一种是以 React 为代表的 **immutability + referential equality testing**。即不可变的数据结构，配合反复执行的渲染函数，以及在函数执行过程中，通过数据的引用相等性判断，所以react非常依赖vdom，找出变更部分，只应用变化的部分到 UI 上。

**mutable vs immutable**

https://zhuanlan.zhihu.com/p/79245530

然而 React Immutable 特性带来的可预测性非常利于调试和维护：

1. 断点调试时变量的值与当前执行位置无关，已创建过的值不会突然 Mutable 突变，非常可预测。
2. 在 React 框架下组件更新机制单一，只有引用变化才触发重渲染，而没有 Mutable 模式下 ForceUpdate 的心智负担。

当然 Immutable 模式下存在一定编码心智负担，所以各有优劣。

**immutable**

https://zhuanlan.zhihu.com/p/163590288

1. vue依赖追踪，更新精准，少了react需要不断render的过程。react immutable的特性，好处在于便于清晰清楚引发视图渲染的原因，而mutable因为可以在任何组件调用修改，复杂项目，容易引起bug。但是immutable会使得代码可读性比较差很多修改原引用类型的方法不能用push，经常用到扩展操作符...，结构嵌套深了，就失去了代码可读性。vue更符合js开发者自然操作方式，修改-》响应。
2. 所以React的hooks种种反直觉的问题，主要还是在于javascript的对象和函数默认不是immutable的
3. 至于其它诸如api设计真就无关痛痒。

---

虽然并不清楚并发模式的具体代码实现，但可以肯定的是目前的并发模式，是打断当前的render执行高优任务的render，这是为了解决复杂交互场景的性能，所以现在大多数场景的性能，并不能得到改善，使用react还是需要技巧，注意一些优化细节，所以react把优化方式交给用户来控制，看似更灵活，实则不仅开发难度上升，而且还需要写一些usecallback、memo啥的，确实是有一些不厚道，并不是每一个开发者都是fackbook前端的水准，而且国内业务任务还是比较重的，并不是有很多精力投入到代码重构里，大半年学习、使用的感受就是这样的，document.title=a就会改变，才是符合js开发的自然，并不是被知乎、文章啥洗脑的，相信大家也能有部分感受。不过这可能也是理论上的，因为实际想想，hook比较灵活，便底层，更新时可控，vue封装度高，api多，更新可能会找不到哪里导致出现了bug。

Vue作者尤雨溪：以匠人的态度不断打磨完善Vue

https://zhuanlan.zhihu.com/p/108899766



**¥重点需要阅读的！！特别是react hook和vue hook的对比**

Vue3 究竟好在哪里？（和 React Hook 的详细对比）

https://zhuanlan.zhihu.com/p/133819602



### **React Hook 和 Vue Hook 对比**

其实 React Hook 的限制非常多，比如官方文档中就专门有一个**[章节](https://link.zhihu.com/?target=https%3A//zh-hans.reactjs.org/docs/hooks-rules.html)**介绍它的限制：

1. 不要在循环，条件或嵌套函数中调用 Hook
2. 确保总是在你的 React 函数的最顶层调用他们。
3. 遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 useState 和 useEffect 调用之间保持 hook 状态的正确。

而 Vue 带来的不同在于：

1. 与 React Hooks 相同级别的逻辑组合功能，但有一些重要的区别。 与 React Hook 不同，`setup` 函数仅被调用一次，这在性能上比较占优。
2. 对调用顺序没什么要求，每次渲染中不会反复调用 Hook 函数，产生的的 GC 压力较小。
3. 不必考虑几乎总是需要 useCallback 的问题，以防止传递`函数prop`给子组件的引用变化，导致无必要的重新渲染。
4. React Hook 有臭名昭著的闭包陷阱问题（甚至成了一道热门面试题，omg），如果用户忘记传递正确的依赖项数组，useEffect 和 useMemo 可能会捕获过时的变量，这不受此问题的影响。 Vue 的自动依赖关系跟踪确保观察者和计算值始终正确无误。
5. 不得不提一句，React Hook 里的「依赖」是需要你去手动声明的，而且官方提供了一个 eslint 插件，这个插件虽然大部分时候挺有用的，但是有时候也特别烦人，需要你手动加一行丑陋的注释去关闭它。

React Hook 的心智负担是真的很严重