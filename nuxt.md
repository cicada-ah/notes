# 									nuxt

## 一、为什么要使用nuxt

> vue单页面应用渲染是从服务器获取所需js，在客户端将其解析生成html挂载于 id为app的DOM元素上，这样会存在两大问题。

* 由于资源请求量大，造成网站首屏加载缓慢，不利于用户体验。

* 由于页面内容通过js插入，对于内容性网站来说，搜索引擎无法抓取网站内容，不利于SEO。 Nuxt.js 是一个基于Vue.js的通用应用框架，预设了利用Vue.js开发服务端渲染的应用所需要的各种配置。可以将html在服务端渲染，合成完整的html文件再输出到浏览器。

#### 关于同构SSR

1. 虽然使用了服务端渲染，但是这个只能叫同构SSR，和传统的服务端渲染还是有区别的。目前同构SSR的本质就是集成页面组件，路由，前端状态，在服务端中运行生成快照，将生成的快照HTML传给客户端。需要注意的是，由于同构的这种快照所需的计算量远大于传统服务端渲染，所以单机性能上，可能要弱于传统服务端渲染。
2. 同构SSR的实现得意于虚拟DOM的出现，虚拟DOM的最大好处并非Diff算法而是为前端赋能，把HTML的DOM抽象化，可以在服务端、IOS、安卓甚至智能家电上运行。
3. 同构SSR的实质是当用户首次请求时，通过node端生成一个HTML快照给前端，之后用户在当前页面上的操作，其实都是一个SPA的操作交互，前端的路由交互还是依靠history路由去处理，而非传统路由，所以其实还是一个“`SPA`”。这样的处理，可以在保证首屏速度时，同时，减少服务器压力，提升用户体验，弥补同构渲染性能问题。

### Nuxt入门

### 构建

#### npm

```
npx create-nuxt-app <项目名>
复制代码
```

#### yarn

```
yarn create nuxt-app <项目名>
复制代码
```

### 目录结构

#### 简写

src

```
~ or @
复制代码
```

root folder

```
~~ or @@
复制代码
```

默认root和src是一致的

### Nuxt.config.js

踩坑：

1.`Nuxt.config.js`文件未使用`babel处理`

`nuxt.config.js`是nuxt提供的核心配置文件

开发时，`nuxt.config.js`中的修改不会直接热更新，需要手动在命令行中输入`rs`重新执行一次

### asyncData

> asyncData方法会在组件（限于页面组件）每次加载之前被调用。它可以在服务端或路由更新之前被调用。 在这个方法被调用的时候，第一个参数被设定为当前页面的上下文对象，你可以利用 asyncData方法来获取数据，Nuxt.js 会将 asyncData 返回的数据融合组件 data 方法返回的数据一并返回给当前组件。

#### 关于asyncData的理解

想象一下同构渲染的场景，当首次访问的时候，服务端返回一个HTML的快照；后续用户在改页面上操作，则是用户直接从浏览器发出请求到服务端。那么我们需要对数据的操作，为了避免写两套代码，运行在node端和浏览器端，我们需要这样一个函数，能够判断浏览器端还是服务端，自动化的处理数据请求。

所以，nuxt提供了`asyncData`这样一个方法，用来处理同时会在服务端以及浏览器端进行的数据请求，`asyncData`方法第一个参数被定义为nuxtjs的上下文对象，通过nuxtjs的上下文对象，可以获取到路由参数，使用自定义的nuxtjs插件,对错误参数进行处理等等。

即：同构逻辑执行的接口函数

#### 上下文对象中的参数

##### app

##### params

##### res,req

##### $axios等

获取异步数据

默认axios，需要返回res中的data

#### 如何使用asyncData

##### 使用 async或await

