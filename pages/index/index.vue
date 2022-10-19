<template>
	<view>


		<div v-if="!placeid">
			<div  >
				<div class="emptyData">请先扫一扫座位上的二维码</div>
				<!-- #ifndef H5 -->
				
				
				<div class="flex flex-center">
					<div @click="saoma()" class="btn-small btn-icon icon-qr_code_light">扫一下</div> 
				</div>
				<!-- #endif -->
			</div>

		</div>
		<div v-else>
			<div class="side">
				<div @click="setCat(0)" :class="catid==0?'side-active':''"  class="side-menu">推荐</div>
				<div @click="setCat(item.catid)"  :class="catid==item.catid?'side-active':''" class="side-menu"
					v-for="(item,index) in  catList" :key="index">{{item.title}}</div>
			</div>
			<div class="main">
				<div @click="gourl('/cy2c/cy2c_order/confirm')" class="goOrder">下单</div>
				<div class="pd-10 flex">
					<div>当前座位：{{place.title}}</div>
					<div class="cl-danger mgl-5">请对号下单</div>
				</div>
				<div class="flexlist">
					<div class="flexlist-item" v-for="(item,index) in  list" :key="index">
						<img @click="goProduct(item.id)" class="flexlist-img pointer bd-radius-5" :src="item.imgurl" />
						<div class="flex-1">
							<div @click="goProduct(item.id)" class="flexlist-title pointer">{{item.title}}</div>
							<div class="flex mgb-5">
								<div class="flex-1 cl3 none">库存 {{item.total_num}}</div>
								<div class="cl-money mgr-10">￥{{item.price}}</div>
								<div class="cl3">已售 {{item.buy_num}}</div>
							</div>
							<div class="flex">
								<div class="flex-1"></div>
								<div v-if="item.isksid>0">
									<div @click="ksBox(item.id)" class="btn-small btn-outline-success">选规格</div>
								</div>
								<div class="pdb-5" v-else>

									<div v-if="item.incart" class="numbox prolist-numbox">
										<div @click="minusCart(item.id,item.cart_amount)" class="numbox-minus">-</div>
										<input type="text" name="amount" :value="item.cart_amount" class="numbox-num" />
										<div @click="plusCart(item.id,item.cart_amount)" class="numbox-plus">+</div>
									</div>
									<div @click="addCart(item.id)" class="btn-buy   iconfont icon-cart" v-else></div>
								</div>
							</div>
						</div>
					</div>
					<div style="height:80px"></div>
				</div>
				<div v-if="ksShow">
					<div class="modal-mask" @click="ksHide"></div>
					<div class="modal">
						<div class="modal-header">
							<div class="modal-title">选择款式</div>
							<div class="modal-close icon-close" @click="ksHide"></div>
						</div>
						<div class="modal-body pdt-10">
							<div class="ksBox mgb-10">
								<div class="kslist mgb-10">
									<div class="kslist-label">{{ksproduct.ks_label_name}}</div>
									<div @click="ks1(item.id)" v-bind:class="{'kslist-active':item.id==ksid1}"
										class="kslist-item" v-for="(item,index) in ksList" :key="index">{{item.title}}
									</div>
								</div>
								<div class="kslist">
									<div class="kslist-label">{{ksproduct.ks_label_size}}</div>
									<div class="kslist-item" @click="ks2(item.id)"
										v-bind:class="{'kslist-active':item.id==ksid}" v-for="(item,index) in ksList2"
										:key="index">{{item.size}}</div>
								</div>
							</div>
							<div class="flex">
								<div class="cl2 mgr-10 mgl-10">价格</div>
								<div class="cl-money">￥{{ksproduct.price}}</div>
								<div class="flex-1"></div>
								<div v-if="ksproduct.incart" class="numbox prolist-numbox">
									<div @click="ksMinusCart(ksproduct.id,ksproduct.cart_amount)" class="numbox-minus">-
									</div>
									<input type="text" name="amount" :value="ksproduct.cart_amount"
										class="numbox-num" />
									<div @click="ksPlusCart(ksproduct.id,ksproduct.cart_amount)" class="numbox-plus">+
									</div>
								</div>
								<div @click="ksAddCart(ksproduct.id)" class="btn-buy" v-else>买</div>
							</div>
						</div>
					</div>
				</div>
			</div>
		</div>
		<mt-footer tab="home"></mt-footer>
	</view>
</template>

