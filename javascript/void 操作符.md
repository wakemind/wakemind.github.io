# 谈谈Javascript中的void操作符

## 1 引言

void 唯一的目的就是让一个表达式的计算结果为 undefined

## 2 void 运算符有两种写法，类似于 typeof
- void 表达式
 
 > void 0

- void(表达式)

 > void(0)
 
## 3 用 void 0 代替 undefined

### 3.1 谈谈 undefined

undefined 并不是保留词（reserved word），它只是全局对象的一个属性，在低版本 IE 中**能被重写**

```
var undefined = 10;

// undefined -- chrome
// 10 -- IE 8
alert(undefined);
```
事实上，undefined 在 ES5 中已经是全局对象的一个只读（read-only）属性了，它不能被重写。但是**在局部作用域中，还是可以被重写的**

```
(function() {
 
 //局部变量
  var undefined = 10;

  // 10 -- chrome
  alert(undefined);
})();

(function() {

  // 全局变量
  undefined = 10;

  // undefined -- chrome
  alert(undefined);
})();
```
### 3.2 为何用 void 0 来代替
我们来看看 [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/void) 的解释：

> The void operator evaluates the given expression and then returns undefined.

意思是说 **void 运算符能对给定的表达式进行求值，然后返回 undefined**。也就是说，void 后面你随便跟上一个表达式，返回的都是 undefined，都能完美代替 undefined！那么，这其中最短的是什么呢？毫无疑问就是 void 0 了。其实用 void 1，void (1+1)，void (0) 或者 void "hello"，void (new Date()) 等等，都是一样的效果。更重要的前提是，void 是不能被重写的（cannot be overidden）。

那么，ES5 大环境下，void 0 就没有用武之地了吗？答案是否定的，用 void 0 代替 undefined 能节省不少字节的大小，事实上，不少 JavaScript 压缩工具在压缩过程中，正是将 undefined 用 void 0 代替掉了。

## 4 运用

### 4.1 立即调用的函数表达式

在使用「立即执行的函数表达式」时，可以**利用 void 运算符让 JavaScript 引擎把一个 function 关键字识别成函数表达式而不是函数声明（语句）**

```
void function Fn(){
	alert(1);
}();
```

### 4.2 超链接使用
当用户点击一个以 javascript: URI 时，它会评估URI中的代码，然后用返回的值替换页面内容，除非返回的值是undefined。void运算符可用于返回undefined。例如：

```
<a href="javascript:void(0);">
  这个链接点击之后不会做任何事情，如果去掉 void()，
  点击之后整个页面会被替换成一个字符 0。
</a>
```
>【 ！注意】
>虽然这么做是可行的，但利用 javascript: 伪协议来执行 JavaScript 代码是「不推荐」的，推荐的做法是为链接元素绑定事件 obj === void 0 // 判断对象是否未定义

### 4.3 检查对象是否是 undefined

underscore.js

```
 _.isUndefined = function(obj) {
    return obj === void 0;
  };
```
##【学习参考】
- [为什么用「void 0」代替「undefined」](https://github.com/hanzichi/underscore-analysis/issues/1)

