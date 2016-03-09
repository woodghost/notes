# Notes
Taking notes via learning, in this case especially in dejafm encountered problems.

###### 2016.3.9
```javascript
var tipId = 'cloth_tip_' + VIEW.models.Basic.getUserId();
    if(!els.guideTip && !Core.localStorage.get(tipId)){
      els.guideTip = new Tooltip({
        body: '<i class="icon icon-arrow"></i>' + '<p>scroll up to see the street snaps</p>',
        bottom: 30,
        left: 140,
        rootCls: 'black',
        bodyCls: 'view-cloth-guide-tip box-h-c-c',
        autoShow: true,
        showDelay: 1000,
        autoHide: false
      });
      Core.localStorage.set(tipId, new Date().getTime());
    }
```

`var tipId = 'cloth_tip_' + VIEW.models.Basic.getUserId();` and `Core
.localStorage.set(tipId, new Date().getTime());` are for cache storage.
