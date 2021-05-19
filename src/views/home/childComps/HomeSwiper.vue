<template>
  <swiper>
    <swiper-item v-for="item in banners" :key="item.index">
      <a :href="item.link">
        <img :src="item.image" alt="" @load="imageLoad">
      </a>
    </swiper-item>
  </swiper>
</template>

<script>
import {Swiper, SwiperItem} from 'components/common/swiper'

export default {
  props: {
    banners: {
      type: Array,
      default(){
        return []
      },
    }
  },
  data () {
    return {
      isLoad: false  
    }
  },
  components: {
    Swiper,
    SwiperItem,
  },
  methods: {
    // 这个事件只需要发送一次就可以了
    imageLoad() {
      if(!this.isLoad) {
        this.$emit('swiperImageLoad')
        this.isLoad = true
      }
    }
  }
}
</script>

<style>

</style>