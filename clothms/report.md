# Report module

## Report Management页面 in react
passing parameters 只能从VIEW—> CTRL
主要的逻辑都在var Main = React.createClass({})这里，相当于RN里面container
   react里面所有model都updated同一个render方法，react会自己解析，如果json里缺少fields也不会报错。
这个页面主要分3 parts，包含2个API，需要require following blocks:

```javascript
var ProductModel = require('app/model/ProductModel'); 
var StatusModel = require('app/model/StatusModel');  
var React = require('react'); 
var ReactDOM = require('react-dom'); 
var DashboardHeader = require('app/view/component/DashboardHeader’);
//改module menu名字component
 var DatePicker = require('app/view/component/DatePicker’);//slelect date widget 
var Dropdown = require('app/view/component/Dropdown’);//dropdown widget

var FilterBar = React.createClass({})
```

reactJS在渲染时有很多规则与之前不一样，ES2015（ES6）语法也与ES5有差别：

1. 用`className`替换`class`
2. pagingStore model必须调用getStore method才能render第二页之后的data，<br/>
否则总是第一页data。invoke getStore method 需要__STORE_ID, 需要动态创建一下
3. array traversal（遍历）从之前的`forEach`变为`map`


## 关于API：
首先要知道怎么设计的，要实现什么功能，为什么这么设计。
1. 拿report来说，AllReasonReport这个接口的作用就是返回json进行页面渲染，json内容静态不改变，不用每次render的时候都调用，不需要给server传id，仅仅起到的就是一个取json渲染页面的作用。


