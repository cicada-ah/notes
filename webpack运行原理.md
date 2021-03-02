https://segmentfault.com/a/1190000014221014

1. compiler和Compilation

compiler和compilation类似于Vue和new Vue实例.

compiler原型prototype挂载了各种生命周期和处理方法,和webpack传入的options，以及在传入plugin.apply内，被挂载上的各种信息，之后通过原型run方法，执行this.compile(onCompiled)产生compilation，

onCompiled执行const stats = new Stats(compilation)

最后返回finalCallback(null, stats)

