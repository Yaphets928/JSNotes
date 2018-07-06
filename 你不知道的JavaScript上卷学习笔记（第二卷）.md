# Ch1 关于this
## 为什么用this
可以在不同的上下文对象中重复使用函数，不需要针对每个对象定义不同的函数。  
不使用this就需要显示的给函数传参，传入一个上下文对象，this提供了一种优雅的方法来隐式传递一个对象的引用。  
this是在运行时绑定，不是编写时。  
当一个函数被调用时，会创建一个活动记录，即执行上下文，运行时上下文。  
调用位置：就似乎函数在代码中被调用的位置（而不是声明的位置），某些编程模式会隐藏真正的调用位置，  
要去分析调用栈，就是为了达到当前执行位置所调用的所有函数，我们关心的调用位置就在当前正在执行的函数的前一个调用中。  
# Ch2 this全解析
## this绑定四条规则
1. 默认绑定
>独立函数调用，非严格模式下，window。
2. 隐式绑定
```    
function foo(){
        console.log(this.a);
    }
var obj = {
    a:2,
    foo:foo
};
obj.foo();//2
```    
本例说明：
- 函数foo，无论是直接在obj中定义还是先定义再添加为引用属性，严格来说都不属于obj对象；
- 调用位置会使用obj上下文来引用函数，可以说是在函数被调用时obj对象“拥有”它
- 当函数引用有上下文对象时，隐式绑定规则会把函数调用中this绑定到这个上下文对象
- 这种对象属性链调用只在最后一层调用位置生效。


```
function foo(){
    console.log(this.a);
}
var obj = {
    a:2,
    foo:foo
};
var bar = obj.foo;//这一步很关键
var a = "global";
bar();//"global"
```
bar其实是给foo这个函数取了个别名，foo在内存中有自己的一块地方，bar指向了这块地方。bar()直接调用时，this指向了window。  
这称为隐式丢失。
被隐式绑定的函数会丢失绑定对象。
这里bar是obj.foo的一个引用，但实际上，它引用的是foo函数本身。因此此时bar()是一个不带任何修饰的函数调用。  
```
function foo(){
    console.log(this.a);
}
function doFoo(fn){
    //fn引用的其实是foo
    fn();//<--调用位置
}
var obj={
    a:2;
    foo:foo
};
var a = "global";
doFoo(obj.foo);//"golbal"
```
传入回调时更微妙了，参数的传递其实是一种隐式赋值，相当于fn=obj.foo，然后fn传给了doFoo();因此我们传入函数时也会被隐式赋值。！！！
**这很重要，多看几遍**！
把函数传入内置函数也是一样的效果
如：
```
setTimeout(obj.foo,100);
```
js中的setTimeout()函数实现和下面伪代码类似：
```
function setTimeout(fn,delay){
    //等待delay毫秒
    fn();
}
```
3. 显式绑定
call()方法和apply()方法
如果给call或apply传递字符串类型，布尔类型或者是数字类型来当作this绑定对象，这个原始值会被转换成他的对象形式，也就是new String(...),new Boolean(...),new Number(...)，这被称为装箱。  
显示绑定无法解决丢失绑定的问题
显示绑定的变种可以解决：
    - 硬绑定
    ```
    function foo(){
    console.log(this.a);
    }   
    var obj = {
      a:2
    };
    var bar = function(){
        foo.call(obj);
    };
    bar();//2
    setTimeout(bar,100);//2
    bar.call(window);//2
    //硬绑定下的bar无法再修改他的this值
    ```
    无论之后如何调用bar，它总会手动在obj上调用foo。
    这是一种显示的强制绑定。
    应用场景：
    包裹函数，负责接收参数并返回值
    ```
    function foo(somethimng){
        console.log(this.a,somenthon);
        return this.a+something;
    }
    var obj={
        a:2
    };
    var bar = function(){
        return foo.apply(obj,arguments);
    };
    //这里的bar就是一种包裹函数
    var b = bar(3);//2 3
    console.log(b);//5
    ```
    另一种应用场景：
    创建一个可以重复使用的辅助函数
    ```
    function foo(somethimng){
        console.log(this.a,somenthon);
        return this.a+something;
    }
    //简单的辅助绑定函数
    function bind(fn,obj){
        return function(){
            return fn.apply(obj,arguments);
        };
    }
    var obj={
        a:2
    ;}
    var bar = bind(foo,obj);
    var b = bar(3);
    console.log(b);//5
    ```
    这就是ES5方法：bind()
    - Api
    第三方库，以及js的许多新的内置函数，都提供了一个可选参数，“上下文”，像bind那样确保你的回调使用制定的this。
    ```
    [1,2,3].forEach(foo,obj);
    ```
    

