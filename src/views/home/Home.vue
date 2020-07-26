<template>
  <div id="home">
    <nav-bar class="home-nav" ref="home"></nav-bar>
    <tab-control
      :titles="['流行','新款','精选']"
      class="tab-control"
      @tabClick="tabClick"
      ref="tabControl1"
      v-show="tabFixed"
    ></tab-control>
    <scroll
      class="content"
      ref="scroll"
      :probe-type="3"
      :pull-up-load="true"
      @scroll="contentScroll"
      @pullingUp="loadMore"
    >
      <home-swiper :banners="banners" @imageLoad="SwiperImageLoad"></home-swiper>
      <recommend-view :recommends="recommends"></recommend-view>
      <feature-view></feature-view>
      <tab-control
        :titles="['流行','新款','精选']"
        class="tab-control"
        @tabClick="tabClick"
        ref="tabControl2"
      ></tab-control>
      <good-list :goods="goods[currentType].list"></good-list>
    </scroll>
    <back-top @click.native="bindClick" v-show=" isShowBackTop" />
  </div>
</template>

<script>
import HomeSwiper from "./childComps/HomeSwiper";
import RecommendView from "./childComps/RecommendView";
import FeatureView from "./childComps/FeatureView";

import NavBar from "components/common/navbar/NavBar";
import TabControl from "components/content/tabControl/TabControl";
import GoodList from "components/content/goods/GoodsList";

import { getMultiData, getHomeGoods } from "network/home.js";
import scroll from "components/common/scroll/Scroll";

import BackTop from "components/content/backTop/BackTop";
import { debounce } from "components/common/utils.js";
export default {
  name: "Home",
  components: {
    HomeSwiper,
    RecommendView,
    FeatureView,
    NavBar,
    TabControl,
    GoodList,
    scroll,
    BackTop,
  },
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        pop: { page: 1, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 1, list: [] },
      },
      currentType: "pop",
      isShowBackTop: false,
      tabOffsetTop: 0,
      tabFixed: false,
    };
  },
  created() {
    this.getMultiData();
    this.getHomeGoods("pop");
    this.getHomeGoods("new");
    this.getHomeGoods("sell");
  },
  mounted() {
    const refresh = debounce(this.$refs.scroll.refresh, 2000);
    this.$bus.$on("ItemImageLoad", () => {
      refresh();
    });
   
  
  },
  methods: {
    /**
     * 事件监听相关的方法
     */
    SwiperImageLoad() {
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop;
    },
    tabClick(index) {
      switch (index) {
        case 0:
          this.currentType = "pop";
          break;
        case 1:
          this.currentType = "new";
          break;
        case 2:
          this.currentType = "sell";
          break;
      }
     this.$refs.tabControl2.currentIndex = index
      this.$refs.tabControl1.currentIndex = index;
    },
    bindClick() {
      this.$refs.scroll.scrollTo(0, 0);
    },
    contentScroll(position) {
      this.isShowBackTop = -position.y > 1000;
      this.tabFixed = -position.y > this.tabOffsetTop;
    },
    loadMore() {
      this.getHomeGoods(this.currentType);
    },
    /**
     * 网络请求相关的方法
     */
    getMultiData() {
      getMultiData().then((res) => {
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      });
    },
    getHomeGoods(type) {
      const page = this.goods[type].page + 1;
      getHomeGoods(type, page).then((res) => {
        console.log(res);
        this.goods[type].list.push(...res.data.list);
        this.goods[type].page += 1;
        this.$refs.scroll.finishPullUp();
      });
    },
  },
};
</script>

<style scoped>
#home {
  height: 100vh;
  position: relative;
}
.content {
  overflow: hidden;
  position: absolute;
  top: 44px;
  bottom: 49px;
  left: 0;
}
.home-nav {
  background-color: var(--color-tint);
  color: #fff;
}
.tab-control {
  background-color: #ffffff;
  position: relative;
  left: 0;
  right: 0;
  z-index: 99;
}
</style>
