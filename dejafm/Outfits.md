Codes in outfits
================

#### Model
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
      var d = this.data,ids = [];
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


#### Controller
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
      CTRL.views.Outfits.hideStyles();
      //CTRL.views.Basic.msgbox.showLoading();
      CTRL.models.Style.outfits.resetPage();
      beforeRequestOutfits();
    }

    CTRL.views.Outfits.show();

    //追加统计
    analyticsCurView();
  }

  function updateOutfitFilter(options){
    if (options) {
      CTRL.models.Style.outfits.filter.set(options, cur__STORE_ID);
    }else{
      CTRL.models.Style.outfits.filter.set(null);
    }
  }

  function beforeRequestOutfits(options, reload) {
    var data = {
      __STORE_ID: cur__STORE_ID,
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
  }

  function afterRequestOutfits(success) {
    CTRL.views.Basic.msgbox.hideLoading();
    var data = CTRL.models.Style.outfits.get();
    if (!success || !data || data.ret != 0) {
      CTRL.views.Basic.msgbox.showFailed({
        msg: success && data.msg
      });
    }else{
      //set filter
      if(data.daily_recommend){
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
    var ids = [];
    item.clothes.forEach(function(key){
      ids.push(key.id);
    });
    var data = {
      cloth_ids: ids.join(',')
    };
    CTRL.models.Style.styleReviewHistory.request(data)
  }

  function forwardOutfits(arg) {
    Core.Router.forward('/outfits/' + (arg || ''));
  }

  function analyticsCurView(params, title) {
    if (!Core.Router.currentMatch(['/outfits/'])) {
      return;
    }
    params = params ? ('&' + params) : '';
    title = title || viewNames[curViewId] || document.title;

    Core.Event.trigger('analytics', 'viewid=' + curViewId + params, title);
  }
}
module.exports = new OutfitsController;
```
