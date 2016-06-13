# Notes
Taking notes via learning, including CSS3, HTML5, JavaScript, jQuery...(adding in process)
> This readme file is for exploring markdown syntax and application in reality
> Below will display some specific examples of markdown, and show the guide map of my learning notes, it'll make you directly locate to the field u r interested

**All of my notes are related to web front-end techniques and the structure [generator-webappstarter](https://github.com/unbug/generator-webappstarter) provided by [@unbug](https://github.com/unbug)**

#Guide Map
Today's task is to be familiar with markdown for the future usage.
(Updating & organising)


|File name | Content|
|--------- |:-------:|
|[style](https://github.com/woodghost/notes/blob/master/StyleTips/) | CSS3 & HTML5|
|[Javascript](https://github.com/woodghost/notes/blob/master/JsTips/)| javascript or node.js, and any other structures using js|   
|[dejafm](https://github.com/woodghost/notes/blob/master/dejafm/)| some encountered issues in dejafm project& MVC generator.|
|[supplement](https://github.com/woodghost/notes/blob/master/RelatedTips/)| Other CS knowledge (Updating & organising)|
|[clothms](https://github.com/woodghost/notes/blob/master/clothms/)|cloth management system(using React JSX & Semantic UI)|
|[dejastie](https://github.com/woodghost/notes/blob/master/dejasite/)| desktop website (has gulpfile and after refactor in individually modules)|

#Test

# This is an `<h1>` tag
## This is an `<h2>` tag
###### This is an `<h6>` tag

### This is a third-tier heading
============================================


You can use  one `#` all the way up to `######(in `` share the same btn with tilde(~))` six for different heading sizes.
here are samples of keyboard btn:
<kbd>command</kbd>
<kbd>fn</kbd>

If you'd like to quote someone, use the > character before the line:

> This is a quote line, 
> - woodghost

It's very easy to make some words **bold** and other words *italic* with Markdown. You can even [link to Markdown 
tutorial!](https://guides.github.com/features/mastering-markdown/)

* Item 1
* Item 2
  * Item 2a
  * Item 2b
  
![GitHub Logo](https://fleep.io/blog/wp-content/uploads/2014/07/github_icon.png)

```python
def foo():
    if not bar:
        return True
```
        
- [x] @mentions, #refs, [links](), **formatting**, and <del>tags</del> supported
- [x] list syntax required (any unordered or ordered list supported)
- [x] this is a complete item
- [ ] this is an incomplete item


First Header | Second Header
------------ | -------------
Content from cell 1 | Content from cell 2
Content in the first column | Content in the second column

The table head must be snap to grid.

~~I am a demo  strikethrough~~

~~l a la la  23333~~

hello \***\* ~\~

:ghost:



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
    

> *chapter 9*

> **chapter 10**



