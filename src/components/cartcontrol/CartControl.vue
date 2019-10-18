<template>
	<div class="cartcontrol">
		<!-- 减少 添加动画效果   v-show 当 food.count有值 的时候才让减号 显示 -->
		<transition name="move">
			<div 
				class="cart-decrease icon-remove_circle_outline"
				@click.stop.prevent="decreaseCart"
				v-show="food.count"
			></div>
        </transition>
		<!-- 显示 商品的数量  v-show 当 food.count有值 的时候才让数量 显示 -->
		<div class="cart-count" v-show="food.count">{{food.count}}</div>
		<!-- 增加  -->
		<div 
			class="cart-add icon-add_circle"
			@click.stop.prevent="increaseCart"
			>
			<i class="bg"></i>
		</div>
	</div>
</template>

<script>
import Vue from 'vue'
	export default {
    props:{
	  // 接收 父级 传递过来的商品数据
      food:{
        type:Object
      }
    },
	methods:{
	  // 减 方法
      decreaseCart(){
        this.food.count--
	  },
	  // 加 方法
      increaseCart(){
		// 判断 数据中有没有 count属性  如果没有就给商品数据 添加 count 属性
        if(!this.food.count){
          Vue.set(this.food,"count",1)
        }else{
          this.food.count++
        }
      }
    }
	}
</script>

<style>

.cartcontrol{
	/* 解决 子元素样式中的使用 inline-block 出现间隙的问题，设置父组元素的 font-size: 0 */
	font-size: 0;
}

.cartcontrol .cart-decrease{
	display: inline-block;
	width: 26px;
	height: 26px;
	font-size: 26px;
	color: #b4b4b4;
}

.cartcontrol .cart-add .bg{
	width: 20px;
	height: 20px;
	border-radius: 50%;
	background: black;
	position: absolute;
	left: 3px;
	top: 3px;
	z-index: -1;
}


.cartcontrol .cart-count{
	display: inline-block;
	width: 25px;
	text-align: center;
	font-size: 12px;
	line-height: 26px;
	vertical-align: top;
}

.cartcontrol .cart-add{
	display: inline-block;
	width: 26px;
	height: 26px;
	font-size: 26px;
	color: #ffd161;
	position: relative;
}

.move-enter-active,.move-leave-active{
	transition: all 0.5s linear;
}
.move-enter,.move-leave-to{
	transform: translateX(20px) rotate(180deg);
}
</style>
