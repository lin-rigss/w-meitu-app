<template>
  <div class="goods">
    <!-- 分类列表 -->
    <div class="menu-wrapper" ref="menuScroll">
      <ul>
        <!-- 左侧菜单列表   专场 -->
        <!-- currentIndex === 0  为什么条件 === 0   因为专场是默认选中状态，而且右侧分类也是默认展示的内容，不用滚动 -->
        <li 
          class="menu-item"           
          :class="{'current':currentIndex === 0}"  
          @click="selectMenu(0)">
          <p class="text">
            <img class="icon" :src="container.tag_icon" v-if="container.tag_icon">
            {{container.tag_name}}
          </p>
        </li>

        <!--左侧菜单列表   热销和早餐等  -->
        <!-- currentIndex === index + 1  为什么是 index+1  因为 下面的分类是遍历出来的， 从0开始，所以 +1 才是真正的热销第一个。  -->
        <li 
            class="menu-item" 
            :class="{'current':currentIndex === index + 1}" 
            v-for="(item,index) in goods" :key="index"
            @click="selectMenu(index+1)"
            >
          <p class="text">
            <img class="icon" :src="item.icon" v-if="item.icon">
            {{item.name}}
          </p>

          <!-- 左侧菜单列表的 小红点 提示商品数量   要将对应 的商品数据传入   -->
          <i class="num" v-show="calculateCount(item.spus)">
            {{calculateCount(item.spus)}}
          </i>

        </li>
      </ul>
    </div>
    <!-- 右侧 商品列表 -->
    <div class="foods-wrapper" ref="foodScroll">
      <ul>
        <!-- 专场的 2个图片 -->
        <li class="container-list food-list-hook">
          <div v-for="(item,index) in container.operation_source_list" :key="index">
            <img :src="item.pic_url">
          </div>
        </li>

        <!-- 热销的具体分类 -->
        <li v-for="(item,index) in goods" :key="index" class="food-list food-list-hook">
          <h3 class="title">{{item.name}}</h3>

          <!--  分类下面  具体的商品列表 -->
          <ul>
            <li 
              v-for="(food,index) in item.spus" 
              :key="index" 
              @click="showDetail(food)"
              class="food-item">
              <div class="icon" :style="head_bg(food.picture)"></div>
              <div class="content">
                <!-- 商品名称 -->
                <h3 class="name">{{food.name}}</h3>
                <!-- 商品描述     数据中有描述就展示，没有就不展示-->
                <p class="desc" v-if="food.description">{{food.description}}</p>
                <div class="extra">
                  <!-- 月售 -->
                  <span class="saled">{{food.month_saled_content}}</span>
                  <!-- 点赞 -->
                  <span class="praise">{{food.praise_content}}</span>
                </div>
                <!-- 网友推荐图标     -->
                <img class="product" :src="food.product_label_picture" alt="">
                <!-- 价格 -->
                <p class="price">
                  <span class="text">￥{{food.min_price}}</span>
                  <span class="unit">/{{food.unit}}</span>
                </p>
              </div>
              <div class="cartcontrol-wrapper">
                <app-cart-control :food="food"></app-cart-control>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>

    <!-- 购物车   poiInfo获取的商品数据   selectFoods（通过计算属性获取的加入购物车的数据）-->
    <app-shopcart :poiInfo="poiInfo" :selectFoods="selectFoods"></app-shopcart>

    <!-- 商品详情组件 -->
    <app-product-detail :food="selectFood" ref="foodView"></app-product-detail>
  </div>
</template>

