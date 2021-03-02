#                                ts学习笔记

## 一、 基础类型

* **布尔值:boolean**

  ```typescript
  let isDone: boolean = false
  ```

* **数字:number**

  ```typescript
  let decLiteral: number = 6
  ```

* **字符串:string**

  ```tsx
  let name: string = 'Great'
  ```

* **数组: [] || Array\<number>**

  第一种方式：

  ```tsx
  let list: number[] = [1, 2, 3]
  ```

  第二种数组范型方式：

  ```tsx
  let list: Array<number> = [1, 2, 3]
  ```

* **元组：Tuple**

  元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。 比如，你可以定义一对值分别为`string`和`number`类型的元组。

  ```ts
  // Declare a tuple type
  let x: [string, number];
  // Initialize it
  x = ['hello', 10]; // OK
  // Initialize it incorrectly
  x = [10, 'hello']; // Error
  ```

  当访问一个已知索引的元素，会得到正确的类型：

  ```ts
  console.log(x[0].substr(1)); // OK
  console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
  ```

  当访问一个越界的元素，会使用联合类型替代：

  ```ts
  x[3] = 'world'; // OK, 字符串可以赋值给(string | number)类型
  
  console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString
  
  x[6] = true; // Error, 布尔不是(string | number)类型
  ```

* **枚举: enum**

  用于定义一组数值名字：

  ```ts
  enum Color {Red = 1, Green, Blue}
  let colorName: string = Color[2];
  
  console.log(colorName); // 显示'Green'因为上面代码里它的值是2
  ```

* **任意值：any**

  ```ts
  let list: any[] = [1, true, "free"];
  
  list[1] = 100;
  ```

* **空值：void**

  某种程度上来说，`void`类型像是与`any`类型相反，它表示没有任何类型。 当一个函数没有返回值时，你通常会见到其返回值类型是`void`：

  ```ts
  function warnUser(): void {
      console.log("This is my warning message");
  }
  ```

  声明一个`void`类型的变量没有什么大用，因为你只能为它赋予`undefined`和`null`：

  ```ts
  let unusable: void = undefined;
  ```

* **对象：bject**

  `object`表示非原始类型，也就是除`number`，`string`，`boolean`，`symbol`，`null`或`undefined`之外的类型。使用`object`类型，就可以更好的表示像`Object.create`这样的API。例如：

  ```ts
  declare function create(o: object | null): void;
  
  create({ prop: 0 }); // OK
  create(null); // OK
  
  create(42); // Error
  create("string"); // Error
  create(false); // Error
  create(undefined); // Error
  ```

* **断言：<>和as**

  通过*类型断言*这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 TypeScript会假设你，程序员，已经进行了必须的检查。

  类型断言有两种形式。 其一是“尖括号”语法：

  ```ts
  let someValue: any = "this is a string";
  
  let strLength: number = (<string>someValue).length;
  ```

  另一个为`as`语法：

  ```ts
  let someValue: any = "this is a string";
  
  let strLength: number = (someValue as string).length;
  ```

  两种形式是等价的。 至于使用哪个大多数情况下是凭个人喜好；然而，**当你在TypeScript里使用JSX时，只有`as`语法断言是被允许**的。

  ****

  ## 

## 二、接口

## `readonly` vs `const`

最简单判断该用`readonly`还是`const`的方法是看要把它做为变量使用还是做为一个属性。 做为变量使用的话用`const`，若做为属性则使用`readonly`。

### 额外的属性检查

对象字面量会被特殊对待而且会经过*额外属性检查*，当将它们赋值给变量或作为参数传递的时候。 如果一个对象字面量存在任何“目标类型”不包含的属性时，你会得到一个错误。

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

绕开这些检查非常简单。 最简便的方法是使用类型断言：

```ts
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

然而，最佳的方式是能够添加一个字符串索引签名，前提是你能够确定这个对象可能具有某些做为特殊用途使用的额外属性。 如果`SquareConfig`带有上面定义的类型的`color`和`width`属性，并且*还会*带有任意数量的其它属性，那么我们可以这样定义它：

```ts
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

