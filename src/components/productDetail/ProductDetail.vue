<template>
    <transition name="food-detail">
    <div class="food" ref="foodView" v-show="showFlag">
      <div class="food-wrapper">
        <div class="food-content">
          <div class="img-wrapper">

            <!-- 展示的大图 -->
            <img class="food-img" :src="food.picture"/>

            <!-- 关闭的小图 icon -->
            <span 
              class="close-bt icon-close"
              @click="closeView"
              ></span>
              <!-- 分享小图 icon -->
            <img class="share-bt" src="./img/share.png" />
              <!-- 更多小图  icon -->
            <img class="more-bt" src="./img/more.png" />
          </div>

           <div class="content-wrapper">
              <h3 class="name">{{food.name}}</h3>
              <p class="saled">{{food.month_saled_content}}</p>
              <img class="product" v-show="food.product_label_picture" :src="food.product_label_picture"/>
              <p class="price">
                <span class="text">￥{{food.min_price}}</span>
                <span class="unit">/{{food.unit}}</span>
              </p>


              <!-- 如果有商品数量 （商品数量 不等于 0 ） 就显示此结构 -->
              <div class="cartcontrol-wrapper">
                <!-- 加  减 的组件 -->
                <CartControl :food="food"></CartControl>
              </div>
              <!-- 如果没有商品数量数据  并且  商品数量 等于0  就显示此结构  -->
              <div class="buy" @click="addProduct" v-show="!food.count || food.count == 0">
                选规格
              </div>

            </div>
        </div>

        <!-- 使用 分割线组件 -->
        <Split></Split>


        <!-- 外卖评价 -->
        <div class="rating-wrapper">
          <!-- 评价头部 -->
          <div class="rating-title">
            <div class="like-ratio" v-if="food.rating">
              <span class="title">{{food.rating.title}}</span>
              <!-- 好评度 -->
              <span class="retio">
                (
                  {{food.rating.like_ratio_desc}}
                  <i>{{food.rating.like_ratio}}</i>
                )
              </span>
            </div>
            <div class="snd-title" v-if="food.rating">
                  <span class="text">{{food.rating.snd_title}}</span> 
                  <!-- 右箭头  icon -->
                  <span class="icon icon-keyboard_arrow_right"></span> 
            </div>
          </div>

          <!-- 展示评价 -->
          <ul class="rating-content" v-if="food.rating">
            <li 
              v-for="(comment,index) in food.rating.comment_list" 
              :key="index"
              class="comment-item"
              >
              <!-- 展示  评价用户名头像 -->
              <div class="comment-header">
                <!-- 后台数据中有头像数据  -->
                <img :src="comment.user_icon" v-if="comment.user_icon" />
                <!-- 后台数据中 没有头像数据，使用本地图片替换 -->
                <img src="./img/anonymity.png" v-if="!comment.user_icon"  />
              </div>

              <div class="comment-main">
                <div class="user">
                  {{comment.user_name}}
                </div>
                <div class="time">
                  {{comment.comment_time}}
                </div>
                <div class="content">
                  {{comment.comment_content}}
                </div>
              </div>
            </li>
          </ul>
        </div>


      </div>
    </div>
  </transition>
</template>

<script>
import Vue from 'vue';
import BScroll from 'better-scroll'
import CartControl from '../cartcontrol/CartControl'
import Split from '../split/Split'
	export default {
		data(){
      return {
        showFlag:false  // 定义展示详情页的开关
      }
    },
    props:{
      food:{
        type:Object
      }
    },
    methods:{
      // 展示详情页方法
      showView(){
        this.showFlag = true

        // 初始化 滚动  （当一进入详情页 页面 就让有滚动效果）
        this.$nextTick(() => {
          if(!this.scroll){
            this.scroll = new BScroll(this.$refs.foodView,{
              click:true
            })
          }else{
            // 实时更新方法
            this.scroll.refresh()
          }
        })
      },

      // 关闭 详情页
      closeView(){
        this.showFlag = false
      },

      // 添加商品数量  （添加 商品数量 food.count 数据）
      addProduct(){
        // this.food.count = 1  不能直接这样写，因为 数量的数据是后set添加的， 如果此时没有展示数据，就证明此商品没有count 数据， 所以要使用  Vue.set 方法添加数据。
        Vue.set(this.food,"count",1)
      }

    },
    components:{
      CartControl,
      Split
    }
	}
</script>

<style>
.food{
	position: fixed;
	left: 0;
	top: 0;
  /* 空出底部 51px的 购物车栏 */
	bottom: 51px; 
	background: white;
	width: 100%;
	z-index: 90;
}

.food-detail-enter-active, .food-detail-leave-active {
  transition:  1.0s
}
.food-detail-enter, .food-detail-leave-to {
  transform: translateX(100%);
}

.food .food-wrapper .food-content{
	
}

