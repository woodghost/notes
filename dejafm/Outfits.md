Codes in outfits
================
Contains model, controller, view, three parts in total. 
### Model
model是存放数据的模型，所有数据结构上面需要处理的问题的设计，都要在这里做
if server side wants to add new field or new segment stuff, you need to dispose that here.
```javascript
Style.prototype.formatHelper = {
  orderByCategoryWeight: function (data) {
    return data.sort(function (a, b) {
      return a.category_weight - b.category_weight;
    });
  }
};

Style.prototype.outfits = new Mdl({//outfits的分页model
  page: 0,
  page_size: 20,
  resetPage: function () {
    this.page = 0;
  },
  //model里面嵌套的model
  filter: new Mdl({//改版之后没有顶部filter tags那一部分内容了，但是这个字段还是能起到作用的
    getIds: function(){//所有filter出的tags的id们
      var d = this.data,ids = [];//ids in tag means occasion_ids
      if (d && d.tags) {
        d.tags.forEach(function (key) {
          ids.push(key.id);
        });
        return ids.join(',');
      }
      return null;//加return null的意义在于？
    },
    removeTagById: function (id) {//感觉更像是返回不相等的id
      var d = this.data;
      if (d && d.tags) {
        d.tags = d.tags.filter(function (key) {
          return key.id != id;
        });
      }
    }
  }),
  有高级缓存功能的request
  request: function (data, callback) {//model里面的function，request
    data = data || {};
    var _this = this;//带前下划线的是我们自己定义的，不是从后台传过来的。
    var __STORE_ID;
    __STORE_ID = data.__STORE_ID;
    delete data.__STORE_ID;
    data.page = this.page;
    data.page_size = this.page_size;
    RequestHelper.getJSON({
      data: data, //数据
      action: Actions.outfits, //url
      complete: function (data) { //完成一页之后的callback
        if (data.success) {
          _this.set(data.data, __STORE_ID);
          _this.page++;
        }
        callback && callback(data.success);
      }
    });
  }
});

Style.prototype.likeStyle = new Mdl({//普通的post接口
  post: function (data, callback) {
    RequestHelper.post(Actions.likeStyle, data, callback, this);
  }
});

Style.prototype.unLikeStyle = new Mdl({
  post: function (data, callback) {
    RequestHelper.post(Actions.unLikeStyle, data, callback, this);
  }
});

Style.prototype.styleReviewHistory = new Mdl({
  request: function (data, callback) {
    RequestHelper.request(Actions.styleReviewHistory, data, callback, this);
  }
});
```


