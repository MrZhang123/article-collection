## 第一章：什么是作用域？

### 编译器理论

在传统的编译型语言处理中，一块儿源代码，你的程序，在它被执行 _之前_ 通常将会经历三个步骤，大致被称为“编译”：

1.  **分词/词法分析：** 将一连串字符打断成（对于语言来说）有意义的片段，称为 token（记号）。举例来说，考虑这段程序：`var a = 2;`。这段程序很可能会被打断成如下 token：`var`，`a`，`=`，`2`，和 `;`。空格也许会被保留为一个 token，这要看它是否是有意义的。
    **注意：** 分词和词法分析之间的区别是微妙和学术上的，其中心在于这些 token 是否以 _无状态_ 或 _有状态_ 的方式被识别。简而言之，如果分词器去调用有状态的解析规则来弄清`a`是否应当被考虑为一个不同的 token，还是只是其他 token 的一部分，那么这就是 **词法分析**。

2.  **解析：** 将一个 token 的流（数组）转换为一个嵌套元素的树，它综合地表示了程序的语法结构。这棵树称为“抽象语法树”（AST —— <b>A</b>bstract <b>S</b>yntax <b>T</b>ree）。
    `var a = 2;` 的树也许开始于称为 `VariableDeclaration`（变量声明）顶层节点，带有一个称为 `Identifier`（标识符）的子节点（它的值为 `a`），和另一个称为 `AssignmentExpression`（赋值表达式）的子节点，而这个子节点本身带有一个称为 `NumericLiteral`（数字字面量）的子节点（它的值为`2`）。

3.  **代码生成：** 这个处理将抽象语法树转换为可执行的代码。这一部分将根据语言，它的目标平台等因素有很大的不同。
    所以，与其深陷细节，我们不如笼统地说，有一种方法将我们上面描述的 `var a = 2;` 的抽象语法树转换为机器指令，来实际上 _创建_ 一个称为 `a` 的变量（包括分配内存等等），然后在 `a` 中存入一个值。
    **注意：** 引擎如何管理系统资源的细节远比我们要挖掘的东西深刻，所以我们将理所当然地认为引擎有能力按其需要创建和存储变量。

和大多数其他语言的编译器一样，JavaScript 引擎要比这区区三步复杂太多了。例如，在解析和代码生成的处理中，一定会存在优化执行效率的步骤，包括压缩冗余元素，等等。

### LHS & RHS

| 查询方式 | 定义                                         | 详情                                               | 概念上理解 |
| -------- | -------------------------------------------- | -------------------------------------------------- | ---------- |
| LHS      | 变量出现在赋值操作的**左**手边时会进行该查询 | LHS 查询是试着找到变量容器本身，以便它可以**赋值** | 赋值的目标 |
| RHS      | 变量出现在赋值操作的**右**手边时会进行该查询 | 难以察觉，因为它简单地**查询某个变量**的值         | 赋值的源   |

从这种意义上说，RHS 的含义实质上不是 真正的 “一个赋值的右手边”，更准确地说，它只是意味着“不是左手边”。
“RHS”意味着“取得他/她的源（值）”，暗示着 RHS 的意思是“去取……的值”

**注意：** LHS 和 RHS 意味着“赋值的左/右手边”未必像字面上那样意味着“ = 赋值操作符的左/右边”。赋值有几种其他的发生形式，所以最好在概念上将它考虑为：“赋值的目标（LHS）”和“赋值的源（RHS）”

## 第三章：函数与块儿作用域

立即调用表达式

```js
(fuction aa ( ) { })( )
(function aa ( ) { } ( ) )
```

IIFE 还有另一种变种，它将事情的顺序倒了过来，要被执行的函数在调用和传递给它的参数 之后 给出。这种模式被用于 UMD（Universal Module Definition —— 统一模块定义）项目。一些人发现它更干净和易懂一些，虽然有点儿繁冗。

```js
var a = 2

;(function IIFE(def) {
  def(window)
})(function def(global) {
  var a = 3
  console.log(a) // 3
  console.log(global.a) // 2
})
```

def 函数表达式在这个代码段的后半部分被定义，然后作为一个参数（也叫 def）被传递给在代码段前半部分定义的 IIFE 函数。最后，参数 def（函数）被调用，并将 window 作为 global 参数传入。

