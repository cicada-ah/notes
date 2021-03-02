# react的生僻使用细节

1. **dangerouslySetInnerHTML** 插入富文本。`dangerouslySetInnerHTML` 是吧 `**html**字符串`转成 `innerHTML` 让`**html**` 能解析的样子，不然你看到得到就是一堆**html**标签。

2. **`findDomNode`**。新版本的React已经不推荐我们使用ca**ref** string转而使用**ref** callback，在说ReactDOM.findDOMNode，当参数是**DOM**，返回值就是该**DOM**（这个没啥卵用）；当参数是Component获取的是该Component render方法中的**DOM**

3.  createRef vs callback ref vs string ref.  http://www.sosout.com/2018/11/30/react-refs.html

4. # [react-router-dom多路由共用一个组件时，切换页面地址，页面不刷新的问题](https://segmentfault.com/a/1190000017410974)

5. setState 返回的value，是浅拷贝，传对象须注意

6. ellipsis 子元素是div 不生效

7. p,div等块级标签，数字和英文不会自动换行。white-space、word-break、word-wrap

8. event emit好处：1. 减少组件传递的props携带数量， 处理状态更加统一。2. 可能减少子组件调用渲染次数(因为props传递的参数少了，子组件会复用)（虽然用usecallback+memo解决一些性能问题，但是也不一定，usecallback不会缓存变化的值，这种情况）3.event  emit 便于处理一些状态发生变化时的回掉，特别当回调的内容写在父组件时，状态会更加统一。

9. css属性:pointEvents：auto||none，鼠标事件穿透某些div，很好用,例如：穿透蒙层，点击蒙层下的模块。

10. memo能作为shouldcompupdate使用，在hook中

11. Usecontext的，context的value发生了改变，，那么所有provider消费者都会被重新rerender，无论是引用了被修改的对象，那么在hook中，由于函数是被重新创建的，所有有了新的地址，那么就会触发provider中的子组件更新，比如作为props时。再class组件就不会，因为class是调用的render方法，不会让函数重新被创建