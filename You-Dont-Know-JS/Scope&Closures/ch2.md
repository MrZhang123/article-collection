# 第三章：函数与块儿作用域

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