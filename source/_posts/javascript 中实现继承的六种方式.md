---
title: javascript 中实现继承的六种方式
---
javascript 中对于继承的描述：

            许多面向对象语言都支持两种继承的方式：接口继承和实现继承。接口继承只继承方法签名，而实现继承则继承实际的方法。在 javascript 中由于函数没有签名也就无法实现接口继承，而只支持实现继承，而且实现继承主要通过原型链来实现的。

            先引述下官方文档对于原型链的描述：其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。要理解这个概念要先弄清楚构造函数，原型，和实例的关系：每个构造函数（只要是函数）都有一个 prototype 属性该属性指向一个对象（这个对象就是构造函数的原型对象）；原型对象（只要是对象）中都有一个 constructor 属性该属性指向一个构造函数；而实例中都包含一个指向原型对象的内部指针 `Prototype`。说白了就是原型链的构建是通过将一个类型的实例赋值给另一个构造函数的原型实现的。这样子类型就可以访问定义在超类型上的所有属性和方法了。每个对象都有自己的原型对象，以原型对象为模板从原型对象中继承属性和方法，原型对象也可以有自己的原型并从中继承属性和方法，一层一层，以此类推，这种关系被称为原型链它解释了为何一个对象会拥有定义在其他对象上的属性和方法。  

javascript 中实现继承的六种方式：

            1、原型链

            2、借用构造函数  

            3、组合继承（组合使用原型链和借用构造函数）  

            4、原型式继承  

            5、寄生式继承  

            6、寄生组合式继承（组合使用组合继承和寄生式继承）

1、原型链  

```
具体示例：
// 实现原型链的一种基本模式
function SuperType(){
            this.property = true;
}
SuperType.prototype.getSuperValue = function(){
            return this.property;
};
function SubType(){
            this.subproperty = false;
}

// 继承，用 SuperType 类型的一个实例来重写 SubType 类型的原型对象
SubType.prototype = new SuperType();
SubType.prototype.getSubValue = function(){
            return this.subproperty;
};
var instance = new SubType();
alert(instance.getSuperValue());     // true
PS：SubType 继承了 SuperType，而继承是通过创建 SuperType 的实例，并将该实例赋值给 SubType 的原型实现的。实现的本质是重写子类型的原型对象，代之以一个新类型的实例。子类型的新原型对象中有一个内部属性 `Prototype` 指向了 SuperType 的原型，还有一个从 SuperType 原型中继承过来的属性 constructor 指向了 SuperType 构造函数。最终的原型链是这样的：instance 指向 SubType 的原型，SubType 的原型又指向 SuperType 的原型，SuperType 的原型又指向 Object 的原型（所有函数的默认原型都是 Object 的实例，因此默认原型都会包含一个内部指针，指向 Object.prototype）。
原型链的缺点：
            1、在通过原型来实现继承时，原型实际上会变成另一个类型的实例。于是，原先的实例属性也就顺理成章地变成了现在的原型属性，并且会被所有的实例共享。这样理解：在超类型构造函数中定义的引用类型值的实例属性，会在子类型原型上变成原型属性被所有子类型实例所共享。
            2、在创建子类型的实例时，不能向超类型的构造函数中传递参数。

```

2、借用构造函数（也称伪造对象或经典继承）

```
具体示例：
// 在子类型构造函数的内部调用超类型构造函数；使用 apply() 或 call() 方法将父对象的构造函数绑定在子对象上
function SuperType(){
            // 定义引用类型值属性
            this.colors = ["red","green","blue"];
}
function SubType(){
            // 继承 SuperType，在这里还可以给超类型构造函数传参
            SuperType.call(this);
}
var instance1 = new SubType();
instance1.colors.push("purple");
alert(instance1.colors);     // "red,green,blue,purple"

var instance2 = new SubType();
alert(instance2.colors);     // "red,green,blue"
PS：通过使用 apply() 或 call() 方法，我们实际上是在将要创建的 SubType 实例的环境下调用了 SuperType 构造函数。这样一来，就会在新 SubType 对象上执行 SuperType() 函数中定义的所有对象初始化代码。结果 SubType 的每个实例就都会具有自己的 colors 属性的副本了。
借用构造函数的优点是解决了原型链实现继承存在的两个问题；借用构造函数的缺点是方法都在构造函数中定义，因此函数复用就无法实现了。而且，在超类型的原型中定义的方法，对子类型而言也是不可见的，结果所有类型都只能使用构造函数模式。

```

3、组合继承（也称伪经典继承）