```js
export default {
  async asyncData ({ params }) {
    let { data } = await axios.get(`https://my-api/posts/${params.id}`)
    return { title: data.title }
  }
}
复制代码
```

##### 使用 Promise

```js
export default {
  asyncData ({ params }) {
    return axios.get(`https://my-api/posts/${params.id}`)
    .then((res) => {
      return { title: res.data.title }
    })
  }
}
复制代码
```

#### 不能使用this

### SSR逻辑

首次请求页面，会触发SSR，当在当前页面进行跳转时，则会通过AJAX的方式去请求接口，CSR的方式去生成新的页面。

### 预处理

nuxt继承了vue cli3的预处理配置，如果想使用pug，scss，stylus等只需要在使用时执行npm intall 或者yarn add

```
npm install --save-dev pug@2.0.3 pug-plain-loader coffeescript coffee-loader node-sass sass-loader
复制代码
```

### 跨域请求

```js
npm i @nuxtjs/proxy -D
复制代码
  modules: [
    '@nuxtjs/axios',
    '@nuxtjs/proxy'
  ],
  axios: {
    proxy: true
  },
  proxy: {
    '/api': {
      target: 'http://example.com',
      pathRewrite: {
        '^/api' : '/'
      }
    }
  }
复制代码
```

### noSSR

可以通过noSSR包裹，来实现CSR，场景，当页面很长时，可以通过底部使用CSR渲染来减少服务器负载。

支持文字形式以及插槽形式

```vue
<no-ssr placeholder="Loading...">
      <!-- 此组件仅在客户端呈现 -->
      <comments />
    </no-ssr>
复制代码
  <no-ssr>
    <!-- 此组件仅在客户端呈现 -->
    <comments />

    <!-- loading indicator -->
    <comments-placeholder slot="placeholder" />
  </no-ssr>
复制代码
```

### 路由跳转

NuxtLink

```vue
    <NuxtLink :to="'/users/'+user.id">
      {{ user.name }}
    </NuxtLink>
复制代码
```

router.push

```js
 this.$router.push(`/detail/${topicItem.postid}`);
复制代码
```

### 全局CSS

```js
nuxt.config.js
  css: [
    'element-ui/lib/theme-chalk/index.css',
    '~/assets/main.scss'
  ],
复制代码
```

### 页面跳转间的loading

```js
loading: '~/components/loading.vue',
复制代码
```

### layout

对应layout目录下自定义的vue文件名

```
 layout: 'dark',
复制代码
```

动态布局适应移动端

```js
layout: (context) => context.isMobile ? 'mobile' : 'desktop'
复制代码
```

### 中间件

```js
nuxt.config.js
  router: {
    middleware: ['visits', 'user-agent']
  }
复制代码
export default function (context) {
  const userAgent = process.server ? context.req.headers['user-agent'] : navigator.userAgent
  context.isMobile = /Android|webOS|iPhone|iPad|BlackBerry/i.test(userAgent)
}
复制代码
```

中间件基于路由，路由改变时将执行，执行流程顺序：

1. nuxt.config.js
2. 匹配布局
3. 匹配页面

nuxt中的中间件概念是基于路由层面的方法，可以分别在`nuxt.config.js`、`layouts`、`pages`中配置，分别对应全部页面的中间件、所有使用同一布局的中间件、单一页面的中间件。如果同时配置一个中间件在三个位置，则具体到单个页面，会执行三次。

使用举例 `nuxt.config.js`中

```js
  //nuxt.config.js中router的配置
  router: {
    middleware: 'auth',
  },
复制代码
```

`layouts`和`pages`中

```js
middleware:'auth'
复制代码
```

注意

场景：在页面中清空cookie（这时vuex状态并不会清空），然后点击链接进行spa操作，执行middleware时，vuex中状态还是原来状态

### 插件机制

#### 三方库，例如axios

```js
import axios from "axios";
...
  async asyncData() {
    let { data } = await axios.get(
      `https://api.isoyu.com/api/News/new_list?type=1&page=20/new_list?type=1&page=20}`
    );
    return { topicList: data.data };
  }
复制代码
nuxt.config.js
 plugins: [
     { src:  '@/plugins/element-ui' },
    { src: '@/plugins/vue-notifications.js', mode: 'client' }
  ],
复制代码
```

mode 可以选择client以及server。

#### 注入vue实例

```js
plugins/vue-inject.js
import Vue from 'vue'