### Controller
```javascript
var Actions = require('../resources/Actions');
var BasicModel = require('app/model/Model');
var StyleModel = require('app/model/StyleModel');
var BasicView = require('app/view/View');
var OutfitsView = require('app/view/OutfitsView');

function OutfitsController() {
  this.models = {
    Basic: BasicModel,
    Style: StyleModel
  }
  this.views = {
    Basic: BasicView,
    Outfits: OutfitsView
  }

  var CTRL = this,
    viewNames,
    curViewId = '',
    cur__STORE_ID, //为了做高级缓存而定义的variable
    viewOutfitsQuery = {};

  viewNames = {
    'outfits': 'Outfits'
  }
  Core.Router.subscribe('/outfits/', onViewOutfits, unViewOutfits);

  Core.Event.on('beforeRequestOutfits', beforeRequestOutfits);
  Core.Event.on('beforeRequestStyleReviewHistory', beforeRequestStyleReviewHistory);
  Core.Event.on('updateOutfitFilter', updateOutfitFilter);

  //统计视图
  Core.Event.on('analyticsCurView', analyticsCurView);
  //forwardOutfits
  Core.Event.on('forwardOutfits', forwardOutfits);

  function unViewOutfits() {
    CTRL.views.Outfits.hide();
  }
//此处开始做高级缓存
  function onViewOutfits(req) {
  
    curViewId = 'outfits';
    viewOutfitsQuery = req.query;
    Core.Event.trigger('appModifyTitle', viewNames[curViewId]);

    cur__STORE_ID = viewOutfitsQuery.cloth_id +''+ viewOutfitsQuery.occasion_ids;
    //e.g. 12305 32243
    cur__STORE_ID = cur__STORE_ID || 'tmp__id';
    
    //model的method都在core文件夹下class.js文件，比如这个getFromStoreById
    //调getFromStoreById，返回的是通过storeId从缓存里面拿回缓存的数据
    var filterStoreData = CTRL.models.Style.outfits.filter.getFromStoreById(cur__STORE_ID);
    //cache timeout 12*5= 60 min，默认是5min，时间以5的倍数增加
    if (filterStoreData && !CTRL.models.Style.outfits.filter.timer.isTimeout(12,cur__STORE_ID)) {
    //
      CTRL.models.Style.outfits.filter.set(filterStoreData, cur__STORE_ID);
    } else {
      updateOutfitFilter(null);
    }

    var storeData = CTRL.models.Style.outfits.getFromStoreById(cur__STORE_ID),
      curData = CTRL.models.Style.outfits.get();
    //cache timeout 12*5= 60 min
    //timer是updateFactory这个function，里面有很多method, 最常用的包括isTimeOut，update...
    if (storeData && curData && !CTRL.models.Style.outfits.timer.isTimeout(12,cur__STORE_ID)) {
      curData.__STORE_ID != storeData.__STORE_ID && CTRL.models.Style.outfits.set(storeData, cur__STORE_ID);
    } else {
      CTRL.views.Outfits.hideStyles();//定义子this上面的method就可以这样调用
      //CTRL.views.Basic.msgbox.showLoading();
      CTRL.models.Style.outfits.resetPage();//分页model都要有reset这个操作
      beforeRequestOutfits();//再向server发请求。
    }

    CTRL.views.Outfits.show();

    //追加统计
    analyticsCurView();
  }
  
  //更新cache的function
  function updateOutfitFilter(options){//从view传过来的参数
    if (options) {//options是一个object，用在loading，style loading star那部分
      CTRL.models.Style.outfits.filter.set(options, cur__STORE_ID);
      options存在就set一下，不存在就是set null
    }else{
      CTRL.models.Style.outfits.filter.set(null);
    }
  }

  function beforeRequestOutfits(options, reload) {
    var data = {
      __STORE_ID: cur__STORE_ID,//id of data in cache
      specified_id: viewOutfitsQuery.cloth_id,
      refine_ids: viewOutfitsQuery.occasion_ids
    };
    if (options || reload) {
      CTRL.models.Style.outfits.resetPage();
      //CTRL.views.Basic.msgbox.showLoading();
      CTRL.views.Outfits.hideStyles();
    }
    updateOutfitFilter(options);
    options = options || CTRL.models.Style.outfits.filter.get();
    if (options && options.tags) {
      data.refine_ids = [];
      options.tags.forEach(function (key) {
        data.refine_ids.push(key.id);
      });
      data.refine_ids = data.refine_ids.join(',');
    }
    CTRL.models.Style.outfits.request(data, afterRequestOutfits)
  }//after function is a callback function

  function afterRequestOutfits(success) {
    CTRL.views.Basic.msgbox.hideLoading();
    var data = CTRL.models.Style.outfits.get();
    if (!success || !data || data.ret != 0) {
      CTRL.views.Basic.msgbox.showFailed({
        msg: success && data.msg
      });//if unsuccess, then alter error
    }else{
      //set filter
      if(data.daily_recommend){
      //这个最基本的功能就是set filter data & update filter data
        updateOutfitFilter({
          tags: [{
            id: data.daily_recommend.occasion_id + '',
            name: data.daily_recommend.occasion_name
          }]
        });
      }
    }
  }

  function beforeRequestStyleReviewHistory(item) {
  //为了在点change的时候吧每套outfits的ids再给server传一遍。
    var ids = [];
    item.clothes.forEach(function(key){
      ids.push(key.id);
    });
    var data = {
      cloth_ids: ids.join(',')
    };
    CTRL.models.Style.styleReviewHistory.request(data)
  }
}
module.exports = new OutfitsController;
```

