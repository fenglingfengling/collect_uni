<template>
	<view class="content">
		<!-- 地区选择和个人中心 
		  首页 当前定位模式模块  个人头像模块  目录模块 
		  猜你喜欢模块 shopList模块 卡片图文
		  前两个都是navigator 跳转到 city页面 和跳转到personal个人页面
		  下面目录是 图文卡片模式 
		  商品列表  图文 左右排列模式
		  
		  -->
		<view class="title_content">
			<!-- 点击跳转城市页city?address
					address 当前的城市位置 
			 -->
			<navigator class="area_name" url="../city/city">
				{{address}}
			</navigator>
			<image class="downIcon" src="/static/images/down.png" /> 
			<!-- 点击跳转用户中心页 personal -->
			<navigator class="people" url="/pages/personal/personal">
				<image class="people-img" src="/static/images/people.png" />
			</navigator>
		</view>
		<!-- 目录 -->
		<view class="menu_content">
			<view class="menu_item">
				<image src="/static/images/eat.png" />
				<text>美食</text>
			</view>
			<view class="menu_item">
				<image src="/static/images/ktv.png" />
				<text>KTV</text>
			</view>
			<view class="menu_item">
				<image src="/static/images/foot.png" />
				<text>足疗/按摩</text>
			</view>
			<view class="menu_item">
				<image src="/static/images/hotel.png" />
				<text>酒店</text>
			</view>
			<view class="menu_item">
				<image src="/static/images/improve.png" />
				<text>丽人/美发</text>
			</view>
		</view>
		<!-- 灰线 -->
		<view class="line_gray"></view>
		<!-- 
			ShopList 猜你喜欢
				title 标题,"猜你喜欢"
				:list 商铺列表
				:loading 数据是否加载中
		 -->
		<ShopList title="猜你喜欢" :list="shops" :loading="loading"></ShopList>
		
	</view>
</template>

<script>
	/* 引入ShopList 店铺列表组件 */
	import ShopList from '../../components/ShopList.vue'
	
	import QQMap from '../../utils/qqmap-wx-jssdk.js';
	
	const map = new QQMap({
		key:"EEYBZ-QK2KG-ELJQJ-IKXXY-7AHES-VIB7K"
	});
	const db = wx.cloud.database();
	
	export default {
		data() {
			return {
				address:'解析中',
				page:0,
				shops:[],
				loadOff: true,
			};
		},
		
		components: {ShopList},
		
		/*
			尝试在这里去获取openId
		*/
		onLoad() {
			
		},
		
		onShow() {
			
			let address = uni.getStorageSync('city');
			// uni.getStorage({
			//     key: 'city',
			//     success: function (res) {
			//        address = res.data;
			//     }
			// });

			if(!address){
				this.getLocation()
				.then(res=>this.getPosition(res))
				.then((res)=>{
					this.address = res;
					uni.setStorage({
					    key: 'city',
					    data: res
					});
				});
			}else{
				this.address = address;
			}
			
			// this.loadOff = true;
		},
		
		watch:{
			// address:function(){
			// 	this.shops = [];
			// 	this.page = 0;
			// 	this.updateShops(true);
			// }
			address:function(){
				this.shops=[];
				this.page = 0;
				this.updateShops(true);
				/*进入城市页选择 this.updateShops(true); 布尔值是管理 是否加载数据 或者下拉加载数据  this.shops为空 保持数据可以清空 
				不造成视图存储 显示错乱问题 page页面为0 返回视图重新获取页面*/ 
			}
		},
		// 下拉刷新 优化
		onReachBottom() {
			if(this.loadOff){
				this.updateShops(false);
			}
		},
		
		onPullDownRefresh(){
			//TODO: 下拉刷新
			//https://uniapp.dcloud.io/collocation/pages?id=pages
		},

		methods: {
			/* 获取用户的经纬度 */
			getLocation(){
				return new Promise((resolve)=>{
					uni.getLocation({
						success:(res)=>{
							console.log(res);
							resolve(res);
						}
					})
				});
			},
			
			/*根据经纬度获取用户当前的地址*/
			getPosition({latitude,longitude}){
				return new Promise((resolve)=>{
					map.reverseGeocoder({
						location:{latitude,longitude},
						success:(res)=>{
							console.log(res);
							resolve(res.result.address_component.city);
						}
					})
				});
			},
			/*获取店铺数据*/
			getShops(){
				uni.showLoading({
				    title: '加载中...'
				});
				const n = 5;
				const start = this.page * n;
				
				return new Promise((resolve)=>{
					db.collection('favorList')
					.where({
						place:this.address
					})
					.skip(start)
					.limit(n)
					.get({
						success:(res)=>{
							console.log(res);
							let newData = this.parseShops(res.data);
							resolve(newData);
							uni.hideLoading();
						}
					})
				});
			},	
			updateShops(isFirst){
				this.getShops().then((data)=>{
					this.shops = [...this.shops,...data];// 更新展开成新数组
					if(data.length<5){
						if(!isFirst){
							uni.showToast({
								title:"没有更多数据了"
							})
							this.loadOff = false;
						}
					}else{
						this.page++;
					}
				})
			},
			// image_path 空的情况下 补404图片
			parseShops(data){
				data.forEach((item)=>{
					if(item.image_path===null){
						item.image_path = 'mdata/img/404.jpg';
					}
				})
				return data
			}
		},
	};
</script>

<style>
	.home_container {
		width: 750upx;
		height: 300upx;
		background: red;
	}

	.title_content {
		width: 750upx;
		height: 80upx;
		background: #00acec;
		color: #fff;
		font-size: 24upx;
		padding-top: 24upx;
	}

	.bar_title {
		color: white;
		margin-top: 60upx;
		margin-left: 34.5upx;
		font-size: 28upx;
	}

	.area_name {
		float: left;
		margin-left: 36upx;
		margin-right: 9upx;
	}
	.downIcon {
		width: 18upx;
		height: 10upx;
	}
	.people{
		margin-right: 28upx;
		float: right;
	}
	.people-img {
		width: 48upx;
		height: 48upx;
	}

	.menu_content {
		width: 750upx;
		height: 198upx;
		display: flex;
		flex-direction: row;
	}
	
	.menu_item {
		flex: 1;
		text-align: center;
	}
	/* 靠margin来布局到字体位置 */
	.menu_item image {
		width: 88upx;
		height: 88upx;
		margin: 20upx 22upx;
	}

	.menu_item text {
		font-size: 24upx;
	}


	
</style>