4. new绑定
js中构造函数与其他面向对象语言不同，只是一些new操作符调用的普通函数而已，不属于某个类，也不会实例化一个类。  
包括内置对象在内的所有函数都可以用new调用，这种称为构造函数调用。  
这里有一点：实际上不存在构造函数，只有对函数的构造调用。  
**使用new时会执行以下操作：**
1. 创建一个全新的对象
2. 这个新对象会被执行prototype连接
3. 这个新对象会被绑定到函数调用的this
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象  
```
var bar = new foo(2);
```
所有使用new创建的对象，会在堆中开辟一个新空间存储。
使用new来调用foo(...)时，我们会构造一个新的对象，并绑定到foo中的this上。

## this优先级
1. new
函数是否在new中调用，如果是则绑定的是新创建的对象
2. 是否通过call或apply应绑定，如果是，则this指向指定对象
3. 函数是否在某个上下文对象中调用
4. 如果都不是就是默认绑定，严格模式下undefined，非严格模式下是全局对象

>bing()的功能之一就是可以把除了第一个参数（第一个参数用来绑定this）之外的所有参数传给下层函数  
这是柯里化的一种，称为部分应用。
```
function foo(p1,p2){
    this.val=p1+p2;
} 
var bar = foo.bind(null,"p1");
var baz = new bar("p2");
baz.val;//p1p2
```
>这里用null是因为我们并不关心硬绑定的this是谁，反正会被new修改。  
## 绑定例外
### 被忽略的this
如果把null或者undefined作为this的对象传入call，apply或者bind，这些值在调用时会被忽略，
实际应用的是默认绑定规则（就是严格undefined非严格全局）  
常见应用：
- 使用appply(...)来展开一个数组当作参数传入函数
```angular2html
function foo(a,b){
    console.log('a:'+a+' b:'+b);
  }
 foo.apply(null,[2,3]);
 //其实就是利用apply传参，apply接数组
```
- 使用bind进行柯里化
```angular2html
var bar = foo.bind(null,2);
bar(3);
//先用bind给foo传过去一个参数，
//这样在执行的时候只要传另一个参数就好了
```
>柯里化作用：延迟执行，参数服用，提前返回
>es6里可以用...arr展开数组，arr是数组，...arr是数组的元素

使用null来忽略this在函数确实使用了this时会产生副作用。  
但是可以创建一个DMZ对象（非军事区对象）——就是一个空的非委托对象，在忽略this绑定是总是传入  
那么对于this的使用都会被限制在这个空对象中，不会对全局对象产生任何影响  
在js中创建一个空对象的最简单的方法都是Object.create(null)  
这种放发不会创建prototype这个委托，所以比{}更空
```
Object.create(null);
{} 
    No properties
```
```
{}
    {}
    __proto__: 
        Objectconstructor: ƒ 
        Object()hasOwnProperty: ƒ 
        hasOwnProperty()

        ...
        
```
```
    function foo(a,b){
        console.log(...);
    }
    //创建DMZ空对象
    var o = Object.create(null);
    foo.apply(o,[2,3]);
    var bar = foo.bind(o,2);
```
### 间接引用
有可能有意或无意的创建于给函数的间接引用
最容易在赋值时发生！这也很重要 多看几遍记住并理解这个例子
```angular2html
function foo(){
    console.log(this.a);
}
var a = 2;
var o ={a:3,foo:foo};
var p = {a :4};
o.foo();//3
(p.foo=o.foo)();//2
```
赋值表达式p.foo=o.foo返回值是目标函数的引用，o.foo其实是一段地址 
指向了foo这个函数，赋值之后p.foo也指向了这个函数，然后直接调用了p.foo；
这里this就是默认绑定了。
### 软绑定
硬绑定可以在new以外的情况把this强制绑定到指定的对象上，  
防止函数应用默认绑定，但是这样会大大降低函数的灵活性，一旦绑定后就无法修改this。  
如果给默认绑定制定一个全局对象或undefined以外的值，就可以实现和硬绑定相同的效果，同时保留隐式绑定或者显示绑定修改this的能力。
可以通过一种被称为软绑定的方法来实现我们想要的效果：  
```angular2html
if（！Function.prototype.softBind){
    Function.protoype.softBind = function(obj){
        var fn = this;
        //捕获参数
        var curried = [].slice.call(arguments,1);
        var bound = function(){
            return fn.apply(
            (!this||this===(window||global))?obj:this.curried.concat.apply(curried,arguments)
            );
        }
        bound.prototype = Object.create(fn.prototype);
        return bound;
    };
}
```
除了软绑定之外，softBind(...)的其他原理和ES5内置的bind(...)类似。  
它还会对指定的函数进行封装
### 箭头函数的this词法
箭头函数不使用this的四种规则，而是根据外层（函数或者全局）作用域来决定this。  
箭头函数的绑定无法被修改 new也不行。  
```angular2html
function foo(){
    //返回一个箭头函数
    return(a) => {
        console.log(this.a);
    };
}
var obj1 = {
a:1
};
var obj2= {
a:2
};
var bar = foo.call(obj1);
bar.call(obj2);//输出1！！！
```
这里foo()内部创建的箭头函数会捕获调用时foo()的this。  
由于foo()的this绑定到了obj1，bar(引用箭头函数)的this也会绑定到obj1.  
箭头函数的绑定无法被修改！
箭头函数常被用于回调函数中
```
function foo(){
    setTimeout(() => {
        //这里的this继承自foo
        console.log(this.a);
    },1000)
}
var obj={a:2};
foo.call(obj);//2
```
>题外话，有没有理解为什么用call呢？
>前面有个obj里有foo:foo，所以可以直接obj.foo，这里obj里没有foo,所以就要foo.call(obj)  

