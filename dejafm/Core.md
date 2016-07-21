## JSONP, CORS, advanced JSON analyzing & concat

> 跨域技术, 处理跨域请求, json拼接

这是最早在newList project里面处理movie list用到的，deja website里面blog preview也用到。
最大的方便是可以直接用DDMS编辑后台数据，先不用server开发相应的api
系统的过了一遍如何解析DDMS里请求到的jsonp data并拼接成容易解析的json数据结构

这种jsonp方式url里带一个表单id，每换一个id都能读不同的表单信息