### View
```javascript
var VIEW = this,
    isApp = Core.NativeBridge.isApp(),
    Tpl, els, viewQuery,
    tap = VIEW._BasicView.tapEvent,
    likeAlertId,
    firstLikeAlerted,
    filterAlertId,
    firstFilterAlerted;

  Core.Router.subscribe('/outfits/', updateViewQuery);
  //model listeners
  VIEW.models.Style.outfits.updated(render);
  VIEW.models.Style.outfits.filter.updated(renderFilterTags);
  //这个filter model也是需要监听一下
```
接下来是bind event和render部分，代码略长
```javascript
function bindEvent() {
    els.body.on('touchstart',hideGuideTip);//任意touch一处，tips消失
    els.filterTagsList.on(tap, '.del', removeFilterTag);
    els.stylesList.on(tap, '.c-style-tpl__item', VIEW._StyleTemplateView.onForwardCloth);
    //点任意衣服去该衣服的detail page页面
    els.styleOccasion.on(tap, '.occasion__imgs .item', renderStyleOccasionItem);//点小图看图下小字
    els.styleBars.on(tap, '.like', onToggleLike);
    els.styleBars.on(tap, '.tryon', function () {
      Core.Event.trigger('appAPI', 'tryon', null, null, this.getAttribute('data-id'), 0);//try on Event!
    });
    els.styleMenu.on(tap, '.remix', function(){
      renderNextStyle();
      setTimeout(showGuideTip2,1000);
    });
    els.styleMenu.on(tap, '.help', beforeAddMission);
    
    //以下两个都是调client的filter出来
    els.styleMenu.on(tap, '.filter', function(){
      Core.Event.trigger('appAPI', 'styleFilter', VIEW.models.Style.outfits.filter.get(), beforeFilter);
    });
    els.stylesEmpty.on(tap, '.more', function(){
      Core.Event.trigger('appAPI', 'styleFilter', VIEW.models.Style.outfits.filter.get(), beforeFilter);
    });
  }//end bindEvent

  this.show = function () {
    initResources();
    /*
    Core.Event.trigger('appActionFilterButton', function () {
      Core.Event.trigger('appAPI', 'styleFilter', VIEW.models.Style.outfits.filter.get(), beforeFilter);
    });
    */
    Core.Event.trigger('trigerAnimate', els.main);
    VIEW._BasicView.show(VIEW.viewCls);
  }

  this.hide = function () {
    if (!els) {
      return;
    }

    //Core.Event.trigger('appActionDefaultButton');
    hideGuideTip();
    renderStyleInfo(true);//是最顶部的xxx outfits we found
    VIEW._BasicView.msgbox.hideDialog();
  }

  this.hideStyles = function () {//hideStyle基本就是让几个主要的部分都透明，看不见
    initResources();
    els.stylesInfo.css({opacity: 0});
    els.stylesList.css({opacity: 0});
    els.styleBars.css({opacity: 0});
    els.styleOccasion.css({opacity: 0});
    renderEmpty(true);//empty segment也隐藏
    renderStyleLoading(false,{infinite: true});//star is always glitering, stay in loading status
  }
  this.showStyles = function () {
    initResources();
    els.stylesInfo.css({opacity: 1});
    els.stylesList.css({opacity: 1});
    els.styleBars.css({opacity: 1});
    els.styleOccasion.css({opacity: 1});
  }
  function showGuideTip1(){//这些tips要require util里面的tooltips
    var el = els.styleMenu.find('.remix'),
        tip1Id = 'outfit_tip1_' + VIEW.models.Basic.getUserId();//localStorage
    if(!els.guideTip1 && !Core.localStorage.get(tip1Id)){
      els.guideTip1 = new Tooltip({
          body: 'Change outfits with just one simple click.',
          top: el.offset().top-160,
          left: 240,
          rootCls: 'black',//if need to change styles, u could write a relative class
          bodyCls: 'view-outfit-guide-tip',
          arrow: {
            type: 'bottom',
            position: 320
          },
          autoShow: true,
          autoHide: false
      });
      Core.localStorage.set(tip1Id, new Date().getTime());//set localStorage
    }
  }
  function hideGuideTip1() {
    els.guideTip1 && els.guideTip1.hide();
  }
  function showGuideTip2() {
    var tip2Id = 'outfit_tip2_' + VIEW.models.Basic.getUserId();
    if(!els.guideTip2 && !Core.localStorage.get(tip2Id)) {
      els.guideTip2 = new Tooltip({
        body: 'Choose the item you like to see street snaps and other details.',
        top: 480,
        left: 40,
        rootCls: 'black',
        bodyCls: 'view-outfit-guide-tip',
        arrow: {
          type: 'top',
          position: 220
        },
        autoShow: true,
        autoHide: false
      });
      Core.localStorage.set(tip2Id, new Date().getTime());
    }
  }
  function hideGuideTip2() {
    els.guideTip2 && els.guideTip2.hide();
  }
  function showGuideTip3(){
    var el = els.styleMenu.find('.help'),
      tip3Id = 'outfit_tip3_' + VIEW.models.Basic.getUserId(),
      last =  Core.localStorage.get(tip3Id),
      timeout = !last || (new Date().getTime() - last) > 1000*60*60*24;
      //超时一天之后还会重新显示这个地方
    if(!els.guideTip3 && timeout){
      els.guideTip3 = new Tooltip({
        body: 'Can’t find a suitable outfit? Invite your friends to help you create more outfits. ',
        top: el.offset().top-200,
        left: 40,
        rootCls: 'black',
        bodyCls: 'view-outfit-guide-tip',
        arrow: {
          type: 'bottom',
          position: 180
        },
        autoShow: true,
        autoHide: false
      });
      Core.localStorage.set(tip3Id, new Date().getTime());
    }
  }
  function hideGuideTip3() {
    els.guideTip3 && els.guideTip3.hide();
  }
  function hideGuideTip(){
    hideGuideTip1();
    hideGuideTip2();
    hideGuideTip3();
  }

  function updateViewQuery(req) {
    viewQuery = req.query;
  }

  function updateStoreIds() {更新localStorage
    likeAlertId = 'outfit_first_like_' + VIEW.models.Basic.getUserId();
    firstLikeAlerted = Core.localStorage.get(likeAlertId);
    filterAlertId = 'outfit_first_filter_' + VIEW.models.Basic.getUserId();
    firstFilterAlerted = Core.localStorage.get(filterAlertId);
  }

  function render(data) {//每一次需要render都会进来，点一下change也会进来
    initResources();
    updateStoreIds();//render之前先要update localStorage

    if (!firstFilterAlerted) {//不是第一次call filter的话要set filter里面的数据
      firstFilterAlerted = true;
      Core.localStorage.set(filterAlertId, true);
      Core.Event.trigger('appAPI', 'styleFilter', VIEW.models.Style.outfits.filter.get(), beforeFilter, 'guide');
    }

    data = data || VIEW.models.Style.outfits.get();
    if (!data || !data.data || !data.data.length) {
      renderEmpty();//no data, display empty
      renderStyleLoading(true);//hide loading
      return;
    }

    els.mainData = data;主data对象
    els.mainStylesData = data.data;data数组
    els.mainStylesIndex = 0;//第几套outfits
    //以下都是初始化render
    resetTmpVals();//重置临时变量
    renderEmpty(true);//隐藏empty
    VIEW.showStyles();//显示outfits相关内容

    renderStyleDaily();//推荐outfits时的动画页面
    renderStyleInfo();//顶部说明，xxx new outfits found
    renderStyleBar(els.mainStylesData);//是like + try on那一条
    renderStyleOccasion(els.mainStylesData);//render occasion

    if (els.mainStylesData.length > 0) {//outfits套数大于零
      els.stylesSlider = els.stylesSlider || new Slider({//这个展示outfits还要用到slider plugin
          listEl: els.stylesList,
          enableDrag: true,
          enableLoop: false,
          onTouchend: function () {
            VIEW._BasicView.GlobalTouch.preventMove = false;
          },
          onTouchmove: function (x, y) {
            VIEW._BasicView.GlobalTouch.preventMove = true;//x > 5 && y < 30;
          },
          onMove: function (index) {
            renderStyleBarsNav(index);
            renderStyleOccasionNav(index);
          }
        });
      renderRemixBtn();//这个render btn就是判断一下这个change btn是正常还是disable状态
      renderNextStyle();主要是styleList那部分的渲染
    }
  }

  function renderStyleDaily(data){
    data = els.mainData && els.mainData.daily_recommend;
    if(data) {
      data.description = data.description || data.dscription;
      els.styleDailyBd.html(Tpl.styleDaily(data));
      els.styleDaily.removeClass('hide');
      setTimeout(function(){
        els.styleDaily.addClass('hide');
      },3800);
    }
  }

  function renderStyleInfo(hide){
    if(els.mainData){
      if(els.mainData.daily_recommend){
      // if daily_recommend field existed && not null, render num
        els.stylesInfo.html(Tpl.stylesInfo({num: els.mainStylesData.length}));
        //daily_recommend是object
        els.stylesInfo.removeClass('hide');
      }
      if(hide){
        els.stylesInfo.addClass('hide');
        //remove daily
        //styleinfo 和 daily是共存亡的
        els.mainData.daily_recommend = null;
      }
    }
  }

//几个状态条都是有lock状态的, 
//cause when pressing change btn, during the changing period, should keep every bar into forbidden status
//另，每一套outfit都对应着自己的style bar和occasion，渲染的时候要一次性全都渲染完，再在每次next的时候都判断index保证一一对应
  function renderStyleBar(data) {
    renderStyleBarLock(true);
    var htm = [],
      appendFn = VIEW.models.Style.outfits.page ? 'append' : 'html';
    data.forEach(function (key) {
      var ids = [];
      key.clothes.forEach(function (key) {
        ids.push(key.id);
      });
      key._itemIds = ids.join(',');
      htm.push(Tpl.styleBar(key));
    });
    els.styleBars[appendFn](htm.join(''));
  }

  function renderStyleBarsNav(idx) {
    var curEl;
    els.styleBarListEls = els.styleBarListEls || els.styleBars.children();
    els.styleBarListEls.addClass('hide');
    curEl = els.styleBarListEls.eq(idx);
    curEl.removeClass('hide');
    //els.lastOutfitOccasionName = curEl.attr('data-occasion');
    //原本改名称要取到场景名，才有这个变量这个属性，现在不需要
  }
  function renderStyleOccasion(data){
    renderStyleOccasionLock(true);
    var htm =[],
      appendFn = VIEW.models.Style.outfits.page ? 'append' : 'html';
    data.forEach(function (key) {
      key.occasions = key.occasions || [];
      htm.push(Tpl.styleOccasion(key));
    });
    els.styleOccasion[appendFn](htm.join(''));
  }
  function renderStyleOccasionNav(idx){
    var curEl;
    els.styleOccaListEls = els.styleOccaListEls || els.styleOccasion.children();
    els.styleOccaListEls.addClass('hide');
    curEl = els.styleOccaListEls.eq(idx);
    curEl.removeClass('hide');
    //els.lastOutfitOccasionName = curEl.attr('data-occasion');
  }
  function renderStyleOccasionItem(){
    var el = $(this),
      pel = el.parents('.occasion'),
      names = pel.find('.occasion__names .item'),
      idx = el.index();
    names.addClass('hide');
    names.eq(idx-1).removeClass('hide');
    els.styleOccasionItemTimer && clearTimeout(els.styleOccasionItemTimer);
    //这是为了防止快速点击造成的显示延迟，所以存在styleOccasionItemTimer的时候要清除那个延时
    els.styleOccasionItemTimer = setTimeout(function(){
      names.addClass('hide');
    },1500);
    //延迟1.5m隐藏occasion name
  }

  function renderNextStyle() {
    //0. hide guide
    if (els.isRenderNextStyle) {
      return;
    }
    els.isRenderNextStyle = true;
    var appendFn = els.mainStylesIndex ? 'append' : 'html',
      item = els.mainStylesData[els.mainStylesIndex],
      nextItem = els.mainStylesData[els.mainStylesIndex + 1],
      afterNextItem = els.mainStylesData[els.mainStylesIndex + 2];

    if (item) {
      //1. append outfit style
      var htm = VIEW._StyleTemplateView.getStylesHtm([item]);
      els.stylesList[appendFn](htm);
      //2. show cover
      renderStyleCover(htm);
      renderStyleLock(true);
      setTimeout(function () {
        renderStyleLoading();
      }, 50);
      (!viewQuery || !viewQuery.cloth_id) && setTimeout(function(){
        renderStyleCover();
      },640);
      //4. animate stars
      setTimeout(function () {
        //5. show outfit style
        els.isRenderNextStyle = false;
        els.stylesList.find('.slider-current').addClass('animated');
        //6. show guide
        showGuideTip1();
        if(els.mainStylesIndex>4){
          showGuideTip3();
        }
        //7. hide stars & hide cover
        renderStyleCover();
        setTimeout(function () {
          renderStyleLock();
          renderStyleLoading(true);
        }, 350);
      }, 1000);

      //3. move to next outfit
      if (els.mainStylesIndex) {
        els.stylesSlider.refresh();
        els.stylesSlider.last();
      } else {
        els.stylesSlider.reset();
      }
      els.mainStylesIndex++;
      if (els.mainStylesIndex > els.mainStylesData.length - 1) {
        renderRemixBtn(true);
      }

      //pre load images
      [].concat(afterNextItem ? afterNextItem.clothes : [], nextItem ? nextItem.clothes : [], item.clothes).forEach(function (key) {
        var img = new Image();
        img.src = key.image_trans_crop + '/360.png';
      });
    }
    Core.Event.trigger('beforeRequestStyleReviewHistory', item);
  }

  function renderStyleCover(htm) {
    els.stylesList.find('.c-style-tpl__item.hide').removeClass('hide');
    if(htm){
      els.stylesList.addClass('frameless');
    }else{
      els.stylesList.removeClass('frameless');
    }
    if (viewQuery && viewQuery.cloth_id && htm) {
      els.stylesCover.removeClass('hide');
      els.stylesList.find('.c-style-tpl__item[data-id="' + viewQuery.cloth_id + '"]').addClass('hide disanimated');
      var dataid = '[data-id="' + viewQuery.cloth_id + '"]',
        last = els.stylesCover.children(),
        lastItem = last.find('.c-style-tpl__item' + dataid),
        cur = $(htm),
        curItem = cur.find('.c-style-tpl__item' + dataid);

      if (lastItem[0]) {
        setTimeout(function () {
          last[0].className = cur[0].className;
          lastItem[0].className = curItem[0].className;
        }, 1000);
      } else {
        cur.find('.c-style-tpl__content').html(curItem[0]);
        els.stylesCover.html(cur);
      }
    } else {
      els.stylesCover.addClass('hide');
    }
  }

  function renderStyleLoading(hide,options) {
    if (hide) {
      renderStyleLoadingStars();
      els.stylesLoading.addClass('hide');
    } else{
      if(!options){
        var minRatio = 0,
          maxRatio = 100;
        if (viewQuery && viewQuery.cloth_id) {
          var idx = els.stylesList.find('.c-style-tpl__item[data-id="' + viewQuery.cloth_id + '"]').index();
          if (idx > 0) {
            maxRatio = 65;
          } else {
            minRatio = 50;
            maxRatio = 100;
          }
        }
        options = {minRatio: minRatio,maxRatio: maxRatio};
      }
      renderStyleLoadingStars(options);
      els.stylesLoading.removeClass('hide');
    }
  }

  function renderStyleLoadingStars(options){
    options = options || {};

    els.stylesLoadingBd.html('');
    var i = 0,
      len = 10,
      htm = [],
      minRatio = options.minRatio || 0,
      maxRatio = options.maxRatio || 100,
      infinite = options.infinite;

    for (i; i < len; i++) {
      htm.push(Tpl.stylesLoading({
        top: Math.random() * 100,
        left: Core.Num.randomArbitrary(minRatio, maxRatio),
        delay: 0.1 * i,//Math.min(Math.random(),0.2),
        duration: 0.3//+Math.min(Math.random(),0.2)
      }));
    }

    els.stylesLoadingBd.html(htm.join(''));

    els.styleLoadingStarsTimer && clearTimeout(els.styleLoadingStarsTimer);
    els.styleLoadingStarsTimer = !infinite?0:setTimeout(function(){
      renderStyleLoadingStars(options);
    },1200);
  }
  function renderEmpty(hide){
    els.stylesEmpty[hide?'addClass':'removeClass']('hide');
    renderRemixBtn(true);
  }

  function renderStyleBarLock(lock) {
    els.styleBars[lock ? 'addClass' : 'removeClass']('lock');
  }
  function renderStyleOccasionLock(lock) {
    els.styleOccasion[lock ? 'addClass' : 'removeClass']('lock');
  }
  function renderStyleMenuLock(lock) {
    els.styleMenu[lock ? 'addClass' : 'removeClass']('lock');
  }
  function renderRemixBtn(disabled){
    els.styleMenu.find('.remix')[disabled?'addClass':'removeClass']('disabled');
  }

  function renderStyleLock(lock){
    renderStyleBarLock(lock);
    renderStyleOccasionLock(lock);
    renderStyleMenuLock(lock);
  }

  function renderFilterTags(data) {
    initResources();

    data = data || VIEW.models.Style.outfits.filter.get();
    renderFilterBtn(data && data.tags && data.tags.length);
    return;
    if (!data || !data.tags) {
      els.filterTags.addClass('hide');
    } else {
      var htm = [];
      data.tags.forEach(function (key) {
        htm.push(Tpl.filterTag(key));
      });
      els.filterTagsList.html(htm.join(''), true);
      els.filterTags[htm.length ? 'removeClass' : 'addClass']('hide');
    }
  }

  function renderFilterBtn(isOn){
    els.styleMenuFilter[isOn ? 'addClass' : 'removeClass']('on');
  }
  function resetTmpVals() {
    els.isLoadingNextStyleList = false;
    els.styleBarListEls = null;
    els.styleOccaListEls = null;
  }
  function removeFilterTag() {
    var el = $(this),
      id = el.attr('data-id');
    VIEW.models.Style.outfits.filter.removeTagById(id);
    Core.Event.trigger('beforeRequestOutfits', VIEW.models.Style.outfits.filter.get());
  }
  function onToggleLike() {
    var el = $(this),
      ids = el.attr('data-id'),
      isLike = !el.hasClass('on');
    el.toggleClass('on');
    if (isLike) {
      Core.Event.trigger('StyleBookController.beforePostLikeStyle', ids);
      if (!firstLikeAlerted) {
        VIEW._BasicView.msgbox.showDialog({
          msg: 'Excellent! You can view it in your Favourites > Located at the sidebar in your wardrobe.',
          yesText: 'OK'
        });
        firstLikeAlerted = true;
        Core.localStorage.set(likeAlertId, 1);
      }
    } else {
      Core.Event.trigger('StyleBookController.beforePostUnLikeStyle', ids);
    }
  }

  function beforeFilter(data) {
    if (data) {
      Core.Event.trigger('beforeRequestOutfits', data);
    }
  }

  function beforeAddMission(){
    var sub = [],
      occasion = VIEW.models.Style.outfits.filter.getIds();
    //viewQuery.cloth_id && sub.push('ids='+viewQuery.cloth_id);
    occasion && sub.push('occasion='+occasion);

    Core.Event.trigger('appAPI', 'addMission', null, null, sub,0);
  }

}//end View
module.exports = new OutfitsView();
```
