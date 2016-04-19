## message list

#### View
```javascript
function bindEvent() {
    els.outfits.on('touchstart',hideOutfits);
    els.msgListBd.on(tap,'.mission',function () {
      var missionId = this.getAttribute('data-missionId');
      if(missionId!=0){
        Core.Event.trigger('forwardMissionStyleDetail','mission_id='+missionId);
      }else{
        showOutfits.call(this);
      }
    });
    els.msgListEnd.on(tap, '.more', function () {
      els.msgListBd.find('.item.hide').removeClass('hide');
      Core.Event.trigger('MessageListController.beforeRequestMessageList');
      renderListEnd(els.msgListEnd, '.loading');
    });
  }//end bindEvent

  this.show = function () {
    initResources();

    Core.Event.trigger('trigerAnimate',els.main);
    VIEW._BasicView.show(VIEW.viewCls);
  }
  this.hide = function () {
    if (!els) {
      return;
    }
    hideOutfits();
  }
  function showOutfits(){
    els.outfits.removeClass('hide');
    renderOutfit(this.getAttribute('data-idx'));
  }
  function hideOutfits(){
    els.outfits.addClass('hide');
  }

  function render(data) {
    initResources();
    data = data || VIEW.models.Message.messageList.get();

    if(!data || data.ret != 0 || !data.data){
      return;
    }
    var list = [],
      appendFn = VIEW.models.Message.messageList.page ? 'append' : 'html';
    els.mainMsgs = VIEW.models.Message.messageList.page?els.mainMsgs:[];

    data.data.forEach(function(key) {
      key.old = VIEW.models.Message.messageList.page?'':(key.new?'':'hide');
      key._idx = els.mainMsgs.length;
      key.mission_id = key.mission_id || 0;
      key.user_info.avatar =  !!key.user_info.avatar?key.user_info.avatar:Actions.dejaUserAvatar;
      key._mission_body = key.style && VIEW._StyleTemplateView.getStylesHtm([key.style]);
      list.push(Tpl.msgListItem(key));
      els.mainMsgs.push(key);
    });
    els.msgListBd[appendFn](list.join(''));
    renderListEnd(els.msgListEnd, data.end ? '.end' : '.more');
  }//end render

  function renderOutfit(idx){
    var data = els.mainMsgs[idx];
    if(data.style){
      els.stylesList.html(VIEW._StyleTemplateView.getStylesHtm([data.style]));
      els.occasion.html(Tpl.occasion(data.style));
    }
  }
  function renderListEnd(el,cls){
    el.children().removeClass('show');
    el.find(cls).addClass('show');
  }
```
