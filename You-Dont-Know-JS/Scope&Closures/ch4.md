# 第五章：作用域闭包

> 定义：闭包就是函数能够记住并访问它的词法作用域，即使当这个函数在它的词法作用域之外执行时

**函数在它被编写时的词法作用域之外被调用。**`闭包`使这个函数可以继续访问它在编写时被定义的词法作用域。

```js
function foo() {
	var a = 2;
	function bar() {
		console.log( a );
	}
	return bar;
}
var baz = foo();
baz();
```

`foo()` 被执行之后，一般说来我们会期望 `foo()` 的整个内部作用域都将消失，因为我们知道 引擎 启用了 垃圾回收器 在内存不再被使用时来回收它们。因为很显然 `foo()` 的内容不再被使用了，所以看起来它们很自然地应该被认为是 消失了。

但是闭包的“魔法”不会让这发生。内部的作用域实际上 依然 “在使用”，因此将不会消失。谁在使用它？函数 `bar()` 本身。

有赖于它被声明的位置，`bar()` 拥有一个词法作用域闭包覆盖着 `foo()` 的内部作用域，闭包为了能使 `bar()` 在以后任意的时刻可以引用这个作用域而保持它的存在。

**`bar()` 依然拥有对那个作用域的引用，而这个引用称为闭包。**

以下事例也是闭包

```js
var fn;

function foo() {
	var a = 2;

	function baz() {
		console.log( a );
	}

	fn = baz; // 将`baz`赋值给一个全局变量
}

function bar() {
	fn(); // 看妈妈，我看到闭包了！
}

foo();

bar(); // 2
```

>无论我们使用什么方法将内部函数 传送 到它的词法作用域之外，它都将维护一个指向它最开始被声明时的作用域的引用，而且无论我们什么时候执行它，这个闭包就会被行使。

## 无处不在的闭包

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

## 循环+闭包

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

## 模块

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

# 动态作用域

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

## 填补块儿作用域

在ES6中声明一个作用域块儿的代码如下：

```js
{
	let a = 2;
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

但是，如果在ES6之前的环境中声明一个作用域块儿，则需要如下代码：

```js
try{throw 2}catch(a){
	console.log( a ); // 2
}

console.log( a ); // ReferenceError
```

可以看到一个 `try/catch` 似乎强制抛出一个错误，但是这个它抛出的“错误”只是一个值 2。然后接收它的变量声明是在 catch(a) 子句中，`catch`子句拥有块儿作用域，这意味着它可以在ES6之前的环境中创建一个块儿作用域。

事实上代码转换工具将ES6转换成ES5时候，块儿作用域就是被转换成这样的。

## 词法this

```js
var obj = {
	id: "awesome",
	cool: function coolFn() {
		console.log( this.id );
	}
};

var id = "not awesome";

obj.cool(); // awesome

setTimeout( obj.cool, 100 ); // not awesome
```

这个问题就是在 `cool()` 函数上丢失了 `this` 绑定。有各种方法可以解决这个问题，但一个经常被重复的解决方案是 `var self = this;`。

```js
var obj = {
	count: 0,
	cool: function coolFn() {
		var self = this;

		if (self.count < 1) {
			setTimeout( function timer(){
				self.count++;
				console.log( "awesome?" );
			}, 100 );
		}
	}
};

obj.cool(); // awesome?
```

用不过于深入细节的方式讲，`var self = this` 的“解决方案”免除了理解和正确使用 this 绑定的整个问题，而是退回到我们也许感到更舒服的东西上面：词法作用域。self 变成了一个可以通过词法作用域和闭包解析的标识符，而且一直不关心 this 绑定发生了什么。

ES6的解决方案，箭头函数：

```js
var obj = {
	count: 0,
	cool: function coolFn() {
		if (this.count < 1) {
			setTimeout( () => { // 箭头函数能好用？
				this.count++;
				console.log( "awesome?" );
			}, 100 );
		}
	}
};

obj.cool(); // awesome?
```

简单的解释是，**当箭头函数遇到它们的 `this` 绑定时，它们的行为与一般的函数根本不同。它们摒弃了 `this` 绑定的所有一般规则，而是采用它们的直接外围词法作用域的 `this` 的值，无论它是什么。**

于是，在这个代码段中，箭头函数不会以不可预知的方式丢掉 `this` 绑定，它只是“继承” `cool()` 函数的 `this` 绑定（如果像我们展示的那样调用它就是正确的！）。

正确使用`this`机制的解决办法：

```js
var obj = {
	count: 0,
	cool: function coolFn() {
		if (this.count < 1) {
			setTimeout( function timer(){
				this.count++; // `this` 因为 `bind(..)` 所以安全
				console.log( "more awesome" );
			}.bind( this ), 100 ); // 看，`bind()`!
		}
	}
};

obj.cool(); // more awesome
```

`bind()`方法创建一个新的函数, 当被调用时，将其`this`关键字设置为提供的值，在新函数被调用时提供的任何前面的给定序列的参数。
