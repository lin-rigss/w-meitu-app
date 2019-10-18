<template>
<!-- 因为要给购物车列表添加蒙版 效果 ，所以要在最外层添加一个盒子   （购物车  + 蒙版）  -->
<div class="shopcart" :class="{'highligh':totalCount>0}">
	<!-- 购物车容器  （购物车 左 右 和 购物车列表） -->
	<div class="shopcart-wrapper" >

		<!-- 底部左侧 -->
		<div class="content-left">
									<!-- 当totalCount 商品的数量 大于 0 的时候添加 高亮的样式 -->
			<div class="logo-wrapper" :class="{'highligh':totalCount>0}">
			
				<span class="icon-shopping_cart logo" :class="{'highligh':totalCount>0}" @click="toggleList"></span>
					<!-- 购物车上的小红点 提示加入购物车的商品数量   -->
					<i class="num" v-show="totalCount">{{totalCount}}</i>
			</div>
			<div class="desc-wrapper">
				<!--  商品价格 -->
					<p class="total-price" v-show="totalPrice">
						￥{{totalPrice}}
					</p>
					<!-- 配送费提示 -->
					<p class="tip" :class="{'highligh':totalCount>0}">
						另需{{poiInfo.shipping_fee_tip}}
					</p>
			</div>
		</div>
		
		<!-- 底部右侧 -->
		<div class="content-right" :class="{'highligh':totalCount>0}">
		{{payStr}}
		</div>

		<!-- 购物车列表 -->
		<div class="shopcart-list" v-show="listShow" :class="{'show':listShow}">
			<div class="list-top" v-if="poiInfo.discounts2">
				<!-- 获取商品数据中的 info数据  因为poiInfo.discounts2数据是一个数组所以[0] -->
				{{poiInfo.discounts2[0].info}}
			</div>
			<div class="list-header">
				<h3 class="title">1号口袋</h3>
				<div class="empty" @click="clearAll">
					<img src="./img/ash_bin.png" />
					<span>清空购物车</span>
				</div>
			</div>
			<div class="list-content" ref="listContent">
				<ul>
					<!-- 遍历 加入购物车的商品信数据 -->
					<li class="food-item" v-for="(food,index) in selectFoods" :key="index">
						<div class="desc-wrapper">
							<!-- 左 显示的商品名称 及 个数或详情 -->
							<div class="desc-left">
								<p class="name">{{food.name}}</p>
								                <!-- 如果没有详情数据，就显示 量词数据 -->
								<p class="unit" v-show="!food.description">{{food.unit}}</p>
								                <!-- 如果没有量词数据，就显示 详情数据  -->
								<p class="description" v-show="!food.unit">{{food.description}}</p>
							</div>
							<!-- 右 显示的价格 -->
							<div class="desc-right">
								￥{{food.min_price}}
							</div>
						</div>
						<!-- 加 减 商品 插件的容器 -->
						<div class="cartcontrol-wrapper">
							<app-cart-control :food="food"></app-cart-control>
						</div>
					</li>
				</ul>
			</div>
			<div class="list-bottom"></div>
		</div> 

	</div>
	<!-- 蒙版 -->
	<div class="shopcart-mask" v-show="listShow" @click="hideMask"></div>
</div>
</template>

