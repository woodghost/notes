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