<script>
// 引入 better-scroll 插件
import BScroll from 'better-scroll'
import Shopcart from '../shopcart/Shopcart'
import CartControl from '../cartcontrol/CartControl'
import ProductDetail from '../productDetail/ProductDetail'
export default {
  data(){
    return {
      container:{},  // 专场的数据
      goods:[],  // 热销商品的数据
      poiInfo:{}, // 存储购物车组件需要用到的关于商品相关的数据。

      listHeight:[],  // 存储所有获取到元素的可视高度
      menuScroll:{},  // 存储 左侧菜单滚动的 实例对象 
      foodScroll:{},  // 存储 右侧菜单滚动的 实例对象
      scrollY:0,  // 初始化滚动的高度值 
      selectFood:{}  // 存储 当前点击的商品
    }
  },
   created(){
    fetch("/api/goods")
      .then(res => {
        return res.json()
      })
      .then(response =>{
        if(response.code == 0){
          // 获取左侧菜单数据
          this.container = response.data.container_operation_source
          // 获取右侧商品数据
          this.goods = response.data.food_spu_tags
          // 获取 购物车组件需要用到的 数据
          this.poiInfo = response.data.poi_info


          // 当获取到数据后，就执行 初始化滚动插件的方法 
          this.$nextTick(() => {
            // 执行滚动方法
            this.initScroll()

            // 实现左右两侧联动效果   需要 获取
            // 1. 计算分类的区间高度
            this.calculateHeight()
            // 2. 监听滚动的位置  （在初始化 better-scroll 实例中监听）
            // 3. 根据滚动位置确认下标,与左侧对应 （在计算属性中）
            // 4. 通过下标实现点击左侧,对应的滚动右侧  （点击方法，传入点击的索引，通过 better-scroll中的 滚动到指定元素的方法中，定位到右侧对应的商品）
          })
        }
      })
  },


  // 计算属性是不能够接收参数的
  methods:{
    head_bg(imgName){
      return "background-image: url(" + imgName + ");"
    },


    // 初始化 better-scroll 插件 
    initScroll(){
                        // 想让哪个元素有滚动效果 ，就把哪个元素放在实例中
      this.menuScroll = new BScroll(this.$refs.menuScroll,{
        click:true
      })
      this.foodScroll = new BScroll(this.$refs.foodScroll,{
        probeType:3, // 必写的属性， 只有 此属性为3的时候才能触发监听滚动的事件。
        click:true   // 激活子元素的点击事件
      })

      // 监听 foodScroll  滚动事件  是 better-scroll 插件中的方法  pos 是一个对象 内有 x y 轴的坐标值 。
      // 获取 右侧分类商品滚动时的  Y轴坐标（滚动的位置）  
      this.foodScroll.on("scroll",(pos) => {
        // console.log(pos.y)
        // 将获取的 Y轴坐标 取整数 并 四舍五入
        this.scrollY = Math.abs(Math.round(pos.y))
        // console.log(this.scrollY)
      })
    },




    // 获取右侧 每一个分类的 区间高度  从顶部 0 到分类中最后一个商品 之前的高度
    calculateHeight(){
      // 首先在元素上 设置自定义类名food-list-hook， 然后通过获取元素的类名来获取 里面循环的所有div商品元素。 
      let foodlist = this.$refs.foodScroll.getElementsByClassName("food-list-hook")
      // console.log(foodlist)

      let height = 0 // 设置初始值 
      this.listHeight.push(height)

      for(let i = 0; i < foodlist.length; i++){
        let item = foodlist[i] // 获取到了每一个元素
        // 累加  初始高度0   +  每一个元素的可视高度 （每一个分类内容的高度）
        height += item.clientHeight
        // 然后把每一个元素的高度都push到  listHeight的数组中。 
        this.listHeight.push(height)
      }
      // console.log(this.listHeight)
    },


    // 点击左侧菜单  及 右侧分类所执行的方法
    selectMenu(index){
      // console.log(index)
      let foodlist = this.$refs.foodScroll.getElementsByClassName("food-list-hook")
      let element = foodlist[index]
      // console.log(element)
      // 使用 better-scroll 插件的方法   滚动到对应元素的位置   （跳转到的元素， 时间：250毫秒）
      this.foodScroll.scrollToElement(element,250)
    },


    // 根据 传入的对应商品的数据中的数量  让 小红点中的数字  显示个数  或 隐藏
    calculateCount(spus){
      let count = 0
      spus.forEach((food) => {
        if(food.count > 0){
          // 根据 food.count数量 累加数量 
          count += food.count
        }
      })
      return count
    },

    // 点击打开商品详情页 方法
    showDetail(food){
      // 点击哪个商品，就把点击商品的数据赋值给一个变量，然后再将变量属性传值 的方式传递给详情页组件
      this.selectFood = food
      // 父调用子组件中的方法   通过ref获取子组件实例  
      this.$refs.foodView.showView()
    }
  },
 
  computed:{

    //  通过获取到右侧所有分类商品的区间值   和  获取到的 Y轴坐标（滚动位置） 进行对比，如果区间值 在 滚动位置里 就生成对应的索引
    currentIndex(){
      for(let i = 0; i < this.listHeight.length; i++){
        // 获取商品区间的范围  上一个分类的区间
        let height1 = this.listHeight[i]
        // 下一个分类的区间
        let height2 = this.listHeight[i+1]

        // 判断滚动Y轴  是否在上述区间中
        if(!height2 || (this.scrollY >= height1 && this.scrollY < height2)){
          // console.log(i)
          // 条件成立 证明滚动在这个区间中 ，将区间值 的索引返回出去。
          return i;
        }
      }
      // 不在区间值 之间，就返回 0 
      return 0
    },



     /**
      *  正常的 购物车的数据，应用 写在 vuex中  
      * 
      *  此项目中的购物车数据是  使用 计算属性来监听  数据中商品的数量来 得到加入购物车的数据。
      *  1. 将购物车数据 存储在一个数组中，然后将数据 通过属性传值 的方式 传递给 购物车组件。      * 
      * 
      */
    // 计算属性 监听 数据中的  商品数量food.count  发生改变  并 获取到购物车中的商品，然后通过属性传值 的方法，传递给购物车组件。  
    selectFoods(){
      // 存储 添加到购物车中的商品 数组  （是根据商品的 > 0 来判断的 ）
      let foods = [] 
      // 因为数据是嵌套数据，要找到商品的count数据，所以要按层次依次遍历获取。
      this.goods.forEach((myfoods) => {
        myfoods.spus.forEach((food) => {
          if(food.count > 0){
            foods.push(food)
          }
        })
      })
      return foods
    }


  },
  components:{
    "app-shopcart":Shopcart,
    "app-cart-control":CartControl,
    "app-product-detail":ProductDetail
  }
}
</script>

