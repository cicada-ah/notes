#                学习useEffect完全指南

## useEffect问题梳理

在使用useEffect总会觉得哪里有点不对劲，类似class的生命周期...但真的是这样吗？

* 如何使用`useEffect `模拟出`componentDidMount`生命周期？
*  如何正确地在`useEffect`里请求数据？`[]`又是什么？
* 我应该把函数当做effect的依赖吗？
* 为什么有时候会出现无限重复请求的问题？
* 为什么有时候在effect里拿到的是旧的state或prop？

**Dan：当我不再透过熟悉的class生命周期方法去窥视`useEffect` 这个Hook的时候，我才得以融会贯通。**

****

### 如何正确地在`useEffect`里请求数据？`[]`又是什么？

```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';
 
function App() {
  const [data, setData] = useState({ hits: [] });
 
  useEffect(async () => {
    const result = await axios(
      'https://hn.algolia.com/api/v1/search?query=redux',
    );
 
    setData(result.data);
  });
 
  return (
    <ul>
      {data.hits.map(item => (
        <li key={item.objectID}>
          <a href={item.url}>{item.title}</a>
        </li>
      ))}
    </ul>
  );
}
 
export default App;
```

effect hook里通过axios来获取数据，在通过setData来更新data。

However,当开始运行的时候，会陷入讨厌的循环里。effect会在，组件挂载运行，同样会在数据更新运行。因为我们写入了setData，会不断获取数据，不断更新。**我们只是希望在mount的时候获取数据**，所以要给effect提供一个空的`[]`。

```jsx
  useEffect(async () => {
    const result = await axios(
      'https://hn.algolia.com/api/v1/search?query=redux',
    );
 
    setData(result.data);
  }, []);
```

第二个参数可用于定义挂钩所依赖的所有变量（在此数组中分配）。如果变量之一更改，则挂钩再次运行。如果带有变量的数组为空，则在更新组件时挂钩根本不会运行，因为它不必监视任何变量。

最后，async会隐式返回一个Promise，然而，useEffect hook应该没有返回值或者返回一个**cleanup function**,返回Promise回报错，所以修改一下：

```jsx
  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        'https://hn.algolia.com/api/v1/search?query=redux',
      );
 
      setData(result.data);
    };
```

### 下面考虑一下第二个参数[]的实际作用

如何主动触发useEffect呢？当我们有如下的场景：

```jsx
import React, { Fragment, useState, useEffect } from 'react';
import axios from 'axios';
 
function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
 
  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        'https://hn.algolia.com/api/v1/search?query=redux',
      );
 
      setData(result.data);
    };
 
    fetchData();
  }, []);
 
  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <ul>
        {data.hits.map(item => (
          <li key={item.objectID}>
            <a href={item.url}>{item.title}</a>
          </li>
        ))}
      </ul>
    </Fragment>
  );
}
 
export default App;
```

在class中，我们通过update来处理data变化，调用axios来获取数据。那在函数组件中呢？如上所说，当我们提供了空的`[]`，那么useEffect只会在mount阶段触发。现在useEffect需要依赖query的变化，进行运行。

```jsx
  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
        `http://hn.algolia.com/api/v1/search?query=${query}`,
      );
 
      setData(result.data);
    };
 
    fetchData();
  }, [query]); //添加query
```

现在更改input内容，会触发effect hook了。但是又产生了一个问题，每次change都会触发调用。我们希望提供一个触发功能，手动控制触发更新。

```jsx
function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [url, setUrl] = useState(
    'https://hn.algolia.com/api/v1/search?query=redux',
  );
 
  useEffect(() => {
    const fetchData = async () => {
      const result = await axios(
      const result = await axios(url);

      );
 
      setData(result.data);
    };
 
    fetchData();
  }, [url]);
 
  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <button type="button" onClick={() => setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`)}>
        Search
      </button>
 
      <ul>
        {data.hits.map(item => (
          <li key={item.objectID}>
            <a href={item.url}>{item.title}</a>
          </li>
        ))}
      </ul>
    </Fragment>
  );
}
```

通过一个button来控制何时触发，把`[query]`改成`[url]`大致思路就是这样。

### 提供loading，和error处理能力

```jsx
import React, { Fragment, useState, useEffect } from 'react';
import axios from 'axios';
 
function App() {
  const [data, setData] = useState({ hits: [] });
  const [query, setQuery] = useState('redux');
  const [url, setUrl] = useState(
    'https://hn.algolia.com/api/v1/search?query=redux',
  );
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);
 
  useEffect(() => {
    const fetchData = async () => {
      setIsError(false);
      setIsLoading(true);
 
      try {
        const result = await axios(url);
 
        setData(result.data);
      } catch (error) {
        setIsError(true);
      }
 
      setIsLoading(false);
    };
 
    fetchData();
  }, [url]);
 
  return (
    <Fragment>
      <input
        type="text"
        value={query}
        onChange={event => setQuery(event.target.value)}
      />
      <button
        type="button"
        onClick={() =>
          setUrl(`http://hn.algolia.com/api/v1/search?query=${query}`)
        }
      >
        Search
      </button>
 
      {isError && <div>Something went wrong ...</div>}
 
      {isLoading ? (
        <div>Loading ...</div>
      ) : (
        <ul>
          {data.hits.map(item => (
            <li key={item.objectID}>
              <a href={item.url}>{item.title}</a>
            </li>
          ))}
        </ul>
      )}
    </Fragment>
  );
}
 
