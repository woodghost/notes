# cloth, street snap
 仔细回顾一下streetsnap怎么做的吧。
## 首先streetsnap model
写model要完全和server定义api吻合。
里面有streetsnap的model，做了缓存之类的，看上去有点复杂，
复杂是因为里面除了常规的分页model之外还有 reportStreet Mdl 以及

1. reportStreetById//这个以及下面那个方法都是因为加了街拍点赞的逻辑而诞生的
2. likeStreetById
3. setWithStoreData等几个function。

还有mystreet那个的model
还有街拍点赞like dislike report（也就是删除并且反馈原因）那几个post方法。

## 接下来Controller
所有的接口都是json格式的，
所有与server的api协议都可以在swagger里面找到
controller里面绝大部分是与server的交互
所有的定义都可以在native与web定义接口的文档里面查找到（我们与客户端的交互）
想要找具体与客户端的交互的方法，或者深究写在框架里的方法
客户端：xxxController———》Controller.js————》NativeBridge.js
MVC：class.js  subject.js，pubsub.js  (总之都是Core文件夹下面的)
引入streetsnap model是肯定的，这个部分的api都在street类
其中这个页面调了
```javascript
clothStreets: Core.localHost + '/apis/street/get_street_of_cloth/v3', 
likeStreet: Core.localHost + '/apis/street/sync_streetsnap_like/v3_1', 
reportStreet: Core.localHost + '/apis/street/report_reason/v3_1',
```
这三个API

```javascript
function beforePostLikeSnap(id) {//if 是report function，pass(id, reason)
    id = id || '';
    var data = {
      street_id: id,
      like: 1 //unlike function和like基本一样，pass 的param是like: 0; report也差不多，pass的是reason: reason
    };
    CTRL.models.StreetSnap.clothStreets.likeStreetById(id,true);
    CTRL.models.StreetSnap.likeStreet.post(JSON.stringify(data), afterPostLikeSnap);
  }

  function afterPostLikeSnap(success) {//after function基本都是一样的：hide loading
    CTRL.views.Basic.msgbox.hideLoading();
    var data = CTRL.models.StreetSnap.likeStreet.get();
    if (!success || !data || data.ret != 0) {//报错处理
      CTRL.views.Basic.msgbox.showFailed({
        msg: success && data.msg
      });
    }else{
      Core.Event.trigger('StyleBookController.resetMyStreets');
      //重置myStreet
    }
  }
```
### 值得注意的是
- beforePostLikeSnap（id）里面调了filter里面的likeStreetById(id,true) function
- data里面写着需要传给server的params
- like unlike call function之后都要resetMyStreet里面的内容Core.Event.trigger('StyleBookController.resetMyStreets');
- post function 记得要用JSON.stringify(data)  历史遗留问题
- 所有的after function记得传success param
- 通过这次尝试，发现beforePostLikeSnap function确实是在after之前执行的，不知道上次是哪里遇到相反的情况

```javascript
Core.Event.on('StreetSnapsController.beforePostLikeSnap', beforePostLikeSnap); 
像这样的Core.Event.on() declaration 是为了你写的controller里面的方法能在view里面调 Core.Event.trigger()
直接在onView function里面判断没请求到数据就去请求。
```

## 然后View
streetsnap 会用到util里的Slider
然后常规的actions和streetsnap model

在var VIEW = this, 区域定义一些变量
likeAlertId, firstLikeAlerted, reportStreetReasonId, reportStreetReason, reportStreetPopMuteId, isReportStreetPopMute;

监听model用updated method render function

bindEvent()
```javascript
function bindEvent() {
    els.clothes.on(tap, '.item', setLastViewQuery);
    //
    els.snapBar.on(tap, '.like', onToggleStreetLike);
    //trigger toggle like
    els.snapBar.on(tap, '.report', beforeReportStreet);
    //report & delete
    els.reportStreetPop.on(tap, '.reason', beforeSelectReportStreetReason);
    //two reason(11 and 12)
    els.reportStreetPop.on(tap, '.dont-show', beforeToggleReportStreetMute);
    //不再弹出report弹窗的逻辑
    //以上都是report的一些逻辑
    els.reportStreetPop.on(tap,function(e){
      if(!$(e.target).is('.msg-bd *')){
        hideReportPop();
      }
    });
    //点击非弹窗区域则弹窗消失
  }//end bindEvent
```


