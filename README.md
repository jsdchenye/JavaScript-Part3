
# 你不知道的JavaScript-下

标题
==
标签（空格分隔）： javascript 闭包 对象原型
---
2017.8.16
#### topic
1.对象原型

#### 背景
前面两部分将JavaScript中基本的概念和比较难以理解的闭包和this绑定都已经讲完了。下面将带来这本书关于JavaScript的精髓部分的分享。通过对类理论、类设计模式的介绍，进而分析JavaScript中的原型链、原型继承、对象关联以及JavaScript中通过各种方法、语法糖实现的模拟类行为模式，从而进一步凸显JavaScript中对象关联的设计思想的优势。揭开了隐藏在JavaScript表面的面纱，带领大家真正切切的去体会JavaScript的思想。

###### 对象原型
1.混合对象“类”
前面我们花了很大的篇幅来深入探讨对象以及针对对象的操作。这部分主要分享跟对象密切相关的“类”。
先从类理论、类的机制和类的继承出发，全方位的介绍类的相关概念以及使用，然后引出JavaScript中实现的模拟“类”的语法和实现方法。JavaScript本质上跟真正的“类”有很大区别，这种模拟“类”的行为埋下了很多隐患，造成代码难以理解和维护。
讲到类理论，离不开它的三个核心概念：实例化、继承、多态。
实例化简单来说就是将抽象的类理论转换为可实际操作的对象的过程；这个实例化的对象是由一个名字跟类名相同的特殊的类方法通过new调用来生成的，这个特殊的方法通常叫做构造函数。
class Student {
    name = nothing
    Student(val){
        name = val
    }
    show(){
        output("My name is",name)
    }
}
Person = new Student('cy');
Person.show()  //My name is cy
从中可以看出，真正的类中，构造函数是属于类的；但是JavaScript中的“类”是属于构造函数的（类似Func.prototype...这样的类型引用）。
Tip：JavaScript中严格说不存在所谓的类，它只有对象，这里用对象的原型prototype来模拟“类”；并且JavaScript中不存在所谓的构造函数，只有函数的构造调用。这里也是为了方便大家意会，所以引入了JavaScript的构造函数。

继承就是指子类复制了一份父类的行为和属性，成为一个独立并且完全不同的类，这个过程就叫做子类继承了父类。多态的概念跟继承密不可分，当子类继承了父类的方法，并且重写了其中的某个方法，这就叫做多态。因为存在继承，才有多态的出现。
//定义了一个父类（宙斯）
class Parent {
    name = 'Zeus'
    done() {
        output("Be God of the gods！")
    }
    doing() {
        done();
        ouput("Start to rule the universe！")
    }
}
//定义了一个子类（雅典娜）
class Child inherits Parent {
    sex = 'female'
    name = 'Athena'
    done() {
        output("Became Jose's daughter！")
    }
    todo(){
        inherited: doing()
        output("Overthrow Jose's rule！")
    }
}
这里的子类利用inherits继承了父类的全部的行为，并重写了父类中的done方法。
有个有趣的问题————当实例化Child子类之后，调用其中的todo方法，它继承了来自Parent父类中的doing方法，而这里doing方法中又引用了done方法，这里的done是来自父类还是子类中的呢？
实际上它会使用Child中的done方法，输出："Became Jose's daughter！"
所以在继承链的不同层次中一个方法名可以被多次定义。当调用该方法时到底选哪个层次中的取决在哪个类的实例中引用它。
多态并不表明子类和父类存在关联，子类得到的只是父类的一份副本。类的继承本质上就是复制。
在实现继承的时候，存在一种情况，叫做多重继承。简单来讲就是：某个子类继承自多个父类，可以融合来自多个父类的方法，看上去好像很有用。但是会带来很多问题，特别是当某两个父类存在同名的方法时，到底继承哪个会混乱。所以不推荐使用，会让问题变得复杂。干干净净，简单可依靠才是我们的追求。
JavaScript中只有对象，不存在可以被实例化的“类”；对象不会被复制到其他对象，只会被关联起来。由于其他语言中类表现出的都是复制行为，JavaScript使用混入来模拟实现类的复制行为。
由于JavaScript中只有对象，所以要实现复制某个对象行为到另一个对象的的方法：
1.for...in遍历源对象的所有属性，并复制到目标对象中
2.sourceObj.fun.call(this)显示多态来实现方法的重写
通过上述方法基本上就可以模拟实现JavaScript的“类”复制行为。
//定义遍历源对象属性的函数
function mixin(sourceObj, targetObj) {
    for(var key in sourceObj) {
        //仅当sourceObj中的属性不在targetObj中进行复制
        if(!(key in targetObj)) {
            targetObj[key] = sourceObj[key];
        }
    }
    return targetObj;
}
//定义一个源对象
var Parent = {
    name : 'Zeus',
    done: function() {
        console.log("Be God of the gods！");
    },
    doing: function() {
        this.done();
        console.log("Start to rule the universe！");
    }
};
//根据源对象，复制生成一个新的目标对象，并重写了doing方法
var Child = mixin(Parent, {
    sex: 'female',
    doing: function() {
        Parent.doing.call(this);
        console.log("Overthrow Jose's rule！");
    }
});
虽然看上去实现了JavaScript中的属性复制功能，但这里只会复制变量以及函数的引用。JavaScript中的函数无法真正的被复制，如果修改了被引用的函数本身，都会受影响。
并且使用显示多态的方法重写了源对象中的方法doing。显示多态会在所有需要的地方创建一个函数关联，导致代码变得复杂、难以阅读和维护。
总的来说JavaScript中存在很多模拟“类”的语法和实现方法，但是本质上跟真正的“类”有很大区别，这些模拟的方法埋下了很多隐患，造成代码难以理解和维护。

