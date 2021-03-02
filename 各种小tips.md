1. img

img标签保持宽高，外面包一个div设置宽高，img，width：auto。

或者div，background-size:contain,background-image:url(xxx)，background-portion: center.

Img:src如果为空，会有灰色边框。

2. requestldleCallback

https://juejin.cn/post/6844903592831238157

3. 富文本 quill,execCommand,contentEditable
4. 简单css虚拟滚动content-visibility,contains-intrinsic-size

https://juejin.cn/post/6908521872577527822

5. script标签 async、defer

   ![请输入图片描述](http://segmentfault.com/img/bVcQV0)

6. webpack分chunk

多个chunk有利于持久化缓存，改动一个chunk不会影响到整体。

7. Less 全局变量

在样式引入时，对于变量的引入，需要在每个文件里都要引入一遍，为了避免每次使用时都需要单独引入一遍的问题，采用了style-resources-loader

8. 阿里云服务

阿里云的服务有很多功能，比如图片压缩、图片旋转。

### js

1. Filter(Boolean)

过滤掉**null,undefind,0,false**的数据

 