ES之前需要显式的去把this的值赋值给一个变量。  
var self = this;然后再用setTimeout函数  
# Ch3对象
## 语法
两种定义形式：声明（文字）形式和构造形式。
字面量表示：
```
var myObj={
    key:value
};
```
构造形式：
```
var myObj = new Object();
myObj.key = value;
```
## 类型
基础类型：null,undefined,string,number,boolean,symbol  
引用类型：object
对象有很多子类型，我们称为复杂基本类型，比如函数，数组。  
函数是一个可调用的对象，本质上和对象是一样的，所以可以像操作对象一样操作函数。  
内置对象：String,Number,Boolean,Object,Function,Array,Date,ReaExp,Error.  
内置对象类似于java中的类class，String类等。  
但是在js中，他们只是一些内置函数，他们可以当作构造函数使用。  
```
var strPrimitive = "I am a string";
typeof strPrimitive;//"string"
strPrimitive instanceOf String;//false

var strObject = new String("I am a string");
typeof strObject;//"object"
strObject instanceOf String;//true

Object.prototype.toString.call(strPrimitive);//[object String]
Object.prototype.toString.call(strObject);//[object String]
```
可以看出，元是指strPrimitive并不是一个对象，只是一个字面量，并且是一个不可变的值。  
如果要在这个字面量上执行一些操作，比如获取长度，访问其中的某个字符，就需要把它转换为String对象。  
必要时语言会自动转换，不需要显式的创建一个对象。  
js社区中大部分人 认为能使用文字形式时就不要使用构造形式。
null和undefined没有构造形式，只有文字形式，Date只有构造，没有文字形式。
Error很少显示创建，一般是抛出异常时自动创建。  
### 内容
对象的内容是由一歇存储在特定命名位置的值组成的。
内容存在对象内部只是表现形式，存储在对象容器内部的是这些属性的名称，他们像指针，指向真正的存储位置。  
访问内容可以.操作符，也可以[]操作符，可以互换。  
.操作符要求属性的名字满足标识符的命名规范，而[]可接收任意UTF-8/Unicode字符串作为属性名。  
比如myObj["Super-fun"].  
也可以a="Super-fun",然后obj[a];  
对象中属性名永远都是字符串，即使是数字也不例外，虽然在数组下表中使用的是数组，  
但是在对象属性名中数字会被转换成字符串，所以不要搞混对象和数组中的用法。  
1. 可计算属性名
```angular2html
myObj[prefix+"bar"]
```
常用场景是ES6符号Symbol，Symbol包含一个不透明且无法预测的值， 
一般来说不会用到符号的实际值（因为理论上来说在不同的js引擎中值是不同的）  
所以通常接触到的是符号的名称，比如Symbol.something
```angular2html
var myObj = {
    [Symbol.Something]:"hello Yaphets";
}
```
2. 属性与方法
对象属性是个函数，通常被称为方法。  
但是js中，函数永远不会“属于”一个对象，所以把对象内部引用的函数称为方法有些不妥。  
即使你在对象的文字形式中声明一个函数表达式，这个函数也不会属于这对象，他们只是对于同一个函数对象的多个引用。  
```angular2html
var myObj={
    foo:function(){
        console.log("foo");
    }
};
var someFoo = myObj.foo;
//个人理解：这个匿名函数对象在堆中有一块内存存储着，myObj.foo存着的不是这个函数，而是这个函数的地址
//在赋值给someFoo之后，其实是把地址赋给了someFoo，这样someFoo中存着的就也是这个地址
```

