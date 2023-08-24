<template>
	<view class="detail_container" v-if="shop">
		<view class="container_up">
			<view class="detail_content">
				<view class="imgContainer">
					<image :src="path+shop.image_path"/>
				</view>
				<view class="store">
					<view class="store_title">{{shop.title}}</view>
					<view class="store_price">
						<Star :score="shop.score"></Star>
						<text class="text-tiao">{{shop.view}}条 </text>
						<text class="text-price">￥{{shop.price}}/人</text>
					</view>
					<view class="store_taste">口味：{{shop.score}} 环境：{{shop.environment}} 服务：{{shop.service}}</view>
					<view class="store_type">{{shop.description}}</view>
				</view>
			</view>
			<view class="local">
				<view class="content-show">
					<image src="/static/images/clock.png" /> 营业至21:00
					<image src="/static/images/wifi.png" /> WIFI
					<image src="/static/images/car.png" /> 停车场
					<image src="/static/images/more.png" />
				</view>
			</view>
			<view class="position">
				<image src="/static/images/position.png" />
				<view class="text-content">
					<text class="text-pos">上地十街辉煌国际西6号楼低商1-2楼(地铁13号线西二旗A口西500米)</text>
				</view>
				<view class="phone-container">
					<image src="/static/images/phone.png" />
				</view>
			</view>
		</view>
		<view class="line_gray"></view>
		<view class="rank">
			<image class="rank-img" src="/static/images/rank.png" />
			「上地小吃快餐热榜」 第1名
			<image class="rank-more" src="/static/images/more.png" />
		</view>
		<view class="line_gray"></view>
		
		<view class="group-buying">
			<image class="group-buying-img" src="/static/images/group-buy.png" alt="团" />

			<view class="show-list-item">
				<image class="food-show" src="/static/images/eatPicture.png" />
				<view class="text-container">
					<text class="food-detail">
						单人商务套餐，提供免费WIFI
						<text>已售28</text>
					</text>
					<p>
						<text class="now-price">￥35</text>
						<text class="original-price">￥57</text>
					</p>
				</view>
			</view>
			<view class="show-list-item">
				<image class="food-show" src="/static/images/eatPicture.png" />
				<view class="text-container">
					<text class="food-detail">
						单人商务套餐，提供免费WIFI
						<text>已售28</text>
					</text>
					<p>
						<text class="now-price">￥35</text>
						<text class="original-price">￥57</text>
					</p>
				</view>
			</view>
、
			<view class="footer">
				<view @click="toggleCollect()">
					<image :src="collected?'/static/images/collect-active.png':'/static/images/collect.png'" /> 收藏
				</view>
				<view><image src="/static/images/remark.png" /> 写点评</view>
			</view>
		</view>
	
	</view>
	

</template>

