ES6_快速入门
============

---

> ECMAScript 6.0（以下简称ES6）是JavaScript语言的下一代标准，已经在2015年6月正式发布了。它的目标，是使得JavaScript语言可以用来编写复杂的大型应用程序，成为企业级开发语言。

-	各大浏览器的最新版本，对ES6的支持可以查看kangax.github.io/es5-compat-table/es6/。

![](./docs/images/es6.jpg)

---

如何玩ES6?
----------

-	Traceur转码器

```javascript
	<script src="https://google.github.io/traceur-compiler/bin/traceur.js"></script>
	<script src="https://google.github.io/traceur-compiler/bin/BrowserSystem.js"></script>
	<script src="https://google.github.io/traceur-compiler/src/bootstrap.js"></script>
	<script type="module">
	import './Greeter.js';  // 除了引用外部ES6脚本，也可以直接在网页中放置ES6代码。
	</script>
```

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

它的用法类似于var，但是所声明的变量，只在let命令所在的代码块内有效

```javascript
console.log(foo); // 输出undefined
console.log(bar); // 报错ReferenceError

var foo = 2;
let bar = 2;
```

不存在变量提升(变量一定要在声明后使用)

```javascript
var tmp = 123;

if (true) {
    tmp = 'abc'; // ReferenceError
    let tmp;
}
```

暂时性死区(temporal dead zone，简称TDZ) ES6明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

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

为什么需要块级作用域？ ES5只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。

```javascript
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
```

let实际上为JavaScript新增了块级作用域。 内层作用域可以定义外层作用域的同名变量。

```javascript
// IIFE 写法 (function () { var tmp = ...; ... }());

// 块级作用域写法 { let tmp = ...; ... }
```

块级作用域的出现，实际上使得获得广泛应用的立即执行函数表达式（IIFE）不再必要了。

```javascript
const PI = 3.1415; PI // 3.1415

PI = 3; // TypeError: Assignment to constant variable.
const foo; // SyntaxError: Missing initializer in const declaration
```

对于const来说，只声明不赋值，就会报错。

```javascript
if (true) { const MAX = 5; }

MAX // Uncaught ReferenceError: MAX is not defined
```

const的作用域与let命令相同：只在声明所在的块级作用域内有效

```javascript
if (true) { console.log(MAX); // ReferenceError const MAX = 5; }
```

const命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。

```javascript
const foo = {};
foo.prop = 123;

foo.prop // 123

foo = {}; // TypeError: "foo" is read-only
```

对于复合类型的变量，变量名不指向数据，而是指向数据所在的地址。const命令只是保证变量名指向的地址不变，并不保证该地址的数据不变.

```javascript
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错 foo.prop = 123;
```

如果真的想将对象冻结，应该使用Object.freeze方法。

```javascript
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, value) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```

除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。

```javascript
var a = 1; // 如果在Node的REPL环境，可以写成global.a
// 或者采用通用方法，写成this.a
window.a // 1

let b = 1;
window.b // undefined
```

ES6为了改变这一点，一方面规定，为了保持兼容性，var命令和function命令声明的全局变量，依旧是顶层对象的属性；另一方面规定，let命令、const命令、class命令声明的全局变量，不属于顶层对象的属性。也就是说，从ES6开始，全局变量将逐步与顶层对象的属性脱钩。

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

解构赋值允许指定默认值。

```javascript 
var [x = 1] = [undefined]; // x // 1

var [x = 1] = [null]; //  x // null

```
 注意，ES6内部使用严格相等运算符（===），判断一个位置是否有值。所以，如果一个数组成员不严格等于undefined，默认值是不会生效的。

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
1. Array.from()  
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
Array.from方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括ES6新增的数据结构Set和Map）。
- 实际应用中，常见的类似数组的对象是DOM操作返回的NodeList集合，以及函数内部的arguments对象。Array.from都可以将它们转为真正的数组。
- 所谓类似数组的对象，本质特征只有一点，即必须有length属性
- 扩展运算符背后调用的是遍历器接口（Symbol.iterator）,不支持将类似数组对象转为数组
```javascript
Array.from('hello')
// ['h', 'e', 'l', 'l', 'o']

let namesSet = new Set(['a', 'b'])
Array.from(namesSet) // ['a', 'b']
```
字符串和Set结构都具有Iterator接口，因此可以被Array.from转为真正的数组。
```javascript
Array.from(arrayLike, x => x * x);
// 等同于
Array.from(arrayLike).map(x => x * x);

Array.from([1, 2, 3], (x) => x * x)
// [1, 4, 9]
Array.from({ length: 2 }, () => 'jack')
// ['jack', 'jack']
```
Array.from还可以接受第二个参数，作用类似于数组的map方法.如果map函数里面用到了this关键字，还可以传入Array.from的第三个参数，用来绑定this