3. 数组
数组支持[]访问，有更结构化的值存储机制。期望的是数值下标，即索引。  
数组也是对象，可以直接添加属性：
```angular2html
var arr =[1,2,3];
arr.baz="baz";//[1,2,3,bar:"baz"]
arr.length;//3
arr.baz;//"baz"
```
可以看到虽然添加了属性（无论是.语法还是[]语法），但是length没有变化。  
最好只用对象来存储键/值对，只用数组来存储数值下标/值对。  
如果给数组添加的属性名是"3"这样的类似数字，会变成数字，因此会修改数组内容而不是仅像上面那样添加了一个属性。  
4. 复制对象
浅拷贝与深拷贝
深拷贝的JSON法：
```angular2html
var newObj = JSON.parse(JSON.stringify(somObj));
```
ES6的浅拷贝方法：

```angular2html
Object.assign();
var newObj = Object.assign({},myObj)
//第一个参数是目标对象，之后还可以跟一个或多个源对象
//会遍历一个或多个源对象的所有可枚举的自有键，并复制（使用=赋值操作）到对象。  
//因为使用=赋值操作，源对象属性的一些特性（writable）不会复制到目标对象。
```

5. 属性描述符
ES5开始所有属性有了属性描述符。
创建普通属性时，属性描述符会使用默认值，我们也可以用Object.defineProperty()来添加要给属性或修改一个已有属性。
修改需要这个属性是可修改的（configurable）。

```
var myObject = {};
Object.defineProperty(myObject,"a",{
    value:2,
    writable:true,
    configurable:true,
    enumerable:true
});
```

一般这种方式只用来修改属性描述符。  
writable：是否可修改value，为false时修改值会静默失败，silently failed。严格模式下出错，typeError。  
configurable：属性是否是可配置的。修改为false单向操作，不可撤销。为false时不可修改不可删除。  
enumerable:控制属性是否出现在对象的属性枚举中。比如说for...in循环。  

6. 不变性
希望属性是不可改变的  
所有的方法创建的都是浅不变性，就是说，他们只会影响目标对象和他的直接属性。如果目标对象是引用了其他对象，其他对象的内容不受影响，
ES5中几种方法实现不变性：
    - 对象常量
    结合writable:false和configurable：false就可以创建一个真正的常量属性，不可修改重定义或删除。
    - 禁止扩展
    若要禁止一个对象添加新属性，并且保留已有属性，用Object.preventExtensions(...)
    ```angular2html
    var myObj = {a:2};
    Object.preventExtensions(myObj);
    ```
    - 密封
    Object.seal(...)会创建一个密封的对象，实际上是调用了Object.preventExtensions并把所有现有属性标记为Configurable：false。  
    密封之后不能添加新属性，不能重新配置或删除现有属性，但是可以修改值。
    - 冻结
    Object.freeze(...)会冻结一个对象，实际上调用Object.seal(...)并把属性的writable:false.  
    这是应用在对象上的级别最高的不可变形，会禁止对于对象本身及其任意直接属性的修改。
    - 总结：
    禁止扩展：Object.preventExtensions(..)不能加属性
    密封：Object.seal(..)不能加，不能删，不能配置，可以修改值。
    冻结：不能加，不能删，不能配置，不能修改值

