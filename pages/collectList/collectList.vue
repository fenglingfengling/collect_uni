<template>
	<view>
		<ShopList title='收藏列表' :list="shops" :loading="loading"></ShopList>
	</view>
</template>

<script>
	import ShopList from '../../components/ShopList.vue';
	/* 收藏列表  */ 
	/* 获取数据库 */
	const db = wx.cloud.database();
	
	export default {
		data() {
			return {
				shops: []
			}
		},
		
		components: {ShopList},
		
		onShow() {
			let openid = uni.getStorageSync('openid');
			this.getShops(openid);
			
		},
		methods:{
			async getShops(openid){
				let collectList = await db.collection('collect').where({
					_openid:openid
				}).get();
				console.log(collectList);
				let ids = [];
				collectList.data.forEach((item)=>{
					ids.push(item.shopid);
				});
				db.collection('favorList').where({
					id: db.command.in(ids)
				}).get({
					success:(res)=>{
						console.log(res);
						let newData = this.parseShops(res.data);
						this.shops = newData;//根据id 查找到shops的产品  
						console.log(this.shops);
					}
				});
				},
				/*
				引入ShopList 店铺列表组件ShopList.vue  里面取出 shops loading加载数据  onShow 获取openid getShops获取参数openid
				  async getShop(openid)  获取openid 然后收藏 await db.collection('collect').where({}).get();  然后一个个id 存储到另一个变量里面
				  然后获取喜欢列表的 id  赋予了 item.image_path为空  就补404图片
				*/ 
			parseShops(data){
				data.forEach((item)=>{
					if(item.image_path === null){
						item.image_path = 'mdata/img/404.jpg';
					}
				})
				return data;
			}
		
		}
	};
</script>


