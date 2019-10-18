<template>
  <div class="seller" ref="sellerView">
    <div class="seller-wrapper">
		<Split></Split>
		<div class="seller-view">

			<!-- 地址容器 -->
			<div class="address-wrapper">
				<div class="address-left">
					{{seller.address}}
				</div>
				<div class="address-right">
					<div class="content"></div>
				</div>
			</div>

        <!-- 可滑动的图片容器 -->
			<div class="pics-wrapper" v-if="seller.poi_env" ref="picsView">
				<ul class="pics-list" ref="picsList">
					<li class="pics-item" v-for="(imgurl,index) in seller.poi_env.thumbnails_url_list" :key="index" ref="picsItem">
					  <img :src="imgurl">
					</li>
				</ul>
			</div>

			<!-- 食品安全档案 -->
			<div class="safety-wrapper">
			    查看食品安全档案
				<span class="icon-keyboard_arrow_right"></span>
			</div>
        </div>

		<Split></Split>


		<div class="tip-wrapper">
			<div class="delivery-wrapper">
              配送服务: {{seller.app_delivery_tip}}
            </div>

            <div class="shipping-wrapper">
              配送时间: {{seller.shipping_time}}
            </div>
		</div>

		<Split></Split>


		<div class="other-wrapper">
			<div class="server-wrapper">
                 商家服务
				<div class="poi-server" v-for="(item,index) in seller.poi_service" :key="index">
				    <img :src="item.icon" > {{item.content}}
				</div>
            </div>
            <div class="discounts-wrapper">
				<div class="discounts-item" v-for="(item,index) in seller.discounts2" :key="index">
					<div class="icon">
					    <img :src="item.icon_url">
					</div>
					<div class="text">
					    {{item.info}}
					</div>
			    </div>
            </div>
		</div>
    </div>
  </div>
</template>

<script>
import Split from '../split/Split'
import BScroll from 'better-scroll'
export default {
  data(){
    return {
      seller:{} // 存储商家数据
    }
  },
  created(){
	// 请求商家数据
    fetch("/api/seller")
      .then(res => {
        return res.json()
      })
      .then(response =>{
        if(response.code == 0){
		  this.seller = response.data		  

          // 获取到数据之后    实现图片和页面的滚动 
          this.$nextTick(() => {
			// 实现图片的左右滚动   首先判断 图片数据中有没有数据， 有数据，才会进行 BScroll的初始化
            if(this.seller.poi_env.thumbnails_url_list){
			  // 获取滚动整个容器的宽度
			  // 1.获取 一张图片的 宽度
			  let imgW = this.$refs.picsItem[0].clientWidth
			  // 2.获取图片的右边距  
			  let marginR = 11
			  // 3. 一张图片的宽度+右边距 * 图片数据的长度 （就是图片总数量）  = 总宽度
              let width = (imgW + marginR) * this.seller.poi_env.thumbnails_url_list.length
              //  获取图片的外层 DOM元素 设置 成 总宽度 
              this.$refs.picsList.style.width = width + "px"
              // 实例化 BScroll  
              this.scroll = new BScroll(this.$refs.picsView,{
				// 允许 横向滚动
                scrollX:true  
              })
            }
            // 实现当前页面整体（最外层大容器）的 上下滚动    
            this.sellerView = new BScroll(this.$refs.sellerView)
          })
        }
      })
  },
  components:{
    Split
  }
}
</script>

