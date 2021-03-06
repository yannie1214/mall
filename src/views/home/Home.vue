<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <!-- 因为用了 better scroll 所以想要这个在页面进行停留就比较难实现，所以这里是做了一个相同的东西让它进行显示和隐藏的，当需要它停留的时候就显示，否则就隐藏 -->
    <tab-control :titles="['流行','新款','精选']" 
                   @tabClick="tabClick" 
                   ref="tabControl1"
                   class="tab-control"
                   v-show="isTabFixed"></tab-control>
    <scroll :probe-type="3"
            :pull-up-load="true"
            class="content"
            @scroll="contentScroll"
            @pullingUp="loadMore"
            ref="scroll">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"></home-swiper>
      <recommend-view :recommends="recommends"></recommend-view>
      <feature-view></feature-view>
      <tab-control :titles="['流行','新款','精选']" 
                   @tabClick="tabClick" 
                   ref="tabControl2"></tab-control>
      <goods-list :goods="showGoods"></goods-list>
    </scroll>
    <!-- 组件是不能监听点击的，想要让它监听点击必须加上 native 的修饰符,不仅仅是点击事件，所有原生事件都是 -->
    <back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
  </div>
</template>

<script>
  import NavBar from 'components/common/navbar/NavBar'
  import TabControl from 'components/content/tabControl/TabControl'
  import GoodsList from 'components/content/goods/GoodsList'
  import Scroll from 'components/common/scroll/Scroll'
  import BackTop from 'components/content/backTop/BackTop'

  import HomeSwiper from './childComps/HomeSwiper'
  import RecommendView from './childComps/RecommendView'
  import FeatureView from './childComps/FeatureView'

  import {
    getHomeMultidata,
    getHomeGoods,
    } from 'network/home'
  import {debounce} from 'common/utils'

  export default {
    name: "Home",
    components: {
      NavBar,
      TabControl,
      GoodsList,
      Scroll,
      BackTop,
      HomeSwiper,
      RecommendView,
      FeatureView,
    },
    data(){
      return {
        banners: [],
        recommends: [],
        goods: {
          'pop': {page: 0, list: []},
          'new': {page: 0, list: []},
          'sell': {page: 0, list: []},
        },
        currentType: 'pop',
        isShowBackTop: false,
        tabOffsetTop: 0,
        isTabFixed: false,
        saveY: 0
      }
    },
    computed: {
      showGoods() {
        return this.goods[this.currentType].list
      }
    },
    activated () {
      this.$refs.scroll.scrollTo(0, this.saveY, 0)
      // 回来时最好进行一次刷新
      this.$refs.scroll.refresh()
    },
    deactivated () {
      this.saveY = this.$refs.scroll.getScrollY()
    },
    // created 函数比较特殊，是当我们组件创建好就会执行的一个函数，最好把里面的东西抽取出去，只写重要的内容和主要的逻辑,这里把里面的方法抽取到了方法里面
    created() {
      // 1. 请求多个数据
      this.getHomeMultidata()

      // 2. 请求商品数据
      this.getHomeGoods('pop')
      this.getHomeGoods('new')
      this.getHomeGoods('sell')
    },
    mounted () {
      // 如果需要拿 scroll 这个东西最好把这部分代码写到 mounted 里面，而不是 created 里面
      // 1. 监听 item 中图片加载完成
      // 防抖动函数(简单可以理解为，等待图片都加载完了才调用函数)
      const refresh = debounce(this.$refs.scroll.refresh, 50)
      this.$bus.$on('itemImageLoad', () => {
        refresh()
      })

      // 这样写的话 refresh 就会被调用 30 次
      // this.$bus.$on('itemImageLoad', () => {
      //   this.$refs.scroll.refresh()
      //   console.log('----------------');
      // })

      // 2. 获取 tabControl 的 offsetTop ,但是注意的是这里的 offsetTop 是不准确的
      // this.tabOffsetTop = this.$refs.tabControl.$el.offsetTop
    },
    methods: {
      /**
       * 事件监听相关的方法
       */
      tabClick(index) {
        switch (index) {
          case 0:
            this.currentType = 'pop'
            break;
          case 1:
            this.currentType = 'new'
            break;
          case 2:
            this.currentType = 'sell'
            break;
        }
        this.$refs.tabControl1.currentIndex = index
        this.$refs.tabControl2.currentIndex = index
      },
      backClick() {
        this.$refs.scroll.scrollTo(0, 0)
      },
      contentScroll(position) {
        // 1. 判断 backTop 是否显示
        this.isShowBackTop = (-position.y) > 1000

        // 2. 决定 tabControl 是否吸顶(position: fixed)
        this.isTabFixed = (-position.y) > this.tabOffsetTop
      },
      loadMore() {
        this.getHomeGoods(this.currentType)
      },
      swiperImageLoad() {
        this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
      },
      /**
       * 网络请求相关的方法
       */
      getHomeMultidata() {
        getHomeMultidata().then(res => {
          // console.log(res);
          // this.result = res
          this.banners = res.data.banner.list
          this.recommends = res.data.recommend.list
        })
      },
      getHomeGoods(type) {
        const page = this.goods[type].page + 1
        getHomeGoods(type, page).then(res => {
          // console.log(res);
          this.goods[type].list.push(...res.data.list)
          this.goods[type].page += 1

          // 完成了上传加载更多
          this.$refs.scroll.finishPullUp()
       })
      },
    }
  }
</script>

<style scoped>
  #home {
    /* padding-top: 44px; */
    height: 100vh;
    position: relative;
  }
  .home-nav {
    background-color: var(--color-tint);
    color: #fff;
    /* 下面的这些属性是在我们使用浏览器原生滚动时设置的样式,为了让导航不跟随一起滚动，现在使用的 better scroll 就不需要这个，因为它可以在指定区域进行滚动 */
    /* position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9; */
  }
  .content{
    /* height: 300px; */
    overflow: hidden;
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }
  /* .content {
    height: calc(100% - 93px);
    overflow: hidden;
    margin-top: 44px;
  } */
  .tab-control {
    position: relative;
    z-index: 9;
  }
</style>
