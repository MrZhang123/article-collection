# 第一章: this 是什么

## 困惑

开发者用太过于字面的方式考虑`this`会产生以下误解：

### 它自己

第一种常见的倾向认为`this`指向它自己，至少这是一种在语法上的合理推测，但是，观察以下代码

```js
function foo(num) {
  console.log('foo: ' + num)

  // 追踪 `foo` 被调用了多少次
  this.count++
}

foo.count = 0

var i

for (i = 0; i < 10; i++) {
  if (i > 5) {
    foo(i)
  }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// `foo` 被调用了多少次？
console.log(foo.count) // 0 ????
```

当执行`foo.count = 0`时，它确实向`foo`添加一个`count`属性，但是对于内部函数`this`指向根本 **不是指向函数本身，而是指向全局的**。

其实代码中的`this.count++`不小心创建了一个全局变量`count` ，而且它当前的值是`NaN`。

将`this`指向`foo`本身的代码如下：

```js
function foo(num) {
  console.log('foo: ' + num)

  // 追踪 `foo` 被调用了多少次
  // 注意：由于 `foo` 的被调用方式（见下方），`this` 现在确实是 `foo`
  this.count++
}

foo.count = 0

var i

for (i = 0; i < 10; i++) {
  if (i > 5) {
    // 使用 `call(..)`，我们可以保证 `this` 指向函数对象(`foo`)
    foo.call(foo, i)
  }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// `foo` 被调用了多少次？
console.log(foo.count) // 4
```

在循环调用中，使用`call`手动将`this`指向`foo`本身

### 它的作用域

对 `this` 的含义第二常见的误解，是它不知怎的指向了函数的作用域。这是一个刁钻的问题，因为在某一种意义上它有正确的部分，而在另外一种意义上，它是严重的误导。

明确的说，`this`不会以任何方式指向函数的 `词法作用域`，作用域好像是一个将所有可用标识符作为属性的对象，这从内部来说是对的。但是 JavasScript 代码不能访问作用域“对象”。它是 **引擎** 的内部实现。

```js
function foo() {
  var a = 2
  this.bar()
}

function bar() {
  console.log(this.a)
}

foo() //undefined
```

以上代码希望通过`this`在`bar()`和`foo()`的词法作用域之间建立一座桥，使得`bar()`可以访问`foo()`内部的`a`，这是不可能做到的，不能使用 `this` 引用在词法作用域中查找东西。

## 什么是`this`?

`this`不是编写时绑定的，而是运行时绑定的，它依赖于函数调用的上下文条件。`this`绑定与函数声明的位置没有任何关系，而是与函数调用的方式有关系。

当一个函数被调用时，会建立一个成为执行环境的活动记录。这个记录包含函数从何处（调用栈——call-stack）被调用的，函数是`如何`被调用的，被传递了什么参数信息。这个记录的属性之一，就是函数执行期间被使用的`this`引用。

所以，`this`是一个完全根据`调用点`（函数是如何被调用的）而为每次函数调用建立的绑定。

# 第二章: `this`豁然开朗！

## 调用点（Call-site）

调用点：函数在代码中 **被调用** 的位置（**不是被声明的位置**）

调用栈：使我们到达当前执行位置而被调用的馊油方法的堆栈

```js
function baz() {
  // 调用栈是: `baz`
  // 我们的调用点是 global scope（全局作用域）

  console.log('baz')
  bar() // <-- `bar` 的调用点
}

function bar() {
  // 调用栈是: `baz` -> `bar`
  // 我们的调用点位于 `baz`

  console.log('bar')
  foo() // <-- `foo` 的 call-site
}

function foo() {
  // 调用栈是: `baz` -> `bar` -> `foo`
  // 我们的调用点位于 `bar`

  console.log('foo')
}

baz() // <-- `baz` 的调用点
```

在分析代码来寻找（从调用栈中）真正的调用点时要小心，因为它是影响`this`绑定的唯一因素。

如果想调查`this`绑定，可以使用开发者工具取得调用栈，之后从上到下找到第二个记录，那就是调用点

## 仅仅是规则

通过调用点寻找`this`指向的四条规则

### 1.默认绑定（Default Binding）

函数调用最常见的一种情况：独立函数调用，可以认为这种`this`规则是在没有其他规则适用时的 **默认规则**。

```js
function foo() {
  console.log(this.a)
}

var a = 2

foo() // 2
```

