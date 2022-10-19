<template>
	<view>
		<view class="footer-row"></view>
		<view class="footer">
			<view class="footer-item icon-home" v-bind:class="{'footer-active':tab=='home'}" @click="goHome()">首页</view>
			 
			<view class="footer-item icon-order" v-bind:class="{'footer-active':tab=='order'}" @click="goOrder()">已点</view>
			<view class="footer-item icon-my_light" v-bind:class="{'footer-active':tab=='user'}"  @click="goUser()">我的</view>
		</view>
	</view>
</template>

<script>
	export default{
		props:{
			tab:""
		},
		data:function(){
			return {
				
			}
		},
		methods:{
			goHome:function(){
				uni.redirectTo({
					url:"../../pages/index/index"
				})
			},
			goFenlei:function(){
				uni.reLaunch({
					url:"../../cy2c/cy2c_category/index"
				})
			},
			goProduct:function(){
				uni.reLaunch({
					url:"../../cy2c/cy2c_product/index"
				})
			},
			goCart:function(){
				uni.reLaunch({
					url:"../../cy2c/cy2c_cart/index"
				})
			},
			goOrder:function(){
				uni.reLaunch({
					url:"../../cy2c/cy2c_placeorder/index"
				})
			},
			goUser:function(){
				uni.reLaunch({
					url:"../../cy2c/cy2c_user/index"
				})
			}
		}
	}
</script>

<style>
</style>
