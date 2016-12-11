ES6_快速入门
============

---

ECMAScript 6.0（以下简称ES6）是JavaScript语言的下一代标准，已经在2015年6月正式发布了。它的目标，是使得JavaScript语言可以用来编写复杂的大型应用程序，成为企业级开发语言。

-	各大浏览器的最新版本，对ES6的支持可以查看kangax.github.io/es5-compat-table/es6/。

![](./docs/images/es6.jpg)

---

如何玩ES6?
----------

-	Traceur转码器`javascript
	<script src="https://google.github.io/traceur-compiler/bin/traceur.js"></script>
	<script src="https://google.github.io/traceur-compiler/bin/BrowserSystem.js"></script>
	<script src="https://google.github.io/traceur-compiler/src/bootstrap.js"></script>
	<script type="module">
	import './Greeter.js';  // 除了引用外部ES6脚本，也可以直接在网页中放置ES6代码。
	</script>
	`
-	在线转换
	-	[Traceur 在线编译器](http://google.github.io/traceur-compiler/demo/repl.html#)
	-	[Babel提供一个REPL在线编译器](https://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=es2015%2Creact%2Cstage-2&code=)
-	babel转码(需要配合npm 使用)
-	node直接运行 (对于es6的功能分成了3个部分:shipping, staged 和 in progress. shipping功能:这些功能是已经稳定的。已经写入了node.js中的，直接就可以使用 staged功能:此功能是几乎完成的功能,但是v8团队没有考虑稳定性，需要使用--harmony. in progress功能: 此功能是需要写出标签的,比如你上面写的--harmony_destructuring.你可以通过下面的命令查看)
	-	node xx.js
	-	node --harmony xx.js
	-	node --harmony_destructuring xx.js `node --v8-options | grep 'in progress'`

---

let和const命令
--------------

```javascript
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

它的用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效。\`\`` javascript console.log(foo); // 输出undefined console.log(bar); // 报错ReferenceError

var foo = 2; let bar = 2;`
不存在变量提升(变量一定要在声明后使用)
` var tmp = 123;

if (true) { tmp = 'abc'; // ReferenceError let tmp; }\`\`\` 暂时性死区(temporal dead zone，简称TDZ) ES6明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

```javascript
// 第一种场景，内层变量可能会覆盖外层变量。
var tmp = new Date();

function f() {
  console.log(tmp);
  if (false) {
    var tmp = "hello world";
  }
}

f(); // undefined
// 第二种场景，用来计数的循环变量泄露为全局变量。
var s = 'hello';

for (var i = 0; i < s.length; i++) {
  console.log(s[i]);
}

console.log(i); // 5
```

为什么需要块级作用域？ ES5只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。`javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
` let实际上为JavaScript新增了块级作用域。 内层作用域可以定义外层作用域的同名变量。\`\``javascript // IIFE 写法 (function () { var tmp = ...; ... }());

// 块级作用域写法 { let tmp = ...; ... }`
块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。
` javascript const PI = 3.1415; PI // 3.1415

PI = 3; // TypeError: Assignment to constant variable. const foo; // SyntaxError: Missing initializer in const declaration`
对于const来说，只声明不赋值，就会报错。
`javascript if (true) { const MAX = 5; }

MAX // Uncaught ReferenceError: MAX is not defined`
const的作用域与let命令相同：只在声明所在的块级作用域内有效
`javascript if (true) { console.log(MAX); // ReferenceError const MAX = 5; }`
const命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。
`javascript const foo = {}; foo.prop = 123;

foo.prop // 123

foo = {}; // TypeError: "foo" is read-only`
对于复合类型的变量，变量名不指向数据，而是指向数据所在的地址。const命令只是保证变量名指向的地址不变，并不保证该地址的数据不变.
`javascript const foo = Object.freeze({});

// 常规模式时，下面一行不起作用； // 严格模式时，该行会报错 foo.prop = 123;`
如果真的想将对象冻结，应该使用Object.freeze方法。
`javascript var constantize = (obj) => { Object.freeze(obj); Object.keys(obj).forEach( (key, value) => { if ( typeof obj[key] === 'object' ) { constantize( obj[key] ); } }); };`
除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。
`javascript var a = 1; // 如果在Node的REPL环境，可以写成global.a // 或者采用通用方法，写成this.a window.a // 1

let b = 1; window.b // undefined\`\`\` ES6为了改变这一点，一方面规定，为了保持兼容性，var命令和function命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。也就是说，从ES6开始，全局变量将逐步与顶层对象的属性脱钩。

**总结**：ES5只有两种声明变量的方法：var命令和function命令。ES6除了添加let和const命令，后面章节还会提到，另外两种声明变量的方法：import命令和class命令。所以，ES6一共有6种声明变量的方法。

---

解构赋值
--------

```javascript
var [a,{b,c},[d,e]] = [12,{b:100,c:20},[30,15]];
let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```

```javascript
var [foo = true] = [];
foo // true

[x, y = 'b'] = ['a']; // x='a', y='b'
[x, y = 'b'] = ['a', undefined]; // x='a', y='b'
```

解构赋值允许指定默认值。\`\`` javascript var [x = 1] = [undefined]; x // 1

var [x = 1] = [null]; x // null\`\`\` 注意，ES6内部使用严格相等运算符（===），判断一个位置是否有值。所以，如果一个数组成员不严格等于undefined，默认值是不会生效的。

---

字符串的扩展
------------

```javascript
var x = 1;
var y = 2;

`${x} + ${y} = ${x + y}`
// "1 + 2 = 3"

`${x} + ${y * 2} = ${x + y * 2}`
// "1 + 4 = 5"

var obj = {x: 1, y: 2};
`${obj.x + obj.y}`
// 3
```

---

数组的扩展
----------

```javascript
let arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};

// ES5的写法
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']

// ES6的写法
let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
```

```javascript
var arr = [1,2,3];
var arr2 = [...arr]; // 复制数组(json不行)

function show(...args){
    console.log(args);          //[1,2,3]
}
show(1,2,3)
```

扩展运算符 ...

---

课堂练习：JSX 语法
------------------

浏览器打开`demos/jsx-demo/index.html`，按照[《操作说明》](../demos/README.md#jsx)，完成练习。

```javascript
ReactDOM.render(
  <span>Hello World!</span>,
  document.getElementById('example')
);
```

---

示例：React 组件
----------------

React 允许用户定义自己的组件，插入网页。

浏览器打开`demos/react-component-demo/index1.html`，按照[《操作说明》](../demos/README.md#react-组件语法)，仔细查看源码。

```javascript
class MyTitle extends React.Component {
  render() {
    return <h1>Hello World</h1>;
  }
};

ReactDOM.render(
  <MyTitle/>,
  document.getElementById('example')
);
```

---

课堂练习：组件的参数
--------------------

组件可以从外部传入参数，内部使用`this.props`获取参数。

打开`demos/react-component-demo/index2.html`，按照[《操作说明》](../demos/README.md#react-组件的参数)，完成练习。

```javascript
class MyTitle extends React.Component {
  render() {
    return <h1
      style={{color: this.props.color}}
    >Hello World</h1>;
  }
};

<MyTitle color="red" />,
```

---

示例：组件的状态
----------------

组件往往会有内部状态，使用`this.state`表示。

浏览器打开`demos/react-component-demo/index3.html`，按照[《操作说明》](../demos/README.md#react-组件的状态)，仔细查看源码。

![](./images/react-component-state.png)

---

课堂练习：React 组件实战
------------------------

浏览器打开`demos/react-component-demo/index4.html`，按照[《操作说明》](../demos/README.md#react-组件实战)，完成练习。

---

组件的生命周期
--------------

React 为组件的不同生命阶段，提供了近十个钩子方法。

-	`componentWillMount()`：组件加载前调用
-	`componentDidMount()`：组件加载后调用
-	`componentWillUpdate()`: 组件更新前调用
-	`componentDidUpdate()`: 组件更新后调用
-	`componentWillUnmount()`：组件卸载前调用
-	`componentWillReceiveProps()`：组件接受新的参数时调用

我们可以利用这些钩子，自动完成一些操作。

---

课堂练习：组件的生命周期
------------------------

组件可以通过 Ajax 请求，从服务器获取数据。Ajax 请求一般在`componentDidMount`方法里面发出。

```javascript
componentDidMount() {
  const url = '...';
  $.getJSON(url)
    .done()
    .fail();
}
```

打开`demos/react-lifecycle-demo/index.html`，按照[《操作说明》](../demos/README.md#react-组件的生命周期)，完成练习。

---

React 组件库
------------

React 的一大优势，就是网上有很多已经写好的组件库，可以使用。

React-Bootstrap：https://react-bootstrap.github.io/

![](./images/react-bootstrap.png)

---

示例：ReCharts
--------------

ReCharts 是一个 React 图表组件库。http://recharts.org/

浏览器打开`demos/recharts-demo/index.html`，按照[《操作说明》](../demos/README.md#recharts)，仔细查看源码。

```html
<LineChart width={1000} height={400} data={data}>
  <XAxis dataKey="name"/>
  <YAxis/>
  <CartesianGrid stroke="#eee" strokeDasharray="5 5"/>
  <Line type="monotone" dataKey="uv" stroke="#8884d8" />
  <Line type="monotone" dataKey="pv" stroke="#82ca9d" />
</LineChart>
```

---

React 的核心思想
----------------

View 是 State 的输出。

```javascript
view = f(state)
```

只要 State 发生变化，View 也要随之变化。

---

React 的本质是将图形界面（GUI）函数化。

```javascript
const person = {
  name: "michel",
  age: 31
}

const App = ({ person }) => <h1>{ person.name }</h1>

ReactDOM.render(<App person={person} />, document.body)
```

---

React 应用的架构
----------------

React 只是一个 DOM 的抽象层，并没有解决应用程序的架构问题：大型应用程序应该如何组织代码？

Facebook 提出 Flux 架构的概念。

![](./images/flow.png)

最大特点：数据单向流动

---

目前最流行的两个 React 架构
---------------------------

React 架构的最重要作用：管理 Store 与 View 之间的关系。

-	MobX：响应式（Reative）管理，state 是可变对象，适合中小型项目
-	Redux：函数式（Functional）管理，state 是不可变对象，适合大型项目

---

MobX 架构
---------

MobX 的核心是观察者模式。

-	Store 是被观察者（observable）
-	组件是观察者（observer）

一旦`Store`有变化，会立刻被组件观察到，从而引发重新渲染。

---

MobX 的最简单例子
-----------------

```javascript
const {observable, computed} = mobx;
const {observer} = mobxReact;

const person = observable({name: "张三", age: 31});

const App = observer(
  ({ person }) => <h1>{ person.name }</h1>
);

ReactDOM.render(<App person={person} />, document.body);
person.name = "李四";
```

代码：`demos/mobx-demo/browser-demo`目录

---

示例：MobX
----------

进入`demos/mobx-demo`目录，按照[《操作说明》](../demos/README.md#mobx)，理解 MobX 框架。

---

UI 层是观察者，Store 是被观察者。

Store 所有的属性，分成两大类：直接被观察的属性和自动计算出来的属性。

```javascript
class Store {
  @observable name = 'Bartek';
  @computed get decorated() {
    return `${this.name} is awesome!`;
  }
}
```

UI 会观察到 Store 的变化，自动重新渲染。

---

Redux 架构
----------

Redux 的核心概念

-	所有的状态存放在`Store`。组件每次重新渲染，都必须由状态变化引起。
-	用户在 UI 上发出`action`。
-	`reducer`函数接收`action`，然后根据当前的`state`，计算出新的`state`。

![](./images/redux-architecture.png)

---

示例：Redux
-----------

进入`demos/redux-demo`目录，按照[《操作说明》](../demos/README.md#redux)，理解 Redux 框架。

---

-	Redux 将组件分成 UI 组件和容器组件两类。
-	UI 组件是纯组件，需要用户自己写。

```javascript
<div className="index">
  <p>{this.props.text}</p>
  <input
    defaultValue={this.props.name}
    onChange={this.props.onChange}
  />
</div>
```

---

容器组件在用户给出配置以后，由 Redux 生成。

```javascript、
// MyComponent 是纯的 UI 组件
const App = connect(
  mapStateToProps,
  mapDispatchToProps
)(MyComponent);
```

-	mapStateToProps: 定义 UI 组件参数与 State 之间的映射
-	mapDispatchToProps：定义 UI 组件与 Action 之间的映射

---

`reducer`是一个纯函数，用来接收`action`，算出新的`state`。

```javascript
function reducer(state = {
  text: '你好，访问者',
  name: '访问者'
}, action) {
  switch (action.type) {
    case 'change':
      return {
        name: action.payload,
        text: '你好，' + action.payload
      };
  }
}
```

---

-	`Store`由 Redux 提供的`createStore`方法生成，该方法接受`reducer`作为参数。
-	为了把`Store`传入组件，必须使用 Redux 提供的`Provider`组件在应用的最外面，包裹一层。

```javascript
const store = createStore(reducer);

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.body.appendChild(document.createElement('div'))
);
```
