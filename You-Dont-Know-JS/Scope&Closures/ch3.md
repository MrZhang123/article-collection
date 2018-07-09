# 第四章：提升

在你的代码的任何部分被执行之前，所有的声明，变量和函数，都会首先被处理。

变量和函数声明被从它们在代码流中出现的位置“移动”到代码的顶端。这就产生了“提升”这个名字。

**注意：**只有声明本身被提升了，而任何赋值或者其他的执行逻辑都被留在 原处。

```js
foo()

function foo() {
  console.log(a) // undefined

  var a = 2
}
```

> 注：提升是以作用域为单位的

以上代码相当于

```js
function foo() {
  var a

  console.log(a) // undefined

  a = 2
}

foo()
```

**函数声明会被提升，但函数表达式不会**

```js
foo() // 不是 ReferenceError， 而是 TypeError!

var foo = function bar() {
  // ...
}
```

变量标识符 `foo` 被提升并被附着在这个程序的外围作用域（全局），所以 `foo()` 不会作为一个 `ReferenceError` 而失败。但 `foo` 还没有值（如果它不是函数表达式，而是一个函数声明，那么它就会有值）。所以，`foo()` 就是试图调用一个 `undefined` 值，这是一个 `TypeError` —— 非法操作。

```js
foo() // TypeError
bar() // ReferenceError

var foo = function bar() {
  // ...
}
```

这个代码段可以（使用提升）更准确地解释为：

```JS
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
 var bar = ...self...
 // ...
}
```

## 函数优先

函数声明和变量声明都会被提升。但一个微妙的细节（可以在拥有多个“重复的”声明的代码中出现）是，**函数会首先被提升，然后才是变量**。

```js
foo() // 1

var foo

function foo() {
  console.log(1)
}

foo = function() {
  console.log(2)
}
```

`1` 被打印了，而不是 `2` 这个代码段被 引擎 解释执行为：

```js
function foo() {
  console.log(1)
}

foo() // 1

foo = function() {
  console.log(2)
}
```

那个 `var foo` 是一个重复（因此被无视）的声明，即便它出现在 `function foo()...` 声明之前，因为函数声明是在普通变量之前被提升的。

虽然多个/重复的 var 声明实质上是被忽略的，但是后续的函数声明确实会覆盖前一个。

```js
foo() // 3

function foo() {
  console.log(1)
}

var foo = function() {
  console.log(2)
}

function foo() {
  console.log(3)
}
```

在同一个作用域内的重复定义经常会导致令人困惑的结果

**在普通的块儿内部出现的函数声明一般会被提升至外围的作用域**，而不是像这段代码暗示的那样有条件地被定义：

```js
foo() // "b"

var a = true
if (a) {
  function foo() {
    console.log('a')
  }
} else {
  function foo() {
    console.log('b')
  }
}
```

我们可能被诱导而将 var a = 2 看作是一个语句，但是 JavaScript 引擎 可不这么看。它将 var a 和 a = 2 看作两个分离的语句，第一个是编译期的任务，而第二个是执行时的任务。

这将导致在一个作用域内的所有声明，不论它们出现在何处，都会在代码本身被执行前 首先 被处理。你可以将它可视化为声明（变量与函数）被“移动”到它们各自的作用域顶部，这就是我们所说的“提升”。

声明本身会被提升，但不是赋值，即便是函数表达式的赋值，也 **不会** 被提升。