Vue.prototype.$myInjectedFunction = (string) => console.log("This is an example", string)
复制代码
nuxt.config.js
export default {
  plugins: ['~/plugins/vue-inject.js']
}
复制代码
```

#### 注入 context

```js
plugins/ctx-inject.js
export default ({ app }, inject) => {
  // Set the function directly on the context.app object
  app.myInjectedFunction = (string) => console.log('Okay, another function', string)
}
复制代码
nuxt.config.js
export default {
  plugins: ['~/plugins/ctx-inject.js']
}
复制代码
```

使用

```js
export default {
  asyncData(context){
    context.app.myInjectedFunction('ctx!')
  }
}
复制代码
```

#### 同时注入

```js
plugins/combined-inject.js
export default ({ app }, inject) => {
  inject('myInjectedFunction', (string) => console.log('That was easy!', string))
}
复制代码
nuxt.config.js
export default {
  plugins: ['~/plugins/combined-inject.js']
}
复制代码
```

使用

```js
export default {
  mounted(){
    this.$myInjectedFunction('works in mounted')
  },
  asyncData(context){
    context.app.$myInjectedFunction('works with context')
  }
}
复制代码
store/index.js
export const state = () => ({
  someValue: ''
})

export const mutations = {
  changeSomeValue(state, newValue) {
    this.$myInjectedFunction('accessible in mutations')
    state.someValue = newValue
  }
}

