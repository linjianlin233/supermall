<template>
  <div id="home">
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>
     <tab-control :titles="['流行','新款','精选']" class="tab-control"
       @tabClick="tabClick" 
       v-show="isTabFixed" ref="tabControl1"/>
    <scroll class="content" ref="scroll" :probe-type="3" 
      @scroll="contentscroll" :pull-up-load="true" 
      @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"/>
      <recommend-view :recommends="recommends"/>
      <feature-view />
      <tab-control :titles="['流行','新款','精选']"
       @tabClick="tabClick" ref="tabControl2" />
      <good-list :goods="showGoods" />
    </scroll>

    <back-top @click.native="backclick" v-show="isShowBackTop"/>
    
  </div>
</template>

<script>

import HomeSwiper from './childComps/HomeSwiper'
import RecommendView from './childComps/RecommendView'
import FeatureView from './childComps/FeatureView'

import NavBar from 'components/common/navbar/NavBar';
import TabControl from 'components/content/tabControl/TabControl'
import GoodList from 'components/content/goods/GoodsList'
import Scroll from 'components/common/scroll/Scroll'
import BackTop from 'components/content/backtop/BackTop'

import {getHomeMultidata,getHomeGoods} from 'network/home';


  export default {
    name: "Home",
    components:{
      HomeSwiper,
      RecommendView,
      FeatureView,

      NavBar,
      TabControl,
      GoodList,
      Scroll,
      BackTop
    },
    data(){
      return {
        banners:[],
        recommends:[],
        goods:{
          'pop':{page:0,list:[]},
          'new':{page:0,list:[]},
          'sell':{page:0,list:[]},
        },
        currentTyoe:'pop',
        isShowBackTop:false,
        tabOffsetTop:0,
        isTabFixed:false,
        saveY:0,
      }
    },
    created(){
      this.getHomeMultidata(),
      
     this.getHomeGoods('pop')
     this.getHomeGoods('new')
     this.getHomeGoods('sell')

    },
    mounted(){
      const refresh = this.debounce(this.$refs.scroll.refresh,5000)
      this.$bus.$on('itemImageLoad', () => {
        refresh()
    })
    },deactivated() {
      this.saveY = this.$refs.scroll.scroll.scroll.y 
    },
    activaaated(){
      this.$refs.scroll.scrollTo(0, this.saveY, 0)
      this.$refs.scroll.refresh()
    },

    computed:{
      showGoods(){
        return this.goods[this.currentTyoe].list
      }
    },
    methods:{
      tabClick(index){
        switch(index){
          case 0: this.currentTyoe = 'pop'
          break
          case 1: this.currentTyoe = 'new'
          break
          case 2: this.currentTyoe = 'sell'
          break
        }
        this.$refs.tabControl1.currentIndex = index;
        this.$refs.tabControl2.currentIndex = index;
      },
      backclick() {
        this.$refs.scroll.scrollTo(0, 0)
        
      },
      contentscroll(position){
        this.isShowBackTop = (-position.y)>1000 
        this.isTabFixed = (-position.y) > this.tabOffsetTop
      },
      loadMore(){
        // console.log("dsdsf");
        this.getHomeGoods(this.currentTyoe)
      },
      debounce(func,delay) {
        let timer = null;
        return function(...args){
          if(timer) clearTimeout(timer);
          timer = setTimeout(() => {
            func.apply(this,args)
          },delay)
        }
      },
      swiperImageLoad(){
        this.tabOffsetTop=this.$refs.tabControl2.$el.offsetTop;
      },

      // 网络请求的一些方法
      getHomeMultidata(){
        getHomeMultidata().then(res => {
       
        this.banners = res.data.banner.list;
        this.recommends = res.data.recommend.list;
      })
      },
      getHomeGoods(type){
        const page = this.goods[type].page +1
         getHomeGoods(type,page).then(res =>{
         this.goods[type].list.push(...res.data.list)
         this.goods[type].page += 1
         this.$refs.scroll.scroll.finishPullUp()
      })
      }
    }
  }
</script>

<style scoped>
  #home{
    /* padding-top: 44px; */
    height: 100vh;
    position: relative;
  }
  .home-nav{
    background-color: var(--color-tint);
    color: white;

    /* position: fixed;
    left: 0;
    right: 0;
    top: 0;
    z-index: 9; */
  }
  /* .tab-control{
    position: sticky;
    top:44px;
    z-index: 9;
  } */
  .tab-control{
    position: relative;
    z-index: 9;
  }
 .content{
   
   overflow: hidden;
   position: absolute;
   top: 44px;
   bottom: 49px;
   left: 0;
   right: 0;
 }
  /* .content{
    height: calc(100% - 93px);
    overflow: hidden;
    margin-top: 44px;
  } */
  
</style>
