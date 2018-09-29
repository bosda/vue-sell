<template>
	<div class="goods">
		<div class="menu-wrapper" v-el:menu-wrapper>
			<ul>
				<li v-for="item in goods" @click="selectMenu($index,$event)" class="menu-item" :class="{current: currentIndex === $index}">
					<span class="text border-1px">
						<span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>{{item.name}}
					</span>
				</li>
			</ul>
		</div>
		<div class="foods-wrapper" v-el:foods-wrapper>
			<ul>
				<li v-for="item in goods" class="food-list food-list-hook">
					<h1 class="title">{{item.name}}</h1>
					<ul>
						<li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item">
							<div class="icon">
								<img width="57" height="57" :src="food.icon">
							</div>
							<div class="content">
								<h2 class="name">{{food.name}}</h2>
								<p v-show="food.description" class="desc">{{food.description}}</p>
								<div class="extra">
									<span class="count">月售{{food.sellCount}}份</span>好评率{{food.rating}}%<span></span>
								</div>
								<div class="price">
									<span class="now">￥{{food.price}}</span><span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
								</div>
								<div class="cartcontrol-wrapper">
									<cartcontrol :food="food"></carcontrol>
								</div>
							</div>
						</li>
					</ul>
				</li>
			</ul>
		</div>
		<shopcart v-ref:shopcart :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
	</div>
	<food :food="selectedFood" v-ref:food></food>
</template>

<script>
	import BScroll from 'better-scroll'
	import shopcart from 'components/shopcart/shopcart'
	import cartcontrol from 'components/cartcontrol/cartcontrol'
	import food from 'components/food/food'

	const ERR_OK = 0
	export default {
	  props: {
	    seller: {
	      type: Object
	    }
	  },
	  data () {
	    return {
	      goods: [],
	      listHeight: [],
	      scrollY: 200,
	      selectedFood: {}
	    }
	  },
	  computed: {
	    currentIndex () {
	      for (let i = 0; i < this.listHeight.length; i++) {
	        let height1 = this.listHeight[i]
	        let height2 = this.listHeight[i + 1]
	        if ((this.scrollY > height1 && this.scrollY < height2)) {
	          return i
	        }
	      }
	      return 0
	    },
	    selectFoods () {
	      let foods = []
	      this.goods.forEach(good => {
	        good.foods.forEach(food => {
	          if (food.count) {
	            foods.push(food)
	          }
	        })
	      })
	      return foods
	    }
	  },
	  created () {
	    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']
	    this.$http.get('api/goods').then(res => {
	      if (res.body.error === ERR_OK) {
	        this.goods = res.body.data
	        this.$nextTick(() => {
	          this._initScroll()
	          this._calculateHeight()
	        })
	      }
	    })
	  },
	  methods: {
	    _initScroll () {
	      this.menuScroll = new BScroll(this.$els.menuWrapper, {
	        click: true
	      })
	      this.foodsScroll = new BScroll(this.$els.foodsWrapper, {
	        click: true,
	        probeType: 3
	      })
	      this.foodsScroll.on('scroll', pos => {
	        this.scrollY = Math.abs(Math.round(pos.y))
	      })
	    },
	    _drop (target) {
	      this.$nextTick(() => {
	        this.$refs.shopcart.drop(target)
	      })
	    },
	    _calculateHeight () {
	      let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook')
	      let height = 0
	      this.listHeight.push(height)
	      for (let i = 0; i < foodList.length; i++) {
	        height += foodList[i].clientHeight
	        this.listHeight.push(height)
	      }
	    },
	    selectMenu (index, event) {
	      let el = this.$els.foodsWrapper.getElementsByClassName('food-list-hook')[index]
	      this.foodsScroll.scrollToElement(el, 300)
	    },
	    selectFood (food, event) {
	      if (!event._constructed) {
	        return
	      }
	      this.selectedFood = food
	      this.$refs.food.show()
	    }
	  },
	  components: {
	    shopcart,
	    cartcontrol,
	    food
	  },
	  events: {
	    'cart.add' (target) {
	      this._drop(target)
	    }
	  }
	}
</script>

<style lang="stylus" rel="stylesheet/stylus">
	@import "../../common/stylus/mixin"
	.goods
		display: flex
		position: absolute
		top: 174px
		bottom: 46px
		width: 100%
		overflow: hidden
		.menu-wrapper
			flex: 0 0 80px
			width: 80px
			background: #f3f5f7
			.menu-item
				display: table
				height: 54px
				padding: 0 12px
				font-size: 12px
				line-height: 14px
				color: rgb(240,20,20)
				vertical-align: middle
				.icon
					display: inline-block
					width: 12px
					height: 12px
					vertical-align: top
					margin-right: 2px
					background-size: 12px 12px
					background-repeat: no-rpeat
					&.decrease
						bg-image('decrease_3')
					&.discount
						bg-image('discount_3')
					&.guarantee
						bg-image('guarantee_3')
					&.invoice
						bg-image('invoice_3')
					&.special
						bg-image('special_3')
				.text
					display: table-cell
					width: 56px
					vertical-align: middle
					border-1px(rgba(7,17,27,0.2))
				&.current
					position: relative
					z-index: 10
					background: #fff
					margin-top: -1px
					font-weight: 700
					.text
						border-none()
		.foods-wrapper
			flex: 1
			.title
				padding-left: 16px
				font-size: 12px
				height: 26px
				line-height: 26px
				border-left: 2px solid #d9dde1
				background: #f3f5f7
				color: rgb(149,153,159)
			.food-item
				display: flex
				margin: 18px
				border-1px(rgba(7,17,27,0.2))
				padding-bottom: 18px
				&:last-child
					border-none()
					padding-bottom: 0
				.icon
					flex: 0 0 57px
					margin-right: 10px
					img
						border-radius: 2px
				.content
					flex: 1
					.name
						font-size: 14px
						line-height: 14px
						margin: 2px 0 8px 0
						color: rgb(7,17,27)
					.desc,.extra
						font-size: 10px
						line-height: 10px
						color: rgb(147,153,159)
					.desc
						line-height: 14px
						margin-bottom: 8px
					.extra
						.count
							margin-right: 12px
					.price
						font-weight: 700
						line-height: 24px
						.now
							font-size: 14px
							margin-right: 8px
							color: rgb(240,20,20)
						.old
							font-size: 10px
							text-decoration: line-through
							color: rgb(147,153,159)
					.cartcontrol-wrapper
						position: absolute
						right: 0
						bottom: 12px
</style>