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
所有使用