注意：

1.  全局作用域中声明 `var a = 2`， 是全局对象的同名属性的同义词。
2.  当`foo()`被调用时，`this.a`解析为全局变量`a`，因为在独立函数调用的情况下，`this`实施了 _默认绑定_，所以`this`指向全局对象。

为什么这里的`this`进行来默认绑定？因为在这段代码中`foo()`是被直接调用的，没有其他的我们将要展示的规则适用，所以这里 _默认绑定_ 适用。

如果 `strict mode`在这里生效，那么对于默认绑定来说全局对象是不合法的，所以`this`被设置为`undefined`。

```js
function foo() {
  'use strict'

  console.log(this.a)
}

var a = 2

foo() // TypeError: `this` is `undefined`
```

### 2.隐含绑定（Implicit Binding）

另一种要考虑的规则是：调用点是否有一个环境对象（context object），也成为拥有者或容器对象。

```js
function foo() {
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
}

obj.foo() // 2
```

首先有一点需要注意，`foo()` 被声明然后作为引用属性添加到`obj`上，无论`foo()`是否一开始就在`obj`上被声明，还是后来作为引用属性添加，这个函数都不被`obj`所真正拥有。

然而，调用点 *使用`obj`*环境来 **引用** 函数，所以可以说 **`obj`对象在函数被调用的时间点上“拥有”或“包含”这个 函数引用**。

因为`obj`是`foo()`调用的`this`，所以`this.a`就是`obj.a`的同义词。

<span style="text-decoration: underline">只有对象属性引用链的 **最后一层** 是影响调用点的，</span>例如：

```js
function foo() {
  console.log(this.a)
}

var obj2 = {
  a: 42,
  foo: foo
}

var obj1 = {
  a: 2,
  obj2: obj2
}

obj1.obj2.foo() // 42
```

#### 隐含丢失（Implicitly Lost）

`this` 绑定最常让人沮丧的事情之一，<span style="text-decoration: underline">**就是当一个 _隐含绑定_ 丢失了它的绑定，这通常意味着它会退回到默认绑定**</span>，根据 `strict mode` 的状态，其结果不是全局对象就是 `undefined`。

考虑这段代码：

```js
function foo() {
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
}

var bar = obj.foo // 函数引用！

var a = 'oops, global' // `a` 也是一个全局对象的属性

bar() // "oops, global"
```

尽管`bar`似乎是`obj.foo`的引用，但实际上它只是另一个`foo`本身的引用而已。另外，起作用的调用点是`bar()`，一个直白，毫无修饰的调用，所以 _默认绑定_ 在这里适用。

这种情况发生的更加微妙，更加常见的是考了传递一个回调函数时：

```js
function fo() {
  console.log(this.a)
}
function doFoo(fn) {
  // `fn`只不过是`foo`的另一个引用
  fn()
}
var obj = {
  a: 2,
  foo: foo
}

var a = 'opps, global' // a 也是一个全局对象的属性
deFoo(obj.foo) // "oops, global"
```

参数传递仅仅是一种隐含的赋值，而且因为我们在传递一个函数，它是隐含的引用赋值，所以最终结果和我们签一个代码段一致。

### 3.明确绑定（Explicit Binding）

可以使用`call`和`apply`进行明确的`this`绑定。它们接收的第一个参数都是一个用于`this`的对象，之后使用这个指定的`this`来调用函数。

```js
function foo() {
  console.log(this.a)
}
var obj = {
  a: 2
}
foo.call(obj) // 2
```

通过`foo.call()`使用 _明确绑定_ 来调用`foo`,允许我们强制函数的`this`指向`obj`。

如果传递一个简单基本类型（`string`，`boolean`，`number`）作为`this`绑定，那么这个基本类型值会被包装在它的对象类型中（分别是`new String(...)`，`new Boolean(...)`，`new Number(...)`）。这通常称为“封箱”。

> 注：就`this`绑定的角度而言，`call`和`apply`是完全一样的，它们只是在处理参数的方式上不同。

#### 3.1 硬绑定（Hard Binding）

有一个明确绑定的变种可以实现原本的`this`不丢失。

```js
function foo() {
  console.log(this.a)
}

var obj = {
  a: 2
}

var bar = function() {
  foo.call(obj)
}

bar() // 2
setTimeout(bar, 100) // 2

// `bar` 将 `foo` 的 `this` 硬绑定到 `obj`
// 所以它不可以被覆盖
bar.call(window) // 2
```

