<template>
    <div class="goods">
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li v-for="(item,i) in goods" :key="i" class="menu-item" :class="{'current': currentIndex === i}" @click = "selectMenu(i, $event)">
            <span class="text border-1px">
              <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
            </span>
          </li>
        </ul>
      </div>
      <div class="foods-wrapper" ref="foodsWrapper" style="overflow: hidden">
        <ul>
          <li v-for="(item,i) in goods" :key="i" class="food-list food-list-hook">
            <h1 class="title">{{item.name}}</h1>
            <ul>
              <li v-for="(food,i) in item.foods" :key="i" class="food-item">
                <div class="icon">
                  <img :src="food.icon" width="57" @click="chosFood(food,$event)">
                </div>
                <div class="content">
                  <h2 class="name">{{food.name}}</h2>
                  <p class="desc">{{food.description}}</p>
                  <p class="desc">{{food.description}}</p>
                  <div class="extra">
                    <span class="count">月售{{food.sellCount}}份</span>
                    <span>好评率{{food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">¥{{food.price}}</span>
                    <span class="old" v-show="food.oldPrice">¥{{food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <cartcontrol @cartAdd="cartAdd" :food="food"></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <shopcart ref="shopcart" @cartAdd="cartAdd" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
      <vfood @cartAdd="cartAdd" :food="selectedFood" ref="vfood"></vfood>
    </div>
</template>

<script>
import BScroll from 'better-scroll'
import shopcart from '../shopcart/shopcart'
import cartcontrol from '../cartcontrol/cartcontrol'
import vfood from '../food/food.vue'

const ERR_OK = 0
export default {
  name: 'goods',
  data () {
    return {
      goods: [],
      listHeight: [], // 存储区块的高度
      scrollY: 0,
      selectedFood: {}
    }
  },
  props: ['seller'],
  created () {
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
    this.$http.get('/api/goods').then((result) => {
      result = result.body
      if (result.errno === ERR_OK) {
        this.goods = result.data
      }
      this.$nextTick(() => {
        this._initScroll()
        // 拿到数据以后计算高度
        this._calculateHeight()
      })
    })
  },
  components: {
    vfood,
    cartcontrol,
    shopcart
  },
  methods: {
    _initScroll () {
      // v-el(this.$els.foodWrapper)获取DOM对 象，json对象
      this.menuScroll = new BScroll(this.$refs.menuWrapper, {
        click: true // 阻止默认，允许点击
      })
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
        click: true,
        probeType: 3 // BScroll滚动时，能实时告诉我们滚动的位置，类似探针的效果
      })
      // foodsScroll监听事件,在scroll滚动时能见位置实时暴露出来
      this.foodsScroll.on('scroll', (pos) => {
        this.scrollY = Math.abs(Math.round(pos.y))//  取正值
      })
    },
    _calculateHeight () {
      // food-list-hook类的添加知识为了能拿到food列表,例如，拿到的是多个类似整个粥品的区块
      let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
      let height = 0
      this.listHeight.push(height) // listHeight是一个递增的区间数组，是每个专区高度的累加
      for (let i = 0; i < foodList.length; i++) {
        let item = foodList[i]
        height += item.clientHeight
        console.log(height)
        this.listHeight.push(height)
      }
    },
    selectMenu (index, event) {
      if (!event._constructed) { // 浏览器直接return掉,去掉自带click事件的点击
        return
      }
      let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook')
      let ref = foodList[index] // 取到index对应的DOM
      this.foodsScroll.scrollToElement(ref, 300)// 滚动到DOM所在位置
      // console.log(index);
    },
    cartAdd (target) {
      // 体验优化，异步执行动画
      this.$nextTick(() => {
        this.$refs.shopcart.drop(target)
      })
    },
    chosFood (food, event) {
      if (!event._constructed) {
        return
      }
      this.selectedFood = food
      this.$refs.vfood.show()
    }
  },

  computed: {
    currentIndex () { // currentIndex对应菜单栏的下标
      for (let i = 0; i < this.listHeight.length; i++) { // 不要忘了加this引用
        let height1 = this.listHeight[i]
        let height2 = this.listHeight[i + 1]
        // 获得了一个区间的上下范围，判断scrollY落到这个区间，!height2是判断最后一个区间
        // 避免i溢出，>= 向下的是一个闭区间，这样第一个就会高亮了
        if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
          console.log(i)
          return i // 映射到第5行menu的变化
        }
      }
      return 0
    },
    // 选中的食品
    selectFoods () {
      let foods = []
      this.goods.forEach((good) => {
        good.foods.forEach((food) => {
          if (food.count) {
            foods.push(food)
          }
        })
      })
      return foods
    }
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"

  .goods
    display: flex
    position: absolute
    width: 100%
    top: 174px
    bottom: 46px
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px
      background: #f3f5f7
      overflow: hidden
      .menu-item
        display: table
        height: 54px
        width: 56px
        line-height: 14px
        padding: 0 12px
        .icon
          display: inline-block
          vertical-align: bottom
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
        &.current
          position relative
          margin-top -1px
          background #fff
          font-weight 700
          z-index 10
          .text
            border-none()
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        height: 26px
        line-height: 26px
        border-left: 2px solid #d9dde1
        font-size: 12px
        color: rgb(147, 153 ,159)
        background: #f3f5f7
      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7, 17, 27, 0.1))
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
