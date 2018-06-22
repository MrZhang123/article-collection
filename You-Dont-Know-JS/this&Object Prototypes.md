## 第一章: this 是什么

### 困惑

开发者用太过于字面的方式考虑`this`会产生以下误解：

#### 它自己

第一种常见的倾向认为`this`指向它自己，至少这是一种在语法上的合理推测，但是，观察以下代码

```js
function foo(num) {
	console.log( "foo: " + num );

	// 追踪 `foo` 被调用了多少次
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		foo( i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// `foo` 被调用了多少次？
console.log( foo.count ); // 0 ????
```

当执行`foo.count = 0`时，它确实向`foo`添加一个`count`属性，但是对于内部函数`this`指向根本 **不是指向函数本身，而是指向全局的**。

其实代码中的`this.count++`不小心创建了一个全局变量`count` ，而且它当前的值是`NaN`。

将`this`指向`foo`本身的代码如下：

```js
function foo(num) {
	console.log( "foo: " + num );

	// 追踪 `foo` 被调用了多少次
	// 注意：由于 `foo` 的被调用方式（见下方），`this` 现在确实是 `foo`
	this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
	if (i > 5) {
		// 使用 `call(..)`，我们可以保证 `this` 指向函数对象(`foo`)
		foo.call( foo, i );
	}
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// `foo` 被调用了多少次？
console.log( foo.count ); // 4
```

在循环调用中，使用`call`手动将`this`指向`foo`本身


#### 它的作用域

对 `this` 的含义第二常见的误解，是它不知怎的指向了函数的作用域。这是一个刁钻的问题，因为在某一种意义上它有正确的部分，而在另外一种意义上，它是严重的误导。

明确的说，`this`不会以任何方式指向函数的 `词法作用域`，作用域好像是一个将所有可用标识符作为属性的对象，这从内部来说是对的。但是 JavasScript 代码不能访问作用域“对象”。它是 **引擎** 的内部实现。

```js
function foo() {
	var a = 2;
	this.bar();
}

function bar() {
	console.log( this.a );
}

foo(); //undefined
```

以上代码希望通过`this`在`bar()`和`foo()`的词法作用域之间建立一座桥，使得`bar()`可以访问`foo()`内部的`a`，这是不可能做到的，不能使用 `this` 引用在词法作用域中查找东西。

### 什么是`this`?

`this`不是编写时绑定的，而是运行时绑定的，它依赖于函数调用的上下文条件。`this`绑定与函数声明的位置没有任何关系，而是与函数调用的方式有关系。

当一个函数被调用时，会建立一个成为执行环境的活动记录。这个记录包含函数从何处（调用栈——call-stack）被调用的，函数是`如何`被调用的，被传递了什么参数信息。这个记录的属性之一，就是函数执行期间被使用的`this`引用。

所以，`this`是一个完全根据`调用点`（函数是如何被调用的）而为每次函数调用建立的绑定。

## 第二章: `this`豁然开朗！

### 调用点（Call-site）

调用点：函数在代码中 **被调用** 的位置（**不是被声明的位置**）

调用栈：使我们到达当前执行位置而被调用的馊油方法的堆栈

```js
function baz() {
    // 调用栈是: `baz`
    // 我们的调用点是 global scope（全局作用域）

    console.log( "baz" );
    bar(); // <-- `bar` 的调用点
}

function bar() {
    // 调用栈是: `baz` -> `bar`
    // 我们的调用点位于 `baz`

    console.log( "bar" );
    foo(); // <-- `foo` 的 call-site
}

function foo() {
    // 调用栈是: `baz` -> `bar` -> `foo`
    // 我们的调用点位于 `bar`

    console.log( "foo" );
}

baz(); // <-- `baz` 的调用点
```

在分析代码来寻找（从调用栈中）真正的调用点时要小心，因为它是影响`this`绑定的唯一因素。

如果想调查`this	`绑定，可以使用开发者工具取得调用栈，之后从上到下找到第二个记录，那就是调用点

### 仅仅是规则

通过调用点寻找`this`指向的四条规则

#### 1.默认绑定（Default Binding）

函数调用最常见的一种情况：独立函数调用，可以认为这种`this`规则是在没有其他规则适用时的 **默认规则**。

```js
function foo() {
	console.log( this.a );
}

var a = 2;

foo(); // 2
```

注意：

1. 全局作用域中声明 `var a = 2`， 是全局对象的同名属性的同义词。
2. 当`foo()`被调用时，`this.a`解析为全局变量`a`，因为在独立函数调用的情况下，`this`实施了 *默认绑定*，所以`this`指向全局对象。

