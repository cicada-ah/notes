https://segmentfault.com/a/1190000008500946

每一个dom上的指令(v-xxx:val / {{val}}) 对应一个watch

在watch内触发getter，会让对应属性的dep 收集watch，那么一个dep就对应多个watch。

所以vue1更新颗粒度很小，但是初始化的依赖收集就变多了