创建函数`bar()`在它内部手动调用`foo.call(obj)`，由此强制`this`绑定到`obj`并调用`foo`。

用 _硬绑定_ 将一个函数包装起来的最典型方法，是将所有传入的参数和传出的返回值创建一个通道：

```js
function foo(something) {
  console.log(this.a, something)
  return this.a + something
}

var obj = {
  a: 2
}

var bar = function() {
  return foo.apply(obj, arguments)
}

var b = bar(3) // 2 3
console.log(b) // 5
```

另一种表达这种模式的方法是创建一个可复用的`bind`帮助函数：

```js
function foo(something) {
  console.log(this.a, something)
  return this.a + something
}

// 简单实现的 `bind` 帮助函数
function bind(fn, obj) {
  return function() {
    return fn.apply(obj, arguments)
  }
}

var obj = {
  a: 2
}

var bar = bind(foo, obj)

var b = bar(3) // 2 3
console.log(b) // 5
```

由于 _硬绑定_ 是一个如此常用的模式，因此 ES5 的内奸工具提供：`Function.prototype.bind`来硬绑定`this`。

`bind(...)`返回一个硬编码的新函数，它使用你指定的`this`环境来调用原本的函数。

> 注：在 ES6 中，`bind(...)`生成的硬绑定函数有一个名为`.name`的属性，它源自原始的 _目标函数（target function）_。举例来说：`bar = foo.bind(..)` 应该会有一个 `bar.name` 属性，它的值为 `"bound foo"`，这个值应当会显示在调用栈轨迹的函数调用名称中。

#### 3.2 API 调用“环境”

许多库中的函数和许多在 js 语言以及宿主环境中的内建函数，都提供一个可选参数，通常称为“环境（context）”，这种设计作为一种替代方案来确保回调函数使用特定的`this`而不必非得使用`bind(...)`。

```js
function foo(el) {
  console.log(el, this.id)
}

var obj = {
  id: 'awesome'
}

// 使用 `obj` 作为 `this` 来调用 `foo(..)`
;[1, 2, 3].forEach(foo, obj) // 1 awesome  2 awesome  3 awesome
```

**forEach 的参数如下：**

`callback`：为数组中每个元素执行的函数，该函数接受三个参数：

1.  `currentValue`（当前值）：数组中正在处理的当前元素
2.  `index`（索引）：数组中正在处理的当前元素的索引
3.  `array`：`forEach()`方法正在操作的数组

`thisArg`（可选）：可选参数，当前执行回调函数时用做`this`的值。

### 4.`new`绑定（`new` Binding）

在传统的面向对象类语言中，“构造器”是附着在类上的一种特殊方法，当使用`new`操作符来初始化一个类时，这个类的构造器（constructor）就会被调用。

在 js 中，构造器 **仅仅是一个函数**，它们偶然地与前置的 `new` 操作符一起调用。它们不依附于类，它们也不初始化一个类。它们甚至不是一种特殊的函数类型。它们本质上只是一般的函数，在被使用 `new` 来调用时改变了行为。

<span style="color: red">**可以说任何函数，都可以在前面加上`new`来被调用，这使函数调用称为一个 构造其调用（constructor call）**。这是一个重要而微妙的区别：实际上不存在“构造器函数”这样的东西，而只有函数的构造器调用。</span>

<span style="color: red">
当在函数前面加入`new`调用时，也就是构造器调用时，下面这些事情会自动完成：

1.  一个全新的对象会凭空创建（就是被构建）
2.  这个新构建的对象会被接入原型链（`[[Prototype]]`-linked）
3.  这个新构建的对象被设置为函数调用的`this`绑定
4.  除非函数返回一个它自己的其他 **对象**，否则这被 `new` 调用的函数将 _自动_ 返回这个新构建的对象

也就是说：

以 new 操作符调用构造函数会经历以下四个步骤

1.  创建一个新对象
2.  将构造函数的作用域赋给新对象（因此 this 就指向这个新对象）
3.  执行构造函数中的代码（为这个新对象添加属性）
4.  返回新对象
    </span>

```js
function foo(a) {
  this.a = a
}
var bar = new foo(2)
console.log(bar.a) // 2
```

## 一切皆有顺序

