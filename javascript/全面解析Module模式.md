# 全面解析 Module 模式

## 1 简介
Module 模式是 JavaScript 编程中一个非常通用的模式，首先我们来看看 Module 模式的「基本特征」：

1.  模块化，可重用
2.  封装了变量和 function，和全局的 namaspace 不接触，松耦合
3.  只暴露可用 public 的方法，其它私有方法全部隐藏

## 2 基本用法
先看一下最简单的一个实现，代码如下：

```
var Calculator = function (eq) {
    //这里可以声明私有成员

    var eqCtl = document.getElementById(eq);

    return {
        // 暴露公开的成员
        add: function (x, y) {
            var val = x + y;
            eqCtl.innerHTML = val;
        }
    };
};

```
我们可以通过如下的方式来「调用」：

```
var calculator = new Calculator('eq');
calculator.add(2, 2);
```
每次用的时候都要 new 一下，也就是说**每个实例在内存里都是一份 copy**，如果你不需要传参数或者没有一些特殊苛刻的要求的话，我们可以在最后一个 } 后面加上一个「括号」，来达到自执行的目的，**这样该实例在内存中只会存在一份 copy**

### 2.1 匿名闭包

匿名闭包是让一切成为可能的基础，而这也是 JavaScript 最好的特性，我们来创建一个最简单的闭包函数，**函数内部的代码一直存在于闭包内，在整个运行周期内，该闭包都保证了内部的代码处于私有状态**。

```
(function () {
    // ... 所有的变量和 function 都在这里声明，并且作用域也只能在这个匿名闭包里
    // ...但是这里的代码依然可以访问外部全局的对象
}());
```

注意，匿名函数后面的「括号」，这是 JavaScript 语言所要求的，因为如果你不声明的话，JavaScript 解释器默认是声明一个 function 函数，**有括号，就是创建一个函数表达式，也就是自执行**，当然你也可以这样来声明：

```
(function () { /* 内部代码 */ })();
```


>！不过我们推荐使用第一种方式

### 2.2 引用全局变量

我们可以「将全局变量当成一个参数传入到匿名函数然后使用」

```
(function ($, _) {
    // 这里，我们的代码就可以使用全局的 jQuery 对象了，underscore 也是一样
} (jQuery, _));
```

 不过，有时候可能「不仅仅要使用全局变量，而是也想声明全局变量」，如何做呢？我们可以通过匿名函数的
返回值来返回这个全局变量，这也就是一个基本的 Module 模式，来看一个完整的代码

```
var blogModule = ( function() {
    var my = {}
    var privateName = '徐 sir';
    
    function privateAddTopic(data) {
        // 这里是内部处理代码
    }
    
    my.Name = privateName;
    my.AddTopic = function(data) {
        privateAddTopic(data);
    };
    
    return my;
}() );
```
上面的代码声明了一个全局变量 blogModule，并且带有 2 个可访问的属性：blogModule.AddTopic 和 blogModule.Name，除此之外，其它代码都在匿名函数的闭包里保持着私有状态。同时根据上面传入全局变量的例子，我们也可以很方便地传入其它的全。

## 3 高级用法

### 3.1 扩展

Module 模式的一个限制就是所有的代码都要写在一个文件，但是在一些大型项目里，**将一个功能分离成多个文件是非常重要的，因为可以多人合作易于开发**。再回头看看上面的全局参数导入例子，我们可以**把blogModule 自身传进去，然后再返回对象就达到了我们所说的目的**，上代码：

```
var blogModule = (function (my) {
    my.AddPhoto = function () {
        //添加内部代码  
    };
    return my;
} (blogModule)); 
```

### 3.2 松耦合扩展

上面的代码尽管可以执行，但是必须先声明 blogModule，然后再执行上面的扩展代码，也就是说步骤不能乱，怎么解决这个问题呢？我们来回想一下，我们平时声明变量的都是都是这样的：

```
var cnblogs = cnblogs || {} ;
```
这是确保 cnblogs 对象，在**存在的时候直接用，不存在的时候直接赋值为 {}**，我们来看看如何利用这个特性来实现Module模式的任意加载顺序：

```
var blogModule = (function (my) {

    // 添加一些功能   
    
    return my;
} (blogModule || {}));  
```
通过这样的代码，每个单独分离的文件都保证这个结构，那么我们就可以实现任意顺序的加载，所以，这个时候的 **var 就是必须要声明的，因为不声明，其它文件读取不到**。

### 3.3 紧耦合扩展

虽然松耦合扩展很牛叉了，但是可能也会存在一些限制，比如你没办法重写你的一些属性或者函数，也不能在初始化的时候就是用 Module 的属性。**紧耦合扩展限制了加载顺序，但是提供了我们重载的机会**，看如下例子：

```
var blogModule = (function (my) {
    var oldAddPhotoMethod = my.AddPhoto;

    my.AddPhoto = function () {
        // 重载方法，依然可通过oldAddPhotoMethod调用旧的方法
    };

    return my;
} (blogModule));
```
通过这种方式，我们达到了重载的目的，当然如果你想在继续在「内部使用原有的属性」，你可以调用oldAddPhotoMethod 来用。

### 3.4 克隆与继承

```
var blogModule = (function (old) {
    var my = {},
        key;

    for (key in old) {
        if (old.hasOwnProperty(key)) {
            my[key] = old[key];
        }
    }

    var oldAddPhotoMethod = old.AddPhoto;
    my.AddPhoto = function () {
        // 克隆以后，进行了重写，当然也可以继续调用oldAddPhotoMethod
    };

    return my;
} (blogModule));
```
这种方式灵活是灵活，但是也需要花费灵活的代价，其实**该对象的属性对象或 function 根本没有被复制，只是对同一个对象多了一种引用而已，所以如果老对象去改变它，那克隆以后的对象所拥有的属性或 function 函数也会被改变**

### 3.5 跨文件共享私有对象

通过上面的例子，我们知道，如果一个 module 分割到多个文件的话，每个文件需要保证一样的结构，也就是说**每个文件匿名函数里的私有对象都不能交叉访问，那如果我们非要使用，那怎么办呢？** 我们先看一段代码：

```
var blogModule = (function (my) {
    var _private = my._private = my._private || {},
        
        _seal = my._seal = my._seal || function () {
            delete my._private;
            delete my._seal;
            delete my._unseal;
            
        },
        
        _unseal = my._unseal = my._unseal || function () {
            my._private = _private;
            my._seal = _seal;
            my._unseal = _unseal;
        };
        
    return my;
} (blogModule || {}));
```
**任何文件都可以对他们的局部变量 _private 设属性，并且设置对其他的文件也立即生效。**一旦这个模块加载结束，应用会调用 blogModule._seal() "上锁"，这会阻止外部接入内部的 _private。如果这个模块需要再次增生，应用的生命周期内，任何文件都可以调用 _unseal()  ”开锁”，然后再加载新文件。加载后再次调用 _seal() ”上锁”。

### 3.6 子模块

最后一个也是最简单的使用方式，那就是创建子模块

```
blogModule.CommentSubModule = (function () {
    var my = {};
    // ...
    return my;
} ());
```
尽管非常简单，我还是把它放进来了，因为我想说明的是**子模块也具有一般模块所有的高级使用方式，也就是说你可以对任意子模块再次使用上面的一些应用方法**。

## 【参考资料】

- [深入理解JavaScript系列（3）：全面解析Module模式 - 汤姆大叔 - 博客园](http://www.cnblogs.com/TomXu/archive/2011/12/30/2288372.html)

