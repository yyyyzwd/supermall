<template>
  <router-view v-slot="{Component}">
    <keep-alive include='Home'>
      <component :is="Component" />
    </keep-alive>
  </router-view>

  <main-tab-bar class="main-tab-bar" />
</template>

<script>
  import MainTabBar from "./components/content/mainTabbar/MainTabBar";

  export default {
    name: 'App',
    components: {
      MainTabBar
    }
  }

</script>

<style>
  @import "~@/assets/css/base.css";

  .main-tab-bar{
    z-index: 10;
  }
</style>
