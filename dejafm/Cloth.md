## cloth detail & sub detail
全新改版之后增加了很多功能和逻辑，重新整理
v16.05.16 today going on
### View

.....

### scss
隐藏scroll bar的方法：比父层元素height大20px，刚好可以遮挡scroll bar，总之就看不到scroll bar了
```css
.pic{
      $pic-h: 240px;
      height: $pic-h;
      overflow: hidden;
      .pic-bd{
        @include box-h-c-c;
        height: $pic-h+20;
        overflow: hidden;
        overflow-x: scroll;
        -webkit-overflow-scrolling: touch;
      }
    }
```