为什么这里的`this`进行来默认绑定？因为在这段代码中`foo()`是被直接调用的，没有其他的我们将要展示的规则适用，所以这里 *默认绑定* 适用。

如果 `strict mode`在这里生效，那么对于默认绑定来说全局对象是不合法的，所以`this`被设置为`undefined`。

```js
function foo() {
	"use strict";

	console.log( this.a );
}

var a = 2;

foo(); // TypeError: `this` is `undefined`
```

#### 2.隐含绑定（Implicit Binding）

另一种要考虑的规则是：调用点是否有一个环境对象（context object），也成为拥有者或容器对象。

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

obj.foo(); // 2
```

首先有一点需要注意，`foo()` 被声明然后作为引用属性添加到`obj`上，无论`foo()`是否一开始就在`obj`上被声明，还是后来作为引用属性添加，这个函数都不被`obj`所真正拥有。

然而，调用点 *使用`obj`*环境来 **引用** 函数，所以可以说 **`obj`对象在函数被调用的时间点上“拥有”或“包含”这个 函数引用**。

因为`obj`是`foo()`调用的`this`，所以`this.a`就是`obj.a`的同义词。

<span style="text-decoration: underline">只有对象属性引用链的 **最后一层** 是影响调用点的，</span>例如：

```js
function foo() {
	console.log( this.a );
}

var obj2 = {
	a: 42,
	foo: foo
};

var obj1 = {
	a: 2,
	obj2: obj2
};

obj1.obj2.foo(); // 42
```


##### 隐含丢失（Implicitly Lost）

`this` 绑定最常让人沮丧的事情之一，<span style="text-decoration: underline">**就是当一个 *隐含绑定* 丢失了它的绑定，这通常意味着它会退回到默认绑定**</span>，根据 `strict mode` 的状态，其结果不是全局对象就是 `undefined`。

考虑这段代码：

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2,
	foo: foo
};

var bar = obj.foo; // 函数引用！

var a = "oops, global"; // `a` 也是一个全局对象的属性

bar(); // "oops, global"
```

尽管`bar`似乎是`obj.foo`的引用，但实际上它只是另一个`foo`本身的引用而已。另外，起作用的调用点是`bar()`，一个直白，毫无修饰的调用，所以 *默认绑定* 在这里适用。

这种情况发生的更加微妙，更加常见的是考了传递一个回调函数时：

```js
function fo() {
	console.log(this.a);
}
function doFoo(fn) {
	// `fn`只不过是`foo`的另一个引用
	fn()
}
var obj = {
	a: 2,
	foo: foo
}

var a = 'opps, global'; // a 也是一个全局对象的属性
deFoo(obj.foo) // "oops, global"
```

参数传递仅仅是一种隐含的赋值，而且因为我们在传递一个函数，它是隐含的引用赋值，所以最终结果和我们签一个代码段一致。

#### 3.明确绑定（Explicit Binding）

可以使用`call`和`apply`进行明确的`this`绑定。它们接收的第一个参数都是一个用于`this`的对象，之后使用这个指定的`this`来调用函数。

```js
function foo() {
	console.log(this.a)
}
var obj = {
	a:2
}
foo.call(obj) // 2
```

通过`foo.call()`使用 *明确绑定* 来调用`foo`,允许我们强制函数的`this`指向`obj`。

如果传递一个简单基本类型（`string`，`boolean`，`number`）作为`this`绑定，那么这个基本类型值会被包装在它的对象类型中（分别是`new String(...)`，`new Boolean(...)`，`new Number(...)`）。这通常称为“封箱”。

>注：就`this`绑定的角度而言，`call`和`apply`是完全一样的，它们只是在处理参数的方式上不同。

##### 3.1 硬绑定（Hard Binding）

有一个明确绑定的变种可以实现原本的`this`不丢失。

```js
function foo() {
	console.log( this.a );
}

var obj = {
	a: 2
};

var bar = function() {
	foo.call( obj );
};

bar(); // 2
setTimeout( bar, 100 ); // 2

// `bar` 将 `foo` 的 `this` 硬绑定到 `obj`
// 所以它不可以被覆盖
bar.call( window ); // 2
```

创建函数`bar()`在它内部手动调用`foo.call(obj)`，由此强制`this`绑定到`obj`并调用`foo`。