show, hide, update functions
```javascript
this.show = function () {
    initResources();

    Core.Event.trigger('trigerAnimate', els.main);
    VIEW._BasicView.show(VIEW.viewCls);
    //以上generator自动生成
    
    scrollToStreet();
    renderStreetNav();
    els.isViewVisible = true;
    els.isFirstIn = true;
    //show function里面call的function，define的varibles
  }
  this.hide = function () {
    if (!els) {
      return;
    }
    els.isLoadingNextList = false;
    els.lastViewQueryId = null;
    els.isViewVisible = false;
  }//出this page的view之后要hide的东西
  
  
  function showReportPop(){
    els.reportStreetPop.removeClass('hide');
  }
  function hideReportPop(){
    els.reportStreetPop.addClass('hide');
  }//show hide report pop
  
  
  function updateViewQuery(req) {
    viewQuery = req.query;
  }//auto generated
  function updateStoreIds() {//更新localStorage id
    likeAlertId = 'street_first_like_' + VIEW.models.Basic.getUserId();
    //should get user id when click like(in basic Model)
    firstLikeAlerted = Core.localStorage.get(likeAlertId);
    //record first liked

    reportStreetReasonId = 'street_report_reason_' + VIEW.models.Basic.getUserId();
    reportStreetReason = Core.localStorage.get(reportStreetReasonId);

    reportStreetPopMuteId = 'street_pop_mute_' + VIEW.models.Basic.getUserId();
    isReportStreetPopMute = Core.localStorage.get(reportStreetPopMuteId);
  }
```
  
  
### 接下来是render function 部分
大致上是分snap bar，street snap slider，clothes三大部分，每一部分还有很多细节
```javascript
  function render(data) {
    initResources();//initial templates & elements
    updateStoreIds();
    data = data || VIEW.models.StreetSnap.clothStreets.get();

    if (!data || data.ret != 0 || !data.data) {
      return;
    }
    els.mainData = data;//定义了ele，把data赋值过去了
    resetTmpVals();//调reset临时值的这个function，后面有具体的这个function的内容
    var isAdd = VIEW.models.StreetSnap.clothStreets.page && !VIEW.models.StreetSnap.clothStreets.isFromStore,
      appendFn = isAdd ? 'append' : 'html',
      htm = [];
    data.data.forEach(function (key, idx) {
      if (idx < 2) {
        var IMG = new Image();
        IMG.src = key.image + '/750.jpg'
      }
      htm.push(Tpl.street(key));
    });
    !data.end && htm.push(Tpl.loadingStreet({}));

    els.streetsList.find('.item.loading').remove();
    els.streetsList[appendFn](htm.join(''));

    renderClothes(data.data, isAdd);
    renderSnapBar(data.data);

    if (data.data.length > 0) {
      els.streetsSlider = els.streetsSlider || new Slider({
          listEl: els.streetsList,
          enableDrag: true,
          enableLoop: false,
          itemLen: 658,
          onTouchend: function () {
            VIEW._BasicView.GlobalTouch.preventMove = false;
          },
          onTouchmove: function (x, y) {
            VIEW._BasicView.GlobalTouch.preventMove = x > 5 && y < 30;
          },
          onMove: function (index) {
            loadImgs(index);
            renderClothesListNav(index);
            renderSnapBarNav(index);
            loadNextPage(index);
          }
        });
      if (isAdd) {
        els.streetsSlider.refresh();
      } else {
        els.streetsSlider.reset();
        scrollToStreet();
      }
    }
  }
```


```javascript
  function renderSnapBar(data) {
    var htm = [],
      appendFn = VIEW.models.StreetSnap.clothStreets.page ? 'append' : 'html';
    data.forEach(function (key) {
      htm.push(Tpl.snapBar(key));
    });
    els.snapBar[appendFn](htm.join(''));
  }
  function renderSnapBarNav(idx) {
    els.snapBarListEls = els.snapBarListEls || els.snapBar.children();
    els.snapBarListEls.addClass('hide');
    els.snapBarListEls.eq(idx).removeClass('hide');
  }
```


```javascript
  function renderClothes(data, isAdd) {
    var htm = [],
      appendFn = isAdd ? 'append' : 'html';
    data.forEach(function (key) {
      htm.push(Tpl.clothes(key));
    });
    els.clothesBd[appendFn](htm.join(''));
  }

  function renderClothesListNav(idx) {
    els.clothesListEls = els.clothesListEls || els.clothesBd.children();
    els.clothesListEls.addClass('hide');
    els.clothesListEls.eq(idx).removeClass('hide');
  }
```

