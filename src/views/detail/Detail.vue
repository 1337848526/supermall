<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav-bar" @titleClick="titleClick" ref="navbar"></detail-nav-bar>
    <scroll class="content" ref="scroll" :probeType="3" @scroll="contentScroll">
      <detail-swiper :topImages="topImages"></detail-swiper>
      <detail-base-info :goods="goods"></detail-base-info>
      <detail-shop-info :shop="shop" />
      <detail-goods-info :detailInfo="detailInfo" @imageLoad="imageLoad" />
      <detail-param-info ref="params" :paramInfo="paramInfo" />
      <detail-comment-info ref="comment" :comment-info="commentInfo" />
    </scroll>
    <back-top @click.native="bindClick" v-show="isShowBackTop" />
    <detail-bottom-bar />
  </div>
</template>
<script>
import { debounce } from "components/common/utils";
import DetailCommentInfo from "./childComps/DetailCommentInfo";
import DetailNavBar from "./childComps/DetailNavBar";
import DetailSwiper from "./childComps/DetailSwiper";
import DetailBaseInfo from "./childComps/DetailBaseInfo";
import DetailShopInfo from "./childComps/DetailShopInfo";
import DetailGoodsInfo from "./childComps/DetailGoodsInfo";
import DetailParamInfo from "./childComps/DetailParamInfo";
import DetailBottomBar from "./childComps/DetailBottomBar";
// import DetailParamInfo from './childComps/DetailParamInfo'
import { getDetail, Goods, Shop, GoodsParam } from "network/detail.js";
import Scroll from "components/common/scroll/Scroll";
import BackTop from "components/content/backTop/BackTop";
export default {
  name: "Detail",
  data() {
    return {
      iid: null,
      topImages: null,
      goods: {},
      shop: {},
      detailInfo: {},
      paramInfo: {},
      commentInfo: {},
      themeTopYs: [],
      getThemeTopY: null,
      isShowBackTop:false
    };
  },
  methods: {
    imageLoad() {
      this.$refs.scroll.refresh();
      this.getThemeTopY();
    },
    titleClick(index) {
      this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 200);
      this.$refs.navbar.currentIndex = index;
    },
    contentScroll(position) {
       this.isShowBackTop = -position.y > 1000;
      let length = this.themeTopYs.length;
      for (let i = 0; i < length - 1; i++) {
        let iPos = this.themeTopYs[i];
        if (
          this.$refs.navbar.currentIndex !== i &&
          position >= this.themeTopYs[i] &&
          position < this.themeTopYs[i + 1]
        ) {
          this.$refs.navbar.currentIndex = i;
        }
      }
    },
    bindClick(){
      this.$refs.scroll.scrollTo(0, 0, 200);
    }
  },
  created() {
    this.iid = this.$route.params.iid;
    getDetail(this.iid).then((res) => {
      const data = res.result;
      this.topImages = data.itemInfo.topImages;
      this.goods = new Goods(
        data.itemInfo,
        data.columns,
        data.shopInfo.services
      );
      this.shop = new Shop(data.shopInfo);
      console.log(this.goods);
      this.detailInfo = data.detailInfo;
      this.paramInfo = new GoodsParam(
        data.itemParams.info,
        data.itemParams.rule
      );
      if (data.rate.list) {
        this.commentInfo = data.rate.list[0];
      }
    });
    this.getThemeTopY = debounce(() => {
      this.themeTopYs = [];
      this.themeTopYs.push(0);
      this.themeTopYs.push(this.$refs.params.$el.offsetTop);
      this.themeTopYs.push(this.$refs.comment.$el.offsetTop);
    });
  },
  components: {
    DetailNavBar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    Scroll,
    DetailGoodsInfo,
    DetailParamInfo,
    DetailCommentInfo,
    DetailBottomBar,
    BackTop,
  },
};
</script>
<style scoped='scoped'>
#detail {
  position: relative;
  z-index: 99;
  background-color: #fff;
  height: 100vh;
}
.detail-nav-bar {
  position: relative;
  z-index: 99;
  background-color: #fff;
}
.content {
  height: calc(100%);
}
</style>