<script>
	/* 星级组件 */
	import Star from "../../components/Star.vue";
	/* 数据库 */
	const db = wx.cloud.database();
	/* 全局对象 */
	const {globalData}=getApp();
	/* 当前店铺的id */
	let shopId=null;
	/**
	 * shop 店铺详细数据
	 * collected 是否已收藏，控制下方收藏图表的状态
	 */
	export default {
		data() {
			return {
				shop:null,
				collected: false,
				shopId:null,
				path:globalData.myPath,
				openid:'',
			};
		},
		/* 星级组件 */
		components: {Star},
		
		onLoad({id}) {
			this.shopId = id;
			this.getShopById(id)
			.then((res)=>{
				this.shop = res;
			});
			// 获取openId 存储
			let openid = uni.getStorageSync('openid');
			if(!openid){
				this.getOpenId().then((res)=>{
					this.openid = res
					uni.setStorage({
						key:'openid',
						data:res
					});
				});
			}else{
				this.openid = openid
			}

		},
		// 同步实时监听 获取已经收藏 储存 删除openid 复用 记录收藏记录
		watch:{
			openid:function(){
				this.getCollect();
			}
		},
		
		methods: {
			getShopById(id){
				uni.showLoading({
					title:'加载中...'
				});
				
				return new Promise((resolve)=>{
					db.collection('favorList')
					.where({
						id
					}).get({
						success:(res)=>{
							// console.log(res);
							if(!res.data[0].image_path){
								res.data[0].image_path = 'mdata/img/404.jpg';
							}
							
							resolve(res.data[0]);
							uni.hideLoading();
						}
					})
				});
			},

			toggleCollect(){
				if(this.collected){
					this.delCollect();
				}else{
					this.addCollect();
				}
				this.collected = !this.collected;
			},
			
			delCollect(){
				db.collection('collect').where({
					shopid:this.shopId
				}).remove();
			},
			
			addCollect(){
				db.collection('collect').add({
					data:{
						shopid:this.shopId
					}
				})
			},
			getCollect(){
				console.log(123);
				db.collection('collect').where({
					shopid:this.shopId,
					_openid:this.openid
				}).get({
					success:(res)=>{
						this.collected = !!res.data.length;// 数字转布尔值 !! 1转false到true
					}
				});
			},
			
			// 获取openId
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
	.detail_container {
		display: flex;
		flex-direction: column;
		padding-top: 40upx;
		padding-bottom: 86upx;
	}
	.container_up {
		padding-left: 32upx;
		padding-right: 32upx;
	}
	.container_up {
		padding-left: 32upx;
		padding-right: 32upx;
	}
	.detail_content {
		display: flex;
		border-bottom: 1px solid #ddd;
		padding-bottom: 20upx;

	}
	.imgContainer {
		border: 1upx solid #ddd;
		padding: 1upx;
		border-radius: 15upx;
		margin-right: 32upx;
		height: 212upx;
	}

	.imgContainer image {
		width: 210upx;
		height: 210upx;
		border-radius: 15upx;
	}
	.store {
		display: flex;
		flex-direction: column;
		justify-content: space-between;
	}
	.store_title {
		font-weight: bold;
		font-size: 32upx;
	}
	.store_price {
		color: #7b7b7b;
		display: flex;
		flex-direction: row;
		height: 36upx;
		align-items: center;
	}
	.text-tiao {
		font-size: 24upx;
		padding-left: 15rpx;

	}

	.text-price {
		font-size: 26upx;
		font-weight: bold;
		padding-left: 15rpx;
		color: #ff6633;
	}
	
	.store_taste {
		font-size: 26upx;
		color: #868686;
		font-weight: 200;
	}
	.store_type {
		font-size: 26upx;
		color: #868686;
		font-weight: 200;
	}
	.content-show {
		border-bottom: 1px solid #ddd;
		font-size: 24upx;
	}
	.content-show image {
		width: 24upx;
		height: 24upx;
		padding-right: 12upx;
	}

	.content-show image:nth-child(2) {
		margin-left: 42upx;
	}

	.content-show image:nth-child(3) {
		margin-left: 42upx;
	}

	.content-show image:nth-child(4) {
		margin-left: 60upx;
	}

	.local {
		height: 103upx;
		display: block;
		line-height: 103upx;
	}
	.position {
		display: flex;
		flex-direction: row;
		margin-top: 24upx;
		margin-bottom: 24upx;
	}
	.position image {
		width: 42upx;
		height: 28upx;
		margin-top: 8upx;
		margin-right: 10upx;
	}
	.text-content {}
	.text-pos {
		font-size: 24upx;
		line-height: 32upx;
	}

	/* .text-content text {
  font-size: 30upx;
  color: #111111;
  font-weight: 200;
} */
	.phone-container {
		width: 96upx;
		display: flex;
		justify-content: center;
		align-items: center;
		height: 60upx;
		margin-top: 30upx;
		margin-left: 48upx;
		border-left: 1px solid #e1e1e1;
	}

	.phone-container image {
		width: 32upx;
		height: 35upx;
		padding-left: 24upx;
	}
	.rank {
		height: 110upx;
		width: 750upx;
		display: flex;
		justify-content: flex-start;
		align-items: center;
		font-size: 28upx;
	}
	.rank-img {
		width: 40upx;
		height: 40upx;
		margin-left: 28upx;
	} 
   .rank-more {
		width: 17upx;
		height: 28upx;
		margin-left: 210upx;
	} 
	/* 团列表 */
    .group-buying {
		display: flex;
		flex-direction: column;
		flex: 1;
	}
	.group-buying-img {
		width: 40upx;
		height: 40upx;
		margin-left: 28upx;
		margin-top: 28upx;
		position: absolute;
	}

	/* 食物显示 */
	.food-show {
		width: 120upx;
		height: 120upx;
		border-radius: 15upx;
		margin-right: 21upx;
	}
	
	.show-list-item {
		display: flex;
		flex-direction: row;
		margin-left: 88upx;
		margin-top: 26upx;
		border-bottom: 1px solid #e1e1e1;
		padding-bottom: 17upx;
	}
	.food-detail {
		font-size: 24upx;
		color: #000;
		line-height: 48upx;
	}
	.now-price {
		font-size: 28upx;
		color: #ff6633;
		margin-right: 14upx;
		font-weight: bold;
	}

	.original-price {
		text-decoration: line-through;
		font-size: 24upx;
		color: #777777;
	}

	.text-container {}

	.food-detail text {
		margin-left: 40upx;
	}

	/* 底部样式 */
	.footer {
		display: flex;
		flex-direction: row;
		position: fixed;
		bottom: 0;
		width: 100%;
		height: 86upx;
		background: #fcfcfb;
		align-items: center;
		color: #707070;
	}

	.footer view {
		flex: 1;
		display: flex;
		justify-content: center;
		height: 34upx;
		font-size: 28upx;
	}
	
	.footer view:nth-child(1) {
		border-right: 1px solid;
	}

	.footer image {
		width: 34upx;
		height: 34upx;
		vertical-align: middle;
		margin-right: 10upx;
	}
</style>
