## cloth detail & sub detail
全新改版之后增加了很多功能和逻辑，重新整理
v16.05.16 today going on
v16.05.27 today going on
v16.05.31 dramatically change all the stuff

### Model
------------------
新增组件StoreHelper `var StoreHelper = require('app/model/StoreHelper');`
导致model可以改的更简便明了，focus on Actions就可以了
新的cloth model如下
```javascript
var StoreHelper = require('app/model/StoreHelper');
var Actions = require('app/resources/Actions');

function Cloth() {

}

Cloth.prototype.formatHelper = {
  normal: function (data) {
    if (data) {
      data.price = data.price || 0;
      data.discount_price = data.discount_price || 0;
      data.original_price = data.original_price || 0;
      data.discount_percent = data.discount_percent || 0;
      data._price = Core.Num.formatMoney(data.price / 100);
      data._original_price = Core.Num.formatMoney(data.original_price / 100);
      data._discount_price = Core.Num.formatMoney(data.discount_price / 100);
    }
    return data;
  }
}

Cloth.prototype.clothDetail = StoreHelper.requestStore(Actions.clothDetail);

Cloth.prototype.clothDetailPage = StoreHelper.requestStore(Actions.clothDetailPage);

Cloth.prototype.clothSimilarItem = StoreHelper.pagingStore(Actions.clothSimilarItem);

Cloth.prototype.clothMatchList = StoreHelper.pagingStore(Actions.clothMatchList);

Cloth.prototype.wishList = StoreHelper.pagingStore(Actions.wishList);

Cloth.prototype.addToWardrobe = StoreHelper.postStore(Actions.addClothToWardrobe);

Cloth.prototype.delFromWardrobe = StoreHelper.postStore(Actions.delClothFromWardrobe);

Cloth.prototype.addToWishList = StoreHelper.postStore(Actions.addToWishList);

Cloth.prototype.delFromWishList = StoreHelper.postStore(Actions.delFromWishList);

module.exports = new Cloth;
```
另，如果要做追加，可以Cloth.prototype.delFromWishList = StoreHelper.postStore(Actions.delFromWishList，{写在这个大括号里}); because `function xxxxxStore(action, options) {}`

### Controller
这部分基本没变，按规矩来就行了

### View
------------------------
重头戏是cloth页面的view部分，为了记录页面滚动和tab切换的位置，新添加了一个组件`var TabStatus = require('util/TabStatus');`
这个组件的细节必须全搞明白，应用在wishlist page和当前的cloth page等所有需要记录tab滚动位置的地方。

```javascript
var store = {};

function init(){ //写一个实例，方便在各种需要的地方直接调用
  var data = {curIdx: 0}; //init是在tab0
  this.getStatus  = function () {
    return data;
  }
  this.setCurTabIdx = function (idx) {
    data.curIdx = idx;
  }
  this.getCurTabIdx = function () {
    return data.curIdx;
  }
  this.setTabPosition = function (idx) {
    idx = idx || this.getCurTabIdx() || 0;
    data[idx] = $(window).scrollTop();
  }
  this.getTabPosition = function(idx) {
    return data[idx] || 1;
  }
  this.scrollToTabPosition = function(idx, budget){
    window.scrollTo(0, Math.max(budget||1, this.getTabPosition(idx)))
  }
}
function TabStatus(key){
  store[key] = store[key] || new init(); //如果有记住的存储位置，就取存的，没有就初始化一个新的对象，里面有上面那些属性。
  return store[key];
}

module.exports = TabStatus;
```

调用tabStatus，先require，再init，可以在els里面init，也可以在render的时候init，
不用传什么具体cloth id的页面（不需要根据衣服的变化而变化的），TabStatus传的是VIEW.viewCls,比如stylebook, home page guide等用到tab bar的view





## sub detail

### scss (sub detail)
隐藏scroll bar的方法：比父层元素height大20px，刚好可以遮挡scroll bar，总之就看不到scroll bar了
```scss
.pic{
      $pic-h: 240px;
      height: $pic-h;
      overflow: hidden;
      .pic-bd{
        @include box-h-c-c;
        height: $pic-h+20;//比父层元素height大20px
        overflow: hidden;
        overflow-x: scroll;
        -webkit-overflow-scrolling: touch;
      }
    }
```