## 第四章：提升

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

### 函数优先

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

声明本身会被提升，但不是赋值，即便是函数表达式的赋值，也**不会**被提升。

---

------- 貌似有一部分在 MBP 上没有上传 --------

---

## 第五章：作用域闭包

> 定义：闭包就是函数能够记住并访问它的词法作用域，即使当这个函数在它的词法作用域之外执行时

**函数在它被编写时的词法作用域之外被调用。**`闭包`使这个函数可以继续访问它在编写时被定义的词法作用域。

### 无处不在的闭包

```js
function wait(message) {
  setTimeout(function timer() {
    console.log(message)
  }, 1000)
}
wait('Hello, closure!')
```

我们拿来一个内部函数（名为 `timer`）将它传递给 setTimeout(..)。但是`timer`拥有覆盖`wait(..)`的作用域的闭包，实际上保持并使用着对变量`message`的引用。

在我们执行`wait(..)`一千毫秒之后，要不是内部函数`timer`依然拥有覆盖着`wait()`内部作用域的闭包，它早就会消失了。

在*引擎*的内脏深处，内建的工具`setTimeout(..)`拥有一些参数的引用，可能称为`fn`或者`func`或者其他诸如此类的东西。引擎去调用这个函数，它调用我们的内部`timer` 函数，而词法作用域依然完好无损。

实质上 _无论何时何地_ 只要你将函数作为头等的值看待并将它们传来传去的话，你就可能看到这些函数行使闭包。计时器、事件处理器、Ajax 请求、跨窗口消息、web worker、或者任何其他的异步（或同步！）任务，当你传入一个 _回调函数_，你就在它周围悬挂了一些闭包。

虽然 IIFE 本身 不是一个闭包的例子，但是它绝对创建了作用域，而且它是我们用来创建可以被闭包的作用域的最常见工具之一。所以 IIFE 确实与闭包有强烈的关联，即便它们本身不行使闭包。

### 循环+闭包

```js
for (var i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i * 1000)
}
```

这段代码期待分别打印数字 1，2，...5，一次一个，一秒一个，但实际上会打印五次“6”，一秒一个。

事实上超时的回调函数都将在循环完成之后立即运行，就计时器而言，即使在每次迭代中他是`setTimeout(..., 0)`，所有的这些回调函数也都仍然是严格的在循环之后执行的，因此每次都打印`6`
这段代码本意是想让在迭代期间，每次迭代都获得一份`i`的拷贝。但是，虽然所有这五个函数在每次循环迭代中分离地定义，由于作用域的工作方式，他们**`都闭包在同一个共享的全局作用域上，在这个全局作用域上，只有一个i`**

所以，我们需要为循环的每次迭代都准备一个新的被闭包的作用域，IIF（立即执行函数）通过声明并立即执行一个函数来创建作用域。修改如下：

```js
for (var i = 1; i <= 5; i++) {
  ;(function() {
    setTimeout(function timer() {
      console.log(i)
    }, i * 1000)
  })()
}
```

运行后会发现还是达不到预期的效果

很显然 IIFE 为每次迭代创建了词法作用域。每个超时回调函数确实闭包在每次迭代时分别被每个 IIFE 创建的作用域中。但是拥有一个被闭包的 **空的作用域** 是不够的，目前的 IIFE 只是一个空的什么都不做的作用域，它内部需要一些东西才能变得对我们有用，修改代码如下：

```js
for (var i = 1; i <= 5; i++) {
  ;(function() {
    var j = i
    setTimeout(function timer() {
      console.log(j)
    }, j * 1000)
  })()
}

// or

for (var i = 1; i <= 5; i++) {
  ;(function(j) {
    setTimeout(function timer() {
      console.log(j)
    }, j * 1000)
  })(i)
}
```

在每次迭代内部使用的 IIFE 为每次迭代创建了新的作用域，这让每次迭代时闭包一个新的作用域，这些作用域中的每一个都拥有一个持有正确的迭代值的变量给我们访问。

仔细观察我们前一个解决方案的分析。我们使用了一个 IIFE 来在每一次迭代中创建新的作用域。换句话说，我们实际上每次迭代都 需要 一个 块儿作用域。**`let` 声明劫持一个块儿并且就在这个块儿中声明一个变量。**

