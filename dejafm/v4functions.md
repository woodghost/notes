## cloth detail & sub detail
全新改版之后增加了很多功能和逻辑，重新整理

关于tab切换，记住view最后的tab位置
我一开始用的是黑暗的trigger router forward方法那样，每切换一次tab页面就要刷新，体验灰常差。
这个问题在此处其实有着非常简单的处理方式，因为router里面加了`preQuery`, client处传来的url带`tab=0，1，2`这样的param，我们没必要改变路由，只要模拟记住路由所带来的状态就行了。
最后采用的方法是 `req.query.tab!=req.preQuery.tab` 就跳转到`req.query.tab`的位置，简单又合理。

什么时候做缓存 & 高级缓存
列表需要前进 & 后退到其他view，对该列表会产生频繁的请求，这时候就需要做缓存了，避免这种频繁的request. 要确保理解含义，自己能想得明白创造的出来。

------------------------------

## Model
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

## Controller
这部分基本没变，按规矩来就行了


------------------------


## View
重头戏是cloth页面的view部分，为了记录页面滚动和tab切换的位置，新添加了一个组件`var TabStatus = require('util/TabStatus');`
这个组件的细节必须全搞明白，应用在wishlist page和当前的cloth page等所有需要记录tab滚动位置的地方。

#### 复习一下toggle bars的动画制作方案（所有与屏幕touch event交互的动画异曲同工）
Current status:
triple tabs没在顶部，bottom bar slideInUp

因为在scss里bottom bar自带SlideInUp样式，而且`display:none`，所以一进来要掉`tabBar(true)`,加一个show方法让bottom bar一直显示
只要执行过一次动画，就要`clearTimeout`,否则会闪动，会有动画效果叠加

```javascript
if(!isApp){ return;}//如果非app环境打开直接不显示bbar了，显示顶部download
    els.toggleBottomBarTimer && clearTimeout(els.toggleBottomBarTimer);
```

当bottom bar隐藏时需要setTimeout, 为了动画过后让这个bar彻底隐藏。否则动画结束bar又出现了
```javascript
el.addClass('dismiss');
      els.toggleBottomBarTimer = setTimeout(function(){
        el.removeClass('show dismiss');
      },400);
```
不想继续让bbar toggle的话直接在判定隐藏的位置注释了`toggleBottomBar` function就ok了

#### 记录tab位置：TabStatus
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

现在存在在页面里的具体交互和功能代码

#### switch tabs
首先是controller里面switch tab的源码，认真看一下
```javascript
function switchTab(el, tabs, tabContents, analytics) {
//传四个参数，当前按下去的element，一行所有的jquery tabs，然后所有的tab content
    if (!tabs || !tabContents) {
      return;
    }
    var isClicked = !!el;//转成boolean型
    el = el || tabs[0];
    for (var i = 0; i < tabs.length; i++) {
      if (tabs[i] == el) {
        tabs[i].classList.add('on');
        trigerAnimate(tabContents.eq(i));
        tabContents[i] && tabContents[i].classList.add('show');
        analytics && isClicked && Core.Event.trigger('analyticsCurView', 'tab=' + i);
      } else {
        tabs[i].classList.remove('on');
        tabContents[i] && tabContents[i].classList.remove('show');
      }
    }
    analytics && Core.Event.trigger('analyticsCurView');
  }


function beforeSwitchTab(evt){
    if(evt){
      evt.stopPropagation();
      evt.preventDefault();
    }
    var el = $(this),
      idx = el.index(),//取当前点击的tab的index
      preIdx = els.tab.find('.on').index(),//原本被选中的
      budget= els.cloth.height(),//tab之上那部分的高度(现在多了toggleExpandText, 高度可能会计算不准)
      scrollTop = $(window).scrollTop(),//向上滑的位置的高度，随着scroll不停改变。
      isFix = scrollTop >= budget;

    //record tab status这里调用了那个封装好的util
    els.tabStatus.setTabPosition(preIdx);
    els.tabStatus.setCurTabIdx(idx);

    setTimeout(function () {
      //fix position when the tab bar is fixed
      //
      Core.Event.trigger('switchTab',el[0],els.tabs,els.tabContents);
      setTimeout(function () {
        isFix && els.tabStatus.scrollToTabPosition(idx, els.cloth.height());
        VIEW._BasicView.resizeCalculateWindow();
      }, 0);
    }, 0);
  }
```

