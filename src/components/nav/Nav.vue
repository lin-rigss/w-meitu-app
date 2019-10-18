<template>
  <div class="nav">
    <router-link class="nav-item" to="/goods">
      点菜
      <!-- i 标签 就是 导航下面的 小短线 -->
      <i class="line"></i>  
    </router-link>
    <router-link class="nav-item" to="/ratings">
      评价({{commentNum}})
      <i class="line"></i>    
    </router-link>
    <router-link class="nav-item" to="/seller">
      商家
      <i class="line"></i>   
    </router-link>
  </div>
</template>

<script>
export default {
  props:{
    // 接收 App.vue 父组件传递过来的  评价数量的数据
    commentNum:{
      type:Number,
      default:0
    }
  }
}
</script>

<style scoped>
/* 导航盒子 */
.nav{
  display: flex;
  width: 100%;
  height: 40px;
  line-height: 40px;
  border-bottom: 1px solid #e4e4e4;
}
/* 每一个导航按钮 */
.nav-item{
  flex:1;
  text-align: center;
  text-decoration:none;
  color: #666666;
  position: relative;
}

/* 设置 router-link 的选中样式  */
.nav .active{
  color: #ffbb22;
}
/* 相对应的下划线 选中样式 */
.nav .active .line{
  width: 20px;
  height: 2px;
  display: inline-block;
  background: #ffbb22;
  /* 给对应 i 标签的线 定位设置  居中显示 */
  position: absolute;
  left: 50%; 
  bottom: 0;
  margin-left: -10px;

}



</style>
