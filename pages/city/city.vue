<template>
	<!-- scroll-view 滚动视图
			scroll-y 竖向滚动 -------------------
	 -->
	 <!-- 写了当前定位 address  热门城市模块 城市列表模块 右侧字母模块
	  点击例如热门城市里面 上海 会跳回主页面 当前上海市定位 toIndex
	  uni.setStorage({ })  里面success 参数ele+'市' uni.navigatorBack()返回主页面
	  城市列表，每类城市以首字母为id 做锚点跳转
	  循环cityData获首字母 循环citys 城市
	  右侧字母 点击时 ：updateLetter(v)，data数据响应式里面 letter指向形参letter 更新letter 数据，视图会滚动到id 为相应letter 的元素上
	  /* 生成26个字母  标准公式*/  letters(){let arr=[];
	  	for(let i=65;i<91;i++){arr.push(String.fromCharCode(i));}return arr;}		
	  -->
	<scroll-view class="scrollView" scroll-y :scroll-into-view="letter">
		<!-- 当前位置 -->
		<view class="location">
			<text class="chiose-city">{{address}}</text>
			<text class="gps">GPS定位</text>
		</view>
		<!-- 热门城市-->
		<view class="hot-city">
			<view class="city-title">热门城市</view>
			<view class="hot-city-container">
				<!-- @click点击时：toIndex(ele)返回首页，并用全局属性将所选城市带回去-->
				<view  v-for="ele in hotCities" :key="ele" @click="toIndex(ele)">{{ele}}</view>
			</view>
		</view>
		<!-- 城市列表，每类城市以首字母为id -->
		<view>
			<view v-for="v in cityData" :key="v.letter" :id="v.letter">
				<!-- 城市首字母 -->
				<view class="letter">{{v.letter}}</view>
				<view class="city-ele" v-for="ele in v.citys" :key="ele">
					{{ele}}
				</view>
			</view>
		</view>
		<!-- 右侧字母
				点击时 ：updateLetter(v)，更新letter 数据，视图会滚动到id 为相应letter 的元素上
		-->
		<view class="right-letter">
			<text v-for="v in letters" :key="v" @click="updateLetter(v)">
				{{v}}
			</text>
		</view>
	</scroll-view>
</template>

<script>
	/* 引入所有城市的数据 */
	let cityData = require("../../utils/cityData.js");
	export default {
		/* 数据
			hotCities 热门城市列表，没有带“市”
			address 前所在城市
			letter 当前所选择的字母
			cityData 中国所有城市的数据
		 */
		data() {
			return {
				hotCities: ["上海", "北京", "广州", "深圳", "天津", "杭州", "南京", "苏州", "成都", "武汉", "重庆", "西安"],
				address: '',
				letter: '',
				cityData,
			};
		},
		/* 页面加载时，从全局对象获取address */
		onLoad(){
			// 更新当前地址
			let address = uni.getStorageSync('city');
			this.address = address;
		},
		methods: {
			updateLetter(letter){
				// 点击右侧更新视图 letter
				this.letter = letter;
			},
			toIndex(ele){
				uni.setStorage({
					key:'city',
					data:ele+'市',
					success:()=>{
						uni.navigateBack();//跳转回去 并带有市名称选择城市
					}
				})
			}

		},
		computed:{
			/* 生成26个字母  标准公式*/
			letters(){
				let arr=[];
				for(let i=65;i<91;i++){
					arr.push(String.fromCharCode(i));
				}
				return arr;
			}		
		}
	};

</script>

<style>
	
	.location {
		display: flex;
		width: 750rpx;
		height: 90rpx;
		align-items: center;
	}

	/* 城市标题 */
	.city-title {
		font-size: 24rpx;
		color: #00acec;
		margin-top: 20rpx;
		margin-left: 30rpx;
		margin-bottom: 18rpx;
	}

	.chiose-city {
		font-size: 24rpx;
		margin-left: 30rpx;
		margin-right: 20rpx;
	}

	.gps {
		color: #c0c0c0;
		font-size: 20rpx;
	}
	
	.hot-city {
		display: flex;
		flex-direction: column;
		width: 750rpx;
		height: 400rpx;
		border-bottom: 1px dashed #ddd;
		border-top: 1px dashed #ddd;
		margin-bottom: 20rpx;
	}
	
	.hot-city-container {
		margin-left: 28rpx;
		margin-right: 70rpx;
	}

	.hot-city-container view {
		width: 200rpx;
		height: 60rpx;
		background: white;
		float: left;
		text-align: center;
		margin-right: 8rpx;
		margin-top: 8rpx;
		border: 1px solid #00acec;
		border-radius: 6upx;
		line-height: 60rpx;
		font-size: 24rpx;
	}
 	.letter {
		color: #aeaeae;
		font-size: 22rpx;
		margin-left: 30rpx;
		margin-top: 6rpx;
	}
	.city-ele {
		margin-left: 28rpx;
		margin-right: 68rpx;
		height: 70rpx;
		border-bottom: 1px dashed #ddd;
		line-height: 70rpx;
		font-size: 24rpx;
	}
	/* 右侧的字母 */
	.right-letter {
		width: 40rpx;
		position: fixed;
		right: 20rpx;
		top: 200rpx;
		display: flex;
		flex-direction: column;
		font-size: 18rpx;
		color: #515b67;
	}
	.right-letter text {
		text-align: center;
		margin-bottom: 9rpx;
	}

	.scrollView {
		height: 1334rpx;
	}
</style>