2. Array.of()
```javascript
Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1
Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
````
Array.of方法用于将一组值，转换为数组。
3. copyWithin()
> Array.prototype.copyWithin(target, start = 0, end = this.length)
```javascript
   [1, 2, 3, 4, 5].copyWithin(0, 3)     //[4, 5, 3, 4, 5]
   // 将3号位复制到0号位
   [1, 2, 3, 4, 5].copyWithin(0, 3, 4)
   // [4, 2, 3, 4, 5]
   
   // -2相当于3号位，-1相当于4号位
   [1, 2, 3, 4, 5].copyWithin(0, -2, -1)
````
4. find() && findIndex()
```javascript
[1, 5, 10, 15].find(function(value, index, arr) { //当前的值 当前的位置 原数组
  return value > 9;
}) // 10
[1, 5, 10, 15].findIndex(function(value, index, arr) {
  return value > 9;
}) // 2
```
这两个方法都可以接受第二个参数，用来绑定回调函数的this对象。
```javascript
[NaN].indexOf(NaN)
// -1

[NaN].findIndex(y => Object.is(NaN, y))
```
两个方法都可以发现NaN，弥补了数组的IndexOf方法的不足--indexOf方法无法识别数组的NaN成员

5. fill()
```javascript
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]
['a', 'b', 'c'].fill(7, 1, 2)
// ['a', 7, 'c']
```
fill方法使用给定值，填充一个数组。
fill方法还可以接受第二个和第三个参数，用于指定填充的起始位置和结束位置。

6.数组实例的 entries(), keys(), values()
```javascript
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```
ES6提供三个新的方法——entries()，keys()和values()——用于遍历数组。它们都返回一个遍历器对象,可以用for...of循环进行遍历
```javascript
let letter = ['a', 'b', 'c'];
let entries = letter.entries();
console.log(entries.next().value); // [0, 'a']
console.log(entries.next().value); // [1, 'b']
console.log(entries.next().value); // [2, 'c']
```
如果不使用for...of循环，可以手动调用遍历器对象的next方法，进行遍历。

7. 数组实例的includes()
```javascript
[1, 2, NaN].includes(NaN); // true
[1, 2, 3].includes(3, -1); // true //第二个参数表示搜索的起始位置，默认为0
```
没有该方法以前，采用indexOf方法; indexOf方法有两个缺点
- 不够语义化，它的含义是找到参数值的第一个出现位置，所以要去比较是否不等于-1，表达起来不够直观。
- 它内部使用严格相当运算符（===）进行判断，这会导致对NaN的误判。

**注意** Map和Set数据结构有一个has方法，需要注意与includes区分。
- Map结构的has方法，是用来查找键名的，比如Map.prototype.has(key)、WeakMap.prototype.has(key)、Reflect.has(target, propertyKey)。
- Set结构的has方法，是用来查找值的，比如Set.prototype.has(value)、WeakSet.prototype.has(value)。

8. 数组的空位
```javascript
Array(3) // [, , ,]
```
Array(3)返回一个具有3个空位的数组。
```javascript
0 in [undefined, undefined, undefined] // true
0 in [, , ,] // false
```
**注意** 空位不是undefined，一个位置的值等于undefined，依然是有值的。空位是没有任何值，in运算符可以说明这一点。

---

##Set && Map
```javascript
// 例一
var set = new Set([1, 2, 3, 4, 4]);
[...set]
// [1, 2, 3, 4]

// 例二
var items = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
items.size // 5

// 例三
function divs () {
  return [...document.querySelectorAll('div')];
}

var set = new Set(divs());
set.size // 56

// 类似于
divs().forEach(div => set.add(div));
set.size // 56
```
`Set函数可以接受一个数组（或类似数组的对象）作为参数，用来初始化。`
```javascript
// 去除数组的重复成员
[...new Set(array)]
// 去除数组重复成员的第二种方法
function dedupe(array) {
  return Array.from(new Set(array));
}

dedupe([1, 1, 2, 3]) // [1, 2, 3]
```
```javascript
let set = new Set();
set.add(5);
set.add('5');
console.log(set.size) // 2
```
向Set加入值的时候，不会发生类型转换，所以5和"5"是两个不同的值。
```javascript
let set = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);
set // Set {NaN}
```
Set内部判断两个值是否不同，使用的算法叫做`“Same-value equality”`，它类似于精确相等运算符（===），主要的区别是NaN等于自身，而精确相等运算符认为NaN不等于自身。

