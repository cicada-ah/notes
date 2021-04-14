https://segmentfault.com/a/1190000014221014

1. compiler和Compilation

compiler和compilation类似于Vue和new Vue实例.

compiler原型prototype挂载了各种生命周期和处理方法,和webpack传入的options，以及在传入plugin.apply内，被挂载上的各种信息，之后通过原型run方法，执行this.compile(onCompiled)产生compilation，

onCompiled执行const stats = new Stats(compilation)

最后返回finalCallback(null, stats)

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/add7a4fdc40a410c9cadb97acc10a7a8~tplv-k3u1fbpfcp-watermark.image?imageslim)



2. 当面试官问Webpack的时候他想知道什么

https://juejin.cn/post/6943468761575849992?utm_source=gold_browser_extension#heading-1



### 优化

1. Source-map:

   dev环境：```cheap-module-eval-source-map```

   prod环境：```cheap-module-source-map```

2. Loader oneof

   在oneOf里面的loader一旦匹配成功则会跳出匹配，相当于break语句

3. TypeScript loader

   可以为loader```transpileOnly```，关闭类型检测，提升构建速度

4. pathInfo

   webpack 会在输出的 bundle 中生成路径信息。然而，在打包数千个模块的项目中，这会导致造成垃圾回收性能压力。在 `options.output.pathinfo` 设置中关闭：

   ```js
   module.exports = {
     // ...
     output: {
       pathinfo: false,
     },
   };
   ```

4. IgnorePlugin

   忽略打包```moment```带来的而外体积。

---



### Loader和plugins的区别

loader是一个模块转换器，将不同类型的源码进行转换，有的是为了转化为浏览器能允许的代码类型，例如：less->css，tsx->js。有的是为了支持资源引用，例如：css-loader,url-loader。有的是为了达到开发的某个目的，例如:postcss-loader

plugin是webpack运行生命周期的各个阶段上挂载的事件，会被指定的时间节点被触发，能够改变构建结果、拆分和优化bundle等。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/77ae854e57df4bb692dd3aaf983b23a6~tplv-k3u1fbpfcp-zoom-1.image)

#### Loader

loader是一个具有单一职责的转换器。

Loader的实现是module.exports为一个function的js模块：

```js
// my-loader.js
module.exports = function (content, map, meta) {
  ...
};
```

Webpack 会对loader注入三个参数：

- `content`：资源文件的内容。对于起始loader，只有这一个参数
- `map`：前面loader生成的source map，可以传递给后方loader共享
- `meta`：其他需要传给后方loader共享的信息，可自定义



## webpack4 -> 5

1. plugin和loader都需要升级
2. cli命令传递参数时需要 ```--env```
3. 不再需要clearnPlugin,新版```output```自支持。
4. ```contenthash```真正支持内容哈希，能实现持久话缓存。以及```moduleIds: "deterministic"```支持以往根据chunkId生存的文件，因为id变化而失效。
5. 深度tree shaking

