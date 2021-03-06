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