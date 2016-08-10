
## Invite Module

### scss
1. 全局复用的样式才能写在大模块外面，其余的不带‘>’号写里面整个模块内部就可以复用了
2. title style可以用text relative，一条line absolute这种情况，并且line的z-index比title小就可以了，可以不用写三块
3. invite-bg直接写最外层，js里面渲染的时候加上这个class


### javascript
1. renderShare在View.js里面
2. 有msgbox的地方就要想到hide view的时候要把它隐藏掉