<style scoped>
/* 中间内容区域的 样式  content */
.goods{
  display: flex;
  position: absolute;
  top: 190px;
  bottom: 51px;
  overflow: hidden;
  width: 100%;
}
/* 左边 分类列表 */
.goods .menu-wrapper{
  /* 固定 宽度85%， 不会响应式变化 */
  flex:0 0 85px;
  background: #f4f4f4;
}

/* 右边 对应的商品 */
.goods .foods-wrapper{
  /* 响应式变化 */
  flex:1;
  /* background: blue; */
}

/* Menu item */ 
.goods .menu-wrapper .menu-item{
	padding: 16px 23px 15px 10px;
	border-bottom: 1px solid #E4E4E4;
  position: relative;
}

.goods .menu-wrapper .menu-item .text{
	font-size: 13px;
	color: #333333;
  line-height: 17px;
  /* 居中显示， 并设置2行显示，超出隐藏 ... 代替 */
	vertical-align: middle;
	-webkit-line-clamp: 2;
	display: -webkit-box;
	-webkit-box-orient: vertical;
	overflow: hidden;
}
/* 列表的icon图标 */
.goods .menu-wrapper .menu-item .text .icon{
	width: 15px;
	height: 15px;
	vertical-align: middle;
}

/* 专场样式 */ 
.goods .foods-wrapper .container-list{
	padding: 11px 11px 0 11px;
	border-bottom: 1px solid #E4E4E4;
}

/* 网友推荐的图标 样式 */
.goods .foods-wrapper .container-list img{
	width: 100%;
	margin-bottom: 11px;
	border-radius: 5px;
}

/* 具体分类商品布局 */ 
.goods .foods-wrapper .food-list{
	padding: 11px;
}
/* title 前面的小竖条 */
.goods .foods-wrapper .food-list .title{
	height: 13px;
	font-size: 13px;
	background: url(./img/btn_yellow_highlighted@2x.png) no-repeat left center;
	background-size: 2px 10px;
	padding-left: 7px;
	margin-bottom: 12px;
}

.goods .foods-wrapper .food-list .food-item{
	display: flex;
	margin-bottom: 25px;
  position: relative;
}

/* 设置 商品前面的缩略小图 */
.goods .foods-wrapper .food-list .food-item  .icon{
	flex: 0 0 63px;
	background-position: center;
	background-size: 120% 100%;background-repeat: no-repeat;
	margin-right: 11px;
	height: 75px;
}
.goods .foods-wrapper .food-list .food-item .content{
	flex: 1;
}

/* 具体内容样式 */ 
.goods .foods-wrapper .food-list .food-item .content .name{
	font-size: 16px;
	line-height: 21px;
	color: #333333;
	margin-bottom: 10px;
	padding-right: 27px;
}

.goods .foods-wrapper .food-list .food-item .content .desc{
	font-size: 10px;
	line-height: 19px;
	color: #bfbfbf;
	margin-bottom: 8px;
	
	/* 超出部分显示省略号*/
	-webkit-line-clamp: 1;
	display: -webkit-box;
	-webkit-box-orient: vertical;
	overflow: hidden;
}

.goods .foods-wrapper .food-list .food-item .content .extra{
	font-size: 10px;
	color: #BFBFBF;
	margin-bottom: 7px;
}
.goods .foods-wrapper .food-list .food-item .content .extra .saled{
	margin-right: 14px;
}
.goods .foods-wrapper .food-list .food-item .content .product{
	height: 15px;
	margin-bottom: 6px;
}
.goods .foods-wrapper .food-list .food-item .content .price{
	font-size: 0;
}
.goods .foods-wrapper .food-list .food-item .content .price .text{
	font-size: 14px;
	color: #fb4e44;
}
.goods .foods-wrapper .food-list .food-item .content .price .unit{
	font-size: 12px;
	color: #BFBFBF;
}

/* 当前选中 */ 
.goods .menu-wrapper .menu-item.current{
	background: white;
	font-weight: bold;
	margin-top: -1px;
}
.goods .menu-wrapper .menu-item:first-child.current{
	margin-top: 1px;
}

/* 加 减商品的容器样式 */
.goods .foods-wrapper .food-list .food-item .cartcontrol-wrapper{
	position: absolute;
	right: 0;
	bottom: 0;
}

.goods .menu-wrapper .menu-item .num{
	position: absolute;
	right: 5px;
	top: 5px;
	width: 13px;
	height: 13px;
	border-radius: 50%;
	color: white;
	background: red;
	text-align: center;
	font-size: 7px;
	line-height: 13px;
}

</style>