<style scoped>
.seller {
		position: absolute;
		left: 0;
		top: 191px;
		bottom: 0px;
		width: 100%;
		background: #F4F4F4;
		overflow: hidden;
	}
	
	.seller .seller-wrapper {
		background: white;
	}
	
	.seller .seller-wrapper .seller-view {
		padding-left: 15px;
	}
	
	.seller .seller-wrapper .seller-view .address-wrapper {
		display: flex;
		padding: 16px 0;
		border-bottom: 1px solid #F4F4F4;
	}
	
	.seller .seller-wrapper .seller-view .address-wrapper .address-left {
		flex: 1;
		background: url(address.png) no-repeat left center;
		padding-left: 26px;
		padding-right: 31px;
		background-size: 14px 16px;
		font-size: 14px;
		line-height: 19px;
	}
	
	.seller .seller-wrapper .seller-view .address-wrapper .address-right {
		flex: 0 0 60px;
		background: url(line.png) no-repeat left center;
		background-size: 1px 15px;
	}
	
	.seller .seller-wrapper .seller-view .address-wrapper .address-right .content {
		width: 100%;
		height: 100%;
		background: url(phone.png) no-repeat center center;
		background-size: 18px 18px;
	}
	
	.seller .seller-wrapper .seller-view .pics-wrapper {
		padding: 10px 0;
		overflow: hidden;
		border-bottom: 1px solid #F4F4F4;
		white-space: nowrap;
	}
	
	.seller .seller-wrapper .seller-view .pics-wrapper .pics-list {}
	
	.seller .seller-wrapper .seller-view .pics-wrapper .pics-list .pics-item {
		display: inline-block;
		margin-right: 11px;
		width: 93px;
		height: 75px;
	}
	
	.seller .seller-wrapper .seller-view .pics-wrapper .pics-list .pics-item img {
		width: 100%;
		height: 100%;
		border-radius: 2px;
	}
	
	.seller .seller-wrapper .seller-view .safety-wrapper {
		padding: 15px 14px 15px 25px;
		background: url(safety.png) no-repeat left center;
		background-size: 14px 16px;
		font-size: 14px;
	}
	
	.seller .seller-wrapper .seller-view .safety-wrapper span {
		float: right;
		font-size: 14px;
	}
	
	.seller .seller-wrapper .tip-wrapper {
		padding-left: 15px;
	}
	
	.seller .seller-wrapper .tip-wrapper .delivery-wrapper {
		background: url(delivery.png) no-repeat left center;
		background-size: 14px 16px;
		padding: 15px 0 15px 25px;
		font-size: 14px;
		border-bottom: 1px solid #F4F4F4;
	}
	
	.seller .seller-wrapper .tip-wrapper .shipping-wrapper {
		background: url(time.png) no-repeat left center;
		padding: 15px 17px 15px 25px;
		background-size: 15px 15px;
		font-size: 14px;
		line-height: 18px;
	}
	
	.seller .seller-wrapper .other-wrapper {
		padding-left: 15px;
	}
	
	.seller .seller-wrapper .other-wrapper .server-wrapper {
		background: url(server.png) no-repeat left center;
		background-size: 15px 15px;
		padding: 15px 0 17px 25px;
		font-size: 14px;
		border-bottom: 1px solid #F4F4F4;
	}
	
	.seller .seller-wrapper .other-wrapper .server-wrapper .poi-server {
		display: inline-block;
		margin-left: 17px;
	}
	
	.seller .seller-wrapper .other-wrapper .server-wrapper .poi-server img {
		width: 15px;
		height: 15px;
		margin-right: 6px;
		vertical-align: middle;
	}
	
	.seller .seller-wrapper .other-wrapper .discounts-wrapper {
		padding: 17px 24px 19px 0;
	}
	
	.seller .seller-wrapper .other-wrapper .discounts-wrapper .discounts-item {
		display: flex;
	}
	
	.seller .seller-wrapper .other-wrapper .discounts-wrapper .discounts-item .icon {
		flex: 0 0 25px;
	}
	
	.seller .seller-wrapper .other-wrapper .discounts-wrapper .discounts-item .icon img {
		width: 15px;
		height: 15px;
	}
	
	.seller .seller-wrapper .other-wrapper .discounts-wrapper .discounts-item .text {
		flex: 1;
		font-size: 14px;
	}
</style>
