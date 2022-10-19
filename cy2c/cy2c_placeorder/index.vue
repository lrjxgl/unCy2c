<template>
	<view>
		<div>
			<div class="flexlist">
				 
				<div v-for="(item,index) in list" :key="index" class="flexlist-item">
					<image class="flexlist-img" :src="item.imgurl+'.small.jpg'" mode="widthFix" ></image>
					<div class="flex-1">
						<div class="flexlist-title">{{item.title}}</div>
						<div class="flex">
							<div class="cl-money">￥</div>
							<div class="f18 cl-money">{{item.price}}</div>
							<div class="flex-1"></div>
							 
						</div>
					</div>
					 
				</div>
				 
			</div>
		</div>
		<mt-footer tab="order"></mt-footer>
	</view>
</template>

<script>
	import mtFooter from "../../components/cy2c-footer.vue";
	export default{
		components:{
			mtFooter
		},
		data:function(){
			return {
				list:[],
				placeid:0
			}
		},
		onLoad:function(){
			if(uni.getStorageSync("placeid")!=""){
				this.placeid=uni.getStorageSync("placeid");
			}
			if(this.placeid){
				this.getPage();
			}
			
		},
		methods:{
			getPage:function(){
				var that=this;
				that.app.get({
					url:that.app.apiHost+"/cy2c_placeorder/index",
					data:{
						placeid:this.placeid
					},
					success:function(res){
						that.list=res.data.list;
					}
				})
			}
		}
	}
</script>

<style>
</style>