export default App;
```

### 自定义钩子

例子中，可以抽离effect相关的钩子为自定义的钩子，让业务逻辑跟清晰。我们抽离出`useHackerNewsApi`

```jsx
const useHackerNewsApi = () => {
  const [data, setData] = useState({ hits: [] });
  const [url, setUrl] = useState(
    'https://hn.algolia.com/api/v1/search?query=redux',
  );
  const [isLoading, setIsLoading] = useState(false);
  const [isError, setIsError] = useState(false);
 
  useEffect(() => {
    const fetchData = async () => {
      setIsError(false);
      setIsLoading(true);
 
      try {
        const result = await axios(url);
 
        setData(result.data);
      } catch (error) {
        setIsError(true);
      }
 
      setIsLoading(false);
    };
 
    fetchData();
  }, [url]);
 
  return [{ data, isLoading, isError }, setUrl];
}
function App() {
  const [query, setQuery] = useState('redux');
  const [{ data, isLoading, isError }, doFetch] = useHackerNewsApi();
 
  return (
    <Fragment>
      <form onSubmit={event => {
        doFetch(`http://hn.algolia.com/api/v1/search?query=${query}`);
 
        event.preventDefault();
      }}>
        <input
          type="text"
          value={query}
          onChange={event => setQuery(event.target.value)}
        />
        <button type="submit">Search</button>
      </form>
 
      ...
    </Fragment>
  );
}
```

使用自定义钩子进行数据获取就可以了。挂钩本身对API一无所知。它从外部接收所有参数，并且仅管理必要的状态，例如数据，加载和错误状态。它执行请求，并将数据用作自定义数据获取挂钩，将数据返回给组件。

## 每一次render都有它自己的props和state

任意一次渲染中的`state`常量都不会随着时间改变。渲染输出会变是因为我们的组件被一次次调用，而每一次调用引起的渲染中，它包含的`state`值独立于其他渲染。

## 每一次render都有它自己的事件处理函数

**在任意一次渲染中，props和state是始终保持不变的。**如果props和state在不同的渲染中是相互独立的，那么使用到它们的任何值也是独立的（包括事件处理函数）。它们都“属于”一次特定的渲染。即便是事件处理中的异步函数调用“看到”的也是这次渲染中的`count`值。

## 每一次渲染都有它自己的effects

官网的例子：

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {    document.title = `You clicked ${count} times`;  });
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

q：effect如何拿到conut的最新值？

a：不是类似`vue`的`data binding、watching`，也不是因为count是个可变的值。

我们已经知道`count`是某个特定渲染中的常量。事件处理函数“看到”的是属于它那次特定渲染中的`count`状态值。对于effects也同样如此：

**并不是`count`的值在“不变”的effect中发生了改变，而是\*effect 函数本身\*在每一次渲染中都不相同。**

每一个effect版本“看到”的`count`值都来自于它属于的那次渲染：

```jsx
概念上，你可以想象effects是渲染结果的一部分。// During first render
function Counter() {
  // ...
  useEffect(
    // Effect function from first render    () => {      document.title = `You clicked ${0} times`;    }  );
  // ...
}

// After a click, our function is called again
function Counter() {
  // ...
  useEffect(
    // Effect function from second render    () => {      document.title = `You clicked ${1} times`;    }  );
  // ...
}

// After another click, our function is called again
function Counter() {
  // ...
  useEffect(
    // Effect function from third render    () => {      document.title = `You clicked ${2} times`;    }  );
  // ..
}
```

**概念上，可以想象effects是渲染结果的一部分。**

****

first render过程：

* React: 给到我状态为0的UI。
* 组件：
  * return 需要渲染的<p>You clicked 0 times</p>
  * 告诉React，渲染完成后调用effect: `() => { document.title = 'You clicked 0 times' }`。

* React：收到组件的要求，开始通知浏览器把p标签添加。
* 浏览器：绘制ing。。。
* React：绘制完毕了，开始运行组件给的effect。

之后调用了setState也是这过程。

## 小结

到目前为止，我们可以明确地得出下面重要的事实：**每一个**组件内的函数（包括事件处理函数，effects，定时器或者API调用等等）会捕获某次渲染中定义的props和state。

所以下面的两个例子是相等的：

```jsx
function Example(props) {
  useEffect(() => {
    setTimeout(() => {
      console.log(props.counter);    }, 1000);
  });
  // ...
}
function Example(props) {
  const counter = props.counter;  useEffect(() => {
    setTimeout(() => {
      console.log(counter);    }, 1000);
  });
  // ...
}
```

**在组件内什么时候去读取props或者state是无关紧要的。**因为它们不会改变。在单次渲染的范围内，props和state始终保持不变。（解构赋值的props使得这一点更明显。）

个人理解，hooks就是js闭包的运用。每一次数据发生变化，调用对应的函数组件重新渲染，都会闭包引用data，所以综上，props和state在每一次特定的渲染中都是不会改变的。

