把javascript以前记的笔记放在这里~~
=======================

#My notes (Savvy Javascript)

### Source code module in this project.

##### Text version notes here, some guides, tips, etc.
`this` has so many different objects!
```notes
Javascript object notation  JSON	

JSON initialize  and create an object
var o = {};
          surrounded by {};
var speaker = {text: “hello world”, say: function() {alert(this.text)}}
JSON是javascript最好的序列化形式，
compare to XML,it is much more simple as well as release more space.

In Javascript, we use “new” operator combining with a function form to create object. 
For example: 
function Myfunc(){}; //define a void function;
var anObj1 = new MyFunc(); // use “new” operator assisted by MyFun Function,
then create an object.
var anObj2 = new Myfunc; // function can also be called without brackets
```
** __*NOTICE: Javascript is case sensitive.*__ **

DATA type Javascript  9/14/2015
## Source code JSBibleS2 module.

```shell
undefined: unknown stuffs, code cannot define them. 
*Att: typeof(undefined)   return  undefined

null: void, only has the concept, code can process.
*Att: typrof(null) return object, but `null` is not object, 
the variate which has null value actually is not object.

boolean: yes or no, absolutely explicit. Can control code process.

number: linear type, size and order cascade clearly, easy for code to do batch, 
also can control the cycling(loop) and iteration.
*Att: typeof(NaN) and typeof(Infinity) are both return number.
NaN != NaN    Infinity / Infinity =  NaN
string: object to human beings, it is logical concept, not the mechanism signal.
Human computer interaction, code processing and attempts understanding , 
information communicating will all rely on “String”.


**Attention: undefined, null, “ ”, 0  logical value is “false”. 
excluding those 5 values(contain false itself), 
these left behind values are all “true”. 
It is interesting that they are not equal to any other except undefined == null. 
Follow the regulations, 
we can easily write more precise and simpler logical judge sentence s. 
“123” == 123  return true,  watch out!!  “0123” == 0123. return false.

“123” === 123 return false!!   "===“ means identically equal (congruent),
which means both data value and type should be exactly the same.

“==“ partially equal  “===“ identically equal,   they are totally different!!!
“!==”  logical equvalent ot    A != B || typeof(A) != typeof(B)

Objectification: The ability of letting data and code organise into complex structure.  
In JavaScript, only object and function type provide the capability of objectification.

 Object is class, In general, object- oriented ======> class. 
 However, actually there is no concept of class in JS.
```

## Source code JSBibleS3 module.
Javascript processes session by session. 9/15/2015

```shell
Time and space of codes
只要你记住一点：JavaScript里的代码也是一种数据，
同样可以被任意赋值和修改的，而它的值就是代码的逻辑。
只是，与一般数据不同的是，函数是可以被调用执行的。
不过，如果JavaScript函数仅仅只有这点道行的话，这与C++的函数指针，
DELPHI的方法指针，C#的委托相比，又有啥稀奇嘛！
然而，JavaScript函数的神奇之处还体现在另外两个方面：
一是函数function类型本身也具有对象化的能力，
二是函数function与对象 object超然的结合能力。

当我们写下：var myName＝“leadzen”；
就是定义了window作用域的一个变量 variable myName，
然而当我们写下：myName＝“leadzen”；
就是定义window object的一个属性attribute myName。
JS查找标识符时，除非特别指明对象，
否则会沿着作用域链寻找这一表示符的属性或变量，
甚至会自动创建该标识符。
javascript这一点很灵活，但容易在不知情情况下造成bug，难排查

Solution：要加var，避免重名……
Javascript 提供的调用上下文信息有几个
函数本身
函数的caller属性
this关键字和arguments 隐含对象。
灵活运用caller属性和自身标识，可以简化代码或写一些通用的处理代码。
许多优秀的AJAX框架也都充分利用了JS这些特性。
```


## Chapter 5 Amazing Object

>Javascript中有object和function两种东西有对象化的能力。
>任何一个函数都可以为其动态地添加或去除属性，这些属性可以是简单类型，
>可以是对象，也可以是其他函数。也就是说，函数具有对象的全部特征，
>你完全可以把函数当对象来用。其实，函数就是对象，只不过比一般的对象
>多了一个括号“()”操作符，这个操作符用来执行函数的逻辑。

>object and function: both have properties and methods
>对象和函数可以象数组一样，用属性名或方法名作为下标来访问并处理。
>那么，它到底应该算是数组呢，还是算对象？
>我们知道，数组应该算是线性数据结构，线性数据结构一般有一定的规律，
>适合进行统一的批量迭代操作等，有点像波。而对象是离散数据结构，
>适合描述分散的和个性化的东西，有点像粒子。因此，我们也可以这样问：
>JavaScript里的对象到底是波还是粒子？
>如果存在对象量子论，那么答案一定是：波粒二象性！