<script>
import BScroll from 'better-scroll'
import CartControl from '../cartcontrol/CartControl'
	export default {
		data(){
			return {
				fold:true // 控制 购物车列表显示 的开关  true 为隐藏
			}
		},
		props:{
			poiInfo:{
				type:Object,
				default:{}
			},
			// 子组件接收 的是一个计算属性 的 加入购物车中的商品数据
			selectFoods:{
				type:Array,
				// 默认值：  为一个方法， 返回一个数组。
				default(){
					return []
				}
			}
		},
		computed:{
			// 根据 传过来的购物车中的商品数据selectFoods    计算商品的总数量
			totalCount(){
				let num = 0
				this.selectFoods.forEach((food) => {
					num += food.count
				})
				return num
			},
			// 根据 传过来的购物车中的商品数据selectFoods    计算商品的总价格
			totalPrice(){
				let total = 0
				this.selectFoods.forEach((food) => {
					// 总价 = 单价 * 数量
					total += food.min_price * food.count
				})
				return total
			},
			// 根据 商品的数量 来判断  右侧结构 显示的内容
			payStr(){
				if(this.totalCount > 0){
					return "去结算"
				}else{
					return this.poiInfo.min_price_tip
				}
			},

			//  监听 this.fold 购物车列表开关的属性
			listShow(){
				if(!this.totalCount){ // 如果商品个数为 0 
					this.fold = true // 不展示 列表 
					return false  // this.fold = true  相当于listShow计算属性的false  都是隐藏 购物车列表    
				}

				// 如果商品数量不为0 的情况
				let show = !this.fold

				if(show){
					// 实例化 BScroll 插件 必须包裹在 this.$nextTick 中  
					this.$nextTick(() => {
						// 没有实例化之前
						if(!this.shopScroll){   
							this.shopScroll = new BScroll(this.$refs.listContent,{
								// 让元素里面有 内容有事件的派发
								click:true
							})
						 // 已经有实例化滚动了  实例化.refresh方法     监听数据的变化或页面的变化，对页面进行刷新。
						}else{ 
							this.shopScroll.refresh()
						}
					})
				}
				return show
			}
		},
		methods:{

			// 点击购物车图标 ，显示购物车列表 的方法
			toggleList(){
				// 判断购物车个数是否为空   数量 = 0  false  =》 ！false  代表隐藏购物车列表     如果有商品数量，就不隐藏。 
				if(!this.totalCount){
					return;
				}
				// 每点击一下，就取返 fold   
				this.fold = !this.fold
			},

			// 清空购物车方法
			clearAll(){
				// 遍历 购物车中的商品 ，将商品的count属性赋值为 0 
				this.selectFoods.forEach((food) => {
					food.count = 0
				})
			},

			// 点击蒙版 隐藏购物车列表 的方法
			hideMask(){
				// 直接将this.fold 值为 true. 隐藏购物车列表。
				this.fold = true 
			}
		},
		components:{
			"app-cart-control":CartControl
		}
	}
</script>

<style>
@import url(../../common/css/icon.css);

.shopcart-wrapper{
	width: 100%;
	height: 51px;
	background: #514f4f;
	position: fixed;
	left: 0;
	bottom: 0;
	display: flex;
	/* 注  购物车容器 一定要比 蒙版 层级要高 */
	z-index: 99; 
}

.shopcart-wrapper .content-left{
	flex: 1;
}

.shopcart-wrapper .content-right{
	flex: 0 0 110px;

  font-size: 15px;
	color: #BAB9B9;
	line-height: 51px;
	text-align: center;
	font-weight: bold;
}

.shopcart-wrapper .content-left .logo-wrapper{
	width: 50px;
	height: 50px;
	background: #666666;
	border-radius: 50%;
	position: relative;
	top: -14px;
	left: 10px;
	text-align: center;
	float: left;
}

.shopcart-wrapper .content-left .logo-wrapper .logo{
	font-size: 28px;
	color: #c4c4c4;
	line-height: 50px;
}

.shopcart-wrapper .content-left .desc-wrapper{
	float: left;
	margin-left: 13px;
}

.shopcart-wrapper .content-left .desc-wrapper .tip{
	font-size: 12px;
	color: #bab9b9;
	line-height: 51px;
}

/* 加入购物车后，购物车的高亮样式 */
.shopcart-wrapper .content-left .logo-wrapper.highligh{
	background: #ffd161;
}
.shopcart-wrapper .content-left .logo-wrapper .logo.highligh{
	color: #2D2B2A;
}


