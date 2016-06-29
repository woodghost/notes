# Dejafm stylesheet
Taking notes via encountered SCSS, CSS3, HTML5, problems in dejafm ...(adding in process)

## Cloth module updating

## Compendium issues in dejafm

### scss
----------------------------------
20160629
About slider used in street snap display
if set min-height and make box-align, box-pack, box-orient all center,
the street snap will stay in middle.



----------------------------------

- [x] 列表结构问题
product-item-waterfal这个列表，clearfix, 不能打乱层次结构，一打乱底下list-row就出问题，first-child就没有margin right

* 深刻的检讨一下列表结构的问题

> 为什么会有这种问题呢？放在list的body里面就看不见list end， 放在外面呢又隐藏不了……
> 这是列表结构的问题，script那个模板层和它外面包裹的那一层一定要连在一起

如果结构不对会造成list end看不见，整个列表层级结构有问题
正确的方式是一定要在`<script>`标签外面包一层，这一层的`data-element`名称最好同script的`data-template`命名相同
```html
<section class="tabs-content wish-list tabs-content-default show" data-element="wishList">
    <div class="bd" data-element="wishListBd">
      <section class="list product-item-block" data-element="wishListItem">
        <script type="text/html" data-template="wishListItem">
          <article class="item" data-id="<%=id%>" data-fake-link="#/cloth/id=<%=id%>">
            <div class="img"><img class="lazy animated" data-src="<%=image%>/420.jpg"></div>
            <div class="detail">
              <div class="info">
                <div class="name"><%=name%></div>
                <div class="brand ellipsis"><%=brand_info.name%></div>
                <div class="meta">
                  @@include("include/cloth-price-list.html")
                </div>
              </div>
            </div>
          </article>
        </script>
      </section>
      <!-- list-end-default -->
      @@include("include/list-end-default.html")
      <!-- end list-end-default -->
    </div>
    <section class="empty hide" data-element="emptyList">
      <p>You have no item in wishlist.</p>
      <p>Start by adding an item in your fitting room or save an item by clicking the heart icon.</p>
    </section>
  </section>
```

new version detail page
怎么说呢，有很多我根本不太会的交互，e.g.:
- [x] 滑动到tab triple bar的位置固定在顶端；
固定这个tab bar有很多细节，可以给parent一个高度，这样就不会抖动了，也不用jQuery加个padding这种不够好的方式了
- [x] tab上面有noti气泡，点到tab的时候消失；
- [x] 这里的msgbox得我重新做


----------------------------------
牵扯到三个部分的model
特别是product部分，add to wish list等于添加这件商品去想买列表

想要合并post like，看missionstyledetail的controller
!!! tabs 千万不要在最外层写display样式，会破坏tab的结构，导致显示不正常。

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

## html & scss (sub detail)
隐藏scroll bar的方法：比父层元素height大20px，刚好可以遮挡scroll bar，总之就看不到scroll bar了
```scss
.pic{
      $pic-h: 240px;
      height: $pic-h;
      overflow: hidden;
      .pic-bd{
        @include box-h-c-c;
        height: $pic-h+20;//比父层元素height大20px
        overflow: hidden;
        overflow-x: scroll;
        -webkit-overflow-scrolling: touch;
      }
    }
```
