## myMissionList
>mine
我最早写的那些还是被删掉了，今后应该在这里做个备份，会比较方便。
之前为了分页第零页past list不render第一个，是这么做的，判断是不是第一个object，不是才渲染
其实这个办法比下面那个合理啊，因为不用渲染很多次
这个思路是对的，但是代码写的渣，可以参考底下lulu写的好的代码
```javascript
if(VIEW.models.Mission.myMissionList.page) { 
  data.data.forEach(function(key) {
       htm.push(Tpl.missionBd(key));
     });
   }else {
     data.data.forEach(function(key,idx) {
       if(idx!=0){
         htm.push(Tpl.missionBd(key));
       }   
  }); 
  }
```
这是改了一下之后写的，就是正常渲染了之后把第一个从DOM里面删掉，这样做性能不好
确实是要考虑页面性能的……
你分页之后再删除确实不如判断一下第一页的话past不渲染第一个。
```javascript
data.data.forEach(function(key){
     htm.push(Tpl.missionBd(key));
 });
 els.missionBd[appendFn](htm.join(''));  
if(!VIEW.models.Mission.myMissionList.page){ 
  els.missionBd.find('.item').eq(0).remove(); 
  }
```



然而后来改了逻辑，之前的太慢效率太低
>lulu‘s new code

总体来说，就是改了渲染方式为先渲染第一个,只有一个的时候past tasks这个title就不显示，什么都没有的时候显示no task
底下的备注打得非常清楚了，理解就好，学着写
而且你之前做的是past那一栏没东西为空，那样并不十分合理，要考虑实际情况，不能不考虑ux就直接写了。
```javascript
function render(data) {
    initResources();
    data = data || VIEW.models.Mission.myMissionList.get();

    if(!data || data.ret!=0 || !data.data || !data.data.length){
      renderEmpty(true);
      return;
    }
    var htm = [],
      firstPage = !VIEW.models.Mission.myMissionList.page,
      appendFn = !firstPage ? 'append' : 'html';

    //only render current when it's first page
    firstPage && renderFirstMission(data.data[0]);

    data.data.forEach(function(key,idx){
      if(firstPage){
        //first page remove first mission from past mission
        idx>0 && htm.push(Tpl.missionBd(key));
      }else{
        htm.push(Tpl.missionBd(key));
      }
    });
    if(htm.length>0){
      //only one mission hide past mission
      els.pastTitle.removeClass('hide');
      els.missionBd.removeClass('hide');
      els.missionBd[appendFn](htm.join(''));
    }
    //first page and only one mission, hide list end
    if(firstPage && data.data.length==1){
      els.missionEnd.addClass('hide');
    }else{
      els.missionEnd.removeClass('hide');
      renderListEnd(els.missionEnd, data.end? '.end' : '.more' );
    }
    renderEmpty();
  }//end render

  function renderFirstMission(data){
    //use the large image
    els.firstMission.html(Tpl.firstMission(data));
  }

  function renderEmpty(empty) {//这个renderEmpty跟我之前写的一毛一样，改成了要传个参数的
    if(empty){
      els.emptyList.removeClass('hide');
      els.myMissionList.addClass('hide');
    }else{
      els.emptyList.addClass('hide');
      els.myMissionList.removeClass('hide');
    }
  }

  function renderListEnd(el,cls){
    el.children().removeClass('show');
    el.find(cls).addClass('show');
  }
```

这个onAddMission是与native交互的一处，在右上角，要在一进入页面的show 
```javascript
  function onAddMission()
  {//这个onAddMission是与native交互的一处，在右上角，要在一进入页面的show 
  //function里面调，这里客户端每调一次 回调 myMIssionList timer都要reset一下，页面要刷新一次。
    Core.Event.trigger('appAPI', 'addMission', null, function(){
      VIEW.models.Mission.myMissionList.timer.reset();
      Core.Router.run();
    });
  }
  
  
  function appAPI(name, data, callback, subProtocol, redirect) {
      if (isApp) {
        Core.NativeBridge.trigger.apply(null, arguments);
      } else if (redirect) {
        var proto = [name];
        subProtocol && proto.push(subProtocol);
        redirectToDownload(null, true, Actions.dejafashionSchema + proto.join('/'));
      }
    }
    
  
```



## mission styling
之前的share style点击按钮的逻辑是
```javascript
function beforeAppDownload(){
    var url = Actions.dejaAppAndroid;
    if ($.os.ios && !$.os.android) {
      url = Actions.dejaAppIos;
    }
    window.location = url;
  }
```
后来写成一个event function在Controller.js里面
```javascript
function bindEvent() {
    els.accountInfo.on(tap, '.download-btn',function(){
      // Core.Event.trigger('openAppInStore', Actions.dejafashionSchema+'wardrobeDetail/'+viewQuery.id);
      Core.Event.trigger('openAppInStore');
    })

  }//end bindEvent
  
  
  
Controller.js里的代码
  Core.Event.on('openAppInStore', downloadDejaInApp);


function downloadDejaInApp(schema) {
    var url = Actions.dejaAppAndroid;
    if ($.os.ios && !$.os.android) {
      url = Actions.dejaAppIos;
      schema && redirectToPage(schema);
    }else{
      schema && Core.Navigator.protocol(url, true);
    }
    if(ThirdVendor && ThirdVendor.code=='WX'){
      redirectToPage(Actions.dejaAppQQ);
    }else{
      setTimeout(function () {
        redirectToPage(url);
      }, 50);
    }
  }
  
  
  function redirectToPage(link) {
      if (link) {
        !(/__NativeBridge_target/g.test(link)) && appActionDefaultButton();
        window.location = link;
      }
    }
    
    
    function appActionDefaultButton() {
        appActionButton('', function () {
        });
      }
```