/* 大图的容器样式 */
.food .food-wrapper .food-content .img-wrapper{
	position: relative;
	width: 100%;
	height: 0;
  /* 会根据当前的高度进行撑高 */
	padding-top: 100%;
}
/* 大图样式 */
.food .food-wrapper .food-content .img-wrapper .food-img{
	position: absolute;
	left: 0;
	bottom: 0;
	width: 100%;
	height: 100%;
}

/* 关闭按钮 */
.food .food-wrapper .food-content .img-wrapper .close-bt{
	width: 30px;
	height: 30px;
	position: absolute;
	left: 10px;
	top: 10px;
	text-align: center;
	font-size: 30px;
	color: white;
	background: #7f7f7f;
	border-radius: 50%;
}
/* 分享 和 更多 按钮 */
.food .food-wrapper .food-content .img-wrapper .share-bt,
.food .food-wrapper .food-content .img-wrapper .more-bt
{
	width: 30px;
	height: 30px;
	position: absolute;
	top: 10px;
}
.food .food-wrapper .food-content .img-wrapper .share-bt{
	right: 50px;
}
.food .food-wrapper .food-content .img-wrapper .more-bt{
	right: 10px;
}


/*  */
.food .food-wrapper .food-content .content-wrapper{
	padding: 0 0 16px 16px;
	position: relative;
}
.food .food-wrapper .food-content .content-wrapper .name{
	font-size: 15px;
	color: #333333;
	line-height: 30px;
	font-weight: bold;
}
.food .food-wrapper .food-content .content-wrapper .saled{
	font-size: 11px;
	color: #9d9d9d;
	margin-bottom: 6px;
}
.food .food-wrapper .food-content .content-wrapper .product{
	height: 15px;
	margin-bottom: 16px;
}
.food .food-wrapper .food-content .content-wrapper .price{
	font-size: 0;
}
.food .food-wrapper .food-content .content-wrapper .price .text{
	font-size: 17px;
	color: #FB4E44;
}
.food .food-wrapper .food-content .content-wrapper .price .unit{
	font-size: 11px;
	color: #9D9D9D;
}
.food .food-wrapper .food-content .cartcontrol-wrapper{
	position: absolute;
	right: 12px;
	bottom: 10px;
	padding: 2px;
}
.food .food-wrapper .food-content .buy{
	width: 64px;
	height: 30px;
	font-size: 12px;
	line-height: 30px;
	text-align: center;
	background: #FFD161;
	border-radius: 30px;
	position: absolute;
	right: 12px;
	bottom: 10px;
}



/* 评价样式 */
.food .food-wrapper .rating-wrapper{
	padding-left: 16px;
}
.food .food-wrapper .rating-wrapper .rating-title{
	padding: 16px 16px 16px 0;
}
.food .food-wrapper .rating-wrapper .rating-title .like-ratio{
	float: left;
	font-size: 0;
}
.food .food-wrapper .rating-wrapper .rating-title .like-ratio .title{
	font-size: 13px;
}
.food .food-wrapper .rating-wrapper .rating-title .like-ratio .ratio{
	font-size: 11px;
}
.food .food-wrapper .rating-wrapper .rating-title .like-ratio .ratio i{
	color: #FB4E44;
	font-size: 11px;
}

.food .food-wrapper .rating-wrapper .rating-title .snd-title{
	float: right;
	font-size: 0;
}
.food .food-wrapper .rating-wrapper .rating-title .snd-title .text,
.food .food-wrapper .rating-wrapper .rating-title .snd-title .icon
{
	font-size: 13px;
	color: #9D9D9D;
	display: inline-block;
}
.food .food-wrapper .rating-wrapper .rating-title .snd-title .icon{
	margin-left: 12px;
}


.food .food-wrapper .rating-wrapper .comment-item{
	padding: 15px 14px 17px 0;
	border-bottom: 1px solid #F4F4F4;
	width: 100%;
	box-sizing: border-box;
	display: flex;
}
.food .food-wrapper .rating-wrapper .comment-item .comment-header{
	flex: 0 0 41px;
	margin-right: 10px;
}
.food .food-wrapper .rating-wrapper .comment-item .comment-header img{
	width: 41px;
	height: 41px;
	border-radius: 50%;
}

.food .food-wrapper .rating-wrapper .comment-item .comment-main{
	flex: 1;
	margin-top: 6px;
}
.food .food-wrapper .rating-wrapper .comment-item .comment-main .user{
	width: 50%;
	float: left;
	font-size: 12px;
	color: #333333;
}
.food .food-wrapper .rating-wrapper .comment-item .comment-main .time{
	width: 50%;
	float: right;
	text-align: right;
	font-size: 10px;
	color: #666666;
}
.food .food-wrapper .rating-wrapper .comment-item .comment-main .content{
	margin-top: 17px;
	font-size: 13px;
	line-height: 19px;
}
</style>
