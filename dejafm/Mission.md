### myMissionList
>mine

之前为了分页第零页past list不render第一个，是这么做的，判断是不是第一个object，不是才渲染
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
但是觉得不是个最合理的办法，于是改了一下，就是正常渲染了之后把第一个从DOM里面删掉

```javascript
data.data.forEach(function(key){
     htm.push(Tpl.missionBd(key));
 });
 els.missionBd[appendFn](htm.join(''));  
if(!VIEW.models.Mission.myMissionList.page){ 
  els.missionBd.find('.item').eq(0).remove(); 
  }
```


>lulu‘s new code

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

  function renderEmpty(empty) {
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

  function onAddMission(){
    Core.Event.trigger('appAPI', 'addMission', null, function(){
      VIEW.models.Mission.myMissionList.timer.reset();
      Core.Router.run();
    });
  }
```
