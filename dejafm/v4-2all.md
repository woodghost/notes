## v4.2 All Modules
### Structure
1. 

### scss
1. 全局复用的样式才能写在大模块外面，其余的不带‘>’号写里面整个模块内部就可以复用了
2. title style可以用text relative，一条line absolute这种情况，并且line的z-index比title小就可以了，可以不用写三块
3. invite-bg直接写最外层，js里面渲染的时候加上这个class


### javascript
1. renderShare在View.js里面
2. 有msgbox的地方就要想到hide view的时候要把它隐藏掉

### 改版前一版celebrity list UI

```scss
@charset "UTF-8";
.view-celebritylist {
  width: 100%;
  height: 100%;
  min-height: 100%;
  background-color: #fff;
  font-size: 32px;
  > .celebritylist {
    @include box-h-c-c;
    width: 100%;
    height: 100%;
    padding-left: 46px;
    .list{
      @include box-v;
      width: 100%;
      height: 100%;
      .bd{
        width: 100%;
        height: 100%;
        >.item{
          @include box-v;
          padding-bottom: 20px;
          .info{
            @include box-h-c-l;
            padding: 20px 0;
            .avatar{
              width: 70px;
              height: 70px;
              img{
                width: 100%;
                height: 100%;
                -webkit-border-radius: 100%;
              }
            }
            .meta{
              @include box-v-c-l;
              padding-left: 20px;
              .name{
                font-size: 30px;
                color: $color-black-fourth;
              }
              .item-num{
                font-size: 26px;
                color: $color-black-third;
              }
            }
          }
          .desc{
            font-size: 28px;
            color: $color-black-third;
            padding-bottom: 14px;
          }
          .clothes{
            $clothes-h: 222px;
            @include box-h;
            width: 100%;
            height: $clothes-h;
            overflow: hidden;
            .bd{
              @include box-h;
              overflow: hidden;
              width: 100%;
              height: $clothes-h +20;
              overflow-x: scroll;
              -webkit-overflow-scrolling: touch;
              .cloth-list{
                @include box-h;
                height: $clothes-h;
                .imgs{
                  @include box-h-c-c;
                  width: 220px;
                  height: 220px;
                  overflow: hidden;
                  border: 1px solid $color-gray-frame;
                  border-right: 0;
                  &:last-child{
                    border-right: 1px solid $color-gray-frame;
                    margin-right: 46px;
                  }
                  img{
                    display: inherit;
                    max-width: 100%;
                    max-height: 100%;
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```