2.原型
前面讲述了类相关部分的内容，类是通过继承实现子类和父类、实例和类的关联。JavaScript中只存在对象，对象之间是通过[[prototype]]内置属性实现关联的。
当查找对象的某个属性时，通过[[Get]]操作查找对象的属性时，会从对象本身开始，一直沿着[[Prototype]]原型链查找，直到找到对应属性或查找完整条原型链。
JavaScript中的原型链的尽头指向Object.prototype，这也从侧面说明JavaScript中一切的源头就是对象；
当设置对象的某个属性时，过程相对比较复杂，主要分为：
A.对象本身包含属性时
    a.该属性是访问描述符并且存在setter，直接调用并设置该属性
    b.该属性是普通数据描述符，writable为false时设置失败
    c.该属性是普通数据描述符，writable为true时直接设置对象的该属性
B.对象不包含，上层原型链中包含
    a.该属性是普通数据描述符，writable为true时直接在对象本身添加属性
    b.该属性是普通数据描述符，writable为false时设置失败
    c.该属性是访问描述副且存在setter，直接在原型链中调用这个setter
C.对象不包含，上层原型链中也不存在，该属性直接被添加到对象本身
从上述情况分析来看，有两种情况下会发生属性屏蔽：
A.对象本身包含该属性并且上层原型链中也存在，对象属性会屏蔽原型链中该属性。
B.对象本身不包含，但是上层原型链中存在该数据描述符并且writable为true，会在对象中设置该属性，形成屏蔽。
我们在实际的操作过程中，也很容易产生隐式屏蔽：
var a = {num = 1};
var b = Object.create(a);
a.num;  //1
b.num;  //1
a.hasOwnProperty('num');  //true
b.hasOwnProperty('num');  //false
b.num++;  //隐式屏蔽，实际执行b.num = b.num + 1;对应屏蔽的第二种情况
b.hasOwnProperty('num');  //true

类在创建对象的时候，存在一张类似蓝图的抽象模式，用来复制父类行为，从而实例化对象。而JavaScript中只存在对象，对象之间只存在相互关联。
JavaScript在创建对象之间的关联的时候，一直在用模拟“类”的方法。这里比较常见的有以下两点：
【原型继承】javascript一直使用原型继承来说明对象之间的关系。继承意味着复制操作；而JavaScript中是通过委托行为来实现对象之间的关联的，这里把委托行为说成“原型继承”，给人一种动态版本的类继承的感觉。
【构造函数】javascript中使用构造函数来实例化对象。
functoin F(){}
var a = new F();
var b = new F();
...//
通过使用new+大写的方法函数来新建多个对象。这种写法很容易给人造成一种类中构造类实例的错觉。而JavaScript中不存在构造函数，只能说是函数的构造调用。
这里有一个比较容易出错的点就是constructor属性。
function F(){}
var a = new F();
a.constructor == F //这里a是委托了F.prototype这个对象，存在F.prototype.constructor
很容易被误解为a对象是由Foo构造的。其实这里a.constructor只是通过默认[[Prototype]]委托给F的，和构造无关。
function F(){}
F.prototype={};  //创建一个新的原型对象
var a = new F();
a.constructor === F;  //false
a.constructor === Object;  //true
这里的委托关系：
a ——> Foo.prototype ——> Object.prototype(Object.prototype.constructor === Object)
constructor是一个不可枚举、可变的属性，是一个特别不可靠并且不安全的属性;可以给任意[[Prototype]]链中的对象添加或修改constructor属性。
JavaScript中极力在模拟实现“面向类”的各种行为，造成开发者难以深入理解JavaScript的实质。

