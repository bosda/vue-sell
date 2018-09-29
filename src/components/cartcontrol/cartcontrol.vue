<template>
	<div class="cartcontrol">
		<div @click.stop.prevent="decreaseCart" class="cart-decrease" v-show="food.count>0" transition="move">
			<span class="inner icon-remove_circle_outline"></span>
		</div>
		<div class="cart-count" v-show="food.count>0">{{food.count}}</div>
		<div @click.stop.prevent="addCart($event)" class="cart-add icon-add_circle"></div>
	</div>
</template>

<script>
  import Vue from 'Vue'
  export default {
    props: {
      food: {
        type: Object
      }
    },
    methods: {
      addCart (event) {
        if (!this.food.count) {
          Vue.set(this.food, 'count', 1)
        } else {
          Vue.set(this.food, 'count', ++this.food.count)
        }
        this.$dispatch('cart.add', event.target)
      },
      decreaseCart () {
        Vue.set(this.food, 'count', --this.food.count)
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
	.cartcontrol
		font-size: 0
		line-height: 24px
		.cart-decrease
			display: inline-block
			padding: 6px
			vertical-align: top
			opacity: 1
			transform: translate3d(0,0,0)
			&.move-transition
				transition: all 0.4s linear
				.inner
					display: inline-block
					font-size: 24px
					vertical-align: top
					color: rgb(0,160,220)
					transition: all 0.4s linear
					transform: rotate(0)
			&.move-enter, &.move-leave
				opacity: 0
				transform: translate3d(24px,0,0)
				.inner
					transform: rotate(180deg)
		.cart-count
			display: inline-block
			width: 12px
			font-size: 10px
			text-align: center
			line-height: 24px
			margin-top: 6px
			color: rgb(147,153,159)
		.cart-add
			display: inline-block
			padding: 6px
			font-size: 24px
			vertical-align: top
			color: rgb(0,160,220)
</style>