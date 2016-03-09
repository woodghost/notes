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

### About auto-loading pictures

```javascript
    this.resizeCalculateWindow = function () {
        els.window = $(window);
        els.body = $('body');
        setTimeout(function () {
          els.bodyHeight = els.body.height();
          els.windowMaxScroll = els.bodyHeight - els.window.height() * 2;
        }, 300);
      }
    
      this.isMaxWindowScroll = function (budget) {
        var top = Math.min(Math.min(window.pageYOffset, document.documentElement.scrollTop || document.body.scrollTop), window.scrollY),
          maxScroll = budget ? (els.bodyHeight - budget) : els.windowMaxScroll;
    
        return top > maxScroll;
      }
```
1. If you are trying to find out why the code segment do not work, try
 to put more `console.log()` in ur functions and to watch whether 
 they print or not.
 
```javascript
 function onBeforeLoadingNextPageClothStreet(scroll) {
     if (els.isViewVisible && !els.isLoadingNextPageClothStreet) {
       if (scroll) {
         if (VIEW._BasicView.isMaxWindowScroll()) {
           console.log(1);
           loadingNextPageClothStreet();
         }
       } else {
         loadingNextPageClothStreet();
       }
     }
   }
```
 Try to use `console.log(scroll)` or  `console.log(top, maxScroll)` to 
 discover what stuffs are in these arguments.(Just because the 
 function didn't come from my code :( ).
 
2. Put logs in appropriate locations to detect which sentence is not 
executed(e.g is in above segment)

3. Need to adopt into favorite outfits list, should change all 
functions' and parameters' names.
![screen shot](img/shot.jpg)