其中用到了stopPropagation & preventDefault, 
有个简单易懂的解释在[stack overflow difference between event.stopPropagation and event.preventDefault?](http://stackoverflow.com/questions/5963669/whats-the-difference-between-event-stoppropagation-and-event-preventdefault)：

#### 自动loading
三个tab所对应的分页列表都要用到auto loading
```javascript
function onBeforeLoadingNextPageXxxx(scroll) {
    if (els.isViewVisible && !els.isLoadingNextPageClothStreet && els.tabFirst.hasClass('on')) {
    //1. view可见， 2. 列表没有被loading过，loading过要手动更改这个参数值。 3. 防止其他tab同时被loading，要做一个当前tab的判断。
      if (scroll) {
      //检测滚动状态
        if (VIEW._BasicView.isMaxWindowScroll()) {
        //在main View里面有个方法，计算maxScroll和top的比较结果，top>maxScroll就加载
       //maxScroll就是最大滑动高度，理解上就是滑到列表底部就要加载下一页了，(我们不知道街拍的宽高)
          loadingNextPageClothStreet();
        }
      } else {
      //这位置目前不知道什么时候会触发
        loadingNextPageClothStreet();
      }
    }
  }
  function loadingNextPageClothSimilar(){
      if(!els.isLoadingNextPageSimilar && !els.isSimilarListEnd){
        els.isLoadingNextPageSimilar = true;
        Core.Event.trigger('ClothController.beforeRequestClothSimilar');
        renderListEnd(els.similarListEnd,'.loading');
      }
   }
```


#### delete street snap in the list

```javascript
  function renderRemoveStreet(){
    initResources();
//删除规则是按照id删除，为了删除的准确性，专门在model里面写了getReportId 方法
    var id = VIEW.models.StreetSnap.clothStreets.reportStreet.get();
    els.streetList.find('.item[data-id="'+id+'"]').remove();
  }
```

#### render各个部分

```javascript
      data.data._name = data.data.name.length>50?(data.data.name.substr(0, 50)+'...'):data.data.name;
      //为了防止名字过长，可以用substr()这个方法裁剪一下
```


```javascript
els.main.on('touchstart touchend',function () {//在touchstart, touchend事件下都调用底下两个函数
//为了修正这个事件的准确性
      hideGuideTip();
      stickyTabs();
    });
```


```javascript
els.matchList.on(tap, '.outfit', function () {
      mainData && Core.Event.trigger('forwardOutfits', 'cloth_id=' + mainData.id + ('&order=' + this.dataset.order));
      //dataset.order就是前面html里面写了Attribute：data-order, 调用的时候就要这么用，data里面命名不能出现大写字母，调用的时候
      //the attribute named data-abc-def corresponds to the key abcDef（Refer to MDN）
    });
```
要注意的是‘’里面的是方法的名称，Core.Event.trigger()相当于调用自己写好的api

------------------------

## html & scss (sub detail)
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

* 深刻的检讨一下列表结构的问题

> 为什么会有这种问题呢？放在list的body里面就看不见list end， 放在外面呢又隐藏不了……
> 这是列表结构的问题，script那个模板层和它外面包裹的那一层一定要连在一起

如果结构不对会造成list end看不见，整个列表层级结构有问题
正确的方式是一定要在`<script>`标签外面包一层，这一层的`data-element`名称最好同script的`data-template`命名相同
```html
<section class="tabs-content wish-list tabs-content-default show" data-element="wishList">
    <div class="bd" data-element="wishListBd">
      <section class="list product-item-block" data-element="wishListItem">
        <script type="text/html" data-template="wishListItem">
          <article class="item" data-id="<%=id%>" data-fake-link="#/cloth/id=<%=id%>">
            <div class="img"><img class="lazy animated" data-src="<%=image%>/420.jpg"></div>
            <div class="detail">
              <div class="info">
                <div class="name"><%=name%></div>
                <div class="brand ellipsis"><%=brand_info.name%></div>
                <div class="meta">
                  @@include("include/cloth-price-list.html")
                </div>
              </div>
            </div>
          </article>
        </script>
      </section>
      <!-- list-end-default -->
      @@include("include/list-end-default.html")
      <!-- end list-end-default -->
    </div>
    <section class="empty hide" data-element="emptyList">
      <p>You have no item in wishlist.</p>
      <p>Start by adding an item in your fitting room or save an item by clicking the heart icon.</p>
    </section>
  </section>
```
