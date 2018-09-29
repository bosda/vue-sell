<template>
	<div class="shopcart">
		<div class="content">
			<div class="content-left" @click="this.fold=!this.fold">
				<div class="logo-wrapper">
					<div class="logo" :class="{highlight:totalCount>0}">
						<i class="icon-shopping_cart"></i>
					</div>
					<div class="num" v-show="totalCount>0">{{totalCount}}</div>
				</div>
				<div class="price" :class="{highlight:totalPrice>0}">￥{{totalPrice}}</div>
				<div class="desc">另需配送费￥{{deliveryPrice}}元</div>
			</div>
			<div class="content-right" :class="{highlight:pay.type}">{{pay.title}}</div>
		</div>
		<div class="ball-container">
			<div v-for="ball in balls" class="ball" v-show="ball.show" transition="drop">
				<div class="inner inner-hook"></div>
			</div>
		</div>
		<div class="list-wrapper" v-show="listShow" v-el:cart-list transition="list-wrapper">
			<div class="header">
				<h1 class="title">购物车</h1>
				<span class="empty" @click="empty">清空</span>
			</div>
			<div class="list-content">
				<ul>
					<li class="content-item" v-for="food in selectFoods">
						<h1 class="name">{{food.name}}</h1>
						<div class="price">
							<span>{{food.price*food.count}}</span>
						</div>
						<div class="cartcontrol-wrapper">
							<cartcontrol :food="food"></cartcontrol>
						</div>
					</li>
				</ul>
			</div>
		</div>
	</div>
	<div class="fade" transition="fade" v-show="listShow" @click="this.fold=true"></div>
</template>

