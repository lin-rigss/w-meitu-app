<template>
  <div id="app">

    <!-- 头部    当前组件是父组件， header是子组件，使用自定义属性方式将获取到的数据 传递给 子组件--> 
    <app-header :poiInfo="poiInfo"></app-header>

    <!-- 导航 -->
    <app-nav :commentNum="commentNum"></app-nav>

    <!-- content  根据导航所显示的对应内容 -->
    <keep-alive>
      <router-view></router-view>
    </keep-alive>

  </div>
</template>

<script>
import Header from './components/header/Header'
import Nav from '@/components/nav/Nav'

export default {
  name: 'App',
  data(){
    return{
      poiInfo:{}, // 头部初始化数据
      commentNum:0  // 存储 评价数量 数据
    }
  },
  components: {
    "app-header":Header,
    "app-nav":Nav
  },
  created(){

    // 请求头部数据
    fetch("/api/goods")
      .then(res => {
        return res.json()
      })
      .then(response =>{
        // code = 0 证明数据成功请求回来了
        if(response.code == 0){
          // 将 请求回来的数据，赋值给 初始值 变量
          this.poiInfo = response.data.poi_info
        }
      })

    // 请求 评价数据中的 评价数量数据 赋值给 变量， 然后 传递给 导航组件  
    fetch("/api/ratings")
      .then(res => {
        return res.json()
      })
      .then(response =>{
        if(response.code == 0){
          this.commentNum = response.data.comment_num
        }
      })
  }
}
</script>

<style>

</style>