JavaScript中的原型继承主要是通过Object.create这个函数实现的，Object.create()建立了对象之间的关联，从而将整个链连接了起来。基本实现：
Object.create = function(srcObj){
    function F(){}
    F.prototype = srcObj;
    return new F();
};
Object.create(),不包含任何“类”的诡计，完美创建对象的关联;唯一的缺点就是需要创建一个新对象，抛弃旧对象，无法直接修改已有的默认对象。比如想要将Aoo.prototype关联到Boo.prototype：
Aoo.prototype = Object.create(Boo.prototype);
有两种可以直接修改对象的[[Prototype]]关联。
【I】__proto__
利用该属性可直接修改默认对象。Aoo.prototype.__proto__ = Boo.prototype。只不过这种方法不太标准，无法兼容所有的浏览器。
【II】Object.setPrototypeOf()
直接通过Object.setPrototypeOf(Aoo.prototype,Boo.prototype)实现直接关联。
不过忽略掉Object.create()带来的轻微性能损失（抛弃的对象需要进行垃圾回收），Object.create()更短、更可读。
这里还存在两种错误实现原型继承的方法：
A.直接通过对象赋值 —— Aoo.prototype =Boo.prototype；
只不过直接赋值的方法没有任何意义，还不如直接使用Boo。
B.通过new构造实现 —— Aoo.prototype = new Boo();
确实可以实现原型继承的效果。使用new的构造函数的方法有一些副作用，会生成.prototype和.constructor，创建的Aoo.prototype对象会引用Boo的这些副作用，影响Aoo的后代。
我们在使用原型继承实现对象之间的关联的时候，会造成代码不太好理解。
var Boo = {
    sayHi: function(){
        console.log("I'm xx");
    }
};
var Aoo = Object.create(Boo);
Aoo.sayHi();  //I'm xx
代码可以正常工作，Aoo对象委托了Boo中的sayHi方法。对于后续维护的人而言，Aoo对象中不存在sayHi方法，却可以正常工作，会让人觉得很神奇，不利于后续的维护和理解。可以改进为：
var Boo = {
    sayHi: function(){
        console.log("I'm xx");
    }
};
var Aoo = Object.create(Boo);
Aoo.introduce = function(){
    this.sayHi();  //内部委托
};
Aoo.introduce();  //I'm xx
使用内部的委托来替代直接委托，会让对象的API接口设计的更加清晰。
那如何检查一个实例的继承祖先呢？
【站在类的角度】使用instanceof来判断，它用来检查对象&&函数之间的关系
function F(){}
var a = new F();
a instanceof F;  //true
用来判断：在a的整条原型链中是否有指向Func.prototype的对象
【站在对象角度】使用isPrototypeOf来判断，它用来检查对象&&对象之间的关系
function F(){}
var a = new F();
F.prototype.isPrototypeOf(a)； //true
用来判断：在a的整条原型链中是否出现过Func.prototype
同时我们也可以设置和获取对象之间的[[Prototype]]关联，只针对对象之间
getPrototypeOf:Object.getPrototypeOf(a)==F.prototype【获取对象关联】
setPrototypeOf:Object.setPrototypeOf(a)=F.prototype 【设置对象关联】

3.行为委托
前面讲了那么多关于类、继承的相关概念，以及JavaScript中模拟实现“类”设计模式的各种语法、行为，是时候深入其中揭开现象的本质。
首先记住一个核心概念：JavaScript中[[Prototype]]的本质就是对象之间的关联关系，同时也是行为委托的基础。
不管是后续将介绍的类风格设计模式还是对象关联的委托设计模式，都离不开[[Prototype]]这个内置属性来建立链接。
首先我们从理论角度分析对比类设计和委托设计的区别，再从思维模型角度通过实际的案例进行分析。
【理论角度】
使用类设计来解决问题时，我们会定义一个通用的父类+多个子类。父类中定义一些通用的方法，子类会继承父类的这些通用方法并添加一些特殊的行为来处理对应任务。
如果用集合来意会的话，大概就是子类中包含父类的所有行为和方法。
class Common {
    age;
    Common(Age) {age=Age;}  //定义构造函数Common()
    introduce() {output(age);}
}
class Boy inherits Common {
    girlFriend;
    Boy(Age, GirlFriend) {  //定义构造函数Boy
        super(Age);
        girlFriend = GirlFriend;
    }
    introduce() {
        super();
        output(girlFriend);
    }
}
同时类设计中鼓励在继承时使用方法重写，形成多态。

