javaScript 本身可以用它 typeof 来检测变量的类型，但是有些结果却让人疑惑，例如，数组的类型居然是 "Object" 。

下面是用 typeof 对各种数据类型的判断结果:

```
var myFunction = function() {
    console.log('hello');
};

var myObject = {
    foo : 'bar'
};

var myArray = [ 'a', 'b', 'c' ];

var myString = 'hello';

var myNumber = 3;

typeof myFunction;   // 返回 'function'
typeof myObject;     // 返回 'object'
typeof myArray;      // 返回 'object' -- 小心哦！
typeof myString;     // 返回 'string';
typeof myNumber;     // 返回 'number'
typeof null;         // 返回 'object' -- 小心哦！


if (myArray.push && myArray.slice && myArray.join) {
    // 很可能是一个数组
    // 当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。
}

if (toString.call(myArray) === '[object Array]') {
    // 肯定是一个数组！
    // 这是判断一个变量是否为数组的最可靠方法
}
```
【学习参考】
*   [Javascript数组类型检测：编写更强壮的isArray函数](http://scriptfans.iteye.com/blog/318821)
*   [JavaScript:Object.prototype.toString方法的原理](http://www.cnblogs.com/ziyunfei/archive/2012/11/05/2754156.html)

