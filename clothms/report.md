# Report module

reactJS在渲染时有很多规则与之前不一样，ES2015（ES6）语法也与ES5有差别：

1. 用`className`替换`class`
2. pagingStore model必须调用getStore method才能render第二页之后的data，<br/>
否则总是第一页data。invoke getStore method 需要__STORE_ID, 需要动态创建一下
3. array traversal（遍历）从之前的`forEach`变为`map`


