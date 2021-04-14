# 							ES6学习笔记

## Map 和 Set

### Map

`Map`对象保存键值对。任何值(对象或者原始值) 都可以作为一个键或一个值。构造函数`Map`可以接受一个数组作为参数。

**Map和Object的区别**

- 一个`Object` 的键只能是字符串或者 `Symbols`，但一个`Map` 的键可以是任意值。
- `Map`中的键值是有序的（FIFO 原则），而添加到对象中的键则不是。
- `Map`的键值对个数可以从 size 属性获取，而 `Object` 的键值对个数只能手动计算。
- `Object` 都有自己的原型，原型链上的键名有可能和你自己在对象上的设置的键名产生冲突。

**Map对象的属性**

- size：返回Map对象中所包含的键值对个数

**Map对象的方法**

- set(key, val): 向Map中添加新元素
- get(key): 通过键值查找特定的数值并返回
- has(key): 判断Map对象中是否有Key所对应的值，有返回true,否则返回false
- delete(key): 通过键值从Map中移除对应的数据
- clear(): 将这个Map中的所有元素删除

**遍历方法**

- `keys()`：返回键名的遍历器

- `values()`：返回键值的遍历器

- `entries()`：返回键值对的遍历器

- `forEach()`：使用回调函数遍历每个成员

  

## ES2021-ES2019

### String.protype.replaceAll

在 ES2021 之前，要替换掉一个字符串中的所有指定字符，我们可以这么做：

```ts
const str = "a+b+c+";
const newStr = str.replace(/\+/g, "🤣");
console.log(newStr); //a🤣b🤣c🤣

```

ES2021 则提出了 `replaceAll` 方法，并将其挂载在 String 的原型上，可以这么用：

```ts
const str = "a+b+c+";
const newStr = str.replaceAll("+", "🤣");
console.log(newStr); //a🤣b🤣c🤣

```



### Promise.any

```
Promise.any
```

- 接收一个 Promise 可迭代对象，只要其中任意一个 promise 成功，就返回那个已经成功的 promise
- 如果所有的 promises 都失败/拒绝，就返回一个失败的 promise

`Promise.race` 的对比:

- 只要任意一个 promise 的状态改变(不管成功 or 失败)，那么就返回那个 promise

`Promise.all`的对比

- 只要任意一个 promise 失败，则返回失败的 promise

- 当所有异步操作都成功后，才返回 promise,返回值组成一个数组

  

### Promise.allSettled

`Promise.allSettled()`方法接受一组 Promise 实例作为参数，包装成一个新的 Promise 实例。只有等到所有这些参数实例都返回结果，不管是`fulfilled`还是`rejected`，包装实例才会结束

有时候，我们不关心异步请求的结果，只关心所有的请求有没有结束。这时，`Promise.allSettled()`方法就很有用



### 逻辑赋值运算符

详细信息参考[ts39-proposal-logical-assignment](https://tc39.es/proposal-logical-assignment/)

逻辑赋值运算符结合了逻辑运算符和赋值表达式。逻辑赋值运算符有两种：

- 或等于（`||=`）
- 且等于（`&&=`）
- `??=`

### `||=`

```ts
const giveKey = () => {
  return "somekey";
};
let userDetails = { name: "chika", age: 5, room: 10, key: "" };
userDetails.key ||= giveKey();
console.log(userDetails.key);

//output : somekey
复制代码
```

### &&=

```ts
const deleteKey = () => {
  return " ";
};
let userDetails = { name: "chika", age: 5, room: 10, key: "990000" };
userDetails.key &&= deleteKey();
console.log(userDetails.key);

//output : ""
```

### 可选链操作符（Optional Chaining）

`?.` 也叫链判断运算符。它允许开发人员读取深度嵌套在对象链中的属性值，而不必验证每个引用。当引用为空时，表达式停止计算并返回 `undefined`。比如：

```ts
var travelPlans = {
  destination: "DC",
  monday: {
    location: "National Mall",
    budget: 200,
  },
};
console.log(travelPlans.tuesday?.location); // => undefined
复制代码
```

现在，把我们刚刚学到的结合起来

```ts
function addPlansWhenUndefined(plans, location, budget) {
  if (plans.tuesday?.location == undefined) {
    var newPlans = {
      plans,
      tuesday: {
        location: location ?? "公园",
        budget: budget ?? 200,
      },
    };
  } else {
    newPlans ??= plans; // 只有 newPlans 是 undefined 时，才覆盖
    console.log("已安排计划");
  }
  return newPlans;
}
// 对象 travelPlans 的初始值，来自上面一个例子
var newPlans = addPlansWhenUndefined(travelPlans, "Ford 剧院", null);
console.log(newPlans);
// => { plans:
// { destination: 'DC',
// monday: { location: '国家购物中心', budget: 200 } },
// tuesday: { location: 'Ford 剧院', budget: 200 } }
newPlans = addPlansWhenUndefined(newPlans, null, null);
// logs => 已安排计划
// returns => newPlans object
```

### Array#{flat,flatMap}

数组的成员有时还是数组，`Array.prototype.flat()`用于将嵌套的数组“拉平”，变成一维的数组。该方法返回一个新数组，对原数据没有影响。

```ts
[1, 2, [3, 4]].flat();
// [1, 2, 3, 4]
复制代码
```

`flatMap()`只能展开一层数组。

```ts
// 相当于 [[[2]], [[4]], [[6]], [[8]]].flat()
[1, 2, 3, 4].flatMap((x) => [[x * 2]]);
// [[2], [4], [6], [8]]
```

