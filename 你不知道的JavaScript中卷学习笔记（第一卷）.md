## CH1 类型
基本类型：null,undefined,boolean,number,string,symbol
引用类型：object

ES5中规定，所有的值都有一个对应的语言类型。
语言引擎对42和'42'的处理方式不同，就说明是不同的类型。讨论js有没有类型，主要是强制类型转换的相关问题。

6+1内置类型！

#### typeof
typeof总是返回一个字符串
typeof typeof 42；//string
```
typeof undefined === 'undefined';
typeof true === 'boolean';
typeof 42 === 'number';
typeof '42' === 'string';
typeof {life: 42} === 'object';
typeof Symbol() === 'symbol';

typeof null === 'object';
//历史遗留问题，修复会产生很多bug。
var a = null;
(!a && typeof a === "object"); //true

typeof function a(){} === 'function';
//具体来说，函数是object的一个子类型。是一个可调用的对象，有一个内部属性[[call]]，该属性使其可以被调用
//函数的对象，a.length是其声明的参数的个数

typeof [1,2,3] === 'object';
//数组也是一个对象，是object的一个子类型，
```
null是假值，falsy或falsse-like

#### 值和类型
js中变量没有类型。
值才有类型！
变量可以随时持有任何类型的值。换个角度讲，就是js不做类型强制，语言引擎不要求变量总是持有其初始值同类型的值。

变量未持有值--undefined
大多数开发者（其它语言的）倾向于将undefined等同于undeclared，**但是**js中是两回事
undefined：已声明但未赋值；  
undeclared：未声明的。
关键是：
```
var a;
typeof a; 'undefined'
typeof b; 'undefined'
```
对于undeclared的变量typeof照样返回undeifned。虽然b是一个undeclared变量，但是没有报错，因为typeof有一个特殊的安全防范措施。
如果没有这个防范措施：
```
var a;
a; //undefined
b; //ReferenceError: b is not defined
```
这里要注意浏览器挖的保险套，应该是b is not declared，而不是现在这样会造成b是要给undefined的错觉

全局命名空间不应该有变量，所有的东西都应该封装到模块和私有的命名空间。



## CH4 强制类型转换
#### 值类型转换
- 类型转换 type casting 显示的
- 强制类型转换 coercion 隐式

js中统称为强制类型转换，可以用隐式强制类型转换和显示强制类型转换。
```
var a = 42;
var b = a + '';//隐式
var c = String(a)；//显示
```

#### 抽象值转换 字符串，数字和布尔的转换
##### toString
负责处理非字符串-->字符串
*基本类型*里： null转化为'null',undefined转换为'undefined',true转换为'true'
*数字*遵循通用规则，极小极大的数用指数形式
*普通对象*来说，除非自行定义，否则toString()(Object.prototype.toString())返回内部属性[[class]]的值。如[object object]。
- class属性可以看作一个内部分类，无法直接访问，可以用Object.prototype.toString(..)来查看
- 如Object.prototype.toString.call([1,2,3]);//'[object Array]'
- 如ObObject.prototype.toString.call(/reget-literal/i);//'[object RegExp]'
- null是Null,Undefined是Undefined， 'abc'是String，42是Number，true是Boolean

*数组*会把所有单元字符串化后，再用,连接起来 '1,2,3'，toString()可以显式调用，也可以在需要的时候自动转。
*JSON* JSON.stringify()在将JSON对象序列化为字符串时也用到了ToString()。
- 安全的JSON值JSON-safe都可以用stringify
- 不安全的JSON值是指undefined,function,symbol回合包含循环引用的对象（对象互相引用形成无限循环）。不符合JSON结构标准，其他支持JSON的语言无法处理他们。stringify遇到时会忽略，在数组中会返回null以保证其他位置不受影响。对包含循环引用的stringify会出错哦。
- 对象中定义了toJSON()方法会首先调用这个方法，用它的返回值来序列化。
- 要对不安全的JSON值stringify可以定义toJSON()
- toJSON()返回的是一个适当的值，可以是任何类型，然后JSON.stringify()对其进行字符化
- 