以上就是函数调用中的四种`this`绑定，通过找到调用点判断以上哪一个规则适用于它从而判断`this`绑定。但是调用点可能会适用多种规则，此时需要用规则的优先顺序判断。

*默认绑定*是四种规则中优先权 **最低** 的。

_隐含绑定_ 和 _明确绑定_ 的优先顺序可以用以下代码测试：

```js
function foo() {
  console.log(this.a)
}
var obj1 = {
  a: 2,
  foo: foo
}
var obj2 = {
  a: 3,
  foo: foo
}

obj1.foo() // 2
obj2.foo() // 3

obj1.foo.call(obj2) // 3
obj2.foo.call(obj1) // 2
```

从以上运行结果可以看出 _明确绑定_ 的优先级高于 _隐含绑定_，这就是说要优先判断是否是 _明确绑定_。

现在，我们只需要搞清楚 `new` 绑定 的优先级位于何处。

```js
function foo(something) {
  this.a = something
}

var obj1 = {
  foo: foo
}

var obj2 = {}

obj1.foo(2)
console.log(obj1.a) // 2

obj1.foo.call(obj2, 3)
console.log(obj2.a) // 3

var bar = new obj1.foo(4)
console.log(obj1.a) // 2
console.log(bar.a) // 4
```

从以上可以看出 _new 绑定_ 的优先级高于 _隐含绑定_。

**注意：**   `new` 和 `call/apply` 不能同时使用，所以 `new foo.call(obj1)` 是不允许的，所以不能直接对比 _new 绑定_ 和 _隐含绑定_，但是依然可以使用 _硬绑定_ 来测试两个规则的优先级。

在我们进入代码中探索之前，回想一下 硬绑定 物理上是如何工作的，也就是 `Function.prototype.bind(..)` 创建了一个新的包装函数，这个函数被硬编码为忽略它自己的 `this` 绑定（不管它是什么），转而手动使用我们提供的。

因此，这似乎看起来很明显，硬绑定（明确绑定的一种）的优先级要比 `new` 绑定 高，而且不能被 `new` 覆盖。

检验一下：

```js
function foo(something) {
  this.a = something
}

var obj1 = {}

var bar = foo.bind(obj1)
bar(2)
console.log(obj1.a) // 2

var baz = new bar(3)
console.log(obj1.a) // 2
console.log(baz.a) // 3
```

从以上运行结果看，_new 绑定_ 没有被 _硬绑定_ 覆盖。因为 `new` 创建了一个名为 `baz` 的新对象。

ES5 内建`Function.prototype.bind(...)`的`polyfill`如下：

```js
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // 可能的与 ECMAScript 5 内部的 IsCallable 函数最接近的东西，
      throw new TypeError(
        'Function.prototype.bind - what ' +
          'is trying to be bound is not callable'
      )
    }

    var aArgs = Array.prototype.slice.call(arguments, 1),
      fToBind = this,
      fNOP = function() {},
      fBound = function() {
        return fToBind.apply(
          this instanceof fNOP && oThis ? this : oThis,
          aArgs.concat(Array.prototype.slice.call(arguments))
        )
      }

    fNOP.prototype = this.prototype
    fBound.prototype = new fNOP()

    return fBound
  }
}
```

**注意：** 就将与`new`一起使用的硬绑定函数而言，上面的`bind(...)` polyfill 与 ES5 中内建的`bind(...)`是不同的。因为 polyfill 不能像内建工具那样，没有`.prototype`就能创建函数，这里使用一些微妙而间接的方法近似模拟相同的行为。如果打算将 _硬绑定_ 与 *new 绑定*一起使用而且依赖这个 polyfill，应该多加小心。

允许`new`进行覆盖的部分是这里：

```js
this instanceof fNOP && oThis ? this : oThis

// ... 和：

fNOP.prototype = this.prototype
fBound.prototype = new fNOP()
```

这里判断硬绑定函数是否通过`new`被调用的（导致一个新构建的对象作为它的`this`），如果是，它就用那个新构建的`this`而非先前那个`this`指定的 _硬绑定_

<span style="text-decoration: underline">`new` 覆盖 _硬绑定_ 这种行为 可以创建一个 **预先设置一部分或所有的参数的函数（这个函数可以与`new`一起使用来构建）**，该函数实质上忽略了 `this` 的 _硬绑定_。`bind(...)`的一个能力是，任何在第一个`this`绑定参数之后被传入的参数，默认地作为当前函数的标准参数（技术上称为“局部应用（partial application）”，是一种“柯里化（currying）”）。使用`bind`函数并 **不执行。** </span>