```javascript
var anObject = {};  //一个对象
      anObject.aProperty = "Property of object";  //对象的一个属性
      anObject.aMethod = function(){alert("Method of object")}; //对象的一个方法
      //主要看下面：
      alert(anObject["aProperty"]);   //可以将对象当数组以属性名作为下标来访问属性
      anObject["aMethod"]();          //可以将对象当数组以方法名作为下标来调用方法
      for( var s in anObject)           //遍历对象的所有属性和方法进行迭代化处理
        alert(s + " is a " + typeof(anObject[s]));

      var aFunction = function() {};  //一个函数
      aFunction.aProperty = "Property of function";  //函数的一个属性
      aFunction.aMethod = function(){alert("Method of function")}; //函数的一个方法
      //主要看下面：
      alert(aFunction["aProperty"]);   //可以将函数当数组以属性名作为下标来访问属性
      aFunction["aMethod"]();          //可以将函数当数组以方法名作为下标来调用方法
      for( var s in aFunction)           //遍历函数的所有属性和方法进行迭代化处理
        alert(s + " is a " + typeof(aFunction[s]));
```

## chapter 6 放下对象

```shell
再来看看function与object的超然结合吧。
this并不一定是函数本身所属的对象。
this只是在任意object和function元素结合时的一个概念，
是种结合比起一般对象语言的默认结合更加灵活，显得更加超然和洒脱。
在JavaScript函数中，你只能把this看成当前要服务的“这个”对象。
this是一个特殊的内置参数，根据this参数，
您可以访问到“这个”对象的属性和方法，但却不能给this参数赋值。
在一般对象语言中，方法体代码中的this可以省略的，
成员默认都首先是“自己”的。但JavaScript却不同，由于不存在“自我”，
当访问“这个”对象时，this不可省略！
JavaScript提供了传递this参数的多种形式和手段，
其中，像BillGates.WhoAmI()和SteveJobs.WhoAmI()这种形式，
是传递this参数最正规的形式，此时的this就是函数所属的对象本身。
而大多数情况下，我们也几乎很少去采用那些借花仙佛的调用形式。
但只我们要明白JavaScript的这个“自我”与其他编程语言的“自我”是不同的，
这是一个放下了的“自我”，这就是JavaScript特有的世界观。
```

## Chapter 7 JavaScript Object Notation (JSON)

```javascript
创建一个没有任何属性的对象：
var o = {};

    创建一个对象并设置属性及初始值：
var person = {name: "Angel", age: 18, married: false};

    创建一个对象并设置属性和方法：
var speaker = {text: "Hello World", say: function(){alert(this.text)}};

     创建一个更复杂的对象，嵌套其他对象和对象数组等：
    var company =
    {
        name: "Microsoft",
        product: "softwares",
        chairman: {name: "Bill Gates", age: 53, Married: true},
        employees: [{name: "Angel", age: 26, Married: false}, {name: "Hanson", age: 32, Married: true}],
        readme: function() {document.write(this.name + " product " + this.product);}
    };
其实，JSON就是JavaScript对象最好的序列化形式，它比XML更简洁也更省空间。
对象可以作为一个JSON形式的字符串，在网络间自由传递和交换信息。
而当需要将这个JSON字符串变成一个JavaScript对象时，只需要使用eval函数这个强大的数码转换引擎，
就立即能得到一个JavaScript内存对象。正是由于JSON的这种简单朴素的天生丽质，
才使得她在AJAX舞台上成为璀璨夺目的明星。

除JSON外，在JavaScript中我们可以使用new操作符结合一个函数的形式来创建对象。例如：
    function MyFunc() {};         //定义一个空函数
    var anObj = new MyFunc();  //使用new操作符，借助MyFun函数，就创建了一个对象
 
   其实，可以把上面的代码改写成这种等价形式：
    function MyFunc(){};
    var anObj = {};     》//创建一个对象
    MyFunc.call(anObj); //将anObj对象作为this指针调用MyFunc函数

我们就可以这样理解，JavaScript先用new操作符创建了一个对象，
紧接着就将这个对象作为this参数调用了后面的函数。
其实，JavaScript内部就是这么做的，而且任何函数都可以被这样调用！
但从 “anObj = new MyFunc()” 这种形式，我们又看到一个熟悉的身影，
C++和C#不就是这样创建对象的吗？条条大路通灵山，殊途同归啊！
在JavaScript里，我们可以把这个MyFunc当作构造函数的。
然而，比静态对象语言更神奇的是，
我们可以随时给原型对象动态添加新的属性和方法，
从而动态地扩展基类的功能特性。这在静态对象语言中是很难想象的。
微软在设计AJAX类库的初期，用了一种被称为“闭包”(closure)的技术来模拟“类”.
```


## JSBibleS5  source code of 甘露模型
```notes
In jsbibles5.js
```

Go(chess) Javascript
```notes
In jsbibles1.html  <script>
chapter 8 in jsbibles2.html
```

  
    
    

> *chapter 9*

> **chapter 10**