export const actions = {
  setSomeValueToWhatever ({ commit }) {
    this.$myInjectedFunction('accessible in actions')
    const newValue = "whatever"
    commit('changeSomeValue', newValue)
  }
}
复制代码
```

注意：1.插件应该是按照`nuxt.config.js`中的顺序，依次执行的

### 错误提示

```js
async asyncData({ $axios, params, error, app }) {
error({ statusCode: 404, message: "Topic not found" });
}
复制代码
```

### nuxt深入理解

#### 生命周期



![image](https://user-gold-cdn.xitu.io/2020/5/29/1725f518d0b7ecd5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



首先我们需要了解一下这个框架的生命周期，在开发过程中，可能会碰到一些问题难以定位，需求无法实现，也许，通过nuxt的生命周期就能帮助你比较好的定位问题，解决问题。

首先，请求发生时，首选会执行`nuxtServerInit`，处理vuex中的状态，也就是先处理整个APP的状态，然后才会处理单个具体路由下的周期。首先会处理`middleware`，先后会处理`nuxt.config.js`中的配置，匹配的`layout`(nuxt提供的模版布局)，匹配的页面。在这个环节之后，会处理单个页面中设置的validate函数(用来校验路由参数)。最后，会通过nuxt提供的`asyncData`以及`fetch`的函数，去获取单个`page`的数据。需要注意的是，在这一些列生命周期后，会进入vue的渲染过程。

#### 一次请求过程中nuxt的生命周期顺序

服务端

1.首先执行插件(所有在`nuxt.config.js`中写入的可以在服务端运行插件，不管是否在当前页面)

2.执行`nuxtServerInit`

3.执行`middleware`：

a. `middleware`会先后执行`nuxt.config.js`中配置的`middleware`；

b. `layouts`中配置的`middleware`；

c. `pages`中的`middleware`。

4.执行validate方法，校验页面参数是否正确

5.执行页面中的asyncData以及fetch方法

6.真正进入vue的生命周期中，按先后顺序,`beforecreated`,`created`

浏览器端

7.执行插件(所有在`nuxt.config.js`中写入可以在浏览器端执行的插件)

8.进入vue的生命周期中，再在浏览器端运行`beforecreated`和`created`一遍。

附：

1.plugin和nuxtServerInit仅在首次刷新页面时会执行，后续点击页面内跳转不会再执行plugin和nuxtServerInit中的方法。如果打开新页面会再次触发plugin和nuxtServerInit方法。

理解：

1.插件会在所有模块运行之前运行，而且每次请求都会运行，所以如果项目较大，访问量大的情况下，仅将必要的方法写到插件中，优化性能。

2.同构渲染框架提供的额外生命周期以及方法都是在真正的vue生命周期之前的，原因是vue的生命周期是不可以异步的，所以，为了满足开发的需求，所有生命周期和方法都应该是在vue生命周期之前去执行的，理解好这一点，会方便入手nuxt开发。

3.`beforeCreated`和`created`生命周期其实是同时在服务端和浏览器端执行的，“同构”渲染的来源之一。

4.插件，`nuxtServerInit`,`middleware`,`validate`,`asyncData`，根据执行顺序和具体需求，选择好适合的生命周期和方法去处理开发需求是很有必要的

#### 解决跨域开发的问题（cookie穿透）

问题场景描述：

本地开发如果是`localhost`，服务端API域名为`test.com`。那么当首次请求时，就会涉及到一个cookie（token）“预取”的过程,虽然引入了node端进行了同构渲染，但是cookie和token作为用户身份的凭证还是保存在浏览器上，所以需要一个cookie从浏览器到node端，再由node端到服务端API的这样一个过程。

解决方案：

所以，当首次请求页面时，页面的权限校验以及相关数据，应该是从node端向服务端请求的，我们需要浏览器端的cookie（token）。针对cookie预取（穿透）在早期有很多种不同的方案，有存到状态管理器中的，有修改asyncData方法中的，我的建议是将数据请求单独封装成插件，利用asyncData同时在浏览器和node端工作的能力，在asyncData中，通过封装后的axios插件去处理对应的问题。

同时，当我们首次请求时，面临一个开发问题，如果是localhost首次访问，那么浏览器首次请求时，是不会携带着`test.com`的cookie的。本地开发就存在问题，所以这里想出了有两种方案，1.直接修改本地的ip指向为`test.com`，mac用户推荐helm，简单好用，切换环境。2.可以在首次请求时，先访问一个空白页面，然后通过ajax跨域请求使nuxt得到cookie，然后再从空白页面跳转回访问的页面，从而实现得到想要的cookie。

下面是封装一个axios插件的代码

```js
export default function ({ $axios, redirect, req, route, error }, inject) {
  // Create a custom axios instance
  let cookie = ''

  if (process.server) {
    if (req.headers.cookie) {
      cookie = req.headers.cookie
    }
  } else {
    cookie = document.cookie
  }
    const instance = $axios.create({
    headers: {
      common: {
        Accept: 'text/plain, */*'
      },
    },
    withCredentials: true, // default
  })
//...
  instance.setBaseURL(process.env.API_HOST)
  process.server ? instance.setHeader(cookie) : ''
复制代码
```

需要知道，浏览器端是不可以设置header中的cookie参数，而服务端是可以的，所以我们在服务端手动设置一下cookie即可，通过`process.server`去判断是否为服务端。这里因为项目中是cookie这样处理，如果token的话其实是一样的，只需要多一个把token从cookie中取出放到token里的过程。

这样，就实现了cookie（token）预取的过程，简单方便有效。同时我们也可以将一些错误处理以及请求自定义在这里处理 。例如http错误状态的统一处理

```js
  instance.onResponse(response => {

    if (response.status === 404) {
        //404处理
    }

    return response.data 
  })
复制代码
```

这里需要注意一个问题，在插件中，是可以直接使用error方法去处理跳转错误页面的，但如果`asyncData`中有多次数据请求，并且成功失败不一时，会导致error执行错误，这里可以在插件中使用`redirect`方法去执行，或者在`asyncData`中的`Promise.all()`完成后去处理错误状态。

#### nuxt鉴权

1.如果页面较少，且每个页面都有接口请求，可以直接省略引入vuex的机制，直接在封装的http请求插件中根据和后端约定的状态码，当状态码错误时，判断未登陆，处理相关逻辑即可。

2.如果页面较多，且用户权限不一致，有部分页面无接口请求，比如，静态页面上面有一个header携带用户名称这样的页面。在这种情况下，需要使用vuex并且做vuex持久化，从而方便开发。官网中的例子使用了`express-session`机制，感觉没有必要。使用session存储数据也可以，但感觉数据量不大的情况下，放在cookie中比较简洁。

### nuxt中vuex的使用

1.nuxtServerInit

可以在nuxtServerInit中做一些请求

例子

```js
store/userinfo
export const state = () => ({
  userName: '',
  roleName: '',
  roleType: '',
})

export const mutations = {
  UPDATE_USERINFO(state, { userName, roleName, roleType, netEaseUserEmail }) {
    state.userName = userName
    state.roleName = roleName
    state.roleType = roleType
  }
}

复制代码
store/index
export const actions = {
  async nuxtServerInit(store, { res, req, app }) {
    //处理userinfo信息初始化
    if (!(store.state.userinfo && store.state.userinfo.netEaseUserEmail)) {
      const userinfoRes = await app.$http.get(path.getUserInfo);
      store.commit('userinfo/UPDATE_USERINFO', userinfoRes.data)
    }
  }
}
复制代码
```

引用位置

```js
import { mapState } from 'vuex'
export default {
  computed: {
    ...mapState('userinfo', ['userName', 'roleName'])
  }
}
复制代码
```

2.通过`vuex-persistedstate`,`js-cookie`,`cookie-parser`,`cookie`实现通过cookie的方式持久化`Vuex` 示例

```js
import createPersistedState from 'vuex-persistedstate'
import * as Cookies from 'js-cookie'

export default ({ store, req, res, app }) => {
  createPersistedState({
    key: 'vuexnuxt',
    storage: {
      getItem: key =>
        process.client
          ? Cookies.getJSON(key)
          : cookie.parse(req.headers.cookie || '')[key],
      setItem: (key, value) => process.client ?
        Cookies.set(key, value, { expires: 365 }) : res.cookie(key, value, { expires: new Date(Date.now() + 60 * 60 * 1000 * 24 * 365) }),
      removeItem: key => process.client ? Cookies.remove(key) : res.clearCookie(key) 
    }
  })(store)
}
复制代码
```

通过这种办法，可以实现vuex的持久化支持，不管是在node端还是浏览器端，都可以访问到通过`cookie`持久化的vuex

附：对cookie的操作浏览器端使用`js-cookie`服务端使用`cookie-parser`处理。尝试了`cookie-universal-nuxt`这个插件，放弃。（使用后页面白屏卡死，可能该插件这个场景下出现了死循环）

对于vuex的持久化有多种选择，locaostorage,sessionstorage,session等，但从使用方便行角度来说，cookie是比较好的选择。

## 项目个人理解

router出的路由的`created`和`beforeCreated`会在客户端和服务端各执行一次，所以有些页面可以考虑spa的用法。

```js
    plugins: [
        {
            src: '~/plugins/vant',
        },
        {
            src: '~/plugins/vue-extend'
        },
        {
            src: '~/plugins/axiosSsr'
        },
        {
            src: '~/plugins/axios',
            mode: 'client'
        },
        {
            src: '~/plugins/habo-config',
            mode: 'client'
        },
        {
            src: '~/plugins/event',
            mode: 'client'
        },
        {
            src: '~/plugins/sentry',
            mode: 'client'
        },
        {
            src: '~/plugins/common',
            mode: 'client'
        },
        {
            src: '~/plugins/route',
            mode: 'client'
        },
        {
            src: '~/plugins/hybrid',
            mode: 'client'
        }
    ],
```

在ssr中只考虑在client中打包plugins，在nuxt.config中设置`mode: 'client'`。

**# nuxt-test**

```bash
|-- .nuxt                            // Nuxt自动生成，临时的用于编辑的文件，sdafasdfbuild

|-- api															 // 请求的地址json，通过this.$api获取

|-- assets                           // 用于组织未编译的静态资源入LESS、SASS 或 JavaScript

|-- components                       // 用于自己编写的Vue组件，比如滚动组件，日历组件，分页组件dsfaf

|-- layouts                          // 布局目录，用于组织应用的布局组件，不可更改。

|-- middleware                       // 用于存放中间件

|-- pages                            // 用于存放写的页面，我们主要的工作区域

|-- plugins                          // 用于存放JavaScript插件的地方

|-- static                           // 用于存放静态资源文件，比如图片

|-- store                            // 用于组织应用的Vuex 状态管理。

|-- .editorconfig                    // 开发工具格式配置

|-- .eslintrc.js                     // ESLint的配置文件，用于检查代码格式

|-- .gitignore                       // 配置git不上传的文件

|-- nuxt.config.json                 // 用于组织Nuxt.js应用的个性化配置，已覆盖默认配置

|-- package.json                     // npm包管理配置文件
```