<script>
	import mtFooter from "../../components/cy2c-footer.vue";
	var winWidth = 750;
	export default ({
		components: {
			mtFooter
		},
		data: function() {
			return {

				flashList: [],
				navList: [],
				adList: [],
				pageLoad: false,
				place:{}, 
				swiperHeight: 100,
				catid:0,
				list:[],
				isFirst:true,
				per_page:0,
				catList:[],
				ksShow:false,
				placeid:0
			}
		},
		onLoad: function() {
			if(uni.getStorageSync("placeid")!=""){
				this.placeid=uni.getStorageSync("placeid");
			}
			if(this.placeid){
				this.getPage();
				this.getList(); 
			} 
			
			const res = uni.getSystemInfoSync();
			winWidth = res.windowWidth;
			this.swiperHeight = Math.min(640, res.windowWidth) / 2;
		},
		onPageScroll: function(e) {
			 

		},
		onReachBottom: function() {
			this.getList();
		},
		onPullDownRefresh: function() {
			this.getPage();
		},
		onShareAppMessage: function() {

		},
		methods: {
			gourl: function(url) {
				uni.navigateTo({
					url: url
				})
			},
			saoma:function(){
				uni.scanCode({
					onlyFromCamera: true,
					success: function (res) {
						uni.navigateTo({
							url:res.result
						})
						 
					}
				});
			},
			goDetail: function(id) {
				uni.navigateTo({
					url: "../../pages/article/show?id=" + id
				})
			},

			getPage: function() {
				var that = this;
				this.app.get({
					url: that.app.apiHost + "/module.php?m=cy2c",
					data:{
						placeid:this.placeid
					},
					success: function(res) {
						that.pageLoad = true;
						that.flashList = res.data.flashList;
						that.navList = res.data.navList;
						that.adList = res.data.adList;
						that.place=res.data.place;
						uni.stopPullDownRefresh();
					}
				})
			},
			getList: function() {
				var that = this;
				if (!that.isFirst && that.per_page == 0) return false;
				this.app.get({
					url: that.app.apiHost + "/module.php?m=cy2c_product",
					data: {
						catid: that.catid,
						per_page: that.per_page,
						placeid:this.placeid
					},
					dataType: "json",
					success: function(res) {

						that.per_page = res.data.per_page;
						if(that.catid==0){
							that.catList=res.data.catList;
						}
						if (that.isFirst) {
							that.isFirst = false;
							that.list = res.data.list;
						} else {

							for (var i in res.data.list) {
								that.list.push(res.data.list[i]);
							}

						}

					}
				})
			},
			setCat:function(catid){
				this.catid=catid;
				this.isFirst=true;
				this.per_page=0;
				this.getList();
			},
			addCart: function(id, ksid) {
				var that = this;
			 
				var ksid = ksid == undefined ? 0 : ksid;
				var amount = 1;
				that.app.get({
					url: that.app.apiHost+'/module.php?m=cy2c_cart&a=add',
					data: {
						productid: id,
						amount: amount,
						ksid: ksid,
						placeid:this.placeid
					},
					success: function(res) {
						if(res.error){
							uni.showToast({
								title:res.message
							})
							return false;
						}
						var list = that.list;
						for (var i = 0; i < list.length; i++) {
							if (list[i].id == id) {
								if (res.data.amount > 0) {
									list[i].incart = 1;
								}
								list[i].cart_amount = res.data.amount;
								break;
							}
						}
						 
					}
				})
			},
			plusCart: function(id, amount, ksid) {
				var that = this;
		 
				var ksid = ksid == undefined ? 0 : ksid;
				amount++;
				that.app.get({
					url: that.app.apiHost+'/module.php?m=cy2c_cart&a=add&ajax=1',
					data: {
						productid: id,
						amount: amount,
						ksid: ksid,
						placeid:this.placeid
					},
					success: function(res) {
						if(res.error){
							uni.showToast({
								title:res.message
							})
							return false;
						}
						var list = that.list;
						for (var i = 0; i < list.length; i++) {
							if (list[i].id == id) {
			
								list[i].cart_amount = res.data.amount;
								break;
							}
						}
					}
				})
			},
			minusCart: function(id, amount, ksid) {
				var that = this;
				var id = id;
				var amount = amount;
				var ksid = ksid == undefined ? 0 : ksid;
				amount--;
				var isdelete = 0;
				if (amount == 0) {
					isdelete = 1
				}
				that.app.get({
					url: that.app.apiHost+'/module.php?m=cy2c_cart&a=add&ajax=1',
					data: {
						productid: id,
						amount: amount,
						ksid: ksid,
						isdelete: isdelete,
						placeid:this.placeid
					},
					success: function(res) {
						if(res.error){
							uni.showToast({
								title:res.message
							})
							return false;
						}
						var list = that.list;
						for (var i = 0; i < list.length; i++) {
							if (list[i].id == id) {
								if (res.data.amount == 0) {
									list[i].incart = 0;
								}
								list[i].cart_amount = res.data.amount;
								break;
							}
						}
					}
				})
			},
		}
	})
</script>

<style>
	.mtt10 {
		margin-top: -10px;
	}

	.adBox {
		display: flex;
		flex-direction: row;

	}

	.adBox-item {
		flex: 1;
		margin: 5px;
	}

	.adBox-img {
		width: 100%;
		border-radius: 10px;
	}

	.m-navPic-img {
		border-radius: 50%;
	}
	.side{
		position: fixed;
		top: 45px;
		left: 0px;
		width: 80px;
		bottom: 50px;
		background-color: #fff;
	}
	.side-menu{
		cursor: pointer;
		padding: 10px 5px;
		border-bottom: 1px solid #eee;
		color: #646464;
		font-size: 14px;
	}
	.side-active{
		color: #f60;
	}
	.main{
		margin-left: 82px;
	}
	.goOrder{
		position: fixed;
		right: 5px;
		bottom: 60px;
		width: 40px;
		height: 40px;
		line-height: 40px;
		text-align: center;
		border-radius: 50%;
		background-color:#f60;
		color: #fff;
	}
</style>