<script>
  import BScroll from 'better-scroll'
  import cartcontrol from 'components/cartcontrol/cartcontrol'
  export default {
    props: {
      selectFoods: {
        type: Array,
        default () {
          return []
        }
      },
      deliveryPrice: {
        type: Number,
        default: 0
      },
      minPrice: {
        type: Number,
        default: 0
      }
    },
    data () {
      return {
        balls: [
          {show: false},
          {show: false},
          {show: false},
          {show: false},
          {show: false}
        ],
        dropBalls: [],
        fold: true
      }
    },
    computed: {
      totalPrice () {
        let total = 0
        this.selectFoods.forEach(food => {
          total += food.price * food.count
        })
        return total
      },
      totalCount () {
        let count = 0
        this.selectFoods.forEach(food => {
          count += food.count
        })
        return count
      },
      pay () {
        let price = this.totalPrice
        let obj = {}
        obj.type = false
        if (price === 0) {
          obj.title = '￥' + this.minPrice + '元起送'
        } else if (price > 0 && price < this.minPrice) {
          obj.title = '差￥' + (this.minPrice - price) + '元起送'
        } else {
          obj.type = true
          obj.title = '去结算'
        }
        return obj
      },
      listShow () {
        if (!this.totalCount) {
          this.fold = true
          return false
        }
        let show = !this.fold
        if (show) {
          this.$nextTick(() => {
            this.scroll = new BScroll(this.$els.cartList, {
              click: true
            })
          })
        }
        return show
      }
    },
    methods: {
      drop (el) {
        for (let i = 0; i < this.balls.length; i++) {
          let ball = this.balls[i]
          if (!ball.show) {
            ball.show = true
            ball.el = el
            this.dropBalls.push(ball)
            return
          }
        }
      },
      empty () {
        this.selectFoods.forEach(food => {
          food.count = 0
        })
      }
    },
    transitions: {
      drop: {
        beforeEnter (el) {
          let count = this.balls.length
          while (count--) {
            let ball = this.balls[count]
            if (ball.show) {
              let rect = ball.el.getBoundingClientRect()
              let x = rect.left - 32
              let y = -(window.innerHeight - rect.top - 22)
              el.style.display = ''
              el.style.webkitTransform = `translate3d(0,${y}px,0)`
              el.style.transform = `translate3d(0,${y}px,0)`
              let inner = el.getElementsByClassName('inner-hook')[0]
              inner.style.webkitTransform = `translate3d(${x}px,0,0)`
              inner.style.transform = `translate3d(${x}px,0,0)`
            }
          }
        },
        enter (el) {
          /* eslint-disable no-unused-vars */
          let rf = el.offsetHeight
          this.$nextTick(() => {
            el.style.webkitTransform = 'translate3d(0,0,0)'
            el.style.transform = 'translate3d(0,0,0)'
            let inner = el.getElementsByClassName('inner-hook')[0]
            inner.style.webkitTransform = 'translate3d(0,0,0)'
            inner.style.transform = 'translate3d(0,0,0)'
          })
        },
        afterEnter (el) {
          let ball = this.dropBalls.shift()
          if (ball) {
            ball.show = false
            el.style.display = 'none'
          }
        }
      }
    },
    components: {
      cartcontrol
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
	@import "../../common/stylus/mixin.styl"
	.shopcart
		position: fixed
		left: 0
		bottom: 0
		width: 100%
		height: 48px
		z-index: 50
		.content
			display: flex
			height: 100%
			background: #141d27
			color: rgba(255,255,255,0.4)
			.content-left
				flex: 1
				font-size: 0
				.logo-wrapper
					display: inline-block
					position: relative
					top: -10px
					margin: 0 12px
					padding: 6px
					width: 56px
					height: 56px
					box-sizing: border-box
					border-radius: 50%
					vertical-align: top
					z-index: 100
					background: #141d27
					.logo
						width: 100%
						height: 100%
						border-radius: 50%
						text-align: center
						background: #2b343c
						&.highlight
							background: rgb(0,160,220)
						.icon-shopping_cart
							font-size: 24px
							line-height: 44px
							color: #80858a
						&.highlight
							.icon-shopping_cart
								color: #fff
					.num
						position: absolute
						top: 0
						right: 0
						width: 24px
						height: 16px
						line-height: 16px
						font-size: 9px
						font-weight: 700
						color: #fff
						border-radius: 16px
						text-align: center
						background: rgb(240,20,20)
						box-shadow: 0 4px 8px rgba(0,0,0,0.4)
				.price
					display: inline-block
					margin-top: 12px
					padding-right: 12px
					font-size: 16px
					line-height: 24px
					font-weight: 700
					border-sizing: border-box
					border-right: 1px solid rgba(255,255,255,0.1)
					&.highlight
						color: #fff
				.desc
					display: inline-block
					margin: 12px 0 0 12px
					font-size: 10px
					line-height: 24px
			.content-right
				flex: 0 0 105px
				width: 105px
				font-size: 12px
				line-height: 48px
				font-weight: 700
				text-align:center
				padding: 0 8px
				background: #2b333b
				&.highlight
					background: #00b43c
					color: #fff
		.ball-container
			.ball
				position: fixed
				left: 32px
				bottom: 22px
				z-index: 200
				&.drop-transition
					transition: all 0.4s cubic-bezier(0.49,-.029,0.75,0.41)
					.inner
						width: 16px
						height: 16px
						border-radius: 50%
						background: rgb(0,160,220)
						transition: all 0.4s linear
		.list-wrapper
			position: absolute
			left: 0
			top: 0
			z-index: -1
			width: 100%
			background: #fff
			transform: translate3d(0,-100%,0)
			&.list-wrapper-transition
				transform: translate3d(0,-100%,0)
				transition: all 0.5s
			&.list-wrapper-enter,&.list-wrapper-leave
				transform: translate3d(0,0,0)
			.header
				padding: 0 18px
				height: 40px
				line-height: 40px
				background: #f3f5f7
				border-bottom: 1px solid rgba(7,17,27,0.1)
				.title
					float: left
					font-size: 14px
					color: rgb(7,17,27)
				.empty
					float: right
					font-size: 12px
					color: rgb(0,160,220)
			.list-content
				padding: 0 18px
				max-height: 217px
				overflow: hidden
				.content-item
					position: relative
					padding: 12px 0
					box-sizing: border-box
					border-1px(rgba(7,17,27,0.1))
					.name
						font-size: 14px
						line-height: 24px
						color: rgb(7, 17, 27)
					.price
						position: absolute
						right: 90px
						bottom: 12px
						font-size: 14px
						line-height: 24px
						font-weight: 700
						color: rgb(240, 20, 20)
					.cartcontrol-wrapper
						position: absolute
						right: 0
						bottom: 6px
	.fade
		position: fixed
		left: 0
		top: 0
		width: 100%
		height: 100%
		z-index: 40
		backdrop-filter: blur(10px)
		&.fade-transition
			opacity: 1
			background: rgba(7,17,27,0.6)
			transition: all 0.5s
		&.fade-enter,&.fade-leave
			opacity: 0
			background: rgba(7,17,27,0)
</style>