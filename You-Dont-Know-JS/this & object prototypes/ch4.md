# 第四章: 混合（淆）“类”的对象

## 构造器（Constructor）

类的实例由类的一种特殊方法构造， **这个方法的名称通常与类名相同**， 称为 _构造器（Constructor）_。这个方法的具体工作，就是初始化实例所需要的所有信息（状态）。

```js
class CoolGuy {
	specialTrick = nothing

	CoolGuy( trick ) {
		specialTrick = trick
	}

	showOff() {
		output( "Here's my trick: ", specialTrick )
	}
}
```

为了制造一个`CoolGuy`实例，我们需要调用类的构造器：

```js
Joe = new CoolGuy( "jumping rope" )

Joe.showOff() // Here's my trick: jumping rope
```

`CoolGuy`类有一个构造器`CoolGuy()`，它实际上就是在`new CoolGuy()`时调用的。

## 类继承

在面向类的语言中，你不仅可以定义一个能够初始化它自己的类，你还可以定义另外一个类 继承 自第一个类。

### 多态（Polymorphism）

同一操作作用于不同的对象，可以有不同的解释，产生不同的执行结果。

现实中，比如我们按下 F1 键这个动作：

* 如果当前在 Flash 界面下弹出的就是 AS 3 的帮助文档
* 如果当前在 Word 下弹出的就是 Word 帮助
* 在 Windows 下弹出的就是 Windows 帮助和支持
* 同一个事件发生在不同的对象上会产生不同的结果

多态最常见的两种实现方式：

* 覆盖：子类重新定义父类方法，这正好就是基于`prototype`继承的玩儿法
* 重载：多个同名但是参数不同的方法，这个JS没有

**注意：**传统的面向对象语言通过`super`可以从子类的构造器中直接访问父类构造器，这基本上是正确的，因为对于真正的类来说，构造器属于这个类。但是在JS中，实际上是认为“类”属于构造器（`Foo.prototype..`类型引用）更恰当。因为在 JS 中，父子关系仅存在于它们各自的构造器的两个 `.prototype` 对象间，构造器本身不直接关联，而且没有简单的方法从一个对象中相对的引用另一个。

如果类是继承而来的，对这些类本身（不是它们创建的对象！）又一个方法可以相对地引用他们继承的对象，这个相对引用通常被成为`super`。

假设子类`Bar`继承父类`Foo`，那么从概念上讲，子类`Bar`可以使用相对多态引用（即`super`）访问它的父类`Foo`的行为。但是事实上，_子类不过是被给予一份它从父类继承来的行为拷贝而已_。如果子类“覆盖”一个它继承的方法，原版的方法和覆盖版方法其实都是存在的，所以它们都可以访问。

**类继承意味着拷贝**

### 多重继承（Multiple Inheritance）

JS中并没有多重继承，但是开发者会用各种方法来模拟多重继承。

## 混合（Mixin）

**JS中没有“类”**可以拿来实例化，只有对象。而且对象也不会被拷贝到另一个对象中，而是被链接在一起。

### 明确的 Mixin（Explicit Mixins）

JS不会自动地将行为从一个对象拷贝到另一个对象，我们可以建造一个工具来手动拷贝。这样的工具经常被许多库/框架称为`extend(..)`。便于说明，我们这里叫它`mixin(..)`.

```js
// 大幅简化的 `mixin(..)` 示例：
function mixin( sourceObj, targetObj ) {
	for (var key in sourceObj) {
		// 仅拷贝非既存内容
		if (!(key in targetObj)) {
			targetObj[key] = sourceObj[key];
		}
	}

	return targetObj;
}

var Vehicle = {
	engines: 1,

	ignition: function() {
		console.log( "Turning on my engine." );
	},

	drive: function() {
		this.ignition();
		console.log( "Steering and moving forward!" );
	}
};

var Car = mixin( Vehicle, {
	wheels: 4,

	drive: function() {
		Vehicle.drive.call( this );
		console.log( "Rolling on all " + this.wheels + " wheels!" );
	}
} );
```



