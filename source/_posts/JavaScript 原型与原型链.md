---
title: JavaScript 原型与原型链
---
一、概述
----

在 JavaScript 中，是一种面向对象的程序设计语言，但是 JS 本身是没有 “类” 的概念，JS 是靠原型和原型链实现对象属性的继承。

在理解原型前，需要先知道对象的构造函数是什么，构造函数都有什么特点？

**1. 构造函数**

```
function Person(name, gender) {
    this.name = name;
    this.gender = gender;
}

var person = new Person("周杰伦", "男");


person

{
    name: "周杰伦",
    gender: "男"
}



```

以上代码，普通函数 `Person()`，加上 `new` 关键字后，就构造了一个对象 `person`

所以构造函数的定义就是普通函数加上 `new` 关键字，并总会返回一个对象。

**2. 函数对象**  
同时，JS 中的对象分为一般对象和函数对象。那什么是一般对象，什么又是函数对象呢？

JavaScript 的类型分为基本数据类型和引用数据类型，基本数据类型目前有 6 种（null, undefined, string, number, boolean, Symbol）。 其余的数据类型都统称为 object 数据类型，其中，包括 Array, Date, Function 等，所以函数可以称为函数对象。

```
let foo = function(){

}
foo.name = "bar";
foo.age = 24;
console.log(foo instanceof Function)  
console.log(foo.age)  



```

以上代码就说明了函数其实是一个对象，也可以具有属性。

二、原型链
-----

JavaScript 中的对象，有一个特殊的 `[[prototype]]` 属性, 其实就是对于其他对象的引用（委托）。当我们在获取一个对象的`属性`时，如果这个对象上没有这个属性，那么 JS 会沿着对象的 `[[prototype]]`链 一层一层地去找，最后如果没找到就返回 `undefined`;

这条一层一层的查找属性的方式，就叫做原型链。

```
var o1 = {
    age: 6
}



```

那么，为什么一个对象要引用，或者说要委托另外一个对象来寻找属性呢？

本文开篇的第一句话，就指出来的，JavaScript 中，和一般的 OOP 语言不同，它没有 '类'的概念，也就没有 '模板' 来创建对象，而是通过字面量或者构造函数的方式直接创建对象。那么也就不存在所谓的类的复制继承。

三、原型
----

那什么又是原型呢？

既然我们没有类，就用其他的方式实现类的行为吧，看下面这句话↓↓。

**1. 每个函数都有一个原型属性 prototype 对象**

```
function Person() {

}

Person.prototype.name = 'JayChou';


var person1 = new Person();
var person2 = new Person();

console.log(person1.name) 
console.log(person2.name) 



```

通过构造函数创造的对象，对象在寻找 `name` 属性时，找到了 构造函数的 `prototype` 对象上。

这个构造函数的 `prototype` 对象，就是 **原型**

用示意图来表示：

![](http://upload-images.jianshu.io/upload_images/1753073-e61c37a1833588da.png)

查找对象实例属性时，会沿着原型链向上找，在现代浏览器中，标准让每个对象都有一个 `__proto__` 属性，指向原型对象。那么，我们可以知道对象实例和函数原型对象之间的关系。

![](http://upload-images.jianshu.io/upload_images/1753073-55cdcf70fb2d0bfa.png)

**2. 每个原型对象都有一个 constructor 属性指向关联的构造函数**

为了验证这一说话，举个例子。

```
function Person() {}

Person === Person.prototype.constructor; 



```

那么对象实例是构造函数构造而来，那么对象实例是不是也应该有一个 `constructor` 呢？

```
function Person() {}

const person = new Person();
person.constructor === Person 



```

但事实上，对象实例本身并没有 `constructor` 属性，对象实例的 constructor 属性来自于引用了原型对象的 `constructor` 属性

```
person.constructor === Person.prototype.constructor // true



```

![](http://upload-images.jianshu.io/upload_images/1753073-d283a84be57dff93.png)

**3. 原型链顶层：Object.**proto** == null**

既然 JS 通过原型链查找属性，那么链的顶层是什么呢，答案就是 Object 对象，Object 对象其实也有 `__proto__`属性，比较特殊的是 `Object.__proto__` 指向 `null`, 也就是空。

```
Object.prototype.__proto__ === null



```

![](http://upload-images.jianshu.io/upload_images/1753073-6242db40f1d49531.png)

我们回过头来看函数对象：

> 所有函数对象的 proto 都指向 Function.prototype，它是一个空函数（Empty function）

```
Number.__proto__ === Function.prototype  
Number.constructor == Function 

Boolean.__proto__ === Function.prototype 
Boolean.constructor == Function 

String.__proto__ === Function.prototype  
String.constructor == Function 


Object.__proto__ === Function.prototype  
Object.constructor == Function 


Function.__proto__ === Function.prototype 
Function.constructor == Function 

Array.__proto__ === Function.prototype   
Array.constructor == Function 

RegExp.__proto__ === Function.prototype  
RegExp.constructor == Function 

Error.__proto__ === Function.prototype   
Error.constructor == Function 

Date.__proto__ === Function.prototype    
Date.constructor == Function 



```

> 所有的构造器都来自于 Function.prototype，甚至包括根构造器 Object 及 Function 自身。所有构造器都继承了 ·Function.prototype· 的属性及方法。如 length、call、apply、bind

以图会友，这就是网上经常看到的 JS 原型和原型链关系图：

![](http://upload-images.jianshu.io/upload_images/1753073-8f0555ce6e0342fa.png)

对于以上看似很复杂的关系图，只需要理解 5 点：

1.  每个函数都有一个原型属性 prototype 对象
2.  普通对象的构造函数是 Object()，所以 `Person.prototype.__proto__ === Object.prototype`
3.  函数对象都来自于 Function.prototype
4.  函数对象也是对象，所有 `Function.prototype.__proto__ === Object.prototype`
5.  `Object.__proto__` 是 null

总结
--

以上就是 JavaScript 中原型和原型链的知识。由于 JS 没有'类', 所以采用了原型的方式实现继承，正确的说法是引用或者委托，因为对象之间的关系不是复制，而是委托。在查找属性的时候，引用（委托）原型对象的属性，也就是我们常说的原型继承。