我们稍后会讲到索引签名，但在这我们要表示的是`SquareConfig`可以有任意数量的属性，并且只要它们不是`color`和`width`，那么就无所谓它们的类型是什么。

还有最后一种跳过这些检查的方式，这可能会让你感到惊讶，它就是将这个对象赋值给一个另一个变量： 因为`squareOptions`不会经过额外属性检查，所以编译器不会报错。

```ts
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

**重点：以对象字面量的方式传递参数，需要完全匹配定义的类型属性**

### 函数类型接口

为了使用接口表示函数类型，需要给接口定义一个调用签名。 它就像是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。

```ts
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

这样定义后，我们可以像使用其它接口一样使用这个函数类型的接口。 下例展示了如何创建一个函数类型的变量，并将一个同类型的函数赋值给这个变量。

```ts
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  return result > -1;
}
```

对于函数类型的类型检查来说，函数的参数名不需要与接口里定义的名字相匹配。 比如，我们使用下面的代码重写上面的例子：

```ts
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
  let result = src.search(sub);
  return result > -1;
}
```

### 索引类型接口

可索引类型具有一个*索引签名*，它描述了对象索引的类型，还有相应的索引返回值类型。 例子：

```ts
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

可以将索引签名设置为只读，这样就防止了给索引赋值：

```ts
interface ReadonlyStringArray {
    readonly [index: number]: string;
}
let myArray: ReadonlyStringArray = ["Alice", "Bob"];
myArray[2] = "Mallory"; // error!
```

### 类类型接口

与C#或Java里接口的基本作用一样，TypeScript也能够用它来明确的强制一个类去符合某种契约。

```ts
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface {
    currentTime: Date = new Date();
    constructor(h: number, m: number) { }
}
```

可以在接口中描述一个方法，在类里实现它，如同下面的`setTime`方法一样：

```ts
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date): void;
}

class Clock implements ClockInterface {
    currentTime: Date = new Date();
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

接口描述了类的公共部分，而不是公共和私有两部分。 它不会帮你检查类是否具有某些私有成员。

## 三、类

#### protected

`protected`修饰符与`private`修饰符的行为很相似，但有一点不同，`protected`成员在派生类中仍然可以访问。例如：

```ts
class Person {
    protected name: string;
    constructor(name: string) { this.name = name; }
}

class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name)
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
console.log(howard.name); // 错误
```

注意，不能在`Person`类外使用`name`，但是我们仍然可以通过`Employee`类的实例方法访问，因为`Employee`是由`Person`派生而来的。

#### 静态属性

创建静态成员，只存在于类本身上，而不是实例上。每个实例想要访问这个属性的时候，都要在`origin`前面加上类名。 如同在实例属性上使用`this.`前缀来访问属性一样，这里我们使用`Grid.`来访问静态属性。

```ts
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

#### 抽象类

抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。 抽象方法的语法与接口方法相似。 两者都是定义方法签名但不包含方法体。 然而，抽象方法必须包含`abstract`关键字并且可以包含访问修饰符。

```ts
abstract class Department {

    constructor(public name: string) {
    }

    printName(): void {
        console.log('Department name: ' + this.name);
    }

    abstract printMeeting(): void; // 必须在派生类中实现
}

class AccountingDepartment extends Department {

    constructor() {
        super('Accounting and Auditing'); // 在派生类的构造函数中必须调用 super()
    }

    printMeeting(): void {
        console.log('The Accounting Department meets each Monday at 10am.');
    }