例如：

```js
function foo(p1, p2) {
  this.val = p1 + p2
}

// 在这里使用 `null` 是因为在这种场景下我们不关心 `this` 的硬绑定
// 而且反正它将会被 `new` 调用覆盖掉！
var bar = foo.bind(null, 'p1')

var baz = new bar('p2')

console.log(baz) // foo {val: "p1p2"}
baz.val // p1p2
```

以上代码中 `var bar = foo.bind(null, 'p1')` 给函数 `foo` 传入部分参数（此时参数`p1`被赋值为字符串'p1'，`p2`为`undefined`），这里`this` 的 _硬绑定_ 被忽略；`var baz = new bar('p2')`继续赋值`p2`，设置为字符串p2。

### 判定`this`

现在可以按照优先循序来总结从函数调用的调用点判定`this`的规则。按照这个顺序，在规则适用的第一个地方停下。

1. 函数是否通过 `new` 被调用（**new 绑定**） ，如果是，`this`是新构建的对象。`var bar = new foo()`
2. 函数是否通过 `call` 或 `apply` 被调用（**明确绑定**），甚至是隐藏在`bind` *硬绑定* 之中。如果是，`this`就是那个被明确指定的对象。`var bar = foo.call(obj2)`
3. 函数是否通过环境对象（也称为拥有者或容器对象）被调用（**隐含绑定**）。如果是，`this`就是那个环境对象。`var bar = obj1.foo()`
4. 如果以上情况都不符合，则使用默认的`this`（**默认绑定**）。如果在`strict mode`下，就是`undefined`，否则就是`global`对象。`var bar = foo()`

即优先循序如下：

new 绑定 >  明确绑定 > 隐含绑定 > 默认绑定

## 绑定的特例

### 被忽略的`this`

如果传递`null`或`undefined`作为`call、apply`或`bind`的`this`绑定参数，这些值就会被忽略，此时 *默认绑定* 适用于该调用。

这种操作的作用：

一种很常见的做法是，使用`apply(...)`来将一个数组散开，从而作为函数调用的参数。相似地，`bind(...)`可以柯里化参数（预设值）（即给函数传递部分参数）。

```js
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// 将数组散开作为参数
foo.apply( null, [2, 3] ); // a:2, b:3

// 用 `bind(..)` 进行柯里化
var bar = foo.bind( null, 2 );
bar( 3 ); // a:2, b:3
```

将数组散开在ES6中有扩展操作符`...`，但是 **柯里化**在ES6中并没有语法上的替代品。

因为不关心`this`绑定而使用`null`会又一些潜在的问题，如果这样处理一些调用函数，可能会让调用函数的`this`指向 `global` 对象（在浏览器中是 `window`）。

#### 更安全的`this`

更加安全的做法是为`this`传递一个特殊创建好的对象，这个对象保证不产生副作用，这个对象其实就是一个完全为空，没有委托的对象。

创建 **完全为空的对象** 的最简单方法是 `Object.create(null)`。`Object.create(null)` 与 `{}`很相似，但是没有指向 `Object.prototype` 的委托，所有它比 `{}` 空的更彻底。

```JS
function foo(a,b) {
	console.log( "a:" + a + ", b:" + b );
}

// 我们的 DMZ 空对象
var ø = Object.create( null );

// 将数组散开作为参数
foo.apply( ø, [2, 3] ); // a:2, b:3

// 用 `bind(..)` 进行 currying
var bar = foo.bind( ø, 2 );
bar( 3 ); // a:2, b:3
```

###  词法 `this`

ES6 引入了一种不适用于这些规则特殊的函数：箭头函数`（arrow-function）`。

与四种标准的`this`规则不同的是，箭头函数从 _封闭它的（函数或全局）作用域开始绑定`this`_。

```js
function foo() {
  // 返回一个箭头函数
	return (a) => {
    // 这里的 `this` 是词法上从 `foo()` 采用的
		console.log( this.a );
	};
}

var obj1 = {
	a: 2
};

var obj2 = {
	a: 3
};

var bar = foo.call( obj1 );
bar.call( obj2 ); // 2, 不是3!
```

