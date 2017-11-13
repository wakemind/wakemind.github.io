## 先热个身！
this 就是找拥有当前上下文（context）的对象（context object）。

在理解 this 的绑定过程之前，首先要理解调用位置：**调用位置就是函数在代码中被调用的位置（而不是声明的位置）。如果要判断一个运行中函数的this绑定，就需要找到这个函数的直接调用位置**

Excerpt From: Kyle Simpson、赵望野、梁杰. “你不知道的JavaScript（上卷）.” iBooks. **

其可以分为 6 层，层数越高权力越大，this 只会认最大的。

## 第 1 层 ：全局范围内

权力最小的层，在普通情况下就是全局（浏览器里就是window）；在 use strict 的情况下就是 undefined。

```
console.log( this === window );  // true

function showThis () {
  console.log( this === window );  // true
}
function showStrictThis () {
  'use strict'
  console.log(this);  // undefined
}
```

## 第 2 层 ：方法调用

找函数前面的点（ .）， 如果用到 this 的那个函数是属于某个 context object 的，那么这个 context object 绑定到 this。

比如下面的例子，boss 是 returnThis 的 context object ，或者说 returnThis 属于boss：

```
var boss = {
    name: 'xusir',
    returnThis:function () {
        console.dir(this === boss); // true
        return this
    }
}

boss.returnThis() === boss // true
```
下面这个例子就要小心点咯

```
var boss1 = {
    name: 'xusir1',
    returnThis:function () {
        return this // boss1
  }
}

var boss2 = {
    name: 'xusir2',
    returnThis :function () {
        return boss1.returnThis(); // boss1
  }
}

var boss3 = {
    name: 'xusir3',
    returnThis :function () {
        var returnThis = boss1.returnThis
        return returnThis() // window
  }
}
```
只要看使用 this 的那个函数。在 **boss2.returnThis** 里，使用 this 的函数是boss1.returnThis，所以 this 绑定到 boss1；

在 **boss3.returnThis** 里，使用 this 的函数是 returnThis，所以 this 绑定到 window。

## 第 3 层 ：call() 、apply()

fun.call(obj1, arg1, arg2, ...);
fun.apply(obj2, [arrs]);

ECMAScript 规范给所有函数都定义了 call 与 apply 两个方法，它们的应用非常广泛，它们的作用也是一模一样，只是传参的形式有区别而已。

```
var Obj1 = {
    name: 'Object1',
    say: function(p1, p2) {
        console.log(this.name + ' says ' + p1 + ' ' + p2);
    }
};

// logs 'Object1 says Good morning'
Obj1.say('Good', 'morning');

var Obj2 = {
    name: 'Object2'
};

// logs 'Object2 says Good afternoon'
Obj1.say.call(Obj2, 'Good', 'afternoon');

// logs 'Object2 says Good afternoon again'
Obj1.say.apply(Obj2, ['Good', 'afternoon again']);
```

【例】：改变 this 指向

```
var obj = {
    name: 'linxin'
}

function func() {
    console.log(this.name);
}

func.call(obj);       // linxin

```
我们知道，call 方法的第一个参数是作为函数上下文的对象，这里把 obj 作为参数传给了 func，此时函数里的 this 便指向了 obj 对象。此处 func 函数里其实相当于

```
function func() {
    console.log(obj.name);
}
```
【例】：借用别的对象的方法

```
var Person1  = function () {
    this.name = 'linxin';
}
var Person2 = function () {
    this.getname = function () {
        console.log(this.name);
    }
    Person1.call(this);
}
var person = new Person2();
person.getname(); // linxin
```
从上面我们看到，Person2 实例化出来的对象 person 通过 getname 方法拿到了 Person1 中的 name。因为在 Person2 中，Person1.call(this) 的作用就是使用 Person1 对象代替 this 对象，那么 Person2 就有了 Person1 中的所有属性和方法了，相当于 Person2 继承了 Person1 的属性和方法。


> ！请注意函数才能使用 call() 、apply()，对象并不能使用

## 第 4 层 ：bind()

Function.**bind**(thisObj, arg1, arg2, ...);

在 EcmaScript5 中扩展了叫 bind 的方法，在低版本的 IE 中不兼容。它和 call 很相似，接受的参数有两部分，第 1 个参数是是作为函数上下文的对象，第 2 部分参数是个列表，可以接受多个参数。

bind 方法**不会立即执行，而是返回一个改变了上下文 this 后的函数**。而原函数 func 中的 this 并没有被改变，依旧指向全局对象 window。

```
var obj = {
    name: 'linxin'
}

function func() {
    console.log(this.name);
}

var func1 = func.bind(obj);
func1();   // linxin
```

**bind() 会覆盖第 3 层 call() 、apply() 的命令**

```
function returnThis () {
  return this
}
var boss1 = { name: 'boss1'}
var boss1returnThis = returnThis.bind(boss1)
boss1returnThis() // boss1
var boss2 = { name: 'boss2' }
boss1returnThis.call(boss2) // still boss1
```

【！总结】

- apply 、 call 、bind 三者的**第一个参数是 null 或者 undefined**，等于将 this 绑定到全局对象
- 只有「函数」和「对象的方法」才有 apply 、 call 、bind  这3 个方法，其它对象均没有
- bind 是返回对应函数，便于稍后调用；apply 、call 则是立即调用

## 第 5 层：调用构造函数

一个比较容易忽略的会绑定 this 的地方就是 new。**当我们 new 一个函数时，就会自动把 this 绑定在新创建的对象上**，然后再调用这个函数。「它会覆盖 bind 的绑定」。

```
function showThis () {
  console.log(this)
}

showThis() // window
new showThis() // showThis

var boss1 = { name: 'boss1' }
showThis.call(boss1) // boss1
new showThis.call(boss1) // TypeError

var boss1showThis = showThis.bind(boss1) //返回一个改变了上下文 this 后的函数
boss1showThis() // boss1
new boss1showThis() // showThis
```

## 第 6 层：箭头函数

ES2015 的「箭头函数」里的 this 被永远封印到当前词法作用域之中，称作 Lexical this ，在代码运行前就可以确定。

这样的好处就是**方便让回调函数的 this 使用当前的作用域，不怕引起混淆**。

所以对于箭头函数，只要看它在「哪里创建」的就行。

```
function callback (cb) {
  cb()
}
callback(() => { console.log(this) }) // window
var boss1 = {
  name: 'boss1',
  callback: callback,
  callback2 () {
    callback(() => { console.log(this) })
  }
}
boss1.callback(() => { console.log(this) }) // still window
boss1.callback2(() => { console.log(this) }) // boss1
```

下面这种奇葩的使用方式就需要注意：

```
var returnThis = () => this
returnThis() // window
new returnThis() // TypeError
var boss1 = {
  name: 'boss1',
  returnThis () {
    var func = () => this
    return func()
  }
}
returnThis.call(boss1) // still window
var boss1returnThis = returnThis.bind(boss1)
boss1returnThis() // still window
boss1.returnThis() // boss1
var boss2 = {
  name: 'boss2',
  returnThis: boss1.returnThis
}
boss2.returnThis() // boss2
```
如果你不知道最后为什么会是 boss2，继续理解“对于箭头函数，只要看它在哪里创建”这句话。

## 【参考资料】
- [JavaScript This 的六道坎 | CRIMX](https://blog.crimx.com/2016/05/12/understanding-this/)








