## cloth

### ClothView.js
#### bind event
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
与native通信用的event。
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