在`foo()`中创建的箭头函数在 _词法上捕获`foo()`被调用时的`this`_。因为`foo()`被`this`绑定到`obj1`，`bar`也会被`this`绑定到`obj1`，一个箭头函数的词法绑定不能被覆盖（即使是`new`也不能覆盖）。

# 第三章：对象

## 语法

对象的两种形式：

* 声明（字面）形式

```js
var myObj = {
	key: value
	// ...
};
```

* 构造形式

```js
var myObj = new Object();
myObj.key = value;
```

JS 社区的绝大部分人都 **强烈推荐** 尽可能地使用字面形式的值，而非使用构造的对象形式。

## 类型

JS六种主要类型：

* `string`
* `number`
* `boolean`
* `null`
* `undefined`
* `object`

注意 简单基本类型 （`string`、`number`、`boolean`、`null`、和 `undefined`）自身 不是 `object`。`null` 有时会被当成一个对象类型，但是这种误解源自于一个语言中的 Bug，它使得 `typeof null` 错误地（而且令人困惑地）返回字符串 `"object"`。实际上，null 是它自己的基本类型。

<span style="color: red">**一个常见的错误论断是“JavaScript中的一切都是对象”。这明显是不对的。**</span>

JS中存在几种特殊的对象子类型，我们称之为 _复杂基本类型_，包括`function`和`Array`。

### 内建对象

* `String`
* `Number`
* `Boolean`
* `Object`
* `Function`
* `Array`
* `Date`
* `RegExp`
* `Error`

这些内建对象的每一个都是可以通过`new`操作符调用，其结果是一个新的 _构建_ 的相应子类型。

考察 `object` 的子类型可以使用：

```js
var arr = [1,2]
Object.prototype.toString.call( arr );	// [object Array]
```

## 内容

```js
var myObject = {
	a: 2
};1
myObject.a;		// 2
myObject["a"];	// 2
```

访问 `myObject` 在 _位置_ `a` 的值可以使用 `.` 或者 `[]` 操作符。`.a` 语法通常成为 “属性” 访问，`['a']` 语法通常称为 “键（key）”访问。两种语法的主要区别在于，`.` 操作符后面需要一个 `标识符（Identifier）` 兼容的属性名，而 `[".."]` 语法基本可以接收任何兼容 UTF-8/unicode 的字符串作为属性名。

由 `['']` 语法使用字符串的 **值** 来指定位置，这意味着程序可以动态地组件字符串值。

**注意：** 在对象中，属性名 总是 字符串。如果你使用 `string` 以外的（基本）类型值，它会首先被转换为字符串。这甚至包括在数组中常用于索引的数字，所以要小心不要将对象和数组使用的数字搞混了。

### 计算型属性名

使用计算型属性名需要使用 `[]` 语法。

```js
var prefix = "foo";

var myObject = {
	[prefix + "bar"]: "hello",
	[prefix + "baz"]: "world"
};

myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

### 属性（Property） vs. 方法（Method）

如果一个对象中有一个函数，我们常说这个函数属于这个对象，是这个对象的一个方法，但是其实从技术上讲，**函数是不会属于对象的**，所以，说一个偶然在对象的引用上被访问的函数就自动成为一个方法是不合适的。

每次访问一个对象的属性都是一个 **属性访问**，无论得到的是什么类型的值。

**注意：** ES6 加入了 `super` 引用，它通常是和 `class` 一起使用的。`super` 的行为方式（静态绑定，而非像 `this` 一样延迟绑定），给了这种说法更多的权重：一个被 `super` 绑定到某处的函数比起“函数”更像一个“方法”。但是同样地，这仅仅是微妙的语义上的（和机制上的）细微区别。

### 数组

数组也使用 `[]` 访问形式，但数组采用 _数字索引_。

数组也是对象，索引虽然没个索引都是正整数，但还是可以添加属性：

```js
var myArray = [ "foo", 42, "bar" ];
myArray.baz = "baz";
myArray.length;	// 3
myArray.baz;	// "baz"
```

**注意：**如果试图在数组上添加属性，但是这个属性看起来像一个数字，那么最终它会成为一个数字索引：

```js
var myArray = [ "foo", 42, "bar" ];
myArray["3"] = "baz";
myArray.length;	// 4 myArray=["foo", 42, "bar", "baz"]
myArray[3];		// "baz"
```

### 复制对象

关于对象的 _拷贝_，首先要理解是一个 _深拷贝（deep）_ 还是一个 _浅拷贝（shallow）_