而当使用委托设计时，定义一个对象+多个兄弟对象；这个对象包含所有任务都可以使用的行为；兄弟对象存储对应的数据和行为，兄弟对象通过委托使用这个对象中的行为。
如果用集合来意会的话，这个对象和兄弟对象之间是并列的关系，通过委托引用对象的行为。
Girl = {
    setAge: function(Age) {this.age = Age;},
    introduce: function() {console.log(this.age);}
}
Boy = Object.create(Girl);
Boy.defineSelf = function(Age, GirlFriend) {
    this.setAge(Age);
    this.girlFriend = GirlFriend;
}
Boy.introduceSelf = function() {
    this.introduce();
    console.log(girlFriend);
};
通常，有一些好的实践注意点：
A.在[[Prototype]]委托中最好把状态保存在委托者Boy上，而不是被委托者Girl上。
B.尽量避免在[[Prototype]]链的不同级别使用相同的命名，提倡更具描述性的方法名。比如在对象Boy中使用introduceSelf而不是同名的introduce。

当在Boy对象中引用defineSelf方法时，会调用this.setAge。先检测自身是否有该方法，发现Boy中没有该方法，因此会通过[[Prototype]]把请求委托给对象Girl。
对象关联风格的委托设计模式，不关注“构造”相关概念，不是通过new+构造函数实现的；而是直接使用Object.create创建关联的，避免了很多幺蛾子。

【思维模型】
面向对象的类设计模式，经常跟：构造函数、原型对象、new等概念打交道；它强调的是原型对象实体与原型对象实体之间的关系。
function Aoo(val) {
    this.num = val;
}
Aoo.prototype.add = function() {
    return "The result is " + this.num;
}
function Boo(val) {
    Aoo.call(this, val);
}
Boo.prototype = Object.create(Aoo.prototype);
Boo.prototype.check = function() {
    console.log(this.add());
}
var obj1 = new Boo(666);
var obj2 = new Boo(22);
obj1.check();  //The result is 666
obj2.check();  //The result is 22

对象关联的委托设计模式中操作的永远是对象，通过Object.create来创建对象，只关注对象之间的关联关系。
Aoo={
    init: function(val) {this.num = val;},
    add: function() {return "The result is " + this.num;}
};
Boo = Object.create(Aoo);
Boo.check = function() {console.log(this.add());};
var obj1 = Object.create(Boo);
obj1.init(666);
var obj2 = Object.create(Boo);
obj2.init(22);
obj1.check();  //The result is 666
obj2.check();  //The result is 22

这里在最后创建对象的时候，面向对象的类设计模式中直接使用new Boo(666)，而对象关联的委托设计中需要先创建Object.create然后再初始化init。需要执行两部，看上去比较麻烦，实际上更加灵活，更方便更好的支持关注分离原则。

我们在创建某个对象的时候，有时候需要检查实例的类型，就像第二部分原型中讲到的检查“类”关系。
面向对象的类模式中：会用到instanceof、.prototype、isPrototypeOf、函数等概念，比较难以理解。因为涉及到函数和对象的关系；
function Aoo{}{}
Aoo.prototype...
function Boo(){}
Boo.prototype = Object.create(Aoo.prototype);
var obj = new Boo();
使用instanceof、isPrototypeOf检查实体关系：
//检测Aoo和Boo的关系
Boo.prototype instanceof Aoo;  //true
Aoo.prototype.isPrototypeOf(Boo.prototype);  //true
//检测obj和Boo、Aoo的关系
obj instanceof Aoo;  //true
obj instanceof Boo； //true
Aoo.prototype.isPrototypeOf(obj);  //true
Boo.prototype.isPrototypeOf(obj);  //true

对象关联的委托模式：只涉及到对象和对象的关系，关系很清晰；只需要使用isPrototypeOf就可以判断关联关系。
var Aoo = {}
var Boo = Object.create(Aoo);
Boo...
var obj = Object.create(Boo);
只需要使用isPrototypeOf检查实体关系
//检测Aoo和Boo的关系
Aoo.isPrototypeOf(Boo);  //true
//检测obj和Boo、Aoo的关系
Aoo.isPrototypeOf(obj);  //true
Boo.isPrototypeOf(obj);  //true