```javascript
  function renderStreetNav() {
    els.streetsNav.removeClass('hide');
    setTimeout(function () {
      els.streetsNav.addClass('hide')
    }, 2100);
  }
```


```javascript
  function renderReportStreet(){
    hideReportPop();
    var idx = els.reportStreetIndex,
      cloth = els.clothesListEls.eq(idx),
      bar = els.snapBarListEls.eq(idx),
      delay = 300;

    els.isReportingStreet = true;
    els.streetsSlider.removeCurrent(delay,function(idx,count){
      cloth.remove();
      bar.remove();
      resetTmpVals();
      renderSnapBarNav(idx);
      renderClothesListNav(idx);
      !count && Core.Router.back();
      setTimeout(function(){
        els.isReportingStreet = false;
      },delay);
    });
  }
```


```javascript
  function loadNextPage(idx) {
    els.listItemEls = els.listItemEls || els.streetsList.children();
    if (els.isViewVisible && !els.mainData.end && !els.isLoadingNextList && idx == els.listItemEls.length - 1) {
      els.isLoadingNextList = true;
      Core.Event.trigger('ClothController.beforeRequestClothStreet');
    }
  }

  function loadImgs(idx) {
    var simgs = [].slice.call(els.streetsList.find('.img')),
      limgs;
    idx = idx !== undefined ? idx : 0;
    limgs = [simgs[idx], simgs[idx - 1], simgs[idx + 1], simgs[idx - 2], simgs[idx + 2]];
    function load(img) {
      if (img) {
        var timg = new Image(),
          src = img.getAttribute('data-src');
        if (src) {
          timg.setAttribute("src", src);
          timg.onload = function () {
            img.removeAttribute('data-src');
            img.style['background-image'] = 'url(' + src + ')';
            load(limgs.shift());
          }
        } else if (limgs.length) {
          load(limgs.shift());
        }
      } else if (limgs.length) {
        load(limgs.shift());
      }
    }

    load(limgs.shift());
  }
```


```javascript
  function scrollToStreet() {
    if (viewQuery && viewQuery.id && els.streetsSlider) {
      els.lastViewQueryId != viewQuery.id && els.streetsSlider.moveTo(els.streetsList.find('.item[data-id="' + viewQuery.id + '"]').index());
    }
  }
```


```javascript
  function onToggleStreetLike() {
    var el = $(this),
      id = el.attr('data-id'),
      isLike = !el.hasClass('on');
    el.toggleClass('on');
    if (isLike) {
      Core.Event.trigger('StreetSnapsController.beforePostLikeSnap', id);

      if (!firstLikeAlerted) {
        VIEW._BasicView.msgbox.showDialog({
          msg: 'Added to your favorites! We will show you more snaps based on your preferences.',
          yesText: 'OK'
        });
        firstLikeAlerted = true;
        Core.localStorage.set(likeAlertId, 1);
      }
    } else {
      Core.Event.trigger('StreetSnapsController.beforePostUnlikeSnap', id);
    }
  }
```


```javascript
  function beforeReportStreet(){
    if(els.isReportingStreet) return;

    var el = $(this),
      id = el.attr('data-id');
    els.reportStreetId = id;
    els.reportStreetIndex = el.parents('.item').index();
    if(isReportStreetPopMute){
      Core.Event.trigger('StreetSnapsController.beforePostReportStreet', id, reportStreetReason);
      renderReportStreet();
    }else{
      showReportPop();
    }
  }
  function beforeToggleReportStreetMute(){
    els.reportStreetPopCheckbox.toggleClass('on');
  }
  function beforeSelectReportStreetReason(){
    var el = $(this),
      isMuted = els.reportStreetPopCheckbox.hasClass('on');
    reportStreetReason = el.attr('data-reason');
    Core.localStorage.set(reportStreetReasonId,reportStreetReason);
    Core.Event.trigger('StreetSnapsController.beforePostReportStreet', els.reportStreetId, reportStreetReason);
    renderReportStreet();
    if(isMuted){
      isReportStreetPopMute = true;
      Core.localStorage.set(reportStreetPopMuteId,new Date().getTime());
    }else{
      isReportStreetPopMute = false;
      Core.localStorage.del(reportStreetPopMuteId);
    }
  }
```

```javascript
  function resetTmpVals() {
    els.isLoadingNextList = false;
    els.listItemEls = null;
    els.clothesListEls = null;
    els.snapBarListEls = null;
  }

  function setLastViewQuery() {
    setTimeout(function () {
      els.lastViewQueryId = viewQuery && viewQuery.id;
    }, 1000);
  }
```
