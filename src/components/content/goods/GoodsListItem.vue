<template>
  <div class="goods-item" @click="itemClick(goodsItem.iid)">
<!--    v-lazy使用图片懒加载-->
    <img  v-lazy="{src:showImage, loading:'images/placeholder.png'}" alt="">
    <div class="goods-info">
      <p>{{goodsItem.title}}</p>
      <span class="price">{{goodsItem.price}}</span>
      <span class="collect">{{goodsItem.cfav}}</span>
    </div>
  </div>
</template>

<script>
  export default {
    name: "GoodsListItem",
    props: {
      goodsItem:{
        type: Object,
        default(){
          return {}
        }
      }
    },
    //计算属性
    computed: {
      //goodsItem.show.img的图片存放位置不一样，这里用计算属性改一下
      showImage() {
        //判断this.goodsItem.image在就赋值，不在就赋值后面的
        return this.goodsItem.image || this.goodsItem.show.img
      },
    },
    methods:{
      itemClick(iid){
        console.log(iid);
        this.$router.push('/detail/'+iid)
      },
    },
  }
</script>

<style scoped>
  .goods-item {
    padding-bottom: 40px;
    position: relative;

    width: 48%;
  }

  .goods-item img {
    width: 100%;
    border-radius: 5px;
  }

  .goods-info {
    font-size: 14px;
    position: absolute;
    bottom: 5px;
    left: 0;
    right: 0;
    overflow: hidden;
    text-align: center;
  }

  .goods-info p {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    margin-bottom: 3px;
  }

  .goods-info .price {
    color: var(--color-high-text);
    margin-right: 20px;
  }

  .goods-info .collect {
    position: relative;
  }

  .goods-info .collect::before {
    content: '';
    position: absolute;
    left: -15px;
    top: -1px;
    width: 14px;
    height: 14px;
    background: url("~@/assets/img/common/collect.svg") 0 0/14px 14px;
  }
</style>