7. [[Get]]
myObj.a是一次属性访问，语言规范中，在myObj上是实现了[[Get]]操作，  
对象默认的内置[[Get]]操作首先在对象中查找是否有名称相同的属性，如果找到就会返回这个属性的值；  
如果没有找到名称相同的属性，会遍历原型链。  
如果无论如何都没找到，返回undefined。  
8. [[Put]]
如果已存在该属性，[[Put]]会：
- 属性是否是访问描述符?如果是并且存在setter就调用setter
- 属性的数据描述符中writable是否是false，如果是，非严格模式下静默失败，严格模式抛出TypeError  
- 如果都不是，将该值设置为属性的值  
如果对象中不存在这个属性，[[Put]]操作更复杂，原型链中将讲解。  
9. Getter和Setter
ES5中使用getter和setter部分改写默认操作，但是只能用在单个属性上。  
当你给一个属性定义getter，setter或者两者都有时，这个属性会被定义为“访问描述符”（和数据描述符相对）
对于访问描述符，js会忽略他们的value和writable特性，取而代之的是关心set和get和configurable和enumerable特性。  
```angular2html
var myObj ={
    get a(){
        return 2;
    }    
};
Object.defineProperty(
    myObj,//目标对象
    "b",//属性名
{  //描述符
    //给b设置一个getter
    get:function(){
        return this.a*2
    }
    enumerable:true
});
```
get a()或者defineProperty()都会在对象中创建于给不包含值的属性，对于这个属性的访问会调用一个隐藏函数，返回值会被当作属性的返回值。
只定义getter，所以对b的值进行set操作会被忽略，不会抛出错误。
所以还应该给一个set操作
```angular2html
var myObj ={
    get a(){
        return this._a_;
    }  
    set a(val){
                this._a_ = val*2;
            }  
};
myObj.a = 2;
myObj.a;//4
```
_a_只是一种名称惯例，没有特殊行为。

10. 存在性
返回值可能是undefined，但是也可能是赋值的undefined
如何区分存在性呢
```angular2html
var myObj = {a:2};
("a" in myObj);//true
("b" in myObj);//false

myObj.hasOwnProperty("a");//true
myObj.hasOwnProperty("b");//false
```
>in操作符检查属性是否在对象及原型链中,检查的是某个属性名是否存在；  
hasOwnProperty只检查是否在该对象中；  
所有对象都可以通过对于Object.prototype的委托来访问hasOwnProperty()，  
但是有的对象可能没有连接到Object.prototype(通过Object.create(null)创建的)。  
所以可以用更强硬的一种方法：Object.prototype.hasOwnProperty.call(myObject,"a"). 

in操作符检查的是属性名!是否存在，对于数组来说这很重要。  
检查的是索引值。  
##### 枚举
不可枚举的值（enumerable：false）可以判断到存在，但是在for item in myObj这样的循环中不会出现。  
- myObj.propertyIsEnumerable("a")判断a属性是否能枚举，能是true，不能是false。  
检查属性名是否**直接**存在于对象中并且满足enumerable:true。  
- Object.keys(myObj)会返回一个数组，包含所有可枚举属性。
- Object.getOwnPropertyNames(myObj)会返回一个数组，包含所有属性，无论是否可枚举
- in和hasOwnProperty()区别在于是否查找原型链。  
- Object.keys(myObj)和Object.getOwnPropertyNames(myObj)都只查找对象本身。  


##遍历
for..in..循环遍历对象的可枚举属性列表，包括原型链。
如何遍历属性的值：
数组：for循环，forEach(..),every(..),some(..).  
forEach会遍历数组中所有值并忽略掉回调函数的返回值  
every会一直运行到回调函数返回false
some会一直运行到回调函数返回true  
every和some中特殊的返回值和普通for循环中的break语句类似，会提前终止遍历。  

Es6新增遍历数组的for..of语法

```
var arr = [1,2,3];
for(var v of arr){
    console.log(v)
}
//1
//2
//3
```

for..of循环会先向被访问对象请求一个迭代器对象，然后通过调用迭代器对象的next()方法来遍历所有返回值。  
  
数组有内置的@@iterator，因此for..of可以直接应用在数组上，调用内置的@@iterator是下面这样的：

```
var arr = [1,2,3];
var it = arr[Symbol.iterator]();
it.next();//{value:1,done:false}
it.next();//{value:2,done:false}
it.next();//{value:3,done:false}
it.next();//{done:true}
```

@@iterator不是对象，是一个返回迭代器对象的函数  
调用迭代器的next()方法会返回形式为{valueL..,done:..}的值，value是当前的遍历值，done是一个布尔值，表示是否还有可以遍历的值。  


普通对象没有内置@@iterator，无法自动完成for..of
可以自己定义一个函数模仿@@iterator  