整体上从理论、思维模型、实际应用以及判断势力关系角度来讲的话，在JavaScript中对象关联的行为委托模式都更具优势，可以让代码看起来更加简洁、更具扩展性；并且可以简化代码结构。随着加入ES6的语法，可以让对象关联的这种方法更加简洁。

4.class
在使用React开发项目的过程中，经常会用到ES6的class语法来继承React.Component，那这里再稍带介绍下class语法；
ES6中使用class语法糖可以改进JavaScript中模拟实现的面向对象的类设计模式，使代码看上去更加的简洁美观。
相比于传统的模拟类设计模式
//传统的
function Aoo(val) {this.num = val;}
Aoo.prototype.add = function() {return "The result is " + this.num;}
function Boo(val,total) {
    Aoo.call(this, val);
    this.total = total;
}
Boo.prototype = Object.create(Aoo.prototype);
Boo.prototype.add = function() {
    console.log(this.add());
    console.log("The total is " + this.total);
}
Boo.prototype.render = function() {}
//使用class语法糖后
class Aoo {
    constructor(val) {
        this.num = val;
    }
    add() {
        return "The result is " + this.num;
    }
}
class Boo extends Aoo {
    constructor(val,total) {
        super(val);
        this.total = total;
    }
    add() {
        super();
        console.log("The total is " + this.total);
    }
    render() {}
}
可以看到引入class语法糖之后带来的优势：
a.不再引用杂乱的.prototype
b.class声明“子类”时直接继承了“父类”，并且class字面语法不能声明属性（智能声明方法），帮助避免了可能的错误
c.通过extends很自然的扩展对象（子）类型
d.使super来实现相对多态
但同时使用class也会带来很多问题。
1.class令人混淆的语法
2.繁琐的.prototype
3.意外屏蔽
4.super语法
下面详细的解释下：
1.class令人混淆的语法
让人误认为class语法是向js中引入了一种新的“类”机制，其实class只是[[Prototype]]委托机制的语法糖； class是基于[[Prototype]]实时委托的，而不是像传统的面向类语言在声明时静态复制所有行为。
class Test {
    constructor() {this.num = Math.randow();}
    show() {console.log("The result is " + this.num);}
}
var obj1 = new Test();
obj1.show();  //The result is 0.2178787421...
Test.prototype.show = function() {
    console.log("The result is " + (this.num*100).toFixed(2));
}
var obj2 = new Test();
obj2.show();  //The result is 48.29
obj1.show();  //The result is 28.14      实时委托

2.繁琐的.prototype
class语法无法定义类成员属性（只能定义方法），如果确实要定义共享的状态属性，只能使用.prototype
class Test {
    constructor(){
        //确保修改的是共享状态，而不是在实例上创建屏蔽属性
        Test.prototype.num++;  
        //this.num可以通过委托实现想要的功能
        console.log("The result is " + this.num);
    }
}
Test.prototype.count = 0;
var obj1 = new Test();  //The result is 1
var obj2 = new Test();  //The result is 2
obj1.num == obj2.num;
可以实现功能，但是暴露了.prototype，违背了class语法本意。如果在class中使用this.num++，会在obj1和obj2上创建.count属性，而非更新共享状态。

3.意外屏蔽
class Test {
    constructor(input) {this.input = input;} 
    input() {console.log("Input is " + this.input)}
}
var obj = new Test('haha');
obj.input();  //TypeError，这时候obj.input是字符串

4.super语法
class中的super并不是动态绑定的，会在声明时“静态”绑定。将方法定义到不同对象中时，都必须重新绑定super
class A {
    output() {console.log("I'm A");}
}
class B extends A {
    output() {super();}
}
var obj = new B();
obj.output();  //I'm A
var D = {
    output: function(){console.log("I'm D");}
};
var E = {
    output: B.prototype.output
};
//将E委托到D
Object.setPrototypeOf(E, D);
E.output();  //I'm A

总的来说class加深了开发者对于JavaScript中“类”的误解，虽然带来了简介的语法，但产生了很多问题，让更加简洁、优雅的[[Prototype]]机制变得非常别扭。

本部分主要内容介绍了类的相关知识，以及JavaScript中模拟实现类机制的各种语法和行为。最后深入对比了JavaScript中面向对象的类设计模式和对象关联的委托设计模式的优劣势。
希望可以对你加深js的理解方面有所帮助。over！