**Set实例的属性和方法 **
- Set.prototype.constructor：构造函数，默认就是Set函数。
- Set.prototype.size：返回Set实例的成员总数。
- add(value); 返回Set结构本身
- delete(value); 删除某个值，返回一个布尔值，表示删除是否成功。
- has(value) 返回一个布尔值，表示该值是否为Set的成员。
- clear();    清除所有成员，没有返回值。

**遍历操作**
- keys(): 返回键名的遍历器
- values(): 返回键值的遍历器
- entries()：返回键值对的遍历器
- forEach()：使用回调函数遍历每个成员

`需要特别指出的是，Set的遍历顺序就是插入顺序。这个特性有时非常有用，比如使用Set保存一个回调函数列表，调用时就能保证按照添加顺序调用。`

```javascript
let set = new Set(['red', 'green', 'blue']);

for (let item of set.keys()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.values()) {
  console.log(item);
}
// red
// green
// blue

for (let item of set.entries()) {
  console.log(item);
}
// ["red", "red"]
// ["green", "green"]
// ["blue", "blue"]
```
entries方法返回的遍历器，同时包括键名和键值，所以每次输出一个数组，它的两个成员完全相等。
```javascript
Set.prototype[Symbol.iterator] === Set.prototype.values  // true
let set = new Set(['red', 'green', 'blue']);

for (let x of set) {
  console.log(x);
}
// red
// green
// blue
```
Set结构的实例默认可遍历，它的默认遍历器生成函数就是它的values方法。这意味着，可以省略values方法，直接用for...of循环遍历Set。

```javascript
let set = new Set([1, 2, 3]);
set.forEach((value, key) => console.log(value * 2) )
// 2
// 4
// 6
```
`forEach方法的参数就是一个处理函数。该函数的参数依次为键值、键名、集合本身（上例省略了该参数）。另外，forEach方法还可以有第二个参数，表示绑定的this对象。`
```javascript
let set = new Set(['red', 'green', 'blue']);
let arr = [...set];
// ['red', 'green', 'blue']
```
`扩展运算符（...）内部使用for...of循环，所以也可以用于Set结构。`
``javascript
let set = new Set([1, 2, 3]);
set = new Set([...set].map(x => x * 2));
// 返回Set结构：{2, 4, 6}

let set = new Set([1, 2, 3, 4, 5]);
set = new Set([...set].filter(x => (x % 2) == 0));
// 返回Set结构：{2, 4}
```
数组的map和filter方法也可以用于Set了。
```javascript
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);

// 并集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}

// 交集
let intersect = new Set([...a].filter(x => b.has(x)));
// set {2, 3}

// 差集
let difference = new Set([...a].filter(x => !b.has(x)));
// Set {1}
```
使用Set可以很容易地实现并集（Union）、交集（Intersect）和差集（Difference）。
```javascript
// 方法一
let set = new Set([1, 2, 3]);
set = new Set([...set].map(val => val * 2));
// set的值是2, 4, 6

// 方法二
let set = new Set([1, 2, 3]);
set = new Set(Array.from(set, val => val * 2));
// set的值是2, 4, 6
```
`上面代码提供了两种方法，直接在遍历操作中改变原来的Set结构。`

**WeakSet** 
**和Set的两个区别
- WeakSet的成员只能是对象，而不能是其他类型的值。
- WeakSet中的对象都是弱引用，无法引用WeakSet的成员，因此WeakSet是不可遍历的。
 
*Map结构的目的和基本用法 **
```javascript
var data = {};
var element = document.getElementById('myDiv');

data[element] = 'metadata';
data['[object HTMLDivElement]'] // "metadata"
```
- JavaScript的对象（Object），本质上是键值对的集合（Hash结构），但是传统上只能用字符串当作键。这给它的使用带来了很大的限制
- 为了解决这个问题，ES6提供了Map数据结构。它类似于对象，也是键值对的集合，但是“键”的范围不限于字符串，各种类型的值（包括对象）都可以当作键。也就是说，Object结构提供了“字符串—值”的对应，Map结构提供了“值—值”的对应，是一种更完善的Hash结构实现。如果你需要“键值对”的数据结构，Map比Object更合适。

```javascript
var m = new Map();
var o = {p: 'Hello World'};

m.set(o, 'content')
m.get(o) // "content"

m.has(o) // true
m.delete(o) // true
m.has(o) // false
```
```javascript
var map = new Map([
  ['name', '张三'],
  ['title', 'Author']
]);

map.size // 2
map.has('name') // true
map.get('name') // "张三"
map.has('title') // true
map.get('title') // "Author"
```
作为构造函数，Map也可以接受一个数组作为参数。该数组的成员是一个个表示键值对的数组。
```javascript
var m = new Map([
  [true, 'foo'],
  ['true', 'bar']
]);