/* 购物车上的数量提示小红点  样式 */
.shopcart-wrapper .content-left .logo-wrapper .num{
	width: 15px;
	height: 15px;
	line-height: 15px;
	border-radius: 50%;
	font-size: 9px;
	color: white;
	background: red;
	position: absolute;
	right: 0;
	top: 0;
}
.shopcart-wrapper .content-left .desc-wrapper .tip.highligh{
	line-height: 12px;
}


.shopcart-wrapper .content-right.highligh{
	background: #FFD161;
	color: #2D2B2A;
}

.shopcart-wrapper .content-left .desc-wrapper .total-price {
    font-size: 18px;
    line-height: 33px;
    color: white;
}



/* 购物车列表 样式 */
.shopcart-wrapper .shopcart-list{
	position: absolute;
	left: 0;
	top: 0;
	z-index: -1;
	width: 100%;
}

.shopcart-wrapper .shopcart-list.show{
	/* 变形属性  根据里面的有的内容多高，就自动撑开多高 */
	transform: translateY(-100%);
}

.shopcart-wrapper .shopcart-list .list-top{
	height: 30px;
	text-align: center;
	font-size: 11px;
	background: #f3e6c6;
	line-height: 30px;
	color: #646158;
}

.shopcart-wrapper .shopcart-list .list-header{
	height: 30px;
	background: #F4F4F4;
}
.shopcart-wrapper .shopcart-list .list-header .title{
	float: left;
	border-left: 4px solid #53c123;
	padding-left: 6px;
	line-height: 30px;
	font-size: 12px;
}
.shopcart-wrapper .shopcart-list .list-header .empty{
	float: right;
	line-height: 30px;
	margin-right: 10px;
	font-size: 0;
}
.shopcart-wrapper .shopcart-list .list-header .empty img{
	height: 14px;
	margin-right: 9px;
	vertical-align: middle;
}
.shopcart-wrapper .shopcart-list .list-header .empty span{
	font-size: 12px;
	vertical-align: middle;
}

.shopcart-wrapper .shopcart-list .list-content{
	/* 设置 列表内容的最大高度 当内容超出范围 让其可以滚动  使用 scroll 插件 */
	max-height: 360px;
	overflow: hidden;
	background: white;
}
.shopcart-wrapper .shopcart-list .list-content .food-item{
	height: 38px;
	padding: 12px 12px 10px 12px;
	border-bottom: 1px solid #F4F4F4;
}
.shopcart-wrapper .shopcart-list .list-content .food-item .desc-wrapper{
	float: left;
	width: 240px;
}
.shopcart-wrapper .shopcart-list .list-content .food-item .desc-wrapper .desc-left{
	float: left;
	width: 170px;
}
.shopcart-wrapper .shopcart-list .list-content .food-item .desc-wrapper .desc-left .name{
	font-size: 16px;
	margin-bottom: 8px;
	
	/* 超出部分隐藏*/
	-webkit-line-clamp: 1;
	display: -webkit-box;
	-webkit-box-orient: vertical;
	overflow: hidden;
	height: 16px;
}
.shopcart-wrapper .shopcart-list .list-content .food-item .desc-wrapper .desc-left .unit{
	font-size: 12px;
	color: #B4B4B4;
}
.shopcart-wrapper .shopcart-list .list-content .food-item .desc-wrapper .desc-left .description{
	font-size: 12px;
	color: #B4B4B4;
		
	/* 超出部分隐藏*/
	overflow: hidden;
	height: 12px;
}
.shopcart-wrapper .shopcart-list .list-content .food-item .desc-wrapper .desc-right{
	float: right;
	width: 70px;
	text-align: right;	
}
.shopcart-wrapper .shopcart-list .list-content .food-item .desc-wrapper .desc-right .price{
	font-size: 12px;
	line-height: 38px;
}

.shopcart-wrapper .shopcart-list .list-content .food-item .cartcontrol-wrapper{
	float: right;
	margin-top: 6px;
}

.shopcart .shopcart-mask{
	position: fixed;
	top: 0;
	right: 0;
	width: 100%;
	height: 100%;
	z-index: 98;
	background: rgba(7,17,27,0.6);
}


</style>