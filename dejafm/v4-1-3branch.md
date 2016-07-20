
## ClothView.js

```javascript
/**
 * 与客户端交互
 *
 *    前端与客户端交互的流程主要为前端调用，客户端读取，客户端回调三个步骤：
 *    前端调用客户端规则为：dejafashion://NAME（android 前端调用window.__dejafashion_NAME()）
 *    客户端读取前端的数据规则为 window.__dejafashion_data_NAME，前端返回JSON {default:'',...}
 *    【如果有】客户端处理完逻辑回调前端的方法规则为： window.__dejafashion_after_NAME([json])
 *    拿分享为例:
 *    1 -------> 前端调用dejafashion://share（android 前端调用window.__dejafashion_share()）
 *    2 --------> 客户端接收到请求，向前端读取必要参数window.__dejafashion_data_share
 *    3 ------- >【如果有】 客户端处理完逻辑回调通知前端处理完毕调用window.__dejafashion_after_share([json])
 *    4 ------>前端接收客户端传回来的参数该干嘛干嘛
 *    注意，客户端在调用前端方法之前一定要先做判断，如 window.__dejafashion_after_share && window.__dejafashion_after_share([json])（容错）
 *
 */
```

### webview与native通信用的API
=============================

### bind event
==============
现在有很多需要与native通信的地方，所以整理一下bind event
```javascript
function bindEvent() {
    els.main.on('touchstart touchend',function () {
      hideGuideTip();
      toggleBars();
    });
    els.cloth.on(tap, '.tryon', function () {
      Core.Event.trigger('appAPI', 'tryon', null, null, this.getAttribute('data-id'), 1);
    });
    els.cloth.on(tap, '.detail-img img', renderImgViewer);
    els.cloth.on(tap, '.occasion__imgs .item', renderOccasionItem);
    els.cloth.on(tap, '.nav-bottom .wardrobe', onToggleWardrobe);
    els.cloth.on(tap, '.nav-bottom .wishlist', onToggleAddToWishList);
    els.cloth.on(tap, '.description .hd', function(){ //description 这部分目前是隐藏的，是cloth 详情里面一个部分
    //原本是点击展开详情的
      Core.Event.trigger('toggleTextSectionExpand',this.parentNode);
    });

    els.tab.on('click','.item',beforeSwitchTab);
    !isApp && els.tabContents.on('click',function () {
      VIEW._BasicView.msgbox.showOpenapp({
        yesCallback: function () {
          Core.Event.trigger('analyticsCurView','act=sharedpage_click_from_detail');
        }
      });
    });
    els.sectionDownload.on(tap,function () {
      Core.Event.trigger('analyticsCurView','act=sharedpage_click_from_detail');
    });

    els.streetsContent.on(tap, '.item', function () {
      var id = this.getAttribute('data-id');
      id && Core.Event.trigger('forwardStreetSnaps', 'id=' + id + (mainData ? ('&cloth_id=' + mainData.id) : ''));
    });
    els.streetListEnd.on(tap, '.more', onBeforeLoadingNextPageClothStreet);
    els.similarListEnd.on(tap, '.more', onBeforeLoadingNextPageClothSimilar);

    els.matchListEnd.on(tap, '.more', onBeforeLoadingNextPageMatchList);
    els.matchList.on(tap, '.outfit', function () {
      mainData && Core.Event.trigger('forwardOutfits', 'cloth_id=' + mainData.id + ('&order=' + this.dataset.order));
    });
    els.imageViwer.on(tap, hideImgViewer);
    $(window).on('scroll', onViewScroll);
  }//end bindEvent
```




当判断不在app里面的时候(本地模拟debug=0)：
1. 显示顶部download
2. 点击3个tab底下的tab content里面的内容时触发msgbox, 其本质是一张图片

```javascript
!isApp && els.tabContents.on('click',function () {
      VIEW._BasicView.msgbox.showOpenapp({
        yesCallback: function () {
          Core.Event.trigger('analyticsCurView','act=sharedpage_click_from_detail');
        }
      });
    });
```
