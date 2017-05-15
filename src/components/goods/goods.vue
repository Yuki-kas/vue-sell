<template>  <!-- 商品列表组件 -->
  <div class="goods">
    <div class="menu-wrapper" v-el:menu-wrapper>    <!-- 获取dom -->
      <ul>
        <li v-for="item in goods" class="menu-item" :class="{'current':currentIndex===$index}"
            @click="selectMenu($index,$event)">
          <!-- 遍历这个li后得到的索引值$index是否等于currentIndex这个方法返回的值，相等的话就添加current这个class样式 -->
          <span class="text border-1px">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" v-el:foods-wrapper>    <!-- 获取dom -->
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li v-for="food in item.foods" class="food-item border-1px">
              <div class="icon">
                <img width="57" height="57" :src="food.icon">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}</span><span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span><span v-show="food.oldPrice"
                                                                class="old">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart v-ref:shopcart :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
    <!-- 通过props将这两个数据传递到shopcart组件中 -->
  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import shopcart from 'components/shopcart/shopcart';
  import cartcontrol from 'components/cartcontrol/cartcontrol';

  export default {
    props: {
      seller: {   // 这里接收的seller需要在App.vue里的router-view组件中传入，不然会在shopcart（购物车）组件调用时报Undefined
        type: Object
      }
    },
    data() {
      return {
        goods: [],
        listHeight: [],    // 存储_calculateHeight()方法中的数据
        scrollY: 0    // 监听用
      };
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];   // 头
          let height2 = this.listHeight[i + 1];    // 尾
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {    // 判断滚动时的scrollY的值是否落在头尾之间
            console.log(height1);
            console.log(height2);
            console.log(i);
            return i;
          }
        }
        return 0;
      },
      selectFoods() {   // 计算所选商品
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];

      this.$http.get('/api/goods').then((response) => {   // 获取数据
        response = response.body;
        if (response.errno === 0) {   // 判断是否成功
          this.goods = response.data;
          this.$nextTick(() => {    // 更改数据后更新的DOM
            this._initScroll();
            this._calculateHeight();
          });
        }
      });
    },
    methods: {
      selectMenu(index, event) {   // 传一个索引值index，来得知点击的是哪一个li;传event来阻止原生的点击事件
        if (!event._constructed) {    // 原生的点击事件是没有_constructed这个属性的，因此用它来判断并阻止原生的点击事件
          return;
        }
        let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook');   // 拿到food-list，一个数组
        let el = foodList[index];   // 具体点击到的是哪个li
        this.foodsScroll.scrollToElement(el, 300);    // better-scroll的接口，谁（this.foodsScroll）用多少时间（300）滚到指定地点（el）
        console.log(el);
      },
      _drop(target) {
          // 优化体验，异步执行下落动画
          this.$nextTick(() => {
            this.$refs.shopcart.drop(target);
          });
      },
      _initScroll() {
        this.meunScroll = new BScroll(this.$els.menuWrapper, {    // 拿到dom
          click: true   // better-scroll这个插件中默认是关闭click事件的
        });

        this.foodsScroll = new BScroll(this.$els.foodsWrapper, {    // 拿到dom
          click: true,
          probeType: 3    // better-scroll属性，监听实时滚动的位置
        });
        this.foodsScroll.on('scroll', (pos) => {   // 滚动时监听滚动到的位置pos，参考better-scroll文档
          this.scrollY = Math.abs(Math.round(pos.y));   //  将Y值转换成整数并取它的绝对值
          // console.log(this.scrollY);
        });
      },
      _calculateHeight() {
        let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook');   // 拿到food-list，一个数组
        let height = 0;   //
        this.listHeight.push(height);   // push一个初始高度0
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];   //  循环出每个li
          height += item.clientHeight;    // 累加得出每个li的高度
          // console.log(height);
          this.listHeight.push(height);   // 将每个li的高度push到listHeight这个数组中
          // console.log(this.listHeight);
        }
      }
    },
    components: {
      shopcart,
      cartcontrol
    },
    events: {
        'cart.add'(target) {
          this._drop(target);
        }
    }
  };
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .goods
    display: flex
    position: absolute
    top: 174px
    bottom: 46px
    width: 100%
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px
      background: #f3f5f7
      .menu-item
        display: table
        height: 54px
        width: 56px
        padding: 0 12px
        line-height: 14px
        &.current
          position: relative
          z-index: 10
          margin-top: -1px
          background: #fff
          font-weight: 700
          .text
            border-none()
        .icon
          display: inline-block
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          display: table-cell
          width: 56px
          vertical-align: middle
          border-1px(rgba(7, 17, 27, 0.1))
          font-size: 12px
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        height: 26px
        line-height: 26px
        border-left: 2px solid #d9dde1
        font-size: 12px
        color: rgb(147, 153, 159)
        background: #f3f5f7
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7, 17, 27, 0.1))
        before-border-none()
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex: 0 0 57px
          margin-right: 10px
        .content
          flex: 1
          .name
            margin: 2px 0 8px 0
            height: 14px
            line-height: 14px
            font-size: 14px
            color: rgb(7, 17, 27)
          .desc, .extra
            line-height: 10px
            font-size: 10px
            color: rgb(147, 153, 159)
          .desc
            line-height: 12px
            margin-bottom: 8px
          .extra
            .count
              margin-right: 12px
          .price
            font-weight: 700
            line-height: 24px
            .now
              margin-right: 8px
              font-size: 14px
              color: rgb(240, 20, 20)
            .old
              text-decoration: line-through
              font-size: 10px
              color: rgb(147, 153, 159)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 12px
</style>