# CH4 混合对象“类”
实例化，继承，多态
## 类理论
类/继承描述了一种代码的组织结构形式，一种在软件中对真实世界问题的建模方法。  
类，继承，实例化。  
类的另一个核心概念是堕胎，弗雷德通用行为可以被字类用更特殊的行为重写。
类理论强烈建议父类和子类使用相同的方法名来表示特定的行为，从而让子类重写父类。  
## 类的机制
### 建造
许多语言标准库会提供Stack类，“栈”数据结构，支持压入弹出等操作，Stack类内部会有一些变量来存储数据，同时提供一些共有的可访问方法，  
从而让代码可以和隐藏的数据进行交互。  
但是在这些语言中，也并不是直接操作Stack。  
Stack类仅仅是一个抽象的表示，它描述了所有“栈”需要做的事，但是它本身并不是一个“栈”。它必须先实例化Stack类然后再操作。  
“类”和“实例”的概念来源于房屋建造。
建筑师会规划出一个建筑的所有特性：高度，宽度，多少个窗户，窗户位置，这个阶段不需要关心建筑会被建在哪，也不需要关心建多少个。  
建筑蓝图只是建筑计划，不是真正的建筑，还需要一个工人来建造。  
建筑工人按照蓝图建造建筑，他会把规划好的特性从蓝图中复制到现实世界的建筑中。  
完成后，建筑就成为了蓝图的物理实例，本质上就是对蓝图的赋值。  
之后建筑工人就可以到下一个地方，把所有的工作都重复一遍，再创建一份副本。  
一个类就是一个蓝图。为了获得真正可以交互的对象，我们必须按照类来建造（也可以说实例化）。  
### 构造函数
类实例是由一个特殊的类方法构造的，方法名通常会类名相同，被称为构造函数。  
这个方法的任务就是初始化实例需要的所有信息。  
```angular2html
class CoolGuy{
    specialTrick = nothing;
    CoolGuy(trick){
        specialTrick = trick
    }
}
Joe = new CoolGuy("jumping rope");
```
类构造函数属于类，而且通常和类同名。此外，构造函数大多需要new来调，这样语言引擎才知道你想要构造一个新的类实例。

##类的继承
##类的多态

# Ch5 原型
##[[prototype]]
对象都有一个属性[[prototype]]。  
其实就是对于其他对象的引用。
[[prototype]]也可以为空。
当你试图引用对象的一个属性时，会触发[[get]]操作，比如myObj.a。
对于默认的get操作来说，第一步是检查对象本身是否有这个属性，有就用它；但是如果没有，就需要使用原型链了。

```
var a = Object.create(b);
```
上面这段代码会创建一个新的对象a，并把他的[[prototype]]指向b。
### Object.prototype
尽头在哪？所有普通的原型链最终都会Object.prototype。
由于所有的普通对象都源于这个对象，所以Object.prototyoe中包含了许多通用功能，如.toString()和.valueOf(),.hasOenProperty(..),.isPrototypeOf(..)等。
### 属性设置和屏蔽
给对象设置属性不简单是添加一个新属性或者说修改已有属性：
myObj.foo = "bar";
如果myObj中包含foo的普通数据访问属性，则只修改已有属性；
如果foo不是直接存在于myObj中，[[prototype]]链就会被遍历，原型链上找不到foo，就会把foo添加到myObj上；
然而，如果原型链上层存在foo：
1. 如果在上层存在名为foo的普通数据访问属性，并且没有被标为只读（writable：false），就是可以改动，就会在myObj上添加一个foo新属性，它是屏蔽属性。
2. 如果在上层存在foo，但是标记为只读，即不可写，那么无法修改已有属性或者在myObj上创建屏蔽属性，在严格模式下报错，普通模式下会被忽略，不会发生屏蔽。
3. 如果在桑曾存在foo，并且他是个setter，就一定会调用这个setter，foo不会欸添加到myObj，也不会重新定义foo这个setter。
如果foo既在myObj上又在原型链上，那么就会屏蔽上层的，总是选择最底层的。

>如果你希望在上面的2，3步骤也屏蔽foo，就不能用=，而使用OBject.defineProperty(..)来添加foo属性。
情况2有些意外，只读属性会阻止prototype链下层隐式创建（屏蔽）同名属性。这样做主要是为了模拟类属性的继承.
可以把原型链上层的foo看作是父类中的属性，会被myObj继承（复制），这样一来myObj中的foo属性也是只读，所以无法被创建。
但是要注意的是！实际上不会发生这样的继承复制。
这看起来很奇怪，myObj竟然因为其他对象中有一个只读foo就不能包含foo属性，更奇怪的是只限制了=，而没有限制defineProperty()。

