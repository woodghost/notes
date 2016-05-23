# JS tips
Taking notes via learning, including CSS3, HTML5, JavaScript, jQuery...(adding in process)

#Guide Map
Today's task is to be familiar with markdown for the future usage.
(Updating & organising)


|File name | Content|
|--------- |:-------:|
|[savvyjscn.md](https://github.com/woodghost/notes/blob/master/JsTips/savvyjscn.md)| Notes of Savvy Javascript, the book writing by Chinese, Author: Zhan Li| 


# Reading Notes
### Professional Javascript for Web Developers (高级程序设计)

##### 5.2 Array type
* slice(start,end)  切取一片，from start location to destination
* filter() return an array contain true elements
* forEach() no return value

```javascript
function fancyAlert(arg) {
  if(arg) {
    $.facebox({div:'#foo'})
  }
}
```

## js syntax common knowledge
* js双感叹号是为了转化为Boolean值，一般用来判断元素是否存在。


#### js语法相关

```javascript
getAttribute() vs .attr()
<input type="text" id="example" tabindex="3">
The following line does indeed show "number", not "string":
alert(typeof $("#example").attr("tabindex")); //Number
Now, the thing that's confusing me is that when using the DOM method getAttribute, you get a different result:

（getAttribute这个操作DOM特性的方法在高级程序设计10.1节点层次深入讲到了）
alert(typeof $("#example")[0].getAttribute("tabindex")); //String
```
why & how to use .call(this) method

```javascript
function Foo() {

  (function () {
    console.log(this);
    // > Foo
  }).call(this);

  (function () {
    console.log(this);
    // > undefined in strict mode, or Window in non strict mode
  })();
}

var bar = new Foo;
```

### getAttribute vs attr
```javascript
Explanation from MDN
elem.checked	true (Boolean) Will change with checkbox state
$( elem ).prop( "checked" )	true (Boolean) Will change with checkbox state
elem.getAttribute( "checked" )	"checked" (String) Initial state of the checkbox; does not change
$( elem ).attr( "checked" ) (1.6)	"checked" (String) Initial state of the checkbox; does not change
$( elem ).attr( "checked" ) (1.6.1+)	"checked" (String) Will change with checkbox state
$( elem ).attr( "checked" ) (pre-1.6)	true (Boolean) Changed with checkbox state
```


关于三元运算符以及更多我早该知道的语法问题！！ in javascript

```javascript
function renderFilterBtn(isOn){   els.styleMenuFilter[isOn ? 'addClass' : 'removeClass']('on'); }

甚至更变态些的。
$('.item')[ flag ? 'addClass' : 'removeClass']('hover')
上面的代码看着比较困惑。因为当flag = true 的时候 ，代码就变成以下代码：
$('.item')['addClass']('hover')
这样写法等同于。
$('.item').addClass('hover')
具体解释 详见 豪妹的文章 // 一开始我一直搞不懂为啥子可以这样写，经过沟通突然领悟了，咩哈哈
再升华一下
可以根据需要来调用自己想要的function来处理更多的事情。
function a(){
      do something
}
function b(){
      do something
}

flag ? a() : b();
那么为师的完全体
于是有了这么个案例，两个按钮 一个向前的行为，一个向后的行为。操作的功能都差不多。
    var action_turn = function(e, type){
        var self = $(e).closest('li');
        var target = self[type === 'prev' ? 'prev' : 'next']();
        target.addClass('has-img');
        self.removeClass('has-img')
    }
    
    var btn_next = $('#item-photo-panel a.next')
    btn_next.click(function(){
        action_turn(this, 'next');
        return false;
    });
    var btn_prev = $('#item-photo-panel a.prev')
    btn_prev.click(function(){
        action_turn(this, 'prev');
        return false;
    });
```
