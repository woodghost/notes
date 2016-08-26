## JSONP, CORS, advanced JSON analyzing & concat

> 跨域技术, 处理跨域请求, json拼接

这是最早在newList project里面处理movie list用到的，deja website里面blog preview也用到。
最大的方便是可以直接用DDMS编辑后台数据，先不用server开发相应的api
系统的过了一遍如何解析DDMS里请求到的jsonp data并拼接成容易解析的json数据结构

这种jsonp方式url里带一个表单id，每换一个id都能读不同的表单信息


```javascript
function render(data) {
    initResources();
    data = data || VIEW.models.StreetSnap.inspireDetail.get();

    if (!data || data.code!=1 || !data.data) {
      return;
    }
    var newData = {
      slider: null,
      product: [],
      desc: []
      },
      finalData = {
        id: data.data._id,
        title: data.data.title,
        desc: data.data.desc,
        data: []
      };

    //pre format
    function formatSchemata(mata){
      var s = {};
        s.title = mata.title;
      mata.child.forEach(function(key){
        s[key.name] = key.title;
      });
      return s;
    }

    function formatSchemataSlider(mata){
      var s = {};
      mata.child.forEach(function(key){
        s[key.name] = key.title;
      });
      return s;
    }

    data.data.schemata.forEach(function(key){
      if(key.name=='slider'){
        newData.slider = formatSchemataSlider(key);
      }
      key.name=='product' && newData.product.push(formatSchemata(key));
      key.name=='desc' && newData.desc.push(formatSchemata(key));
    });

    //console.log(newData);
    //final format
    function getProducts(sliderKey){
      var products = [];
      newData.product.forEach(function(key){
        if(key.slider_key==sliderKey){
          products.push(key);
        }
      });
      return products;
    }
    function getDesc(sliderKey){
      var desc = '';
      newData.desc.forEach(function(key){
        if(key.slider_key==sliderKey){
          desc = key.title;
        }
      });
      return desc;
    }
    for(var key in newData.slider){
      var slider = {image: newData.slider[key],clothes: [], desc: ''};
      slider.clothes = getProducts(key);
      slider.desc = getDesc(key);
      finalData.data.push(slider);
    }
    data = finalData;
    mainData = data;
    resetTmpVals();
    var htm = [];
    data.data.forEach(function (key, idx) {
      key.id = idx;
      if (idx < 2) {
        var IMG = new Image();
        IMG.src = key.image + '/750.jpg'
      }
      htm.push(Tpl.inspireDetailList(key));
    });

    els.inspireDetailTitle.html(data.title);
    els.inspireDetailList.html(htm.join(''));
    !isApp && els.inspireDetailTitle.removeClass('hide');

    renderProduct(data.data);
    renderTextBar(data.data);

    if (data.data.length > 0) {
      els.inspireSlider = els.inspireSlider || new Slider({
          listEl: els.inspireDetailList,
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
            //loadImgs(index);
            //VIEW._BasicView.lazyLoadImg(els.inspireDetailList);
            renderProductListNav(index);
            renderTextBarNav(index);
            renderNavArrow(index);
          }
        });
    }
    els.inspireSlider.reset();

    VIEW._BasicView.renderShare({
      title: 'Deja',
      text: data.title,
      summary: data.title,
      imageurl: data.data.length>0? data.data[0].image:null,
      thumburl: null,
      link: Actions.main +'#/inspiredetail/id='+ data.id
    });

  }//end render

  function renderTextBar(data) {
    var htm = [],
      total = data.length;
    data.forEach(function (key,idx) {
      key.total = total;
      key.desc = key.desc || null;
      key._idx = idx+1;
      key.total = total;
      htm.push(Tpl.textBar(key));
    });
    els.textBar.html(htm.join(''));
  }
  function renderTextBarNav(idx) {
    els.textBarListEls = els.textBarListEls || els.textBar.children();
    els.textBarListEls.addClass('hide');
    els.textBarListEls.eq(idx).removeClass('hide');
  }

  function renderProduct(data) {
    var htm = [];
    data.forEach(function (key) {
      if(!key.clothes){
        return;
      }
      key.clothes.forEach(function(key){
        //key.name = (key.name.length>15?(key.name.substr(0, 15)+'...'):key.name) || '';
      });
      htm.push(Tpl.productListBd(key));
    });
    els.productListBd.html(htm.join(''));
  }

  function renderProductListNav(idx) {
    els.productListEls = els.productListEls || els.productListBd.children();
    els.productListEls.addClass('hide');
    els.productListEls.eq(idx).removeClass('hide');
  }

```

