# Ch1 关于this
## 为什么用this
可以在不同的上下文对象中重复使用函数，不需要针对每个对象定义不同的函数。  
不使用this就需要显示的给函数传参，传入一个上下文对象，this提供了一种优雅的方法来隐式传递一个对象的引用。  
this是在运行时绑定，不是编写时。  
当一个函数被调用时，会创建一个活动记录，即执行上下文，运行时上下文。  
调用位置：就似乎函数在代码中被调用的位置（而不是声明的位置），某些编程模式会隐藏真正的调用位置，  
要去分析调用栈，就是为了达到当前执行位置所调用的所有函数，我们关心的调用位置就在当前正在执行的函数的前一个调用中。  

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
obj.foo();
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
