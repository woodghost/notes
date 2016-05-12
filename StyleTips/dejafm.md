# notes
Taking notes via encountered SCSS, CSS3, HTML5, problems in dejafm ...(adding in process)

## Cloth module

- [ ] 列表结构问题
product-item-waterfal这个列表，clearfix, 不能打乱层次结构，一打乱底下list-row就出问题，first-child就没有margin right

————————————————
new version detail page
怎么说呢，有很多我根本不太会的交互，e.g.:
- [x] 滑动到tab triple bar的位置固定在顶端；
固定这个tab bar有很多细节，可以给parent一个高度，这样就不会抖动了，也不用jQuery加个padding这种不够好的方式了
- [x] tab上面有noti气泡，点到tab的时候消失；
- [x] 这里的msgbox得我重新做
- [ ] 还有就是调API

其实也就这些

——————————----------
牵扯到三个部分的model
特别是product部分，add to wish list等于添加这件商品去想买列表

想要合并post like，看missionstyledetail的controller
tabs 千万不要在最外层写display样式，会破坏tab的结构，导致显示不正常。

tab们的red dot notification

```html
<section class="tabs-title-triple tabs">
      <div class="first on" data-element="firstTab"><p>Street Snaps</p></div>
      <div class="second" data-element="secondTab">
        <p>Outfits<span class="nd-tab">(7)</span></p>
        <!--<div class="notify nd-tab">7</div>-->
      </div>
      <div class="last" data-element="lastTab">
        <p>Best Match<span class="rd-tab">(999+)</span></p>
        <!--<div class="notify rd-tab">3600</div>-->
      </div>
    </section>
```

scss样式保留，html先注销，有可能用得到。