## 类
js中类无法描述对象的行为，因为其实根本就不存在类，对象直接定义自己的行为。
！！！js中只有对象！！！
### “类”函数
js中一种行为称为模仿类，类似类，利用函数的一个特殊的特性：所有的函数都会有一个名为prototype的公有且不可枚举的属性指向另一个对象。
这个对象称为原型。
这个对象就是在调用New Foo()时创建的，最后会被关联到Foo.prototype这个对象上。
Foo.prototype也会有一个公有并且不可枚举的属性.constructor，这个属性引用的是对象关联的函数。
js惯例类名大写。 
### 构造函数还是调用
函数本身并不是构造函数，当你在普通函数前面加上new时，这个函数调用变成一个构造函数调用。new会劫持所有普通函数并用构造对象的形式来调用它。
函数不是构造函数，但是当且仅当使用new时，函数调用会变成构造函数调用。  
### new同一个构造函数时
```
function Foo(name) {
    this.name = name'
}
Foo.prototype.maName = function() {
    return this.name;
};
var a = new Foo("a");
var b = new Foo("b");
a.myName();//a
b.myName();//b
```
this.name = name给每个对象都添加了.name属性。
Foo.prototypoe.myName给每个对象添加了一个属性（函数）。  
看起来创建a和b的时候很把Foo.prototype对象复制到这两个对象中，然而事实不是这样的。  
创建的过程中，a和b的内部[[prototype]]都会关联到Foo.prototype上，当a和b无法找到myName时，它会通过委托在Foo.prototype上找到。

a.constructor === Foo是true，但这不意味着a有一个指向Foo的属性！constructor引用同样被委托给了Foo.prototype，而Foo.prototype.constructor默认指向Foo。   
Foo.prototype的constructor属性只是Foo函数在声明时的默认属性，如果创建一个新对象替换了函数默认的prototype引用，那么新对象不会自动获得constructor属性。  
```
function Foo(){}
Foo.prototype = {};//创建一个新的原型对象
var a1 = new Foo();
a1.constructor === Foo;//false
a1.constructor === Object;//true
```
a1并没有constructor属性，所以它会委托[[prototype]]链上的Foo.prototype，但是这个对象给也没有.constructor属性（默认的会有），所以会继续委托，给顶端的Object.prototype。
当然我们也可以给Foo.prototype手动添加一个.constructor属性。
```
function Foo(){}
Foo.prototype = {};//创建一个新的原型对象

//需要在Foo.prototype上修复丢失的constructor属性
//新对象属性起到Foo.prtotype的作用
Object.defineProperty(Foo.prototype,"constructor", {
    enumerable: false,
    writable: true,
    configurable: true,
    value: Foo//让.constructor指向Foo.
})
```
修复constructor需要很多手动操作，这些工作都源于把constructor错误的理解为由...构造。
所以记住这一点：
***constructor并不表示被构造***
constructor可变不可枚举

### 原型继承
Object.create(Foo.prototype)这句话调用时会凭空创建一个新对象并把新对象内部的prototype关联到你指定的对象，这里就是Foo.prototype。
我们创建一个函数。比如function Bar(){}，和其他函数一样，Bar会iyouyige.prototype关联到了默认的对象，但是这个对象不是我们想要的，于是我们创建于一个新对象把它关联到我们希望的对象上。
下面两种做法不可取！！！
```
Bar.prototype = Foo.prototype;
Bar.prototype = new Foo();
```
第一个，并不会创建一个关联到Bar.prototype的新对象，只是让Bar.prototype直接引用Foo.prototype对象本身。显然这不是想要的结果，否则你根本不需要Bar对象，直接用Foo就可以了。
第二个，的确会创建一个关联到Bar.prototype的新对象，但是它使用了Foo的构造函数调用，如果Foo有一些副作用，比如写日志，修改状态，注册到其他对象，给this添加数据属性等，就会影响到Bar的后代。
因此，要创建一个合适的关联对象，必须使用Object.create()。这样做唯一的缺点就是需要创建一个新对象并且把就对象抛弃掉，不能修改已有的默认对象。
ES5之前只能通过设置.__proto__属性实现修改对象的[[prototype]]，ES6添加了辅助函数Object.setPrototypeOf(..)
```
//ES6之前需要抛弃默认的Bar.prototype
Bar.prototype = Object.create(Foo.ptototype);

//ES6开始可以直接修改现有的Bar.prorotype
Object.setPrototypeOf(Bar.prototype, Foo.prototype);
```
不过如果忽略掉Object.create()方法带来的轻微性能损失（抛弃的对象需要进行垃圾回收），它实际上比es6方法更短可读性更高。
#### 检查类关系
检查对象a委托的对象，传统方法是检查一个实例的继承祖先，被称为内省或反射。
```
function Foo() {

}
Foo.prototype.blah = ..;
var a = new Foo();
```
1. a instanceof Foo;//true
左边是一个普通对象，右边是一个函数，在a的整条prototype链中是否有指向Foo.prototype的对象。
instanceof方法只能处理对象和函数之间的关吸，如果你想判断对象，不能实现。
> bind方法生成硬绑定函数的话，是么有.prototype属性的，这时使用instanceof的话，目标函数的.prototype会代替硬绑定函数的.prototype。  
通常我们不会在构造函数调用中使用硬绑定函数，如果这么做相当于直接调用目标函数。  
同理，在硬绑定函数上使用instanceof也相当于直接在目标函数上使用instanceof
2.  Foo.prototype.isPrototypeOf(a); //true
意思是在a的整条prototype链中是否出现过Foo.prototype。
这个方法直接是两个对象之间的关系。
我们要获取一个对象的prototype链：
ES5中，标准的方法是Object.getPrototypeOf(a);
大多数！（不是所有）也支持一种方法a.__proto__
__proto__在es6前不是标准，引用了protytype对象。
和constructor属性一样，__proto__并不存在于你正在使用的对象中，他和toString()等方法一样，存在于内置的Object.prototype中。
此外，.__proto__更像是一个setter/getter。
```
Object.defineProperty(Object.portotype,"__proto__",{
    get: function() {
        return Object.getPrototypeOf(this);
    },
    set: function(o) {
        Object.setPrototypeOf(this, o);
        return o;
    }
});
```
访问a.__proto__时，调用了getter函数。虽然getter函数存在于Object.prototype对象中，但是他的this指向对象a。

