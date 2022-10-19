<template>
	<view>
		<div class="row-box">
			<div class="flex mgb-10">
				<div class="cl-red">*每次提交都要重新审核！</div>
				<div class="flex-1"></div>
				<div class="mgl-10">当前状态：</div>
				<div class="">
					<span class="cl-status" v-if="user.is_auth">已认证</span>
					<template v-else>
						<span class="cl-status" v-if="data.id==0">未认证</span>
						<span class="cl-status" v-else-if="ddata.status==0">新认证待审核</span>
						<span class="cl-status" v-else>新认证被拒绝</span>
					</template>
					
				</div>
			</div>


			<form>
				<div class="input-flex">
					<div class="input-flex-label input-flex-require">身份证：</div>
					<input type="text" v-model="data.user_card" class="input-flex-text" />
					<span class="f12 cl-red">*(审核通过后不可修改)</span>
				</div>
				<div class="input-flex">
					<div class="input-flex-label input-flex-require">姓名：</div>
					<input type="text"   v-model="data.truename" class="input-flex-text" />
					<span class="f12 cl-red">*(审核通过后不可修改)</span>
				</div>
				<div class="input-flex flex-ai-center">
					<div class="input-flex-label input-flex-require">本人照片</div>
					<div class="flex-1">
						<div class="upimg-box bg-fff">

							<div @click="upload()"  v-if="data && data.true_user_head!=''" class="upimg-item js-upimg-btn">
								<image class="upimg-img" :src="data.true_true_user_head+'.100x100.jpg'" mode="widthFix">
								</image>
							</div>

							<div @click="upload()" class="upimg-btn js-upimg-btn">
								<i class="upimg-btn-icon"></i>
							</div>

							<input type="text" v-model="data.true_user_head" class="none" />
							 
						</div>
					</div>
				</div>
				<div class="input-flex">
					<div class="input-flex-label">手机：</div>
					<input type="text" v-model="data.telephone" class="input-flex-text" />
					<div @click="sendSms" class="input-flex-btn">发送验证码</div>
				</div>
				<div class="input-flex">
					<div class="input-flex-label">验证码：</div>
					<input type="text"  v-model="yzm" class="input-flex-text" />

				</div>
				<div class="flex flex-center">
					<div v-if="!user.is_auth" class="btn mgr-20" @click="submit()">提交认证</div>
					<div class="btn btn-warning" @click="app.goHome()">返回首页</div>
				</div>

			</form>
		</div>	
	</view>
</template>

<script>
	export default {
		data: function() {
			return {
				data: {
					id:0,
					user_card:"",
					truename:"",
					true_user_head:"",
					telephone:"",					
				},
				user:{},
				yzm:""
			}
		},
		onLoad: function() {
			this.getPage()
		},
		methods: {
			getPage: function() {
				var that = this;
				that.app.get({
					url: that.app.apiHost + "/index.php?m=user_auth&ajax=1",
					success: function(res) {
						if(res.data.data!=null){
							that.data = res.data.data;
						}
						
						that.user=res.data.user;
					}
				})
			},
			submit:function(){
				var that=this;
				that.app.post({
					url: that.app.apiHost + "/index.php?m=user_auth&a=save&ajax=1",
					data:{
						user_card:that.data.user_card,
						truename:that.data.truename,
						telephone:that.data.telephone,
						yzm:that.yzm
					},
					success: function(res) {
						uni.showToast({
							title:res.message,
							icon:"none"
						})
					}
				})
			},
			sendSms:function(){
				var that=this;
				that.app.get({
					url: that.app.apiHost + "/index.php?m=user_auth&a=sendsms&ajax=1",
					data:{
						telephone:that.data.telephone
					},
					success: function(res) {
						uni.showToast({
							title:res.message,
							icon:"none"
						}) 
					}
				})
			},
			upload:function(){
				var that=this;
				uni.chooseImage({
					success: (chooseImageRes) => {
						const tempFilePaths = chooseImageRes.tempFilePaths;
						uni.uploadFile({
							url: that.app.apiHost+"/index.php?m=upload&a=img&ajax=1&loginToken="+that.app.getToken(),
							filePath: tempFilePaths[0],
							name: 'upimg',
							
							success: function(res){
								var e=JSON.parse(res.data);
								that.data.true_user_head=e.data.imgurl;
								that.data.true_true_user_head=e.data.trueimgurl;
								
							}
						});
					}
				});
			}
		}
	}
</script>

<style>
</style>