用 *硬绑定* 将一个函数包装起来的最典型方法，是将所有传入的参数和传出的返回值创建一个通道：

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

var obj = {
	a: 2
};

var bar = function() {
	return foo.apply( obj, arguments );
};

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

另一种表达这种模式的方法是创建一个可复用的`bind`帮助函数：

```js
function foo(something) {
	console.log( this.a, something );
	return this.a + something;
}

// 简单实现的 `bind` 帮助函数
function bind(fn, obj) {
	return function() {
		return fn.apply( obj, arguments );
	};
}

var obj = {
	a: 2
};

var bar = bind( foo, obj );

var b = bar( 3 ); // 2 3
console.log( b ); // 5
```

由于 *硬绑定* 是一个如此常用的模式，因此ES5的内奸工具提供：`Function.prototype.bind`来硬绑定`this`。

`bind(...)`返回一个硬编码的新函数，它使用你指定的`this`环境来调用原本的函数。

>注：在ES6中，`bind(...)`生成的硬绑定函数有一个名为`.name`的属性，它源自原始的 *目标函数（target function）*。举例来说：`bar = foo.bind(..)` 应该会有一个 `bar.name` 属性，它的值为 `"bound foo"`，这个值应当会显示在调用栈轨迹的函数调用名称中。

##### 3.2 API调用“环境”

许多库中的函数和许多在js语言以及宿主环境中的内建函数，都提供一个可选参数，通常称为“环境（context）”，这种设计作为一种替代方案来确保回调函数使用特定的`this`而不必非得使用`bind(...)`。

```js
function foo(el) {
	console.log( el, this.id );
}

var obj = {
	id: "awesome"
};

// 使用 `obj` 作为 `this` 来调用 `foo(..)`
[1, 2, 3].forEach( foo, obj ); // 1 awesome  2 awesome  3 awesome
```

**forEach的参数如下：** 

`callback`：为数组中每个元素执行的函数，该函数接受三个参数：

1. `currentValue`（当前值）：数组中正在处理的当前元素
2. `index`（索引）：数组中正在处理的当前元素的索引
3. `array`：`forEach()`方法正在操作的数组

`thisArg`（可选）：可选参数，当前执行回调函数时用做`this`的值。

#### 4.`new`绑定（`new` Binding）

在传统的面向对象类语言中，“构造器”是附着在类上的一种特殊方法，当使用`new`操作符来初始化一个类时，这个类的构造器（constructor）就会被调用。

在js中，构造器 **仅仅是一个函数**，它们偶然地与前置的 `new` 操作符一起调用。它们不依附于类，它们也不初始化一个类。它们甚至不是一种特殊的函数类型。它们本质上只是一般的函数，在被使用 `new` 来调用时改变了行为。

<span style="color: red">**可以说任何函数，都可以在前面加上`new`来被调用，这使函数调用称为一个 构造其调用（constructor call）**。这是一个重要而微妙的区别：实际上不存在“构造器函数”这样的东西，而只有函数的构造器调用。</span>

<span style="color: red">
当在函数前面加入`new`调用时，也就是构造器调用时，下面这些事情会自动完成：

1. 一个全新的对象会凭空创建（就是被构建）
2. 这个新构建的对象会被接入原型链（`[[Prototype]]`-linked）
3. 这个新构建的对象被设置为函数调用的`this`绑定
4. 除非函数返回一个它自己的其他 **对象**，否则这被 `new` 调用的函数将 *自动* 返回这个新构建的对象

也就是说：

以new操作符调用构造函数会经历以下四个步骤

1. 创建一个新对象
2. 将构造函数的作用域赋给新对象（因此this就指向这个新对象）
3. 执行构造函数中的代码（为这个新对象添加属性）
4. 返回新对象
</span>

```js
function foo(a) {
	this.a = a;
}
var bar = new foo( 2 );
console.log( bar.a ); // 2
```


### 一切皆有顺序

以上就是函数调用中的四种`this`绑定，通过找到调用点判断以上哪一个规则适用于它从而判断`this`绑定。但是调用点可能会适用多种规则，此时需要用规则的优先顺序判断。

*默认绑定*是四种规则中优先权 **最低** 的。

*隐含绑定* 和 *明确绑定* 的优先顺序可以用以下代码测试：

```js
function foo() {
	console.log(this.a);
}
var obj1 = {
	a: 2,
	foo: foo
}
var obj2 = {
	a: 3,
	foo: foo
}

obj1.foo();	// 2
obj2.foo();	// 3

obj1.foo.call(obj2);
```