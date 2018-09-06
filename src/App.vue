<template>
  <div id="app">
    <v-header :seller="seller"></v-header>
    <div class="tab">
      <div class="tab-item">
        <router-link to="/goods" @click.native="active">商品</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/ratings" @click.native="active">评价</router-link>
      </div>
      <div class="tab-item">
        <router-link to="/seller" @click.native="active">卖家</router-link>
      </div>
    </div>
    <keep-alive>
      <router-view :seller="seller"></router-view>
    </keep-alive>
  </div>
</template>

<script>
import {urlParse} from './common/js/util'
import header from './components/header/header.vue'
const ERR_OK = 0
export default {
  name: 'App',
  data () {
    return {
      seller: {
        id: (() => {
          let queryParam = urlParse()
          console.log(queryParam)
          return queryParam.id
        })()
      }
    }
  },
  created () {
    this.$http.get('/api/seller?id=' + this.seller.id).then((result) => {
      result = result.body
      if (result.errno === ERR_OK) {
        this.seller = Object.assign({}, this.seller, result.data)
      }
    })
  },
  components: {
    'v-header': header
  },
  methods: {
    active (e) {
      var links = document.querySelectorAll('#app>.tab>.tab-item>a')
      for (var a of links) {
        a.style.color = ''
      }
      e.target.style.color = 'rgb(255,20,20)'
    }
  }
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "./common/stylus/mixin.styl"
  @import "./common/stylus/base.styl"
  @import "./common/stylus/icon.styl"
  #app
    .tab
      display: flex
      width: 100%
      height: 40px
      line-height: 40px
      border-1px(rgba(7,17,27,0.1))
      .tab-item
        flex: 1
        text-align: center
        & > a
          display : block
          font-size : 14px
          color: rgb(77,85,93)

</style>
