<template>

  <div id="home">
      <nav-bar class="home-nav">
          <template #center >购物街</template>
      </nav-bar>

      <!--流行综合版块，解决无法吸顶问题-->
      <tab-control :titles="['流行', '新款', '精选']"
                   @tabClick="tabClick"
                   ref="tabControl1"
                   class="tabControl"
                    v-show="isTabFixed"
      ></tab-control>
      <!--滚动-->
      <scroll class="content" ref="scroll"
              :probe-type="2"
              @scroll="contentScroll"
              :pull-up-load="true"
              @pullingUp="loadMore"
      >

          <!--轮播图-->
          <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad" ></home-swiper>
          <!--推荐-->
          <recommend-view :recommend="recommends"></recommend-view>
          <!--一张图片-->
          <feature-view/>
          <!--流行综合版块-->
          <tab-control :titles="['流行', '新款', '精选']"
                       @tabClick="tabClick"
                       ref="tabControl2"
                       v-show="!isTabFixed"
                        ></tab-control>
          <!--商品-->
          <goods-list :goods="showGoods" />
      </scroll>
      <!--返回顶部-->
      <back-top @click="backClick" v-show="isShowBackTop" />
  </div>
</template>

<script>
//【公共组件】
//最上面的购物街
import NavBar from "components/common/navbar/NavBar";
//标签控制 也就是综合销量这些
import TabControl from "components/content/tabControl/TabControl";
//商品数据
import GoodsList from "components/content/goods/GoodsList";
//滚动插件
import Scroll from "components/common/scroll/Scroll";

//【网络请求】
import {getHomeMultidata, getHomeGoods} from "../../network/home";

//【chlidComps组件】
//封装的轮播图
import HomeSwiper from './chlidComps/HomeSwiper';
//推荐视图
import RecommendView from "./chlidComps/RecommendView";
//本周热门
import FeatureView from "./chlidComps/FeatureView";
//回到顶部小图片
import BackTop from "../../components/content/backTop/BackTop";

import {onMounted, onUnmounted} from 'vue'

export default {
    name: 'Home',
    setup() {
        // mounted
        onMounted(() => {
            console.log('Component is mounted!')
        })
        onUnmounted(()=>{
            console.log('home组件被摧毁!')
        })
    },
    data(){
      return {
          //把请求的数据存起来
          banners: null,
          recommends: null,
          //流行精选等等3个数据
          goods:{
            'pop':{pape:0, list: []},
            'new':{pape:0, list: []},
            'sell':{pape:0, list: []},
          },
          //当前显示流行精选属性
          currentType: 'pop',
          //回到顶部按钮是否显示
          isShowBackTop: false,
          //吸顶和顶部的距离
          tabOffsetTop: 0,
          //是否吸顶
          isTabFixed: false,
          //记住滚动的位置，方便返回时定位到指定位置
          saveY: 0,
      }
    },
    components: {
        NavBar,
        TabControl,
        GoodsList,
        Scroll,

        HomeSwiper,
        RecommendView,
        FeatureView,
        BackTop,
    },
    //计算属性
    computed:{
      //这里老师说太长，用个计算属性包一下，我不喜欢这样搞，拆太散了
      showGoods(){
          return  this.goods[this.currentType].list
      }
    },
    //生命周期函数，创建组件时执行
    created() {
        //获取轮播图推荐
        this.getHomeMultidata()
        //获取流行精选等3个数据
        this.getHomeGoods('pop')
        this.getHomeGoods('new')
        this.getHomeGoods('sell')
    },
    //生命周期函数
    mounted() {
        // console.log('home组件挂载dom');
    },
    //路由活跃状态刷下高度
    activated(){
        this.$refs.scroll.refresh();
        // console.log('选中home');
        this.$refs.scroll.scrollTo(0, this.saveY, 0);

    },
    //路由不活跃
    deactivated() {
        this.saveY = this.$refs.scroll.getScrollY();
        // console.log(this.saveY);
    },
    //组件销毁时调用
    destroyed() {
        // console.log('home组件销毁');
    },
    methods: {
        /**
         * 事件监听相关方法
         */
        //根据点击流行精选后，子组件传递参数，这边接受后显示对应的商品数据
        tabClick(index){
            // console.log("父组件监听到到数据");
            // console.log(index);
            //根据index为currentType赋值
            switch (index) {
                case 0:
                    this.currentType = 'pop'
                    break
                case 1:
                    this.currentType = 'new'
                    break
                case 2:
                    this.currentType = 'sell'
                    break
            }
            this.$refs.tabControl1.currentIndex = index
            this.$refs.tabControl2.currentIndex = index

            this.$refs.scroll.scrollTo(0, -this.tabOffsetTop);
            console.log('ddd');
        },

        //返回顶部
        backClick(){
            //console.log('点击返回顶部');
            /*
            * 通过ref获取子组件的的方法 scrollTo ，跳转到指定位置
            * 参数一：x轴
            * 参数二：y轴
            * 参数三：跳转到位置的延迟时间,默认300
            */
            this.$refs.scroll.scrollTo && this.$refs.scroll.scrollTo(0,0,500);
        },

        //监听滑动的位置，控制回到顶部是否显示
        contentScroll(position){
            // console.log(position);
            this.isShowBackTop = position.y < -1000
            this.isTabFixed = position.y < -this.tabOffsetTop
        },

        //上拉加载更多
        loadMore(){
            //console.log('上拉了');
            this.getHomeGoods(this.currentType)
        },

        //监听轮播图加载，计算吸顶与顶部的距离
        swiperImageLoad(){
            //console.log(this.$refs.tabControl2.$el.offsetTop);
            this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop
        },


        /**
         * 网络请求相关方法
         */
        //调用封装的网络请求方法,获取轮播图，推荐
        getHomeMultidata(){
            getHomeMultidata().then(res => {
                // console.log(res);
                //把请求到的数据存储起来
                this.banners = res.data.banner.list;
                this.recommends = res.data.recommend.list;
            }).catch(err => {
                console.log(err);
            })
        },

        //获取精选，综合，流行这些商品数据
        getHomeGoods(type){
            //需要传递type，所以这个对象的写法像数组
            const page = this.goods[type].pape + 1
            getHomeGoods(type, page).then(res => {
                //console.log(res);
                //...用于多个值传递，看js的3.1
                this.goods[type].list.push(...res.data.list)
                this.goods[type].pape += 1

                //上拉更新下
                this.$refs.scroll.finishPullUp()

                //重构高度
                this.$refs.scroll.refresh()

            }).catch(err => {
                console.log(err);
            })
        }
    }
}
</script>

<style scoped>
    #home{
        height: 100vh;
        position: relative;
    }
    .home-nav{
        /*这里使用之前base设置的变量*/
        background: var(--color-tint);
        color: white;
        font-size: 18px;

        /*position: fixed;*/
        /*left: 0;*/
        /*right: 0;*/
        /*top: 0;*/
        /*z-index: 9;*/
    }
    .content{
        overflow: hidden;

        position: absolute;
        top: 44px;
        bottom: 49px;
        left: 0;
        right: 0;
    }
    .tabControl{
        position: relative;
        z-index: 9999;
        background: white;
    }
</style>