    generateReports(): void {
        console.log('Generating accounting reports...');
    }
}

let department: Department; // 允许创建一个对抽象类型的引用
department = new Department(); // 错误: 不能创建一个抽象类的实例
department = new AccountingDepartment(); // 允许对一个抽象子类进行实例化和赋值
department.printName();
department.printMeeting();
department.generateReports(); // 错误: 方法在声明的抽象类中不存在
```

### 高级技巧

#### 构造函数

在Typescript里声明了一个类的时候，实际上同时声明了很多东西。 首先就是类的*实例*的类型。

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter: Greeter;
greeter = new Greeter("world");
console.log(greeter.greet());
```

这里，写了`let greeter: Greeter`，意思是`Greeter`类的实例的类型是`Greeter`。

#### 把类当作接口

如上一节里所讲的，类定义会创建两个东西：类的实例类型和一个构造函数。 因为类可以创建出类型，所以你能够在允许使用接口的地方使用类。

```ts
class Point {
    x: number;
    y: number;
}

interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```

*****

## 四、泛型

### 介绍

软件工程中，我们不仅要创建一致的定义良好的API，同时也要考虑可重用性。 组件不仅能够支持当前的数据类型，同时也能支持未来的数据类型，这在创建大型系统时为你提供了十分灵活的功能。

在像C#和Java这样的语言中，可以使用`泛型`来创建可重用的组件，一个组件可以支持多种类型的数据。 这样用户就可以以自己的数据类型来使用组件。

### 泛型holleo wrold

不用泛型的话，这个函数可能是下面这样：

```ts
function identity(arg: number): number {
    return arg;
}
```

或者，我们使用`any`类型来定义函数：

```ts
function identity(arg: any): any {
    return arg;
}
```

使用`any`类型会导致这个函数可以接收任何类型的`arg`参数，这样就丢失了一些信息：传入的类型与返回的类型应该是相同的。 如果我们传入一个数字，我们只知道任何类型的值都有可能被返回。

因此，我们需要一种方法使返回值的类型与传入参数的类型是相同的。 这里，我们使用了*类型变量*，它是一种特殊的变量，只用于表示类型而不是值。

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

如果我们想同时打印出`arg`的长度。 我们很可能会这样做：

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

如果这么做，编译器会报错说我们使用了`arg`的`.length`属性，但是没有地方指明`arg`具有这个属性。 记住，这些类型变量代表的是任意类型，所以使用这个函数的人可能传入的是个数字，而数字是没有`.length`属性的。

### 泛型类型

```ts
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity；
```

### 泛型类

泛型类看上去与泛型接口差不多。 泛型类使用（`<>`）括起泛型类型，跟在类名后面。

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

### 泛型约束

你应该会记得之前的一个例子，我们有时候想操作某类型的一组值，并且我们知道这组值具有什么样的属性。 在`loggingIdentity`例子中，我们想访问`arg`的`length`属性，但是编译器并不能证明每种类型都有`length`属性，所以就报错了。

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

相比于操作any所有类型，我们想要限制函数去处理任意带有`.length`属性的所有类型。 只要传入的类型有这个属性，我们就允许，就是说至少包含这一属性。 为此，我们需要列出对于T的约束要求。

为此，我们定义一个接口来描述约束条件。 创建一个包含`.length`属性的接口，使用这个接口和`extends`关键字来实现约束：

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

现在这个泛型函数被定义了约束，因此它不再是适用于任意类型：

```ts
loggingIdentity(3);  // Error, number doesn't have a .length property
```

我们需要传入符合约束类型的值，必须包含必须的属性：

```ts
loggingIdentity({length: 10, value: 3});
```

## 五、实用类工具

ts提供了一些全局可见的工具类型。

### Partial<T>

构造类型`T`，并将它所有的属性设置为可选的。它的返回类型表示输入类型的所有子类型。

```ts
interface allKeys {
    ka: string,
    kb: number,
    kc: Array<number|string>
}
const func = (res: Partial<allKeys>) => {
    return res
}
func({ka: '1', kc: [1, '1']}) //{ ka: '1', kc: [ 1, '1' ] }
```

`partial`源码：

```ts
type Partial<T> = {
  [P in keyof T]?: T[P]
}
```

### Pick<T,K>

从类型`T`中挑选部分属性`K`来构造类型。

```ts
interface Todo {
    title: string;
    description: string;
    completed: boolean;
}

type TodoPreview = Pick<Todo, 'title' | 'completed'>;

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
};
```



## 六、*各种ts总结

ts是来公司，跟进项目需求时，开始照猫画虎的书写:number,:string,:any等，没有理解为什么要这样做，这样做是否上佳，如何**不死板、灵活的**书写强壮、可复用、可扩展的ts代码，这在日后开发或是重构项目，会是潜在灾难。这次在细细学习ts中，有了一些更深刻的理解。

### *简单实用性使用的总结

#### 定义对象类型接口里的函数

这是定义函数类型接口：

```ts
interface funcPointDel {
  (x: number, y: number): Array<number>
}

const createPointArr: funcPointDel = (x: number, y: number): Array<number> => [x, y] 
```

在定义对象里的函数时，要带上函数名，不能以签名的形式定义，这是定义对象类型接口里的函数：

```ts
interface pointDel<T> {
  x: number,
  y: number,
  point(x: T, y: T): Array<T> // 添加point命名 /point: (x: T, y: T) => Array<T>
}

const createPointobj: pointDel<number> = {x: 1, y: 2, point: (x, y) => [x, y]} 
const createPointobj: pointDel<number> = {x: 1, y: 2, z: 3, point: (x, y) => [x, y]} // error

// eg2
interface pointDel<T> {
    x: number,
    y: number,
    point(x: T, y: T): Array<T>
}
let likePointDel = {x: 1, y: 2, z: 3, point: (x: number, y: number) => [x, y]}
const createPointobj: pointDel<number> = likePointDel

// eg3：索引签名
//...
interface pointDel<T> {
    x: number,
    y: number,
    point(x: T, y: T): Array<T>,
    [propsName: string]: number
}
//...

// eg4: as
//...
const createPointobj: pointDel<number> = {x: 1, y: 2, z: 3, point: (x, y) => [x, y]} as pointDel
```

同时引出，当以对象字面量赋值，会被严格检查，需要内容完全相同才能通过，如上error处。处理方式有三种：如上，eg2，以变量赋值形式；或者用as强制转换，以及索引签名的方式。如上eg3，4。

#### 善用索引签名和泛型处理array和object

我的理解，范型和签名在书写ts非常高效，也是抽象到较高层面，相对于写死常用的类型，使用泛型能更好的创建**复用性**组件，同时支持未来不可知的数据类型变化，更加灵活。

当存在多个重复的数组item：

```ts
interface userArr<T> {
  [index: string]: T
}
const arrRes: userArr<string> = ['123', '321', '112']
```

当可能是较少几种类型的item：

```ts
interface userArr<T, K, Y> {
    [index: number]: T | K | Y
  }
  const arrRes: userArr<string, number, Array<number|string>> = [1, '321', '112', [1, 2]]
```

如果更多可能，那就考虑用any了。

同样处理obj接口如下：

```ts
interface userObj<T> {
  [key: string]: T
}
const userRes: userObj<string> = ['1', '2']
```

#### 接口和类型别名

Type和interface都可以定义类型，但是有一些区别：

1. Interface 用来描述对象类型接口，type可以用来描述原始值、联合类型、交叉、组合类型。
2. interface可以实现extends和implements。

```tsx
type name = string
type func<T> = (T) => string
type obj<T> {
	a: <T>
}
type funcAndName = name & func<T>
type funcOrName = name | func<T>

```

#### keyof

Keyof 和 object.keys 很类似，取interface的键：

```ts
interface obj {
  a: string,
  b: number
}

type keys = keyof obj // a|b
```

有下应用场景：

```ts
const func = (o: object, key: string) => {
  return o[key]
}
```

会造成并不能确定对象o里是否有传入的属性key。所以要通过keyof来约束：

```ts
const func <T extends object, K extends keyof T> = (o: T, key: k): T[K] {
  return o[key]
}
```



### 复杂问题的总结

#### extends和implements

Extends,是实现类或者接口的继承，在ts里只能继承一个类；而implements需要依赖类来具体实现，同时implements可以使类实现多个接口。

```tsx
interface a {
    dock(): void
}
interface b {
    mock(): void
}

class implementsClass implements a,b {
    constructor () {
    }
    dock= () => {
        console.log(1)
    }
    mock= () => {
    }
}

class sonClass extends implementsClass {
    constructor () {
        super()
        this.dock()
    }
}
new sonClass()
```







