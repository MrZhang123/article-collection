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

```JS
function anotherFunction() { /*..*/ }
var anotherObject = {
	c: true
};
var anotherArray = [];
var myObject = {
	a: 2,
	b: anotherObject,	// 引用，不是拷贝!
	c: anotherArray,	// 又一个引用!
	d: anotherFunction
};
anotherArray.push( anotherObject, myObject );
```

关于对象的 _拷贝_，首先要理解是一个 _深拷贝（deep）_ 还是一个 _浅拷贝（shallow）_。

一个 _浅拷贝（shallow copy）_ 会得到一个新的对象。它的 `a` 是值 `2` 的拷贝，但 `b`、`c` 和 `d` 属性 **仅仅是引用**，它们指向被拷贝对象中引用的相同位置。一个 _深拷贝（deep copy）_ 不仅复制 `myObject`，还会复制 `anotherObject`，`anotherArray`。

一个简单的 _深拷贝（deep copy）_ 方案是，JSON 安全的对象（也就是，可以被序列化为一个 JSON 字符串，之后还可以被重新解析为拥有相同的结构和值的对象）可以简单地这样 复制：

```js
var newObj = JSON.parse( JSON.stringify( someObj ) );
```

浅拷贝相当易懂，而且没有那么多问题，所以ES6定义了 `Object.assign(...)`。`Object.assign(...)` 接收 _目标对象_ 作为第一个参数，然后是一个或多个 _源_ 对象作为后续参数。它会在 _源_ 对象上迭代所有的 _可枚举属性（enumerable）_ ，_直接拥有的键（owned keys）_，并把它们拷贝到 _目标对象上（仅通过 `=` 赋值）_。

<span style="color: red">**注意：**</span>在 `Object.assign(...)` 发生的复制是单纯的 `=` 式赋值，所以在源对象属性的特殊性质（比如 `writable`） 在目标对象上 **都不会做保留**。

### 属性描述符（Property Descriptors）

对象的属性描述符包括：可写性（Writable），可配置性（Configurable），可枚举性（Enumerable）

可以使用`defineProperty(...)`修改属性描述符

可配置性（Configurable）默认是`true`，且修改是不可逆的，即如果设置`configurable:false`，则无法再设置回`true`，且此时`delete`会操作失败。

### 不可变性（Immutability）

ES5加入了让属性或对象设置不可变的功能支持。

<span style="color: red">但是需要注意一点：</span>所有这些方法创建的都是浅不可变性，即仅仅影响对象和它的直属属性。如果对象拥有对其他对象（数组，对象，函数等）的引用，那个对象的内容依旧可变。

#### 对象常量（Object Constant）

通过`writable:false`与`configurable:false`组合，可以创建一个对象常量（不能被改变，重新定义或删除）。

#### 防止扩展（Prevent Extensions）

如果想防止一个对象被添加新的属性，但是另一个方面保留其存在的对象属性，可以调用`Object.preventExtensions(...)`。

#### 封印（Seal）

`Object.seal(...)`创建一个“封印”对象，这意味着它实质上在当前的对象上调用`Object.preventExtensions(...)`，同时也将所有的既存属性标记为`configurable:false`（不可配置），所以此时既不能添加更多的属性，也不能重新配置或删除既存属性（但是可以修改它们的值）。

```js
const object1 = {
  property1: 42
};

Object.seal(object1);
object1.property1 = 33;
console.log(object1.property1);
// expected output: 33

delete object1.property1; // cannot delete when sealed
console.log(object1.property1);
// expected output: 33
```

#### 冻结（Freeze）

`Object.freeze()` 方法可以冻结一个对象，冻结指的是不能向这个对象添加新的属性，不能修改其已有属性的值，不能删除已有属性，以及不能修改该对象已有属性的可枚举性、可配置性、可写性。也就是说，这个对象永远是不可变的。该方法返回被冻结的对象。

### [[GET]]

```js
var myObject = {
	a: 2
};
myObject.a; // 2
```

以上代码访问`myObject`的一个属性`a`。

从语言规范上讲，上吗的代码实际上在`myObject`上执行了一个`[[GET]]`操作（像是`[[GET]]()`操作）。

对一个对象进行默认的内建`[[GET]]`操作，会首先检查对象，寻找一个拥有被请求的名称的属性，如果找到，就返回相应的值。如果`[[GET]]`操作通过热河方法都找不到被请求的属性值，则返回`undefined`。

### [[Put]]

给一个对象的属性赋值，将会在这个对象上调用 `[[Put]]` 来设置或创建这个属性。

