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