### 对象关联
#### 创建关联Object.create()
Object.create()创建一个新对象，并把它关联到我们指定的对象上，这样就可以充分发挥[[prototype]]的威力，并且避免不要的麻烦（比如new的构造函数调用会生成.prototype和.constructor引用）。
>Object.create(null)会创建一个用有空[[prototype]]链接的对象，这个对象无法进行委托，由于这个对象没有原型链，所以instanceof操作符无法进行判断，总是会返回false。
这些特殊的空[[prototype]]对象被称为字典，他们不会受到原型链的干扰，因此很适合用来存储数据。

我们不需要类来创建两个对象之间的关系，只需要通过委托。create()不包括任何“类的诡计”，所以可以完美的创建我们想要的关联关系。


# Ch6 委托

上一章结论：[[prototypr]]链就是指对象中的一个内部链接引用了另一个对象。
换句话说，js中的这个机制就是**对象之间的关联关系**

##面向委托的设计
这和面向类有些不同。
类设计模式鼓励你在继承时使用方法重写和多态。现在可以实例化子类的一些副本然后是这些执行任务。构造完成后你通常只需要操作这些实例（不是类）。

委托行为呢，首先定义一个Task对象（不是类也不是函数）。Task包含了所有任务都可以使用的具体行为，接着对每个任务都会定义一个对象来存储对应的数据和行为。你会把特定的任务对象都关联到Task功能对象上，让他们在需要的时候进行委托。

```
XYZ = Object.create(Task); //委托
```
XYZ和Task都是对象。XYZ通过create创建，他的[[prototypr]]委托了Task对象。
[[prototype]]委托中最好把状态保存在委托者而不是委托目标（Task）上。
类设计模式中，会让父类和子类有同名的方法，就可以利用重写或多态的优势。而在委托行为中，尽量避免相同的命名。
在XYZ上运行Task的一个方法时，现在XYZ自身查找是否有这个方法名单是XYZ没有，会通过原型链找到Task继续寻找。此时还会出发this的隐式绑定规则，这个方法里的this运行时仍然绑定到XYZ。

在委托设计模式中，对象不是按照父类到子类的关系垂直组织的，而是通过任意方向的委托关联并排组织的。

互相委托行为十进制的，无法在两个或两个以上互相委托的对象之间简历循环委托。如果引用了一个都不存在的属性后者方法，那就会产生一个无限递归循环。

#### 比较类和委托两种设计模式
类设计模式，子类Bar继承父类F哦哦，然后生成了b1和b2两个实例，b1委托了Bar.prototype,Bar.prototype委托了Foo.prototype。
委托设计模式简洁很多：
```
Bar = Object.create(Foo);
var b1 = Object.create(Bar);
```
只是把对象关联起来，并不需要复杂的模仿类的行为。