调用`[[Put]]`时，它会根据几个因素表现不同的行为，包括（影响最大的）属性是否已经在对象中存在了。

如果属性存在，`[[Put]]` 算法将会大致检查：

1. 这个属性是访问器描述符吗？如果是，而且是 `setter`，就调用 `setter`。
2. 这个属性是 `writable` 为 `false` 数据描述符吗？如果是，在非 `strict mode` 下无声地失败，或者在 `strict mode` 下抛出 `TypeError`。
3. 否则，像平常一样设置既存属性的值。

### Getters 与 Setters

ES5 引入了一个方法来覆盖默认操作的一部分，但不是在对象级别而是针对每个属性，就是通过 getters 和 setters。

* Getter 是实际上调用一个隐藏函数来取得值的属性。
* Setter 是实际上调用一个隐藏函数来设置值的属性。

#### 存在性（Existence）

可以使用`in`或者`hasOwnProperty`检查一个对象是否拥有特定的属性，而不必取得那个值：

```js
var myObject = {
	a: 2
};

("a" in myObject);				// true
("b" in myObject);				// false

myObject.hasOwnProperty( "a" );	// true
myObject.hasOwnProperty( "b" );	// false
```

二者区别在于：

* `in`会检查属性是否存在于 **对象** 或 存在于 `[[Prototype]]` 链对象遍历的更高层中。
* `hasOwnProperty(..)`仅仅检查对象是否拥有属性，不会检查 `[[Prototype]]` 链。

##### 枚举（Enumeration）

当可枚举性（enumerability）设置成`false`时候，使用`for...in`循环无法检查到该属性。

区分属性是可枚举属性还是不可枚举属性有两种方法：

1.使用`for...in`循环，该属性是否能出现

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// 使 `a` 可枚举，如一般情况
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// 使 `b` 不可枚举
	{ enumerable: false, value: 3 }
);

myObject.b; // 3
("b" in myObject); // true
myObject.hasOwnProperty( "b" ); // true

// .......

for (var k in myObject) {
	console.log( k, myObject[k] );
}
// "a" 2
```

2.使用`propertyIsEnumerable(..)`检测

`propertyIsEnumerable(..)` 测试一个给定的属性名是否直 接存 在于对象上，并且是 `enumerable:true`。

`Object.keys(..)` 返回一个所有 **可枚举属性** 的数组，而 `Object.getOwnPropertyNames(..)` 返回一个 **所有属性** 的数组，不论能不能枚举。

```js
var myObject = { };

Object.defineProperty(
	myObject,
	"a",
	// 使 `a` 可枚举，如一般情况
	{ enumerable: true, value: 2 }
);

Object.defineProperty(
	myObject,
	"b",
	// 使 `b` 不可枚举
	{ enumerable: false, value: 3 }
);

myObject.propertyIsEnumerable( "a" ); // true
myObject.propertyIsEnumerable( "b" ); // false

Object.keys( myObject ); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

## 迭代（Iteration）

ES6加入有用的`for...of`循环语法，用来迭代数组（和对象，如果这个对象有定义的迭代器）。

`for...of`循环要求被迭代的东西提供一个迭代器对象（从语言规范中叫做`@@iterator`得默认内部函数那里得到），每次循环都调用一次这个迭代器对象的`next()`方法，迭代循环的内容就是这些连续的返回值。

<span style="color: red">注意：</span>`@@iterator` 本身不是迭代器对象， 而是一个返回迭代器对象的方法。

因为数组有内建的`@@iterator`，所以可以在`for...of`中自动迭代，但是 **普通对象没有内建的 `@@iterator`**。

但是可以为想要迭代的对象定义默认`@@iterator`， <span style="color: red">为对象添加`@@iterator`代码如下：</span>

```js
var myObject = {
	a: 'a',
	b: 'b'
};

Object.defineProperty( myObject, Symbol.iterator, {
	enumerable: false,
	writable: false,
	configurable: true,
	value: function() {
		var o = this;
		var idx = 0;
		var ks = Object.keys( o );
		return {
			next: function() {
				return {
					value: o[ks[idx++]],
					done: (idx > ks.length)
				};
			}
		};
	}
} );

// 手动迭代 `myObject`
var it = myObject[Symbol.iterator]();
it.next(); // { value:a, done:false }
it.next(); // { value:b, done:false }
it.next(); // { value:undefined, done:true }

// 用 `for..of` 迭代 `myObject`
for (var v of myObject) {
	console.log( v );
}
// a
// b
```