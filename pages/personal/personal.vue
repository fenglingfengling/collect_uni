<template>
	<view>
		<!-- 页面顶部的头像和昵称 -->
		<!-- 已登录样式 -->
		<block v-if="userInfo">
			<view class="person-top">
				<view class="person-picture">
					<image class="avatarUrl" :src="userInfo.avatarUrl" />
				</view>
				<view class="person-name">
					<text>{{userInfo.nickName}}</text>
				</view>
			</view>

			<!-- 收藏列表 /pages/collectList/collectList -->
			<navigator class="person-opt" url="/pages/collectList/collectList">
				<image class="col-img" src="/static/images/collectAct.png" />
				<text class="collection-text">收藏列表</text>
				<text class="right-arrows"></text>
			</navigator>
		</block>
		<!-- 未登录样式 -->
		<view class="person-top" v-else @click="getUserInfo">
			<!-- 用按钮获取用户数据----------
					open-type="getUserInfo" 微信服务端开放能力	
					@getuserinfo="onGetUserInfo" 用户点击该按钮时，会返回获取到的用户信息
			 -->
			<button class="btn-login">
				<image class="avatarUrl" src="cloud://cloud-k6rjw.636c-cloud-k6rjw-1302747416/mdata/app/head.jpg" />
			</button>
			<view class="person-name">
				<text>点击登录</text>
			</view>
		</view>
	</view>
	
</template>

<script>
	export default {
		data() {
			return {
				userInfo:null,
				openid:''
			};
		},
		onLoad(){
			
			
			
			let openid = uni.getStorageSync('openid');
			
			let userInfo = uni.getStorageSync('userInfo');
			if(userInfo){
				this.userInfo = userInfo
			}
			
			if(!openid){
				this.getOpenId().then((res)=>{
					this.openid = res
					uni.setStorage({
						key:'openid',
						data:res
					});
				});
			}
		},
		methods: {
			getUserInfo(){
				uni.getUserProfile({
					desc:'用于完善个人资料',
					success:(res)=>{
						this.userInfo = res.userInfo
						uni.setStorage({
							key:'userInfo',
							data:res.userInfo
						});
					}
				})
			},
			/* onLoad 加载生命周期  获取openid存储openid和userInfo  如果是UserInfo 那么响应式this.userInfo = userInfo
			 不是openid 那么获取getOpenId  那么this.openid为res  存储uni.setStorage键为openid,数据为res请求数据
			 getUserInfo uni.getUserProfile({}) 获取用户信息 data为res.userInfo请求后的登录信息
			 getOpenId 实例化一个Promise 然后 wx.cloud.callFunction({云函数  名字quickstartFunctions data:{type: "getOpenId" }, 获取openid 
			 resolve(res.result.openid) })
			 */ 
			getOpenId(){
				return new Promise((resolve)=>{
					wx.cloud.callFunction({
						name:"quickstartFunctions",
						data:{
							type: "getOpenId"
						},
						success(res){
							// console.log(res);
							resolve(res.result.openid);
						}
					})
				});
			}
		}
	};
</script>

<style>
	.person-top{
		display: flex;
		flex-direction: column;
		align-items: center;
		padding-top: 24px;
		padding-bottom: 20px;
		background-color: #f2f9ff;
		border-bottom: 1px solid #ddd;
	}

	.btn-login {
		border: none;
		background: none;
		padding: 0;
		line-height: 0;
	}

	.btn-login::after {
		border: none;
	}
	.person-picture {
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
	}
	.person-opt {
		width: 750upx;
		height: 100upx;
		line-height: 100upx;
		border-bottom: 1px solid #ddd;
	}

	.avatarUrl {
		border-radius: 50%;
		width: 200upx;
		height: 200upx;
		border: 2px solid #fff;
	}

	.person-name {
		font-size: 30upx;
		margin-top: 18upx;
	}

	.right-arrows {
		width: 18upx;
		height: 18upx;
		display: block;
		rotate: 45deg;
		border-right: 1px solid;
		border-top: 1px solid;
		float: right;
		margin-top: 40upx;
		margin-right: 35upx;
	}
	.col-img{
		width: 32upx;
		height:32upx;
		margin-left: 30upx;
		margin-right: 14upx;
	}
</style>