m.get(true) // 'foo'
m.get('true') // 'bar'

let map = new Map();

map
.set(1, 'aaa')
.set(1, 'bbb');

map.get(1) // "bbb"
```
如果对同一个键多次赋值，后面的值将覆盖前面的值。
```javascript
var map = new Map();

map.set(['a'], 555);
map.get(['a']) // undefined
```
`注意，只有对同一个对象的引用，Map结构才将其视为同一个键。这一点要非常小心`

** Map结构的实例有以下属性和操作方法 **
- size属性
```javascript
let map = new Map();
map.set('foo', true);
map.set('bar', false);

map.size // 2
```
- set(key,value)
- get(key)  读取key对应的键值，如果找不到key，返回undefined。
- has(key)  返回一个布尔值，表示某个`键`是否在Map数据结构中。
- delete(key)  删除某个键，返回true。如果删除失败，返回false。
- clear()      方法清除所有成员，没有返回值。

**遍历方法**
```javascript
let map = new Map([
  ['F', 'no'],
  ['T',  'yes'],
]);

for (let key of map.keys()) {
  console.log(key);
}
// "F"
// "T"

for (let value of map.values()) {
  console.log(value);
}
// "no"
// "yes"

for (let item of map.entries()) {
  console.log(item[0], item[1]);
}
// "F" "no"
// "T" "yes"

// 或者
for (let [key, value] of map.entries()) {
  console.log(key, value);
}

// 等同于使用map.entries()
for (let [key, value] of map) {
  console.log(key, value);
}
```
`上面代码最后的那个例子，表示Map结构的默认遍历器接口（Symbol.iterator属性），就是entries方法。`
```javascript
let map = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);

[...map.keys()]
// [1, 2, 3]

[...map.values()]
// ['one', 'two', 'three']

[...map.entries()]
// [[1,'one'], [2, 'two'], [3, 'three']]

[...map]
// [[1,'one'], [2, 'two'], [3, 'three']]
```
Map结构转为数组结构，比较快速的方法是结合使用扩展运算符（...）。

```javascript
let map0 = new Map()
  .set(1, 'a')
  .set(2, 'b')
  .set(3, 'c');

let map1 = new Map(
  [...map0].filter(([k, v]) => k < 3)
);
// 产生Map结构 {1 => 'a', 2 => 'b'}

let map2 = new Map(
  [...map0].map(([k, v]) => [k * 2, '_' + v])
    );
// 产生Map结构 {2 => '_a', 4 => '_b', 6 => '_c'}
```
结合数组的map方法、filter方法
```javascript
map.forEach(function(value, key, map) {
  console.log("Key: %s, Value: %s", key, value);
});
```
Map还有一个forEach方法，与数组的forEach方法类似，也可以实现遍历。

## 函数
1. 函数参数的默认值
```javascript
function log(x, y = 'World') {
  console.log(x, y);
}

log('Hello') // Hello World
log('Hello', 'China') // Hello China
log('Hello', '') // Hello
```

2. rest参数
```javascript
// arguments变量的写法
function sortNumbers() {
  return Array.prototype.slice.call(arguments).sort();
}

// rest参数的写法
const sortNumbers = (...numbers) => numbers.sort();
```

3. 扩展运算符
```javascript
console.log(...[1, 2, 3])
// 1 2 3

console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5

[...document.querySelectorAll('div')]
// [<div>, <div>, <div>]
```

4. 严格模式
5. name属性
```javascript
var func1 = function () {};

// ES5
func1.name // ""

// ES6
func1.name // "func1"
```

6. 箭头函数
**使用注意点**
- 函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。
- 不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。
- 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用Rest参数代替。
- 不可以使用yield命令，因此箭头函数不能用作Generator函数。
```javascript
function foo() {
  setTimeout(function(){
  	console.log('id:', this.id);
  }, 100);
}

var id = 21;

foo.call({ id: 42 });
```
如果是普通函数，执行时this应该指向全局对象window，这时应该输出21。但是，箭头函数导致this总是指向函数定义生效时所在的对象（本例是{id: 42}），所以输出的是42
```javascript
function Timer() {
  this.s1 = 0;
  this.s2 = 0;
  // 箭头函数
  setInterval(() => this.s1++, 1000);
  // 普通函数
  setInterval(function () {
    this.s2++;
  }, 1000);
}

var timer = new Timer();

setTimeout(() => console.log('s1: ', timer.s1), 3100);
setTimeout(() => console.log('s2: ', timer.s2), 3100);
// s1: 3
// s2: 0
```

7. 绑定this
ES7提出了“函数绑定”（function bind）运算符，用来取代call、apply、bind调用
8. 尾调用优化




## Class