```
具体示例：
// 将原型链和借用构造函数的技术组合到一块。使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承。这样，既通过在原型上定义方法实现了函数复用，又能够保证每个实例都有自己的属性。
function SuperType(name){
            this.name = name;
            this.colors = ["red","green","blue"];
}
SuperType.prototype.sayName = function(){
            alert(this.name);
};
function SubType(name,age){
            // 借用构造函数方式继承属性
            SuperType.call(this,name);
            this.age = age;
}
// 原型链方式继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;
SubType.prototype.sayAge = function(){
            alert(this.age);
};
var instance1 = new SubType("luochen",22);
instance1.colors.push("purple");
alert(instance1.colors);      // "red,green,blue,purple"
instance1.sayName();
instance1.sayAge();

var instance2 = new SubType("tom",34);
alert(instance2.colors);      // "red,green,blue"
instance2.sayName();
instance2.sayAge();
PS：组合继承避免了原型链和借用构造函数的缺陷，融合了它们的优点，成为 javascript 中最常用的继承模式。而且，使用 instanceof 操作符和 isPrototype() 方法也能够用于识别基于组合继承创建的对象。但是，它也有自己的不足。最大的问题是无论在什么情况下，都会调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是在子类型构造函数内部。

```

4、原型式继承

```
具体示例：
// 借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。
1、自定义一个函数来实现原型式继承
function object(o){
            function F(){}
            F.prototype = o;
            return new F();
}
PS：在 object() 函数内部，先创建一个临时性的构造函数，然后将传入的对象作为这个构造函数的原型，最后返回这个临时类型的一个新实例。实质上，object() 对传入其中的对象执行了一次浅复制。
2、使用 Object.create() 方法实现原型式继承。这个方法接收两个参数：一是用作新对象原型的对象和一个为新对象定义额外属性的对象。在传入一个参数的情况下，此方法与 object() 方法作用一致。
在传入第二个参数的情况下，指定的任何属性都会覆盖原型对象上的同名属性。
var person = {
            name: "luochen",
            colors: ["red","green","blue"]
}; 
var anotherPerson1 = Object.create(person,{
            name: {
                    value: "tom"
            }
});
var anotherPerson2 = Object.create(person,{
            name: {
                    value: "jerry"
            }
});
anotherPerson1.colors.push("purple");
alert(anotherPerson1.name);     // "tom"
alert(anotherPerson2.name);     // "jerry"
alert(anotherPerson1.colors);    // "red,green,blue,purple"
alert(anotherPerson2.colors);    // "red,green,bule,purple";
PS：只是想让一个对象与另一个对象类似的情况下，原型式继承是完全可以胜任的。但是缺点是：包含引用类型值的属性始终都会共享相应的值。

```

5、寄生式继承

```
具体示例：
// 创建一个仅用于封装继承过程的函数，该函数在内部以某种方式来增强对象，最后返回这个对象
function createPerson(original){
            var clone = Object.create(original);   // 通过 Object.create() 函数创建一个新对象
            clone.sayGood = function(){              // 增强这个对象
                        alert("hello world！！！");
            };
            return clone;                                      // 返回这个对象 
}
PS：在主要考虑对象而不是自定义类型和构造函数的情况下，寄生式继承也是一种有用的模式。此模式的缺点是做不到函数复用。

```

6、寄生组合式继承

```
具体示例：
// 通过借用构造函数来继承属性，通过原型链的混成形式来继承方法。本质上，就是使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型的原型
function SuperType(name){
            this.name = name;
            this.colors = ["red","green","blue"];
}
SuperType.prototype.sayName = function(){
            alert(this.name);
};
function SubType(name,age){
            SuperType.call(this,name);
            this.age = age;
}
// 创建超类型原型的一个副本
var anotherPrototype = Object.create(SuperType.prototype);
// 重设因重写原型而失去的默认的 constructor 属性
anotherPrototype.constructor = SubType;
// 将新创建的对象赋值给子类型的原型
SubType.prototype = anotherPrototype;

SubType.prototype.sayAge = function(){
            alert(this.age);
};
var instance1 = new SubType("luochen",22);
instance1.colors.push("purple");
alert(instance1.colors);      // "red,green,blue,purple"
instance1.sayName();
instance1.sayAge();

var instance2 = new SubType("tom",34);
alert(instance2.colors);      // "red,green,blue"
instance2.sayName();
instance2.sayAge();
PS：这个例子的高效率体现在它只调用一次 SuperType 构造函数，并且因此避免了在 SubType.prototype 上面创建不必要，多余的属性。与此同时，原型链还能保持不变；因此还能够正常使用 instance 操作符和 isPrototype() 方法。

```