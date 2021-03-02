1. network

通常可以在network中，找出每个请求的发送时机

(**Queued,Started**),粗略找出影响性能的点。

![image-20201211170821894](/Users/bjhl/Library/Application Support/typora-user-images/image-20201211170821894.png)

2. performance

Api: https://juejin.cn/post/6904517485349830670?utm_source=gold_browser_extension#heading-5

3. 性能图

js执行块，在main线程里，是一个个task，如果有mictask，也可以在task栈里看到。

![image-20201211171708147](/Users/bjhl/Library/Application Support/typora-user-images/image-20201211171708147.png)

**当在html里获取dom元素，会触发提前渲染流程，此时fp会比dcl快。**

