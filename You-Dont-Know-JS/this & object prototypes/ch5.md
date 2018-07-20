# 第五章: 原型（Prototype）

## `[[Prototype]]`

如果使用`in`操作符检测一个属性是否在一个对象上，`in`将会检测对象的整个链条。

### `Object.prototype`

每个 _普通_ 的`[[Prototype]]`顶端是`Object.prototype`。

### 设置与遮蔽

`foo`同时存在于`myObject`和它`[[Prototype]]`的高层，则直接存在于`myObject`上的`foo`会 **遮蔽** `[[Prototype]]`上的`foo`。

假设想要在`myObject`上添加`foo`属性，则存在以下情况：

* `foo`没有直接存在于`myObject`上，`[[Prototype]]`就会被遍历，如果在链条的任何地方都没有找到`foo`，则`foo`会直接被添加到`myObject`上
* `foo`没有直接存在于`myObject`上而是存在于`[[Prototype]]`链条高层的某处，则会出现以下三种情况：
  1. 如果普通的名为`foo`的数据访问属性在`[[Prototype]]`链条的高层某处被找到，且可读写属性并没有标记成只读`writable:false`，则`foo`被直接添加到`myObject`，遮蔽链条更高层的`foo`属性
  2. 如果`foo`在链条的高层被找到且被标记成只读`writable:true`，则创建寄存属性和在`myObject`上创建遮蔽属性是 **不被允许的**，如果代码运行在`strict mode`下，那么会抛出错误，否则这个设置属性值的操作会被忽略
  3. 如果`foo`在链条的高层被找到且是一个`setter`，那么`setter`总是会被调用，`foo`不会被添加到`myObject`上，而且`setter`也不会被定义

从以上场景可以看出，想要造成遮蔽，除第1种情况直接赋值（使用`=`）即可形成之外，第2，3种情况，都是不能实现的。想要对第2，3种情况形成遮蔽，需要使用`Object.defineProperty(..)`。

一般来说，遮蔽与它带来的好处相比太过复杂和微妙了，**所以你应当尽量避免它**。

## “类”

JavaScript没有类但是可以直接创建对象

### “类”函数

```js
function Foo() {
	// ...
}
Foo.prototype; // { }
```

`Foo.prototype;`经常被称为`Foo`的原型，但是这个是一个错误的称谓

假设有如下描绘：

```js
function Foo() {
	// ...
}

var a = new Foo();

Object.getPrototypeOf( a ) === Foo.prototype; // true

```

当通过调用 `new Foo()` 创建 `a` 时，会发生的事情之一是，`a` 得到一个内部 `[[Prototype]]` 链接，此链接链到 `Foo.prototype` 所指向的对象。

在面向对象语言中，可以制造一个类的多个 **拷贝**，就像从模具中冲压出某个东西。这是因为初始化（或者继承）类的处理意味着将“行为计划从这个类拷贝到物理对象中”，对每个新实例都会发生。但是在JS中没有这样的拷贝处理发生，所以不会创建多个实例，只会创建多个对象，它们的`[[Prototype]]`连接到一个共通的对象。但默认的，没有发生拷贝，所以这些对象彼此之间没有完全分离，而是 **链接在一起**。

`new Foo()`创建一个对象`a`,这个新对象`a`内部地被`[[Prototype]]`链接至`Foo.prototype`对象。

“继承”意味着 _拷贝_ 操作，而JS不拷贝对象属性。相反，JS在两个对象间建立链接，一个对象实质上可以将对象属性/函数访问 _委托_ 到另一个对象上。对于JS对象链接机制来说，委托是一个准确的多的术语。

### "构造器"（Constructors）

```js
function Foo() {
	// ...
}

Foo.prototype.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

`Foo.prototype`对象默认的得到一个共有的，称为`constructor`的不可枚举属性，这个属性回头指向这个对象关联函数`Foo`。另外，“构造器”调用`new Foo()`创建的对象`a`看起来也有一个称为`.constructor`的属性，也相似的指向“创建它的函数”。

#### 构造器还是调用？

在JS中，“构造器”是在前面 **用`new`关键字调用的任何函数**，函数不是构造器，当且仅当`new`被使用时，函数调用是一个“构造器调用”。

### 机制






