这实质上将块儿变成了一个我们可以闭包的作用域。所以修改代码如下：

```js
for (var i = 1; i <= 5; i++) {
  let j = i // 呀，给闭包的块儿作用域！
  setTimeout(function timer() {
    console.log(j)
  }, j * 1000)
}
```

在用于 for 循环头部的 let 声明被定义了一种特殊行为。这种行为表明这个变量将不是只为循环声明一次，而是为每次迭代声明一次。并且，它将在每次后续的迭代中被上一次迭代末尾的值初始化。所以优化代码如下：

```js
for (let i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i * 1000)
}
```

### 模块

```js
function CoolModule() {
  var something = 'cool'
  var another = [1, 2, 3]

  function doSomething() {
    console.log(something)
  }

  function doAnother() {
    console.log(another.join(' ! '))
  }

  return {
    doSomething: doSomething,
    doAnother: doAnother
  }
}

var foo = CoolModule()

foo.doSomething() // cool
foo.doAnother() // 1 ! 2 ! 3
```

`CoolModule()`只是一个函数，但它 必须被调用 才能成为一个被创建的模块实例。**没有外部函数的执行，内部作用域的创建和闭包都不会发生。**
`CoolModule()`函数返回一个对象，通过对象字面量语法 { key: value, ... } 标记。这个我们返回的对象拥有指向我们内部函数的引用，但是 没有 指向我们内部数据变量的引用。我们可以将它们保持为隐藏和私有的。可以很恰当地认为这个返回值对象实质上是一个我们 **模块的公有 API。**

更简单地说，行使模块模式有两个“必要条件”：

1.  必须有一个外部的外围函数，而且它必须至少被调用一次（每次创建一个新的模块实例）。
2.  外围的函数必须至少返回一个内部函数，这样这个内部函数才拥有私有作用域的闭包，并且可以访问和/或修改这个私有状态。

一个仅带有一个函数属性的对象不是 真正 的模块。从可观察的角度来说，一个从函数调用中返回的对象，仅带有数据属性而没有闭包的函数，也不是 真正 的模块。

```js
var foo = (function CoolModule(id) {
	function change() {
		// 修改公有 API
		publicAPI.identify = identify2;
	}

	function identify1() {
		console.log( id );
	}

	function identify2() {
		console.log( id.toUpperCase() );
	}

	var publicAPI = {
		change: change,
		identify: identify1
	};

	return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```
通过在模块实例内部持有一个指向公有API对象的内部引用，可以从内部修改这个模块，包括添加和删除方法，属性，和改变它们的值。

## 总结

闭包就是当一个函数即使是在它的词法作用域之外被调用可以记住并访问它的词法作用域。

模块要求两个关键性质：

1. 一个被调用的外部包装函数，来创建外围作用域。
2. 这个包装函数的返回值必须包含至少一个内部函数的引用，这个函数才拥有包装函数内部作用域的闭包。

## 动态作用域

JS拥有词法作用域而非动态作用域，验证如下：

```js
function foo() {
	console.log( a ); // 2
}

function bar() {
	var a = 3;
	foo();
}

var a = 2;

bar();
```

在 `foo()` 的词法作用域中指向 a 的 RHS 引用将被解析为全局变量 a，它将导致输出结果为值 2。

相比之下，动态作用域本身不关心函数和作用域是在哪里和如何被声明的，而是关心 它们是从何处被调用的。换句话说，它的作用域链条是基于调用栈的，而不是代码中作用域的嵌套。

所以，如果 JavaScript 拥有动态作用域，当 foo() 被执行时，理论上 下面的代码将得出 3 作为输出结果。

因为当`foo()`不能为`a`解析出一个变量引用时，它不会沿着嵌套的（词法）作用域链向上走，而是沿着调用栈向上走，以找到`foo()`是从何处被调用的。因为`foo()`是从`bar()`中被调用的，他就会在`bar()`的作用域中检查变量，并且在这里找到`var a = 3`

词法作用域与动态作用域的差别：

**词法作用域是编写时的，而动态作用域（和`this`）是运行时的。词法作用域关心的是函数在何处被声明，但是动态作用域关心的是函数从何处被调用。**